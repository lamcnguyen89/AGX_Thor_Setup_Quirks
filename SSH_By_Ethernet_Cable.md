When connecting two devices directly via an Ethernet cable, you don't have a router dynamically assigning IP addresses via DHCP. The cleanest solution is to configure a **dedicated static IP subnet** on the Ethernet interfaces of both machines.

By leaving the **Gateway blank** on the Ethernet connection, your client machine continues using Wi-Fi for internet access while routing SSH traffic through the direct cable link.

---

1. **Configure Host Linux Device:** Assign static IP and check SSH daemon.
1. Open a terminal on the host Linux device.
2. Identify your Ethernet interface name (e.g., `eth0` or `enp3s0`):

```bash
ip link

```

3. Set up a static IP using NetworkManager (`nmcli`) or GUI settings. For command line:

```bash
sudo nmcli con add type ethernet ifname enP2p1s0 con-name DirectEthernet ip4 192.168.1.14/24
sudo nmcli con up DirectEthernet

```

4. Verify OpenSSH server is installed and running:

```bash
sudo systemctl enable --now ssh

```


2. **Configure Client Computer:** Set static IP in same subnet without default gateway.
Assign an IP address in the **192.168.50.x** range (such as `192.168.50.1`) to your client's Ethernet port.

* **On Linux client (`nmcli`):**

```bash
sudo nmcli con add type ethernet ifname enP2p1S0 con-name DirectToHost ip4 192.168.1.11/24
sudo nmcli con up DirectToHost

```

* **On macOS:**
Go to **System Settings > Network > Ethernet > Details... > TCP/IP**. Set *Configure IPv4* to **Manually**, IP Address to `192.168.50.1`, Subnet Mask to `255.255.255.0`, and leave **Router blank**.
* **On Windows:**
Go to **Control Panel > Network Connections**, right-click Ethernet adapter > **Properties > IPv4 Properties**. Select *Use the following IP address*, enter IP `192.168.50.1`, Subnet mask `255.255.255.0`, and leave **Default Gateway blank**.


3. **Verify Link & Connect via SSH:** Test ping and initiate secure shell.
1. Connect the Ethernet cable directly between both RJ45 ports (modern network cards auto-detect crossover via Auto-MDIX).
2. Ping the host from your client machine to test connectivity:

```bash
ping 192.168.50.2

```

3. Connect via SSH:

```bash
ssh username@192.168.50.2

```


---

### Pro-Tip: Adding an SSH Config Alias

To avoid typing the IP every time, append an alias entry to your client's `~/.ssh/config` file:

```text
Host host-direct
    HostName 192.168.1.14
    User malneyugnfl

```

Then you can connect simply by running:

```bash
ssh host-direct

```