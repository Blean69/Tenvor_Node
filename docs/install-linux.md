# Install on Linux

## Before you start

- Ubuntu 22.04 or newer (it was built on Ubuntu 22.04 and tested on Ubuntu 24.04), x86-64.
- An NVIDIA RTX 20-, 30-, 40- or 50-series card with 10 GB of video memory or more, and a current driver.
- 32 GB of RAM, or 24 GB with a 12 GB or 16 GB card, or 16 GB with a 24 GB card.
- About 60 GB of free disk. The model is 22 GB.
- An invitation code. Request one at [tenvor.io/operators](https://tenvor.io/operators).

## Pick what to install

- **A computer with a desktop:** install the desktop app. It includes a worker and starts it for you.
- **A node without a desktop:** install the worker as a service.

## Desktop app

Needs a graphical desktop session with working OpenGL/EGL drivers.

```sh
sudo apt install ./tenvor-node_0.1.24-alpha.1_amd64.deb
```

Then open "Tenvor Node" from your applications menu.

On first start the app walks you through setup. **Settings > Runtime & GPU Setup** checks and repairs the container runtime and GPU access. When the app says the node needs enrollment, enter your invitation code.

## Worker as a service

Run these as your own user, not as root.

Install the worker under a fixed name:

```sh
install -D -m 0755 tenvor-worker_0.1.24-alpha.1_linux-x64 \
  ~/.local/lib/tenvor/decompute-worker
```

Create `~/.config/systemd/user/tenvor-worker.service` with this content:

```ini
[Unit]
Description=Tenvor worker
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=0

[Service]
Type=simple
WorkingDirectory=%h/.decompute
Environment=TENVOR_SUPERVISOR=systemd
EnvironmentFile=-%h/.decompute/worker.env
ExecStart=%h/.local/lib/tenvor/decompute-worker
Restart=on-failure
RestartSec=30
RestartForceExitStatus=75
TimeoutStopSec=180
KillMode=control-group
SendSIGKILL=no
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=default.target
```

Start it, and keep it running while you are logged out:

```sh
mkdir -p ~/.decompute
systemctl --user daemon-reload
systemctl --user enable --now tenvor-worker
loginctl enable-linger "$USER"
```

Stopping can take up to three minutes while the worker shuts down its serving container. Let it finish.

Follow the logs with:

```sh
journalctl --user -u tenvor-worker -f
```

You can run the desktop app beside a worker service. The app adopts the running worker and never starts a second one.

## RTX 50-series cards

RTX 50-series cards (5060 to 5090) use their own GPU image. On each start, a 50-series node first tries the standard image, then switches by itself, so start-up takes longer than on older cards. Serving on 50-series cards is new and not yet proven end to end.

## Your node identity

`~/.decompute/config.json` holds your node identity. Back it up and keep it private: anyone with this file can act as your node. Removing the package keeps this folder.

Models are stored in `~/.decompute/models` unless you choose another folder in Settings.
