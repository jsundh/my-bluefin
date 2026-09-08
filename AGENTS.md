# AGENTS.md

This file provides guidance to coding agents working in this repository.

## What this repo is

A [BlueBuild](https://blue-build.org) custom OS image ("my-bluefin") based on `ghcr.io/ublue-os/bluefin-dx:stable` (Fedora Atomic / Universal Blue). There is no application code: the repo declaratively describes an OCI image that atomic Fedora systems rebase onto. The published image is `ghcr.io/jsundh/my-bluefin:latest`.

## Commands

The `bluebuild` CLI is installed locally:

```sh
bluebuild validate recipes/recipe.yml   # lint/validate the recipe (fast; run after recipe changes)
bluebuild generate recipes/recipe.yml   # render the Containerfile (output is gitignored)
bluebuild build recipes/recipe.yml      # full local image build (slow, needs container runtime)
```

There are no tests. Real builds happen in GitHub Actions (`.github/workflows/build.yml`): on push (except `**.md`-only changes), on PRs, weekly on Mondays 06:00 UTC, and via manual `workflow_dispatch`. Images are signed with cosign; the private key lives in the `SIGNING_SECRET` repo secret (`cosign.key`/`cosign.private` are gitignored — never commit them), and `cosign.pub` is the committed public key.

## Architecture

`recipes/recipe.yml` is the source of truth for installed software and BlueBuild module configuration.

Modules execute in recipe order. The `files` module copies `files/system/` onto the image root, with source paths mirroring destination paths. Trust-store regeneration must run after CA files are copied, and service enablement must run after any custom unit files are installed.

Supporting directories:

- `files/gschema-overrides/zz1-workspaces.gschema.override` — fixed 4 GNOME workspaces with `<Super>`-key switch/move bindings on the Dvorak home row (h/t/n/s).
- `files/system/usr/share/xkeyboard-config-2/symbols/us` — a full copy of the upstream xkeyboard-config `us` symbols file with a personally modified `dvorak` variant. The destination path matters: it must be `usr/share/xkeyboard-config-2/`, not `usr/share/X11/xkb/` (the latter was tried and reverted — see commits `11b0676`/`acd0946`). When updating the layout, edit the `dvorak` variant inside this file rather than adding a new file.
- `files/system/usr/lib/systemd/user/spacenav-ws.service` — user service running `uvx spacenav-ws@<pinned version> serve` (version bumps happen here), a WebSocket bridge that reads from `spacenavd` and speaks the 3Dconnexion driver protocol on `wss://127.51.68.120:8181` so Onshape in Firefox can use a SpaceMouse. Per-user browser setup steps are documented in `README.md` ("SpaceMouse + Onshape in Firefox"). No loopback alias is needed for the magic IP — Linux routes all of `127.0.0.0/8` to `lo`.
- `files/system/etc/spnavrc` — spacenavd sensitivity tuning (per-axis lines override the group line). Ostree caveat: a machine-locally modified `/etc/spnavrc` shadows this image default on upgrades; refresh with `sudo cp /usr/etc/spnavrc /etc/spnavrc`. Live-reload after edits: `sudo pkill -HUP spacenavd`.
- `files/scripts/example.sh` — template leftover; not referenced by the recipe (the `script` module uses inline snippets).
- `modules/` — placeholder for local custom BlueBuild modules; currently empty.

## Conventions

- `image-version` is pinned to `stable` so builds track Bluefin stable rather than a Fedora major version bump.
- YAML/JSON use 2-space indent, everything else 4-space, LF line endings, final newline (`.editorconfig`).
- To add another image variant, add a recipe file under `recipes/` and list it in the `matrix.recipe` array in `.github/workflows/build.yml`.
