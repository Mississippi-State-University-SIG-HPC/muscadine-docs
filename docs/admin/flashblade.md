# Purestorage Flashblade 

## Connecting to Serial Interface
- each fabric manager on the back of the flashblade has an rj45 serial console port.
- a crossover cable is required for communication
- a baudrate of 115200 must be set
- 8N1 parity is used 
- the username:password is pureuser:pureuser

## Configuring Networking
Due to the pure's intended workload, it has some niche networking requirements, namely, that both of it's fabric managers have a symmetric LACP bond between it's 40GbE QSFP+ ports.
Without the availability of an enterprise 40GbE capable switch, we're left to improvise.

Aganst PureStorage's advice, the following configuration works:
- 2x 40GbE NICs in node-1 connected to CH1.FM1.ETH1 and CH1.FM2.ETH1
On the Node: assuming the 2 NICs in question are ens5 and ens6 (their order doesn't matter)

```bash
nmcli con add \
	type bond \
	con-name bond0 \
	ifname bond0 \
	bond.options "mode=802.3ad,miimon=100,xmit_hash_policy=layer3+4" \
	ipv4.method disabled \
	ipv6.method ignore \
	mtu 9000
nmcli con add \
	type bond-slave \
	con-name bond0-enp5 \
	ifname enp5 \
	master bond0 \
	mtu 9000
nmcli con add \
	type bond-slave \
	con-name bond0-enp6 \
	ifname enp6 \
	master bond0 \
	mtu 9000
nmcli con add \
	type vlan \
	con-name vlan334 \
	ifname 334 \
	dev bond0 \
	id 334 \
	master br0 \
	mtu 9000
nmcli con up bond0
nmcli con up bond0-enp5
nmcli con up bond0-enp6
nmcli con up vlan334
```

Then, on the Pure:
- **NOTE:** since the pure cannot be reset by the user, some of these commands may fail as they've already been run. the desired outcome can be reached by replacing `create` w/ `setattr` in most cases

```bash
purehw connector setattr --port-count 1 --lane-speed 10Gbps CH1.FM1.ETH1
purehw connector setattr --port-count 1 --lane-speed 10Gbps CH1.FM2.ETH1
purelag create uplink --port CH1.FM1.ETH1,CH1.FM2.ETH1
puresubnet create --prefix 192.168.34.0/24 --vlan 334 --gateway 192.168.34.1 --mtu 1500 --lag uplink priv334
purenetwork vip create --address 192.168.34.9 --servicelist data data
```

Now, we should have a vlan tagged, LACP bonded link between the Pure and Node-1 bridged to the rest of muscadine's network.

Test connectivity by pinging 192.168.34.4 for the mgt or 192.168.34.9 for the service

## NFS Shares
We'll need to create the following shares by logging into the webgui:

| share   | provisioned size | user quota | snapshot | nfs rules                                                         |
| ------- | ---------------- | ---------- | -------- | ----------------------------------------------------------------- |
| home    | 20 T             | 2 T        | yes      | 192.168.34.31(rw,no_root_squash) 192.168.34.0/24(rw,root_squash)  |
| work    | 20 T             | 2 T        | yes      | 192.168.34.31(rw,no_root_squash) 192.168.34.0/24(rw,root_squash)  |
| apps    | 20 T             |            | yes      | 192.168.34.31(rw,no_root_squash) 192.168.34.0/24(rw,root_squash)  |
| patches | 20 T             |            | yes      | 192.168.34.31(rw,no_root_squash) 192.168.34.0/24(rw,root_squash)  |
| ww      | 2 T              |            | yes      | 192.168.34.31(rw,no_root_squash) 192.168.34.13(rw,no_root_squash) |

All shares will have the following options:
- NFSv3 - yes
- Hard Limit - no
- No default group quota

## Policies
The pure makes it trivial to protect your data in the event of accidental deletion.
In the web interface, follow these steps:
- in the left pane, click protection
- on the top pane, click policies
- on the top right, click the plus
- name is arbitrary
- click create
- create snapshot every `1d`
- at `12am`
- and keep for `30d`
- click add
- click the new policy
- under members, click the 3 dots
- click add file-systems
- check `apps`, `work`, `home`, and `ww`
- click add


## A Note on Storage Schema
PureStorage prides themselves on their compression and deduplication algorithms.
It is **PARAMOUNT** that users do not compress data on a per-file basis. compressed data is almost impossible to deduplicate or compress again. rely on the appliance to handle these space-saving techniques.