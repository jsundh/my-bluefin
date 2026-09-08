# AGENTS.md

## Repository

This repository defines a custom BlueBuild OS image based on Bluefin DX (Fedora Atomic / Universal Blue). `recipes/recipe.yml` is the source of truth for the base image, installed software, and module configuration. Installation and per-user setup are documented in `README.md`.

## Validation

```sh
bluebuild validate recipes/recipe.yml   # validate after recipe changes
bluebuild generate recipes/recipe.yml   # render the Containerfile for inspection
bluebuild build recipes/recipe.yml      # full local build; requires a container runtime
```

There is no test suite. Recipe validation checks configuration, not package installation or runtime behavior. Image builds run through `.github/workflows/build.yml`; desktop and hardware behavior needs verification on a running system. The generated `Containerfile` is gitignored.

## Architecture

Modules execute in recipe order. The `files` module copies `files/system/` onto the image root, with source paths mirroring destination paths. Trust-store regeneration must run after CA files are copied, and service enablement must run after custom unit files are installed.

- `files/gschema-overrides/` contains GNOME defaults compiled by the `gschema-overrides` module.
- `files/system/usr/share/xkeyboard-config-2/symbols/us` is an upstream keyboard-symbols file with a custom `dvorak` variant. Keep edits within that variant and preserve the `usr/share/xkeyboard-config-2/` destination path.
- `files/system/usr/lib/systemd/user/spacenav-ws.service` pins the SpaceMouse WebSocket bridge version in its `uvx` command. Browser setup is documented in `README.md`.
- Locally modified `/etc` files can shadow image defaults on upgrades; account for this when verifying configuration changes such as `files/system/etc/spnavrc`.

## Conventions

- Follow `.editorconfig` for formatting.
- `image-version: stable` follows the upstream stable channel; it does not pin a Fedora major version.
- Add new recipes to the `matrix.recipe` array in `.github/workflows/build.yml`.
- Image signing uses the `SIGNING_SECRET` repository secret. Never commit private signing keys; `cosign.pub` is the public key.
