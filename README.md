# Homelab Network Setup: MikroTik hEX S + TP-Link TL-SG108E

Setting up the network for my home lab: a MikroTik hEX S router, a TP-Link TL-SG108E managed switch, and an Ubuntu laptop I'm using as a practice server. Everything on the router was done from the command line over SSH from the server.

## Hardware

| Device | Model | Role |
|---|---|---|
| Router | MikroTik hEX S (E60iUGS) | Gateway, DHCP, DNS, firewall |
| Switch | TP-Link TL-SG108E v6 | Managed switch (IP from DHCP) |
| Server | Ubuntu laptop (`laptop`) | Practice server / SSH client (IP from DHCP) |

> IP and MAC addresses are blurred in the screenshots and written as `<router-ip>` / `<switch-ip>` below.

## What I did

1. Connected to the router over SSH
2. Reviewed the default config
3. Changed the admin password
4. Updated RouterOS from 7.20.8 to 7.25.1
5. Checked the connection after the reboot
6. Upgraded the RouterBOARD firmware to match
7. Set the time zone and saved a backup
8. Checked DHCP leases
9. Checked the switch's web UI

---

### 1. First SSH login

```bash
ssh admin@<router-ip>
```

![First SSH login](images/01-first-ssh-login.png)

### 2. Default configuration

On first login, RouterOS shows the default config. The WAN port (`ether1`) runs a DHCP client and is protected by the firewall. The other ports are bridged as the LAN subnet, with a DHCP server, DNS and NAT turned on.

![Default config](images/02-default-config.png)

### 3. Change the admin password

The first thing to do on a new router.

![Password changed](images/03-password-changed.png)

### 4. Update RouterOS

```
/system package update check-for-updates
/system package update install
```

The router downloads the update and reboots on its own, which ends the SSH session.

![RouterOS update](images/04-routeros-update.png)

### 5. Check connectivity after the reboot

```bash
ping <router-ip>
```

33/33 packets came back with 0% loss and about 0.5 ms average latency.

![Ping](images/05-ping-router.png)
![Ping stats](images/06-ping-stats.png)

### 6. Upgrade the RouterBOARD firmware

Updating RouterOS doesn't update the board firmware, so that's a separate step and needs another reboot.

```
/system routerboard upgrade
/system reboot
```

![Firmware upgrade](images/07-firmware-upgrade.png)

To confirm the firmware matches RouterOS:

```
/system routerboard print
```

`current-firmware` and `upgrade-firmware` both show `7.25.1`.

![Firmware verified](images/08-firmware-verified.png)

### 7. Set the time zone and save a backup

```
/system clock set time-zone-name=America/Chicago
/system backup save name=clean-setup
```

My first try failed with `bad parameter` because I ran both commands on one line. They need to be entered separately. The `clean-setup` backup is a known-good restore point.

![Timezone and backup](images/09-timezone-backup.png)

### 8. Check DHCP leases

```
/ip dhcp-server lease print
```

The switch (`TL-SG108E`) and the server (`laptop`) both got addresses from the router.

![DHCP leases](images/10-dhcp-leases.png)

### 9. Switch web UI

I logged into the TL-SG108E at `<switch-ip>`. It shows the router as its default gateway.

![Switch web UI](images/11-switch-webui.png)

---

## What I learned

- RouterOS updates and RouterBOARD firmware are two separate upgrades.
- Save a backup once you have a known-good config, before changing anything else.
- In the RouterOS CLI, each command goes on its own line.
- `/ip dhcp-server lease print` is a quick way to see what's connected and confirm DHCP works.

## Next steps

- [ ] Set up VLANs on the TL-SG108E
- [ ] Give the server a static DHCP lease
- [ ] Harden the router: turn off unused services, use SSH keys
