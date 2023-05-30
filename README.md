# Creating a kubernetes cluster on multiple KVM virtual machines

<sup>FEDORA NOTICE YMMV</sup>

## Prerequisites

* [libvirt](https://libvirt.org/)
* [qemu-kvm](https://www.qemu.org/)
* [Vagrant](https://www.vagrantup.com/)
* [Ansible](https://www.ansible.com/)

## KVM

### Network bridge

1. Show active interfaces, take note of the one you want to bridge

    ```console
    $ nmcli con show --active
    NAME                UUID                                  TYPE      DEVICE
    Wired1              3e1e1b5d-1b0b-4b0e-9e0a-1b0b4b0e9e0a  ethernet  enp2s0
    CENSORED            45b17347-1a0c-4b0e-8d0f-4b0e8d0f4b0e  wifi      wlo1
    ```

1. Create bridge interface

    ```console
    $ nmcli con add ifname br0 type bridge con-name br0
    Connection 'br0' (f5075153-2aef-4a61-9be7-887021b54c8e) successfully added.
    ```

1. Link Bridge Interface to Real Interface (enp2s0 on this recipe)

    ```console
    $ nmcli con add type bridge-slave ifname enp2s0 master br0
    Connection 'bridge-slave-enp2s0' (b9b264d8-59c8-4200-85a6-bc6e221c392d)
    successfully added.
    ```

1. Bring up the new bridge, this'll take down the original interface

    ```console
    $ nmcli con up br0 && nmcli con up bridge-slave-enp2s0
    Connection successfully activated (master waiting for slaves) (D-Bus active
    path: /org/freedesktop/NetworkManager/ActiveConnection/10)
    Connection successfully activated (D-Bus active path: /org/freedesktop/Netwo
    rkManager/ActiveConnection/11)
    ```

1. Verify libvirt networks:

    ```console
    $ sudo virsh net-list --all
    [sudo] password for user:
    Name          State    Autostart   Persistent
    ------------------------------------------------
    default       active   yes         yes
    host-bridge   active   yes         yes
    ```

1. Create a new network for the bridge:

    ```console
    $ cat > bridge.xml <<EOF
    <network>
        <name>host-bridge</name>
        <forward mode="bridge"/>
        <bridge name="br0"/>
    </network>
    EOF

    $ virsh net-define bridge.xml   # delete this file after
    Network host-bridge defined from bridge.xml
    ```

1. Start the network:

    ```console
    $ virsh net-start host-bridge
    Network host-bridge started

    $ virsh net-autostart host-bridge
    Network host-bridge marked as autostarted
    ```

1. Verify libvirt networks:

    ```console
    $ sudo virsh net-list --all
    [sudo] password for user:
    Name          State    Autostart   Persistent
    ------------------------------------------------
    default       active   yes         yes
    host-bridge   active   yes         yes
    ```
