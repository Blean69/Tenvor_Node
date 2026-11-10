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

On first start the app walks you through setup. **Settings > Runtime & GPU Setup** checks and repairs the container runtime and GPU access.

On the Dashboard, type your invitation code into the **Invitation code** box and click **Enroll this node**. After the code is accepted the node fetches its GPU runtime and connects by itself. This can take up to 15 minutes on top of the model download.

## Worker as a service

This path was tested on Ubuntu 24.04. Ubuntu 22.04's Podman is too old for it. Run everything as your own user, not as root.

### What the host needs first

The desktop app sets these up for you. A headless node needs them in place before the worker starts:

- the NVIDIA driver, so that `nvidia-smi` shows your card
- rootless Podman:

  ```sh
  sudo apt install podman uidmap slirp4netns passt fuse-overlayfs crun dbus-user-session jq curl
  ```

- the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html), with its device list written where the worker reads it:

  ```sh
  sudo install -D -m 0644 /usr/share/vulkan/icd.d/nvidia_icd.json /etc/vulkan/icd.d/nvidia_icd.json
  sudo systemctl disable --now nvidia-cdi-refresh.path nvidia-cdi-refresh.service
  sudo nvidia-ctk cdi generate --feature-flag=no-additional-gids-for-device-nodes --output=/etc/cdi/nvidia.yaml \
    || sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
  sudo rm -f /var/run/cdi/nvidia.yaml
  podman run --rm --device nvidia.com/gpu=all docker.io/library/ubuntu:24.04 nvidia-smi -L
  ```

  The last command must print your card.

### Install and start the worker

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

### Prepare the runtime and enroll

A headless worker waits to be told. The desktop app does these two steps from its screens; here you call the worker's local interface yourself:

```sh
api() { curl -s -H "Authorization: Bearer $(jq -r .capability ~/.decompute/ipc.json)" -H 'Content-Type: application/json' "$@"; }
port=$(jq -r .port ~/.decompute/ipc.json)
api -X POST -d '{}' "http://127.0.0.1:$port/api/runtime/prepare"
read -r -p "Invitation code: " code
api -X POST -d "{\"invite_code\":\"$code\"}" "http://127.0.0.1:$port/api/node/enroll"
systemctl --user restart tenvor-worker
```

The enroll call answers `{"enrolled":true}`. After the restart the node downloads the model, fetches its serving image and connects. To see where it is:

```sh
port=$(jq -r .port ~/.decompute/ipc.json)
api "http://127.0.0.1:$port/api/status" | jq .orchestrator_lifecycle
```

`"state": "ready"` means the node is connected. The key in `ipc.json` changes at every start, so the `api` helper reads it each time.

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
