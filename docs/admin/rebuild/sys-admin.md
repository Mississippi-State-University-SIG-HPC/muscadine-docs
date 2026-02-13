# Hardware Configuration
In order to prevent corruption to Muscadine's stable version, we will only be using 2 of the 5 U.2 drives in `muscadine-node-1` configured in a hardware RAID 1 to prevent corruption. New Muscadine will also skip `1.1.2: Scratch` to prevent corruption to active scratch, but once approved will be implemented. Additionally, the image for `muscadine-node-1` will be installed via internal pxe installer.

---

# Basic Cluster Configuration
This part will cover the basics of configuring New Muscadine and show how most Advanced Research Collaboratory (ARC) systems operate.

---

##  Storage
Storage for Muscadine includes:
- 5x U.2 960 GB PCIe 5.0 drives
	- 2x in RAID 1 for New Muscadine root
- 10x M.2 512 GB PCIe 5.0 drives
	- 2x per node, used for BeeGFS
- 6x 16TB PureStorage FlashBlades
	- Connected via 40GbE

###  Home Directory
The home directory is a shared directory hosted via NFS by the PureStorage system. This directory is the directory you are put into when you ssh into Muscadine. The intended use of this directory include: configuration files, scripts, log files, etc. This can be found at `/home/<YourNetID>`. 

To mount this NFS, add the following to `/etc/fstab`

```text
192.168.34.9:/home /home nfs defaults,nconnect=16 0 0
```

###  Scratch Directory
The scratch directory is a distributed flash storage solution provided by BeeGFS. This directory is intended for building programs, storing large datasets, etc. *DO NOT STORE IMPORTANT FILES HERE.* The reason for this is that when storage is close to full, it will automatically start clearing files based on its age. The path for this is `/scratch/`.

```{note}
This will not work until you have deployed [BeeGFS](#beegfs)
```

###  Other NFS via PureStorage
Other than `/home`, the PureStorage system will also provide several other NFS directories such as `/work` and `/apps`. This step very similar to Section 1.1.1, but differs by intended purpose and quota size. 

`/apps` is used for supporting software, modules, and libraries such as compilers, software stacks, etc. Users cannot write to but can read from this directory.

`/work` is used for a safe and decently quick storage for saving binaries for a long time. After building a program, you can store the resulting binary here. The reason to use this vs `/scratch` is the reliability as scratch can be prone to downtime or deleted data due to full capacity. Users can access their respective work directories at `/work/users/<YourNetID>`

To add these NFS mounts, add the following to `/etc/fstab`

```text
192.168.34.9:/apps /apps nfs defaults,nconnect=16 0 0
192.168.34.9:/work /work nfs defaults,nconnect=16 0 0
```


---

##  Warewulf
Warewulf is the node provisioning solution ARC uses for all production clusters. This is where the majority of configuration will take place on muscadine. Warewulf is used to handle the compute node images. all software that can't be made available in the apps tree will need to be present in the compute image. 

###  Building Warewulf
Starting out, we need to pull the latest *release* of Warewulf. But before we can build it, we must first install the dependencies. Run the following command to install the necessary packages.

```bash
dnf  --assumeyes  --setopt=install_weak_deps=False  --nodocs  install  \
	dhcp-server  \
	tftp-server  \
	nfs-utils  \
	golang
```

To keep the VM that Warewulf will be running in small, we'll want to store all of Warewulf's installation and source on an NFS mount. this shouldn't affect performance since the connection between VM and host is virtual and is as fast as loopback. 
#### Mount Setup:

Append the following to `/etc/fstab`

```text
muscadine-node-1.hpc.msstate.edu:/local/ww /opt/warewulf nfs defaults,nodev,noatime 0 0
```

Now we want to mount it with the following commands.

```bash
mkdir /opt/warewulf
mount -a
```
#### Cloning Warewulf Source:

We can continue now to cloning the latest release of Warewulf.

```bash
git clone -b v4.6.x https://github.com/warewulf/warewulf.git /opt/warewulf/src
cd /opt/warewulf/src
```
#### Patching Warewulf

There are a few things we need to change before we build, to suite our environment

First, locate the file `configs/ww/HPC2-dhcpd-ww.patch` in this repo, replace with the path to that file in the following command:

