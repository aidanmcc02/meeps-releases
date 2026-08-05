# Meeps releases

Desktop builds of [Meeps](https://github.com/Fat-Melons/Meeps). This repository
holds **no source** — only the installers and the updater manifest for each
release.

It exists because Meeps' source repository is private, and GitHub does not serve
release assets on a private repository to unauthenticated clients. Tauri's
updater is an unauthenticated HTTP client, so a manifest published only to the
private repository returns `404` to every installed copy of the app, and nobody
can update.

## What's here

Each release carries five files, published by the `create-release` job in the
source repository's `.github/workflows/release.yml`:

| File                            | What it is                                    |
| ------------------------------- | --------------------------------------------- |
| `Meeps_<version>_x64-setup.exe` | NSIS installer — this is the one to download  |
| `Meeps_<version>_x64_en-US.msi` | MSI installer                                 |
| `*.nsis.zip`                    | The bundle the in-app updater downloads       |
| `*.nsis.zip.sig`                | Its minisign signature                        |
| `latest.json`                   | The updater manifest                          |

`latest.json` is fetched from
`https://github.com/Fat-Melons/meeps-releases/releases/latest/download/latest.json`,
which is the endpoint compiled into the app. Every bundle is signed, and the
app verifies the signature against a public key baked into the binary before
installing anything — so these artifacts being public does not let anyone push
an update to a Meeps user.

## Installing

Grab the `.exe` from the [latest release](https://github.com/Fat-Melons/meeps-releases/releases/latest).
After that the app updates itself.

## Reporting a problem

Issues are disabled here. Report Meeps bugs in the source repository, or in the
Discord.
