## Static ip

If you are running a classic Debian server without NetworkManager, you will want to edit the configuration file directly.

1. **Open the network interfaces configuration file:**
```bash
sudo nano /etc/network/interfaces

```


2. **Locate your interface.** You will likely see a line like `iface enp3s0 inet dhcp`.
3. **Change `dhcp` to `static**` and define your network details below it:
```text
# The primary network interface
auto enp3s0
iface enp3s0 inet static
    address 192.168.1.50
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4

```


4. **Save and exit** (In nano, press `Ctrl+O`, `Enter`, then `Ctrl+X`).
5. **Restart the networking service** to apply the changes:
```bash
sudo systemctl restart networking

```



---

## Verification

To verify that your new static IP is active, run:

```bash
ip a s

```

Look at your interface to ensure it shows the new IP address (e.g., `192.168.1.50`).