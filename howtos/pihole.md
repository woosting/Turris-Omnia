# Install Pi-hole (router level add blocker)

> Hyperlinks used in this how-to are pointing to the LuCi interface of the open source Turris Omnia router).

1. Create a **new LXC container** (this how-to assumes either: _Ubuntu xenial_ or _Debian Jessie_).
2. Have your [router's DHCP server][1] assign a **static IP address** to the LXC container:
    1. **Hostname**: `pihole`.
    2. **MAC-Address**: found at:
      - Regular: The lxc container's configuration file (this remains to be tested!!!).
      - Turris Omnia: **[Services/Containers][2] > container > more > configure > ``lxc.network.hwaddr``**.
    3. **IPv4-Address**: Any address available in your network topology (e.g. `192.168.1.2`).
4. Make the containers **startup automatically**:
  - Regular: Add the line `lxc.start.auto = 1` to the container's configuration file.
  - Turris omnia: Edit configuration file `/etc/config/lxc-auto` to include:

    ```shell
    config container
      option name pihole
      option timeout 30
    ```
    Every container configured here will get started at the boot and each of them will be correctly halted during the shutdown. Set timeout option to specify how much time in seconds do the containers have to gracefully shutdown before being killed. The default value is 300.
5. Boot the container and gain entrance:

  ```shell
$ lxc-start -n pihole
$ lxc-attach -n pihole
```
6. Install base packages required to get the installer going:

  ```shell
$ apt-get install -y ca-certificates curl
```
7. Install pi-hole (ncurses installer):

  ```shell
$ curl -sSL https://install.pi-hole.net | bash
```
  1. When asked for (ipv4) nameservers, select `custom` and enter your router's ip (typically: `192.168.1.1`).
  2. Note the password at the end of the installation wizard (although it can be changed later too if you ever forget it: `pihole -a -p new_password`).
8. Check if the pi-hole server is properly running by logging into it; e.g `http://192.168.1.2/admin` (using the password noted in the previous step).
9. Change [DHCP settings] of the LAN interface to use the Pi-hole as a primary DNS and the router's as its fallback:
  - Turris omnia: 
    1. Navigate to: **DHCP Server > Advanced Settings**
    2. **DHCP-Options**: `6,192.168.1.2,192.168.1.1`
    3. Renew your existing IP/leases on your (desktop) clients (to make them aware of the new DNS server).
> Alternatively configure clients themselves to use the pihole as their DNS server statically.


<!-- REFERENCES -->

[1]:http://192.168.1.1/cgi-bin/luci/admin/network/dhcp
[2]:http://192.168.1.1/cgi-bin/luci/admin/services/lxc
[3]:http://192.168.1.1/cgi-bin/luci/admin/network/network/lan