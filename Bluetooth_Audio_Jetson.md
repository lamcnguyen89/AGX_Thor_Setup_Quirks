By default, NVIDIA explicitly disables Bluetooth audio profiles (`audio`, `a2dp`, `avrcp`) in Jetson Linux for Bluetooth compliance and conformance testing. To use Bluetooth headphones or speakers, you need to re-enable these plugins in the BlueZ systemd daemon and install the necessary audio stack modules.

*(Note: Ensure you have an M.2 Key-E Wi-Fi/Bluetooth module or a compatible USB Bluetooth dongle plugged in, as the Jetson Orin Nano Developer Kit carrier board does not include built-in wireless hardware on the PCB itself.)*

---

1. **Modify the BlueZ Service Configuration:**
Open the NVIDIA-specific Bluetooth service override file in `nano` or your preferred text editor:

```bash
sudo nano /lib/systemd/system/bluetooth.service.d/nv-bluetooth-service.conf

```

Locate the `ExecStart` line:

```text
ExecStart=/usr/lib/bluetooth/bluetoothd -d --noplugin=audio,a2dp,avrcp

```

Remove the `--noplugin=audio,a2dp,avrcp` argument so it matches:

```text
ExecStart=/usr/lib/bluetooth/bluetoothd -d

```

Save the file (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`).


2. **Install Bluetooth Audio Support Packages:**
Update your package repository and install the Bluetooth audio integration module:

```bash
sudo apt-get update
sudo apt-get install -y pulseaudio-module-bluetooth

```


3. **Reload and Restart the Bluetooth Service:**
Reload systemd unit files and restart the Bluetooth daemon:

```bash
sudo systemctl daemon-reload
sudo systemctl restart bluetooth

```

*(A system reboot using `sudo reboot` is recommended if PulseAudio doesn't immediately pick up the new interface.)*


4. **Pair and Connect Your Audio Device:**
Use `bluetoothctl` to pair and connect your Bluetooth audio device:

```bash
bluetoothctl

```

Inside the interactive prompt, run:

```text
power on
agent on
default-agent
scan on

```

Once your device MAC address appears (e.g., `XX:XX:XX:XX:XX:XX`), connect and trust it:

```text
pair XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
exit

```


---

### Verification

To confirm audio routing, verify that the A2DP profile sink is active:

```bash
pactl list sinks short

```

You should see an output sink named `bluez_sink...`. You can set it as default or manage output streams using `pavucontrol` or the system sound settings GUI.