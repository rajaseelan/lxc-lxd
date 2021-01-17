# Resource Limits
1. Set memory to 2G for container `myvm`
    ```
    lxc config set myvm limits.memory 2G
    ```

1. Set memory limits on profile `myprofile` to `512MB`
    ```
    # create a new profile
    lxc profile copy default myprofile

    # edit profile
    lxc profile edit myprofile

    config: 
      limits.memory: 512MB
    description: Default LXD profile
    devices:
      eth0:
        name: eth0
        nictype: bridged
        parent: lxdbr0
        type: nic
      root:
        path: /
        pool: dirmap
        type: disk
    name: myprofile
    used_by: []
    ```

1. Launch a `debian/10` based container name `myvm2` with profile `myprofile`
    ```
    lxc launch debian/10 myvm2 --profile myprofile
    ```

Other options:  

Option              | Description
limits.cpu          | 
limits.memory       | 
security.privilege  |
security.nesting    | 