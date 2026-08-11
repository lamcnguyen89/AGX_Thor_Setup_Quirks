That **480 Mb/s (USB 2.0 High-Speed)** speed caps real-world throughput right around **40–42 MB/s**. When SuperSpeed (5 Gbps / 10 Gbps) fails and drops to USB 2.0 on the AGX Thor, it is almost always caused by **UPHY (Universal PHY) lane configuration** or **Realtek hub link training timeouts**.

Here is how to diagnose and force SuperSpeed negotiation on your USB ports.

---

## Step 1: Diagnose the Link State

Check your USB hierarchy to verify where the speed degradation is happening:

```bash
lsusb -t

```

* **If it displays `480M**` under an `xhci-hcd` root hub: The Tegra xHCI controller failed SuperSpeed link training or fell back to the UTMI (USB 2.0) PHY.
* **If it displays `5000M` or `10000M**`: The port itself is operating in USB 3.x mode, but the connected device, cable, or active hub is negotiating at USB 2.0.

---

## Step 2: The Core Fixes

### 1. Re-trigger SuperSpeed Link Negotiation

When the onboard USB-A controller or Type-C PHY misses its initial hardware handshake at boot, you can unbind and re-bind the Tegra xHCI driver to force it to re-train the High-Speed Lanes (UPHY):

```bash
# Unbind the xHCI driver
echo "3550000.usb" | sudo tee /sys/bus/platform/drivers/tegra-xusb/unbind

# Wait 2 seconds
sleep 2

# Re-bind the xHCI driver
echo "3550000.usb" | sudo tee /sys/bus/platform/drivers/tegra-xusb/bind

```

*Run `lsusb -t` after this step to see if the port changes to `5000M` or `10000M`.*

### 2. Disable USB Autosuspend & Power Gating

Power management on JetPack can cause SuperSpeed physical lanes to power down into low-power states (U1/U2) and fail to wake up properly, dropping the link to legacy USB 2.0:

1. Temporarily disable USB runtime power management:
```bash
sudo sh -c 'for i in /sys/bus/usb/devices/*/power/control; do echo "on" > "$i"; done'

```


2. Disable runtime PM permanently by appending `usbcore.autosuspend=-1` to your kernel command line in `/boot/extlinux/extlinux.conf`:
```text
APPEND ${cbootargs} root=/dev/nvme0n1p1 rw rootwait usbcore.autosuspend=-1

```



### 3. Check UPHY Lane Assignment (Device Tree / UEFI Firmware)

On the AGX Thor carrier board, USB 3.0 requires explicit UPHY lane mapping in the device tree. If the device tree maps a port only to UTMI PHYs (`p2u`), it will physically act as a USB 2.0 port regardless of the physical connector.

Check if SuperSpeed PHY nodes are active on your current tree:

```bash
dmesg | grep -i -E "xusb|phy|uphy|ss_port"

```

Look for error lines such as `Phy link training failed` or `Failed to setup SuperSpeed port`. If you see link training errors, updating your UEFI capsule firmware or applying the UPHY device tree overlay is required to align the hardware state with JetPack.

---

## Quick Cable & Hardware Troubleshooting Checklist

* **Orientation Sensitivity:** On Type-C ports running in Host mode without an active hardware mux driver, flippable CC lane logic can drop SuperSpeed on one side. Try unplugging the USB-C cable, **flipping it 180 degrees**, and plugging it back in.
* **USB 3.0 Extension/Hub Check:** Ensure you are testing directly into the AGX Thor connector without unshielded extension cables; USB 3.0 signal integrity on edge devkits is sensitive to noise in the 2.4 GHz–5 GHz band.