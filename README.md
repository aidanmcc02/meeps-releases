# Meeps releases

Desktop builds of Meeps. This repository holds **no source** — it clones the
private source at build time, builds the installers here, and publishes them
along with the updater manifest.

Meeps itself lives on self-hosted Gitea at `git.meeps.chat/Fat-Melons/Meeps`,
which is the source of truth. This is the only part of Meeps still on GitHub,
and it is here for one reason: the updater needs a public host.

It exists because Meeps' source repository is private, and GitHub does not serve
release assets on a private repository to unauthenticated clients. Tauri's
updater is an unauthenticated HTTP client, so a manifest published only to the
private repository returns `404` to every installed copy of the app, and nobody
can update.

## What's here

Each release carries five files, published by
[`.github/workflows/release.yml`](.github/workflows/release.yml):

| File                            | What it is                                   |
| ------------------------------- | -------------------------------------------- |
| `Meeps_<version>_x64-setup.exe` | NSIS installer — this is the one to download |
| `Meeps_<version>_x64_en-US.msi` | MSI installer                                |
| `*.nsis.zip`                    | The bundle the in-app updater downloads      |
| `*.nsis.zip.sig`                | Its minisign signature                       |
| `latest.json`                   | The updater manifest                         |

`latest.json` is fetched from
`https://github.com/aidanmcc02/meeps-releases/releases/latest/download/latest.json`,
which is the endpoint compiled into the app. GitHub only answers that path for
the release it marks **latest**, which is why the workflow passes `--latest`.

Every bundle is signed, and the app verifies the signature against a public key
baked into the binary before installing anything — so these artifacts being
public does not let anyone push an update to a Meeps user.

## How a release happens

The source repo never publishes anything. It bumps the version, rolls the
changelog, merges to `production`, and then dispatches this workflow:

```
gh workflow run release.yml --repo aidanmcc02/meeps-releases \
  --field version=2.0.112 --field ref=<production sha>
```

This repository then clones that commit from **Gitea** — a plain `git fetch`,
because `actions/checkout` only speaks to GitHub — builds and signs the Windows
bundle, generates `latest.json`, cuts the release **here** with its own
`GITHUB_TOKEN`, and posts the one release announcement to Discord.

Building here rather than mirroring in means there is no cross-repository write
token and no GitHub App installation in the path — the credential that used to
carry the release broke silently when this repo changed owner, and the symptom
was a release nobody could install.

You can run it by hand from the Actions tab. The build refuses to start if the
`version` you give does not match the version in the `ref` you give, so a stale
`production` fails loudly instead of publishing a mislabelled installer.

## Required secrets

| Secret                     | Why                                                                       |
| -------------------------- | ------------------------------------------------------------------------- |
| `GITEA_TOKEN`              | Clone the private source from Gitea. Scope: **`read:repository` only**    |
| `TAURI_PRIVATE_KEY`        | Signs the update bundle                                                   |
| `TAURI_KEY_PASSWORD`       | Its password                                                              |
| `VITE_BACKEND_HTTP_URL`    | Baked into the build                                                      |
| `VITE_BACKEND_WS_URL`      | Baked into the build                                                      |
| `DISCORD_POST_EXE_WEBHOOK` | The release announcement                                                  |

Two of those deserve care, because this repository is public:

- **`GITEA_TOKEN` reads the private source.** Give it `read:repository` and
  nothing else — no write, no admin, no user scope. If Gitea tokens are set to
  expire, note the date: when it lapses, releases stop, and it looks like a red
  workflow rather than anything a user reports.
- **`TAURI_PRIVATE_KEY` signs updates every installed client trusts.** Rotating
  it after a leak does not undo the exposure — clients check the pubkey compiled
  into the copy they are already running, so a rotation only protects people who
  update past it. The workflow's trigger surface is deliberately one manual
  dispatch; see the comment at the top of it before adding another.

## Installing

Grab the `.exe` from the [latest release](https://github.com/aidanmcc02/meeps-releases/releases/latest).
After that the app updates itself.

## Reporting a problem

Issues are disabled here. Report Meeps bugs in the source repository, or in the
Discord.
