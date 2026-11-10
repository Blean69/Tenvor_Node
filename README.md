# Tenvor Node

Tenvor Node turns a PC with an NVIDIA graphics card into a worker on the [Tenvor](https://tenvor.io) compute network. It runs AI inference jobs inside a sandboxed container and records what the node earns.

This repository publishes the node's downloads. It doesn't hold the node's source yet: the source will be added here, source-available, once its review is finished.

## Status: alpha, by invitation

- **Installing doesn't make a PC a serving node.** The network admits a node only after it enrolls with an invitation code. You can request one at [tenvor.io/operators](https://tenvor.io/operators).
- **These are alpha builds.** They are unsigned and they don't update themselves. A newer build means downloading and installing again.
- **Earnings are recorded against your node from its first paid job.** Payouts aren't open yet.

## Downloads

Get the files from the [Releases page](../../releases).

| Your PC | File to download | Guide |
|---|---|---|
| Windows 11 | `Tenvor-Node_…_windows-x64-UNSIGNED-setup.exe` | [Install on Windows](docs/install-windows.md) |
| Ubuntu or Debian with a desktop | `tenvor-node_…_amd64.deb` | [Install on Linux](docs/install-linux.md) |
| Linux without a desktop | `tenvor-worker_…_linux-x64` | [Install on Linux](docs/install-linux.md#worker-as-a-service) |

The Windows and Linux files in a release are built from the same source revision.

## What your PC needs

- **System:** Windows 11 (64-bit, with virtualization turned on in the BIOS), or Ubuntu 22.04 or newer.
- **Graphics card:** an NVIDIA RTX 20-, 30-, 40- or 50-series card with at least 10 GB of video memory and a current driver. Serving on 50-series cards is new and not yet proven end to end.
- **Disk:** about 60 GB free. The model alone is 22 GB.
- **Memory:** it depends on the card, because a card with more memory of its own keeps more of the model off the PC's RAM.

| Graphics card memory | RAM your PC needs |
|---|---|
| 10 GB | 32 GB |
| 12 GB or 16 GB | 24 GB |
| 24 GB | 16 GB |

After setup the node measures the card's speed. A card that is too slow for the current model isn't given work.

## Check a download

Each release has a `SHA256SUMS.txt`. Put it in the folder with the files you downloaded.

Linux:

```sh
sha256sum --check --ignore-missing SHA256SUMS.txt
```

Windows (PowerShell), then compare the result with the line in `SHA256SUMS.txt`:

```powershell
Get-FileHash .\Tenvor-Node_0.1.24-alpha.1_windows-x64-UNSIGNED-setup.exe -Algorithm SHA256
```

A matching checksum shows the file arrived intact. It doesn't prove who published it, so download only from this repository's Releases page.

## What the node does on your PC

- It downloads the model and its container images from `huggingface.co` and `ghcr.io`.
- It connects to `orchestrator.tenvor.io` and takes jobs only while the app or worker is running and not paused.
- On Windows it turns on the Windows Subsystem for Linux (WSL2) and sets WSL's memory and processor limits.
- It keeps the node's identity in `.decompute/config.json` in your user folder. Back that file up and keep it private: anyone who has it can act as your node.

## Licences

- Tenvor's own software is offered under [PolyForm Noncommercial 1.0.0](LICENSE.md). Operating an official build as a node on the Tenvor network is covered by the [Provider Terms](https://tenvor.io/legal/provider-terms). By downloading, installing or running a build you accept them.
- The builds include third-party software under its own licences. Every release carries the notices for its files as `THIRD-PARTY-NOTICES` text files.
- The Tenvor name and logo aren't licensed for other products.

## Feedback and security

Open an issue with your operating system, the file you installed, what you did and what happened. An exported diagnostic helps (Logs, then Export diagnostic). It leaves out addresses, folders and keys, but read it before you share it. Don't post invitation codes, your `config.json`, or anything private.

To report a vulnerability, contact the repository owner privately before sharing details in public.
