# my-bluefin &nbsp; [![bluebuild build badge](https://github.com/jsundh/my-bluefin/actions/workflows/build.yml/badge.svg)](https://github.com/jsundh/my-bluefin/actions/workflows/build.yml)

My personal [BlueBuild](https://blue-build.org/how-to/setup/) image based on `bluefin-dx`.

## Installation

> [!WARNING]  
> [This is an experimental feature](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable), try at your own discretion.

To rebase an existing atomic Fedora installation to the latest build:

- First rebase to the unsigned image, to get the proper signing keys and policies installed:

  ```sh
  rpm-ostree rebase ostree-unverified-registry:ghcr.io/jsundh/my-bluefin:latest
  ```

- Reboot to complete the rebase:

  ```sh
  systemctl reboot
  ```

- Then rebase to the signed image, like so:

  ```sh
  rpm-ostree rebase ostree-image-signed:docker://ghcr.io/jsundh/my-bluefin:latest
  ```

- Reboot again to complete the installation

  ```sh
  systemctl reboot
  ```

The `latest` tag will automatically point to the latest build. That build will still always use the Fedora version specified in `recipe.yml`, so you won't get accidentally updated to the next major version.

## SpaceMouse + Onshape in Firefox

The image ships [spacenavd](https://github.com/FreeSpacenav/spacenavd) (device driver, system service) and a [spacenav-ws](https://github.com/RmStorm/spacenav-ws) user service that serves the 3Dconnexion WebSocket protocol Onshape expects on `wss://127.51.68.120:8181`. A few one-time steps remain per user/Firefox profile:

1. Pair the SpaceMouse in GNOME Settings → Bluetooth.
2. In Firefox, install a userscript manager (Tampermonkey/Violentmonkey) and the "Onshape 3D-Mouse on Linux in-page patch" userscript linked from the [spacenav-ws README](https://github.com/RmStorm/spacenav-ws) — it patches Onshape's platform detection so it attempts the driver connection on Linux/Firefox.
3. Visit `https://127.51.68.120:8181` once in Firefox and accept the self-signed certificate (spacenav-ws auto-generates it; the exception is stored per Firefox profile).

Note: the first start of `spacenav-ws.service` downloads the package from PyPI into `~/.cache/uv`; after that it starts offline.

## ISO

If build on Fedora Atomic, you can generate an offline ISO with the instructions available [here](https://blue-build.org/learn/universal-blue/#fresh-install-from-an-iso). These ISOs cannot unfortunately be distributed on GitHub for free due to large sizes, so for public projects something else has to be used for hosting.

## Verification

These images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). You can verify the signature by downloading the `cosign.pub` file from this repo and running the following command:

```bash
cosign verify --key cosign.pub ghcr.io/jsundh/my-bluefin
```
