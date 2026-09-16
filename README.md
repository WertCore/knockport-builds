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
| `RELEASE_TOKEN` | A PAT with **Contents: write** on `WertCore/knockport-releases`, the public repository the bundles are published to |
| `TAURI_SIGNING_PRIVATE_KEY` | Contents of `~/scam/knockport-updater.key` |

There is deliberately no secret for the signing password. The key has none, and
GitHub will not store an empty secret value — so the workflow sets the variable
to an empty string directly. Putting a space there instead makes it a real
password, which Tauri then fails to decrypt with.

### Apple signing and notarisation

All six or none. With none, macOS still builds — unsigned, which Gatekeeper
refuses on first launch with "the app is damaged" rather than a prompt, and
which no `xattr` incantation should be expected of a stranger. With some, the
build fails rather than shipping a signed-but-unnotarised bundle that is
rejected just the same, with nothing in a green log to say why.

| Secret | What it is |
|---|---|
| `APPLE_CERTIFICATE` | The Developer ID Application `.p12`, base64: `base64 -i cert.p12 \| pbcopy` |
| `APPLE_CERTIFICATE_PASSWORD` | The password set when exporting that `.p12` |
| `APPLE_SIGNING_IDENTITY` | `Developer ID Application: NAME (TEAMID)`, exactly as `security find-identity -v -p codesigning` prints it |
| `APPLE_ID` | The Apple ID of the developer account |
| `APPLE_PASSWORD` | An **app-specific** password from appleid.apple.com, not the account password |
| `APPLE_TEAM_ID` | The ten-character team id |

## Variables it reads

| Variable | What it is |
|---|---|
| `TELEMETRY_URL` | Base URL for the install ping, compiled into the bundle as `VITE_TELEMETRY_URL` |

A variable rather than a secret, deliberately: Vite inlines it into the
JavaScript, so it ships inside every copy of the app and anyone can read it.
Masking it in a build log would protect nothing. Unset, the app's
`reportInstall` returns on its first line and no install is ever counted —
which is exactly what every release before this one did.


`SOURCE_TOKEN` is scoped to one repository and read-only on purpose: this repo
is a build service, and a build service that could write to the product source
is a supply-chain hole rather than a convenience.

The signing key is the one that cannot be replaced. Lose it and no existing
install can ever update again — each would need reinstalling by hand.

## Running it

Actions → **release** → Run workflow → enter the tag (`v0.1.0`).
