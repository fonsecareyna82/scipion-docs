# Installing Without Internet

The installer does not need the download server. If the release files are in its folder, it installs from them and **downloads nothing**. This works on machines without Internet, behind a firewall, or when the download server is slow.

!!! note "What still has to be on the machine"
    The system software (Conda, PostgreSQL, Valkey, `unzip`) is not part of the release files. Install it first, from your distribution's packages or a local mirror. `./install.sh --check-only` lists what is missing.

---

## 1. Get the release files

On a machine with Internet, download the four files of a release from the [release server](https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/):

| File | What it is |
|---|---|
| `install.sh` | The installer |
| `ScipionAPI-<version>.zip` | The server (API) |
| `ScipionWeb-<version>-dist.zip` | The web interface |
| `manifest.json` | Optional: the list of releases with their checksums. With it, the installer verifies the files |

For example, for `v4.0.0`:

```bash
BASE=https://scipion.cnb.csic.es/downloads/scipion/scipionWeb
mkdir scipionweb-v4.0.0 && cd scipionweb-v4.0.0
wget $BASE/install.sh $BASE/ScipionAPI-v4.0.0.zip $BASE/ScipionWeb-v4.0.0-dist.zip $BASE/manifest.json
chmod +x install.sh
```

Copy the folder to the target machine: a USB stick, `scp`, a shared folder…

---

## 2. Install

Run the installer from that folder:

```bash
cd scipionweb-v4.0.0
./install.sh
```

It says so when it uses the local files:

```text
[OK]     Using the local release archives in /media/usb/scipionweb-v4.0.0: nothing is downloaded
```

If the files are somewhere else, point to them:

```bash
./install.sh --local-dir /media/usb/scipionweb-v4.0.0
```

Every other option works as in the [Guided Installation](guided-install.md): `--install-dir`, `--user`, `--email`, `--non-interactive`…

!!! tip "Which version does it install?"
    - With `manifest.json`: the release that the manifest calls `latest`, or the one you ask for with `--version`.
    - Without it: the version in the file names (the newest one, if there are several).

    If you copied an older release together with a newer `manifest.json`, ask for it by name: `./install.sh --version v4.0.0`. Otherwise the installer looks for the manifest's latest files and, as they are not in the folder, tries to download them.

!!! warning "Both ZIP files are needed"
    The API and Web files of **the same version** must both be in the folder. If one is missing, the installer tries to download it.

---

## 3. Update later, also without Internet

Put the files of the new release (both ZIP files and `manifest.json`) in a folder, and give it to the updater as a `file://` address:

```bash
~/scipionweb/scripts/scipionapi update --base-url file:///media/usb/scipionweb-v4.1.0/
```

Preview first with `--dry-run`. ➡️ [Upgrade Guide](upgrade.md)

---

## Several machines without Internet

Install the main node as above. Compute nodes download nothing: they use the main node's installation on the shared disk. ➡️ [Several Machines (Multi-Node)](multi-node.md)
