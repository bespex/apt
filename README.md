# be spex packages

**The signed APT repository for all be spex apps.**

Add it once, then install and update the apps with `apt`. Setup instructions are on the page.

**Website:** https://bespex.run/apt/ · part of [be spex](https://bespex.github.io) – small, focused apps for Linux

| Package | Source |
|---|---|
| `marq`, `xaq`, `jayq`, `trisq`, `hyqe`, `cusq`, `dirq`, `maiqe` | [bespex/Qiew](https://github.com/bespex/Qiew) (one repository, releases tagged `<app>-v<version>`) |
| `maillet` | [bespex/Maillet](https://github.com/bespex/Maillet) |
| `hatrack` | [bespex/HatRack](https://github.com/bespex/HatRack) (amd64, arm64) |
| `pockettray` | [bespex/PocketTray](https://github.com/bespex/PocketTray) |
| `pictorix` | [bespex/Pictorix](https://github.com/bespex/Pictorix) |
| `codedeck` | [bespex/CodeDeck](https://github.com/bespex/CodeDeck): the program, recommends Marq, Dirq, Jayq |
| `qiew` | this repository ([`qiew/`](qiew/)): metapackage that recommends every Qiew viewer, with `qiew-setup` |
| `be-spex-archive-keyring` | this repository: signing key and APT source |

## How it works

Nothing is committed here per release. [`publish.yml`](.github/workflows/publish.yml) runs when an
app's release workflow asks for it (see below), on push, by hand, and on a schedule every 30 minutes.
GitHub runs scheduled workflows only on a best-effort basis: in practice every few hours, so the
schedule is a fallback, not the way a release arrives. Each run lists the newest 3 stable releases of each entry in
[`repos.json`](repos.json), and rebuilds only if their `.deb` assets changed
([`scripts/state.sh`](scripts/state.sh)). [`scripts/build.sh`](scripts/build.sh) then downloads the
packages, adds the keyring package, writes the indices with `apt-ftparchive`, signs `Release`
(`InRelease`, `Release.gpg`) and the result is deployed to GitHub Pages.

- **Add an application:** a line in `repos.json` with the package names its releases contain.
  Its release workflow only has to attach the `.deb` files to a GitHub release.
- **A release arrives at once** when its workflow ends with
  `gh workflow run publish.yml -R bespex/apt`, run with the secret `APT_DISPATCH_TOKEN`: a
  fine-grained token for `bespex/apt` with "Actions: read and write" (the workflow's own token
  cannot start a workflow in another repository). Kept as an organisation secret, every app's
  release can use it; without it, the release waits for the schedule. CodeDeck does this. A repository with
  several applications gets one line per application with the `tagPrefix` of its release tags
  (the newest 3 releases are then counted per application) and its `path` for the link on the page.
- **Signing key:** RSA 4096, no expiry, fingerprint `52006D90BDB8272ED2037F6036AE9C5B18B03CC0`.
  Public part in [`keys/`](keys/), private part only in the repository secret `APT_SIGNING_KEY`
  (and the maintainer's backup).
- **Build locally:** `GNUPGHOME=<keyring with the secret key> OUT=/tmp/apt scripts/build.sh`
  (needs `gh`, `jq`, `dpkg-deb`, `apt-ftparchive`, `gpg`).
- **Keyring package:** bump `keyring/VERSION` when the key or the source changes.
- **CodeDeck:** its release brings the package `codedeck` with version `1:<version>` and the viewers
  it opens things in as `Recommends`; a new viewer goes there, in CodeDeck's release workflow.
- **qiew metapackage:** a new Qiew viewer goes into `Recommends` in [`qiew/control`](qiew/control)
  as well, together with a higher `Version` there, so `apt upgrade` brings it to everyone who has
  the set. The build stops if that list and the Qiew entries in `repos.json` differ.
