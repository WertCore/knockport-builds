# knockport-builds

Cross-platform release builds for KnockPort.

**This repository holds no source.** The workflow checks
`transmitworks/knockport` out at a tag and builds it here, for one reason:
Actions minutes are billed to the repository running the workflow, and this org
has its own quota. The source stays private where it lives.

## What it produces

Four bundles per release, from runners that can each build natively — which is
why this exists rather than a laptop:

| Runner | Target |
|---|---|
| `macos-14` | macOS arm64 (.dmg, .app) |
| `macos-13` | macOS x86_64 (.dmg, .app) |
| `windows-latest` | Windows x86_64 (NSIS .exe, .msi) |
| `ubuntu-22.04` | Linux x86_64 (.deb, .AppImage) |

Ubuntu 22.04 and not 24.04 deliberately: the glibc an AppImage is built against
is the *oldest* it will run on, so building on the newer image would silently
drop users still on 22.04.

Each bundle is signed for the updater and a `.sig` is emitted beside it. Those
signatures go into `latest.json` on knockport.com; without them the app refuses
the update, which is the point.

## Secrets it needs

| Secret | What it is |
|---|---|
| `SOURCE_TOKEN` | A fine-grained PAT with **Contents: read** on `transmitworks/knockport` only |
| `TAURI_SIGNING_PRIVATE_KEY` | Contents of `~/scam/knockport-updater.key` |
| `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` | Empty string if the key has no password |

`SOURCE_TOKEN` is scoped to one repository and read-only on purpose: this repo
is a build service, and a build service that could write to the product source
is a supply-chain hole rather than a convenience.

The signing key is the one that cannot be replaced. Lose it and no existing
install can ever update again — each would need reinstalling by hand.

## Running it

Actions → **release** → Run workflow → enter the tag (`v0.1.0`).
