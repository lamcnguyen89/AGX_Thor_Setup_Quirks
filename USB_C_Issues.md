To make USB-C Host Mode persistent across reboots, you need to modify the **Device Tree Blob (DTB)** that Jetson Linux loads during UEFI boot. By setting the dual-role controller to static `host` mode, the OS will bind the XUSB Host Controller driver immediately on startup instead of waiting for dynamic USB Power Delivery (PD) negotiation.





---

## 4. Workaround: Persistent Systemd Service (Alternative)

If you want to avoid modifying device trees, you can force host mode on every boot via a `systemd` startup script:

0. First temporarily Reset the USB-C port to host-mode to recognize drives or peripherals:

```bash
# Reset the USB role controller
echo "none" | sudo tee /sys/class/usb_role/usb2-0-role-switch/role

# Force to host mode
echo "host" | sudo tee /sys/class/usb_role/usb2-0-role-switch/role

```

1. Create a script `/usr/local/bin/enable-usb-host.sh`:
```bash
#!/bin/bash
# Wait for USB role controller sysfs node to appear
while [ ! -d /sys/class/usb_role/usb2-0-role-switch ]; do
    sleep 1
done

# Force role reset and switch to host
echo "none" > /sys/class/usb_role/usb2-0-role-switch/role
sleep 0.5
echo "host" > /sys/class/usb_role/usb2-0-role-switch/role

```


2. Make it executable:
```bash
sudo chmod +x /usr/local/bin/enable-usb-host.sh

```


3. Create a service file `/etc/systemd/system/usb-host-mode.service`:
```ini
[Unit]
Description=Force USB-C Port into Host Mode
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/enable-usb-host.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

```


4. Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable usb-host-mode.service
sudo systemctl start usb-host-mode.service

```



> **Important Reminder:** Once forced into persistent Host Mode, the primary USB-C port will no longer act as a device node. To flash the board again via `l4t_initrd_flash.sh` in the future, you will either need to boot into recovery mode (which overrides device tree roles) or temporarily select your default DTB entry in `/boot/extlinux/extlinux.conf`.

