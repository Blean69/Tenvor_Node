# Install on Windows

## Before you start

- Windows 11, 64-bit, with virtualization turned on in the BIOS (Intel VT-x or AMD SVM).
- An NVIDIA card with 10 GB of video memory or more and a current driver.
- 32 GB of RAM, or 24 GB with a 12 GB or 16 GB card, or 16 GB with a 24 GB card.
- About 60 GB of free disk. The model is 22 GB.
- An administrator password once, to turn on the Linux environment (WSL2). Windows may ask for a restart.
- An invitation code. Request one at [tenvor.io/operators](https://tenvor.io/operators).

## Steps

1. If an earlier build is installed, quit it first (tray icon > Quit).
2. Download the setup file from the [Releases page](../../../releases) and run it. Windows shows "Windows protected your PC" because the build isn't signed. Click **More info**, then **Run anyway**.
3. Open Tenvor Node. It sets up its Linux environment and container runtime. If it asks for a restart, restart, then open it again.
4. On the Dashboard, type your invitation code into the **Invitation code** box and click **Enroll this node**. Dashes and spaces don't matter, and the code works once.
5. Leave it running. It downloads the model, fetches its GPU runtime, measures the card's speed, and then takes work. After the code is accepted this can take up to 15 minutes on top of the download. You can pause or quit from the tray icon.

## If the Dashboard says WSL is running with older resource limits

Click **Restart WSL…**, then **Restart WSL now**. This closes every WSL window and app on the PC, including Docker Desktop, then restarts the node.

Instead, you can quit Tenvor Node, run `wsl --shutdown` in PowerShell, and open it again.

## RTX 50-series cards

RTX 50-series cards (5060 to 5090) use their own GPU image. On each start, a 50-series node first tries the standard image, then switches by itself, so start-up takes about 6 to 7 minutes longer than on older cards. Serving on 50-series cards is new and not yet proven end to end.

## Remove it

Quit the app from the tray icon, then uninstall Tenvor Node from Windows Settings > Apps.

Uninstalling keeps the `.decompute` folder in your user folder. It holds your node identity and the downloaded model, about 22 GB. Delete the folder to free the space, but only if you no longer need that node identity.
