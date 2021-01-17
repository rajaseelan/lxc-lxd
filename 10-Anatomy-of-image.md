# Anatomy of a lxd image

1. Export image `alpine3.11` for inspecting
    ```
    lxc image export alpine3.11 apline/
    ```

1. What is in directory?
    ```
    $ ls -lh
    total 2.5M
    -rw-rw-r-- 1 student student 2.5M Jan 17 10:51        047a3af6882e61b8ac536b3216e60f6b17f83763ca1ee8978b8735d4fcd96ef4.squashfs
    -rw-rw-r-- 1 student student  868 Jan 17 10:51        meta-047a3af6882e61b8ac536b3216e60f6b17f83763ca1ee8978b8735d4fcd96ef4.tar.xz
    ```

    `squashfs`  has everything - can be mounted

1. Mount squashfs filesystem to view it
    ```
    sudo mkdir /dev/expand-alpine
    sudo mount 047a3af6882e61b8ac536b3216e60f6b17f83763ca1ee8978b8735d4fcd96ef4.squashfs /dev/expand-alpine/

    # inspect it 
    ls /dev/expand-alpine
    bin  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
    ```

1. What does the `.tar.xz` archive contain?
    ```
    # Image metadata and templates
    metadata.yaml
    templates/
    templates/hostname.tpl
    templates/hosts.tpl
    templates/inittab.tpl    

    # contents of hosts.tpl
    127.0.1.1	{{ container.name }}
    127.0.0.1	localhost localhost.localdomain
    ::1		localhost localhost.localdomai
    ```

# Publishing Containers

1. Enable public access of lxd server's images
    ```
    lxc config set core.https_address 192.168.x.x
    ```

1. Turn lxc snapshot named `1.0` from container `web01` from  `local` registry in to the registry named `custom` as image `nginx-ubuntu`
    ```
    lxc publish local:web01/1.0

    # the image will be referred to as web01/1.0
    ```

1. List the image in your `custom` remote
    ```
    lxc image list custom:
    ```

1. Launch a new container `web03` using an image named `nginx-ubuntu` from the remote called `custom` 
    ```
    lxc launch custom:nginx-ubuntu web03
    ```
---
# DistroBuilder