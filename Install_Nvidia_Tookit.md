# Make CUDA commands available on the Jetson AGX Thor

If a command such as:

```bash
nvcc --version
```

returns `nvcc: command not found`, either the CUDA development tools are not
installed or the CUDA `bin` directory is not in your shell's `PATH`.

## 1. Check whether `nvcc` is installed

```bash
ls -l /usr/local/cuda/bin/nvcc
```

If that file exists, skip to [Add CUDA to your PATH](#2-add-cuda-to-your-path).
You can also run it immediately with its full path:

```bash
/usr/local/cuda/bin/nvcc --version
```

If `/usr/local/cuda` does not exist, look for a versioned CUDA installation:

```bash
ls -d /usr/local/cuda-* 2>/dev/null
```

For example, if this prints `/usr/local/cuda-13.0`, substitute that path for
`/usr/local/cuda` in the commands below. If no CUDA directory is found, install
the development tools:

```bash
sudo apt update
sudo apt install nvidia-cuda-dev
```

On Thor, use NVIDIA's JetPack package `nvidia-cuda-dev`. Do **not** install
Ubuntu's similarly named `nvidia-cuda-toolkit` package; it may not contain the
CUDA build intended for Jetson. If you want all JetPack SDK components instead
of only the CUDA development tools, install `nvidia-jetpack` (it requires more
than 15 GB of storage).

## 2. Add CUDA to your PATH

Test the setting in the current terminal first:

```bash
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
nvcc --version
```

`PATH` lets the shell find commands such as `nvcc`. `LD_LIBRARY_PATH` lets
programs find CUDA shared libraries at runtime.

To apply these settings to future Bash sessions, add them to `~/.bashrc`:

```bash
echo 'export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}' >> ~/.bashrc
source ~/.bashrc
```

Run the two `echo` commands only once so that duplicate lines are not added to
the file. Open a new terminal, or run `source ~/.bashrc`, after making changes.

## 3. Verify the setup

```bash
command -v nvcc
nvcc --version
```

`command -v nvcc` should print `/usr/local/cuda/bin/nvcc` (or the corresponding
versioned CUDA path), and `nvcc --version` should print the installed CUDA
compiler version.

If the full-path command works but the ordinary command does not, inspect the
active path with:

```bash
printf '%s\n' "$PATH" | tr ':' '\n'
```

Make sure `/usr/local/cuda/bin` is listed. Also note that `sudo` may use a
different restricted `PATH`; normally, compile CUDA programs as your regular
user rather than running `sudo nvcc`.

## References

- [NVIDIA Jetson AGX Thor CUDA setup](https://docs.nvidia.com/jetson/agx-thor-devkit/user-guide/latest/setup_cuda.html)
- [NVIDIA Jetson AGX Thor JetPack SDK setup](https://docs.nvidia.com/jetson/agx-thor-devkit/user-guide/latest/setup_jetpack.html)
