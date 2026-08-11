# Installing jtop on the AGX Thor

## What is jtop?

[`jtop`](https://github.com/rbonghi/jetson_stats) (jetson-stats) is a monitoring and control tool for
NVIDIA Jetson devices. It's the Jetson equivalent of `htop`/`nvtop`, giving you a live terminal
dashboard for:

- CPU, GPU, and memory usage
- Temperatures, power draw, and fan speed
- Jetson power modes (`nvpmodel`) and clock/jetson_clocks status
- L4T/JetPack version, CUDA, cuDNN, TensorRT, and other library versions
- A background `jtop.service` daemon that other tools (and the `jtop` CLI/API) read from

The AGX Thor is new enough that stock `jetson_stats` doesn't fully recognize it yet (missing
L4T/JetPack version mapping, module name, and CUDA table entries). Installing from pip alone will
either fail to identify the board correctly or not install at all, which is why this repo installs
from source with a set of Thor-specific patches applied.

## How `setup_Jtop_thor.sh` installs it

Run the script as root on the AGX Thor:

```bash
sudo bash setup_Jtop_thor.sh
```

It performs the following steps automatically:

1. **`install_prereqs`** – installs `git` and a Python venv package (`python3.12-venv`, falling
   back to `python3-venv`).
2. **`create_venv`** – creates an isolated, root-owned virtual environment at `/opt/jtop/venv` so
   jtop's dependencies don't pollute the system Python.
3. **`clone_repo`** – clones (or updates) the upstream [`jetson_stats`](https://github.com/rbonghi/jetson_stats)
   repository into `/opt/jtop/jetson_stats`.
4. **`apply_thor_edits`** – patches the cloned repo (with `.bak` backups) so it recognizes the
   AGX Thor:
   - Adds JetPack/L4T version mappings (e.g. `38.2.1`, `38.2.0`, `36.4.4`).
   - Adds the `tegra264` entry to the CUDA table.
   - Adds the `p3834-0008` module name for "NVIDIA Jetson AGX Thor (Developer kit)".
   - Rewrites `services/jtop.service` to launch `jtop` from the venv instead of
     `/usr/local/bin/jtop`.
5. **`install_python_fixes`** – runs an additional companion patch script
   (`patch_thor_jp7_in_repo.sh`, expected next to `setup_Jtop_thor.sh`) for any further JP7.0
   Python source fixes needed on Thor.
6. **`install_package`** – installs the patched `jetson_stats` repo into the venv with `pip`.
7. **`install_nvml`** – installs `nvidia-ml-py` (NVML/pynvml bindings) into the venv.
8. **`install_service`** –
   - Writes the `jtop.service` systemd unit to `/etc/systemd/system/jtop.service`.
   - Installs a `/usr/local/bin/jtop` wrapper script so `sudo jtop` works from anywhere on `PATH`.
   - Creates the `jtop` system group.
   - Reloads systemd, enables, and (re)starts the `jtop` service.

At the end it prints a summary with quick verification commands.

## Verifying the install

```bash
systemctl status jtop.service --no-pager
journalctl -u jtop --no-pager -e
```

Then launch the interactive dashboard with:

```bash
sudo jtop
```

## Notes / quirks

- The script must be run as **root** (`sudo bash setup_Jtop_thor.sh`); it exits early otherwise.
- It's safe to re-run: the venv, repo clone, and systemd unit are all idempotent (existing repo is
  reset to `origin/master`, existing venv is reused).
- `apply_thor_edits` keeps timestamped `.bak` copies of every file it patches inside the cloned
  repo, in case you need to diff or revert the Thor-specific changes.
- `install_python_fixes` expects a `patch_thor_jp7_in_repo.sh` script to exist alongside
  `setup_Jtop_thor.sh` — make sure that companion script is present before running the installer.
