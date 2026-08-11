To make USB-C Host Mode persistent across reboots, you need to modify the **Device Tree Blob (DTB)** that Jetson Linux loads during UEFI boot. By setting the dual-role controller to static `host` mode, the OS will bind the XUSB Host Controller driver immediately on startup instead of waiting for dynamic USB Power Delivery (PD) negotiation.

Here is the step-by-step process to compile and deploy a custom DTB.

---

## 1. Locate and Edit the Device Tree Source (DTS)

On your host development PC (or directly on the Jetson if you have the Linux kernel sources installed), locate the main device tree source file for your AGX Thor carrier board (`p4071` / `tegra264`).

1. Navigate to your Linux for Tegra (L4T) kernel source directory:
```bash
cd Linux_for_Tegra/source/hardware/nvidia/t264/boards/p4071/

```


2. Open `tegra264-p4071-0000.dtsi` (or the corresponding USB overlay file `tegra264-p4071-usb.dtsi`):
3. Locate the xHCI / USB-C controller node (`usb@3550000` or the primary Type-C connector node).
4. Change the `dr_mode` property from `"otg"` or `"typec"` to `"host"`:
```dts
/* Inside tegra264-p4071-0000.dtsi */

xusb@3550000 {
    status = "okay";
    phys = <&p2u_hsio_3>, <&p2u_hsio_4>;
    phy-names = "p2u-0", "p2u-1";

    connector {
        compatible = "usb-c-connector";
        label = "USB-C J81";
        /* Change from "otg" / "dynamic" to explicit "host" */
        dr_mode = "host"; 
    };
};

```


5. Disable the dynamic role-switch driver for that node if it is currently overriding hardware state at runtime:
```dts
usb_role_switch {
    status = "disabled";
};

```



---

## 2. Recompile the Device Tree (DTB)

If you are compiling directly on the Jetson dev kit:

1. Decompile your current running device tree to inspect and edit directly:
```bash
sudo dtc -I fs -O dts -o extracted.dts /proc/device-tree

```


2. Open `extracted.dts`, search for `dr_mode`, change its value to `"host"`, and recompile it to binary:
```bash
sudo dtc -I dts -O dtb -o custom_kernel.dtb extracted.dts

```



---

## 3. Deploy the Custom DTB

### Option A: Via UEFI Boot Configuration (Recommended - No Flashing Required)

You can tell UEFI to load your custom DTB directly from disk without re-flashing the entire board:

1. Copy your compiled `custom_kernel.dtb` to the `/boot/` partition:
```bash
sudo cp custom_kernel.dtb /boot/custom_kernel.dtb

```


2. Edit `/boot/extlinux/extlinux.conf`:
```text
TIMEOUT 30
DEFAULT primary

MENU TITLE Jetson AGX Thor Boot Options

LABEL primary
    MENU LABEL Primary Kernel
    LINUX /boot/Image
    FDT /boot/custom_kernel.dtb
    INITRD /boot/initrd
    APPEND ${cbootargs} root=/dev/nvme0n1p1 rw rootwait rootfstype=ext4

```


3. Save and reboot. UEFI will pick up `custom_kernel.dtb` during boot and initialize the USB PHYs in Host Mode.

### Option B: Flash the DTB Partition (Host PC)

If you prefer to write the DTB directly into the boot firmware partition using your host PC:

1. Copy `custom_kernel.dtb` into your host machine's `Linux_for_Tegra/kernel/dtb/` folder.
2. Put the Jetson into Recovery Mode.
3. Flash **only** the device-tree partition:
```bash
sudo ./flash.sh -k kernel-dtb jetson-agx-thor-devkit mmcblk0p1

```



---

## 4. Workaround: Persistent Systemd Service (Alternative)

If you want to avoid modifying device trees, you can force host mode on every boot via a `systemd` startup script:

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

