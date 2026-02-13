# Warewulf

Warewulf is where the majority of configuration will take place on muscadine. warewulf is used to handle the compute node images. all software that can't be made available in the apps tree will need to be present in the compute image.

## Building 

Building warewulf is fairly straightforward, and can be performed by losely following CIQ's docs
the following should be performed in the WW vm discussed on the [main](index.md) page.

### Install Dependencies:

```bash
dnf  --assumeyes  --setopt=install_weak_deps=False  --nodocs  install  dhcp-server  tftp-server  nfs-utils  golang
```

### Mount setup:

to keep the VM small, we'll want to store all of warewulf's installation and source on an NFS mount. this shouldn't affect performance since the connection between VM and host is virtual and is as fast as loopback.

Append the following line to `/etc/fstab`

```text
muscadine-node-1.hpc.msstate.edu:/local/ww                    /opt/warewulf        nfs     defaults,nodev,noatime     0 0
```

now mount it w/

```bash
mkdir /opt/warewulf
mount -a
```

### Cloning the Source

fetch the source code and checkout the correct tag with:
```bash
git clone https://github.com/warewulf/warewulf.git /opt/warewulf/src
git -C /opt/warewulf/src checkout v4.6.x
```

### Patching

there are a few things we need to change before we build, to suite our environment

first, locate the file `configs/ww/HPC2-dhcpd-ww.patch` in this repo, replace <patch> with the path to that file in the following command

```bash
git apply <patch>
```

### Building

*NOTE:* Building requries an http connection. Ask senior admins for the best way to do this

```bash
cd /opt/warewulf/src
make clean defaults PREFIX=/opt/warewulf
make all
make install
```

### Populate iPXE files

```bash
./scripts/build-ipxe.sh
cp -i /opt/warewulf/share/ipxe/bin-x86_64-efi-snponly.efi /var/lib/tftpboot/warewulf/ipxe-snponly-x86_64.efi
```

### Initial Configuration

```bash
wwctl configure --all
systemctl enable --now warewulfd
```

### Building the Containers

Warewulf images are really just specialized docker/podman/containerd/etc... containers. Management can be done in 2 ways:
1. pulling a base image straight into warewulf and managing it by chrooting into it and making stated changes
2. using podman to pull a base image, make a derivative image using dockerfiles, and export that image to warewulf

The former works just fine, and is currently the motis operendi of HPC{sup}`2` admins. The latter is an appealing alternative as it makes updates, and other scripting easier in the long run. As such, we'll be moving forward primarily with the 2nd approach, reserving the first for small tweaks requried to get things working

### Install Podman
<!--TODO-->
```bash
dnf install -y docker
```

### Writing a Dockerfile
TODO

### Exporting Podman Images to Warewulf
<!--TODO-->
Exporting Images to warewulf is simple, both programs support tarball versions of images

```bash
docker save base-rocky9 -o $TMPDIR/base-rocky9.tar
wwctl container import $TMPDIR/base-rocky9.tar base-rocky9
```

### Defining Nodes
TODO

### Overlays
<!--TODO-->
Overlays work similarly to docker, but are used a bit differently here. 
We use Overlays to automatically create any files that may change over time or may be different between nodes. 
Read CIQ's [docs](https://warewulf.org/docs/v4.6.x/overlays/overlays.html) on the subject, then continue.

### Booting Nodes
TODO



