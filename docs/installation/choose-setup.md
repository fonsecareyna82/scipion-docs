---
hide:
  - toc
---

# Choose Your Setup

ScipionWeb can run on one machine or on several. Each setup below takes **one or two commands**. Pick the one that fits your machine and follow its page.

| Your situation | Setup | Commands | Guide |
|---|---|---|---|
| One workstation or server with Internet | **One machine** | `./install.sh` | [Guided Installation](guided-install.md) |
| One machine with **no Internet** (or a slow or blocked connection) | **One machine, offline** | copy 3 files, then `./install.sh` | [Installing Without Internet](offline-install.md) |
| A server for the web plus **more machines to compute** (GPU nodes, a small cluster) | **Several machines** | `multinode enable` on the main one, `install.sh --join` on each node | [Several Machines (Multi-Node)](multi-node.md) |
| Your machines already share a **SLURM** cluster | One machine + SLURM queues | `./install.sh`, then configure a host | [Hosts and Queues](../user-guide/hosts-and-queues.md) |

!!! success "Not sure? Start with one machine"
    Everything you install on one machine stays valid. You can add compute nodes later without reinstalling: the main node keeps its projects, users and settings.

---

## One machine

The usual case: one Linux machine runs the web, the database and the protocols.

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only   # tells you what the machine is missing
./install.sh                # asks for the admin user and installs
```

Then open the address it prints in a browser. ➡️ [Guided Installation](guided-install.md)

---

## One machine, without Internet

Copy the release files next to `install.sh` (from a USB stick, a shared folder…). The installer sees them and installs from there, without downloading anything:

```text
my-folder/
├── install.sh
├── ScipionAPI-v4.0.0.zip
├── ScipionWeb-v4.0.0-dist.zip
└── manifest.json        (optional: adds checksum verification)
```

```bash
cd my-folder
./install.sh
```

➡️ [Installing Without Internet](offline-install.md)

---

## Several machines

One **main node** runs the web, the database and the job queue. **Compute nodes** take protocols from the queue and run them. They all see the same installation on a shared disk.

```text
                 ┌──────────── MAIN NODE ────────────┐
   browser ──────┤  web + API · PostgreSQL · Valkey  │
                 └────────────────┬──────────────────┘
                                  │  job queue
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
       ┌────────────┐      ┌────────────┐      ┌────────────┐
       │  node 1    │      │  node 2    │      │  GPU node  │
       │  protocols │      │  protocols │      │  protocols │
       └────────────┘      └────────────┘      └────────────┘
         all of them see the same installation (shared disk)
```

```bash
# 1. On the main node (once): install it, then open it to the nodes' network
./install.sh
~/scipionweb/scripts/scipionapi multinode enable --network 10.0.0.0/24

# 2. On each compute node
./install.sh --join ~/scipionweb
```

➡️ [Several Machines (Multi-Node)](multi-node.md)

---

## What you need in every setup

- Linux, with `sudo` for the person installing
- Conda (Miniconda is enough)
- PostgreSQL and Valkey on the machine that runs the web (the main node)

`./install.sh --check-only` checks all of it and tells you exactly what to install. ➡️ [Prerequisites](prerequisites.md)
