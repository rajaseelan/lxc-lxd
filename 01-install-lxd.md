# LXD

#### Table of Contents
[Installation](#installation)  
[Initialization](#initialization)  
[Storage Pools](#storagepools)  
[Launch Containers](#launchcontainers)  

<a name="installation"></a>

## Installation

1. Get lxd version
    ```
    lxc --version
    ```
1. Install `lxc`
    ```
    sudo apt install lxc-utils
    ```
1. Install lxd using preferred method (snap packages)
   ```
   sudo snap install lxd --channel=3.0/stable
   ```
1. Remove preinstalled lxd
   ```
   sudo apt remove lxd lxd-client
   ```
1. Add user to lxd group
   ```
   sudo myusername cloud_user -aG lxg
   # logout / login
   ```
1. Test lxd version again
   ```
   lxc --version
   ```
---

<a name="initialization"></a>

## Initialization
1. initialize
    ```
    lxd init
    Would you like to use LXD clustering? (yes/no) [default=no]: 
    Do you want to configure a new storage pool? (yes/no) [default=yes]: 
    Name of the new storage pool [default=default]: 
    Name of the storage backend to use (btrfs, ceph, dir, lvm, zfs) [default=zfs]: 
    Create a new ZFS pool? (yes/no) [default=yes]: 
    Would you like to use an existing block device? (yes/no) [default=no]: 
    Size in GB of the new loop device (1GB minimum) [default=15GB]: 
    Would you like to connect to a MAAS server? (yes/no) [default=no]: 
    Would you like to create a new local network bridge? (yes/no) [default=yes]: 
    What should the new bridge be called? [default=lxdbr0]: 
    What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
    What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
    Would you like LXD to be available over the network? (yes/no) [default=no]: 
    Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
    Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: yes
    config: {}
    networks:
    - config:
        ipv4.address: auto
        ipv6.address: auto
      description: ""
      managed: false
      name: lxdbr0
      type: ""
    storage_pools:
    - config:
        size: 15GB
      description: ""
      name: default
      driver: zfs
    profiles:
    - config: {}
      description: ""
      devices:
        eth0:
          name: eth0
          nictype: bridged
          parent: lxdbr0
          type: nic
        root:
          path: /
          pool: default
          type: disk
      name: default
    cluster: null
    ```
1. How to use preseed file?
    ```
    cat preseed.yaml | lxd init --preseed
    ```
---

<a name="storagepools"></a>

## Storage Pools
1. Create a storage pool using `btrfs` and the block device `/dev/vdb`
    ```
    lxc storage create experimental btrfs  source=/dev/vdb 
    ```
1. List Storage pools
    ```
    lxc storage list 
    +--------------+-------------+--------+--------------------------------------------+---------+
    |     NAME     | DESCRIPTION | DRIVER |                   SOURCE                   | USED BY |
    +--------------+-------------+--------+--------------------------------------------+---------+
    | default      |             | zfs    | /var/snap/lxd/common/lxd/disks/default.img | 1       |
    +--------------+-------------+--------+--------------------------------------------+---------+
    | experimental |             | btrfs  | 5b469cac-35ab-4856-8d8b-8cd4bd192d8c       | 0       |
    +--------------+-------------+--------+--------------------------------------------+---------+
    ```
1. Get info on strage pool `experimental`
    ```
    lxc storage info experimental
    ```
1. Show configuration of storage pool `production`
    ```
    lxd storage show production
    ```
1. Configure `default` profile to use the `experimental` profile by _default_.
    ```
    lxc profile edit defaul1
    ```
1. Delete a storage pool `production`
    ```
    lxc storage delete production
    ```
1. Set the source for storage pool `default` to `/dev/nvmep1`
    ```
    lxc storage set default source /dev/nvmep1
    ```
1. Unset a value of storage pool
    ```
    lxc storage unset default-pool size 
    ```
---
<a name="launchcontainers"></a>

## Launching Containers
1. Where can you view a list of lxc images ?
  [images.linuxcontainers.org](http://images.linuxcontainers.org/)

1. Launch an `alpine:3.10` container
    ```
    lxc launch images:alpine/3.10 -s default
    ```
1. _Only_ pull down an `ubuntu:18.04` image to our `local` remote and assign it an alias `ubuntu-18.04`
    ```
    lxc image copy ubuntu:18.04 local: --alias ubuntu-18.04
    ```

1. launch an `ubuntu:18.04` container with the name `web01`
    ```
    lxc launch ubuntu-18.04 web01
    ```

1. List containers
    ```
    lxc list
    ```

## Instance Configuration

1. Show configuration for container `web01`
    ```
    lxc config show web01
    ```

1. Edit the container configuration
    ```
    lxc config edit web01
    ```

1. Select options that can be configured from the command line:
    ```
    * autostart times
    * Priority
    * enviroment
    * User key /values inside container
    * hardware limits / prioritization
    * graphic card configurations
    * security policies
      * AppArmor
      * SELinux
    * Raw Config for
      * AppArmor
      * LXC
      * ipmap
      * QEMU
      * seccomp
    ```

1. Prevent container fromstarting at boot
    ```
    lxc config set my-container boot.autostart false
    ```

1. Get value of `boot.autostart`
    ```
    lxc config get my-container boot.autostart
    ```

1. Configure physical space used by containers
    ```
    lxc config device
    ```

1. View disk for `my-container`
    ```
    lxc config device list my-container
    ```

1. Get specific configuration values
    ```
    # what pool is the root device of my-container using
    lxc config device get my-container root pool
    ```
---
## Accessing a Container
1. Start a container 
    ```
    lxc start my-container
    ```

1. Access a shell on server
    ```
    lxc exec web01 -- /bin/bash
    ```

1. Run chained commands
    ```
    lxc exec web01 -- sh -c 'systemctl start nginx && systemctl enable nginx'
    ```

1. Delete container
    ```
    lxc stop my-container
    lxc delete my-container

    # or 
    lxc delete my-container --force
    ```
---

<a name="files"></a>

## Working with Files
1. Edit a file on the container
    ```
    lxc file edit web01/etc/nginx/sites-available/default 
    ```

1. Copy file from container
    ```
    mkdir web-seerver-configs
    lxc file pull web01/etc/nginx/sites-available/default webserver-configs/containerhub

    # Continue editing file
    ```

1. Push file back to container
    ```
    lxc file push webserver-configs/containerhub web01/etc/nginx/sites-available/containerhub
    ```

1. Edit file again just to check that it worked
    ```
    lxc file edit web01/etc/nginx/sites-available/containerhub 
    ```

1. Delete file
    ```
    lxc file delete  web01/etc/nginx/sites-enabled/default 
    ```

1. Delete file
    ```
    lxc file delete  web01/etc/nginx/sites-enabled/default 
    ```

---
## Networking
How does it connect?  
```
lxdbr0 ->  veth  ->    veth      ->     eth0
(host)    (host)    (container)      (container)
```

Any additional containers can also talk to each other.  

1. Copy over `alpine/3.11` image as `alpine3.11` in your _local remote_
    ```
    lxc image copy images:alpine/3.11 local: --alias alpine3.11 
    ```

1. Create a container `db01` from local image `alpine3.11`
    ```
    lxc db launch apline3.11 db01
    ```

1. Get information about container `db01`
    ```
    lxc info db01
    ```

1. Ping container `db01` from `web01`
    ```
    # get a shell in web01
    lxc exec web01 -- /bin/bash

    # ping db01 by ip
    ping 10.x.x.x # success

    # ping db01 by hostname
    ping db01 # success as well!!
    ```
---

## Profiles
* Containers a currently being created using the default profile

1. List profiles
    ```
    lxc profile list
    ```

1. Get information about the `default` profile
    ````
    lxc profile show default
    ````

1. Wipeout partitions uzing gdisk
    ```
    sudo gdisk /dev/nvmen1

    # access expert mode
    x

    # wipeout everything, y to delete mbr as well
    y , y
    ```

1. Create a profile that uses the `experimental` storage pool by default
    ```
    # copy default profile
    lxc profile copy default experimental

    # edit profile
    lxc profile edit experimental

    config: {}
    description: Default LXD profile
    devices:
      eth0:
        name: eth0
        nictype: bridged
        parent: lxdbr0
        type: nic
      root:
        path: /
        pool: experimental
        type: disk
    name: experimental
    used_by: []
    ```

1. launch a new `alpine3.11` container using the `experimental` profile
    ```
    lxc launch alpline3.11 test -p experimental
    ```

1. Delete `test` container
    ```
    lxc delete test --force
    ```

1. Create a sample profile for a web server
    ```
    # Copy default profile to web
    lxc profile copy default web

    # set autostart, stop priority
    lxc profile set web boot.autostart.priority 50
    lxc profile set web boot.stop.priority 50
    lxc profile set web environment.EDITOR vim
    ```

1. Add profile to exiting `web01` container
    ```
    lxc profile add web01 web
    ```

1. Remove `default` profile from `web01` container
    ```
    lxc profile remove web01 default
    ```
---

## Snapshots

1. Create a snapshot
    ```
    lxc snapshot mycontainer container-snapshot-name
    ```

1. Create a container based on snapshot
    ```
    lxc copy mycontainer/snapshot-name new-container
    ```

1. Delete a snapshot
    ```
    lxc delete mycontainer/snapshot-name
    ```

1. View infomation about snapshot
    ```
    lxc info mycontainer 

    # look for Snapshots: key
    ```
---
## Images

## Image Remotes
1. What is an Image Remote
    <details>
    <summary>Answer</summary>
    A place to get lxd image from
    </details>

1. List remotes
    ```
    lxc remote list
    ```
1. Add A remote
    ```
    lxc remote add custom 3.4.5.6 

    # verify by running a lxc remote list 
    ```

1. List images on remote named `custom`
    ```
    lxc list custom:
    ```

1. Copy image for `alpine/3.11` from a public remote to our `custom` remote
    ```
    lxc image copy images:alpine/3.11 custom: --alias alpine
    ```

1. List images in `custom` remote
    ```
    lxc image list custom:
    ```

1. Launch an image from our custom remote _in_ the custom remote
    ```
    lxc launch custom:alpine custom:test
    ```

1. View server in `custom` remote created
    ```
    lxc list custom:
    ```

1. Stop the container running in the remote named `custom`
    ```
    lxc stop custom:test
    ```

1. Delete the container in custom
    ```
    lxc delete custom:test
    ```