```bash
git apply <patch>
```
#### Building Warewulf

Finally, its time to actually build Warewulf. Run the following commands to do so.

```bash
cd /opt/warewulf/src
make clean defaults PREFIX=/opt/warewulf
make all
make install
```

>[!NOTE]
>Building requires a http connection. Ask senior admins for the best way to do this.
#### Populating iPXE Files

After building Warewulf, we need to build the appropriate iPXE files for our deployment. Since our systems are x86_64, we need to build the x86_64 iPXE files. Run the following to do so.

```bash
./scripts/build-ipxe.sh
cp -i /opt/warewulf/share/ipxe/bin-x86_64-efi-snponly.efi /var/lib/tftpboot/warewulf/ipxe-snponly-x86_64.efi
```
#### Initial Configuration

Now that everything for Warewulf has been built, we can now proceed with the initial automated configuration of Warewulf.

```bash
wwctl configure --all
systemctl enable --now warewulfd
```

###  Setup Podman
Warewulf images are really just specialized Docker/Podman/Containerd/etc... containers. Management can be done in 2 ways:

1. Pulling a base image straight into Warewulf and managing it by chrooting into it and making stated changes
2. Using Podman to pull a base image, make a derivative image using Dockerfiles, and export that image to Warewulf

The former works just fine, and is currently the motis operendi of HPC2 admins. The latter is an appealing alternative as it makes updates, and other scripting easier in the long run. As such, we'll be moving forward primarily with the 2nd approach, reserving the first for small tweaks required to get things working.

First step is to install Podman, RHEL's version of docker.

```bash
dnf install -y docker
```

###  Configuring Images via Dockerfile

#### Writing a Dockerfile

The following is the basis of the docker file we will be using for the compute nodes. It is here that you can add new packages, mounts, etc. that all will be built into the system. 

```Dockerfile
<enterHere>
```
#### Building the Container in Podman

After completing the Dockerfile, we now need to build the container based on it. Run the following command to do so.

```bash
docker build -t base-rocky9 .
```
#### Exporting Podman Images to Warewulf

After building the Dockerfile into a container image via Podman, we need to upload it to Warewulf so that it can do its magic. Run the following to upload your desired image to Warewulf.

```bash
export TMPDIR=$(mktemp -d)
docker save base-rocky9 -o $TMPDIR/base-rocky9.tar
wwctl container import $TMPDIR/base-rocky9.tar base-rocky9
rm $TMPDIR/base-rocky9.tar
wwctl profile set --image base-rocky9 default --yes
wwctl profile set --netdev eno8303 default --yes
wwctl profile set --netdev ib0
wwctl image syncuser --write --build base-rocky9
wwctl overlay build
```
###  Defining Warewulf nodes

Warewulf makes configuring nodes on mass easy and intuiative. here are a few commands to know:
- `wwctl profile edit <profileName>`: allows you to edit an entire profile. Use default for normal usage.
- `wwctl node add <enterNodeName>`: allows you to create new nodes, one at a time or in mass.
- `wwctl node edit <nodeName>`: allows you to edit a specific node for things such as IP addresses or profile.
#### Profile Configuration

We will start with configuring the default profile to be used for Muscadine. run `wwctl profile edit default` and add/edit the following into it.

```yaml
default:
  system overlay:
    - <addOverlaysHere>
  
  network devices:
    default:
      type: ethernet
      device: eno8303
    ib:
      type: infiniband
      device: ib0

  image name: base-rocky9
```
#### Node Configuration

Here are the commands to run for basic Muscadine node configuration:

```bash
wwctl node add muscadine-node-{2..5}
wwctl node edit muscadine-node-{2..5} \
	--ipaddr=192.168.34.{32..35}
wwctl node edit muscadine-node-{2..5} \
	--netname=ib \
	--ipaddr=192.168.35.{32..35}
wwctl overlay build
```

###  Initial Booting of Muscadine

To get the nodes to get their respective images, run the following for each node and power them on one at a time in respective order so that each node gets the appropriate image.

```bash
wwctl node set --discoverable muscadine-node-<enterNodeNum>
```

##  Slurm
Comming soon

---

##  BeeGFS
Comming soon

---

# Supporting Software

##  Spack
See [Using Spack](../spack.md)

---

# Automation and Metrics
Comming soon