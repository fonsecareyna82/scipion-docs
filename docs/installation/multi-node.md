# Several Machines (Multi-Node)

Run the web on one machine and the protocols on several. You install ScipionWeb **once**, on the main node. Each compute node joins with **one command**, and new nodes can be added or removed at any time.

```text
                 ┌──────────── MAIN NODE ────────────┐
   browser ──────┤  web + API · PostgreSQL · Valkey  │
                 │  (it can run protocols too)       │
                 └────────────────┬──────────────────┘
                                  │  job queue (Valkey)
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
       ┌────────────┐      ┌────────────┐      ┌────────────┐
       │  node 1    │      │  node 2    │      │  GPU node  │
       │ "protocols"│      │ "protocols"│      │"protocols- │
       │   queue    │      │   queue    │      │    gpu"    │
       └────────────┘      └────────────┘      └────────────┘
         all of them see the same installation (shared disk)
```

**How it works:**

1. A user launches a protocol in the web.
2. The API puts it in a **queue**.
3. A free worker on any node takes it, runs it and writes its results to the shared disk and the main node's database.

Nodes listening to the same queue share the work. A node with its own queue (for example, the GPU nodes) only takes the protocols sent to that queue.

---

## Before you start: three requirements

!!! warning "1. One shared disk, at the same path everywhere"
    Every node must see, at **exactly the same paths** as the main node:

    - the installation folder (by default `~/scipionweb`, with its `scipion_home`: settings, Scipion's software and plugins, projects, logs);
    - the Conda environment of ScipionWeb (by default under `~/miniconda3`);
    - the data your users import (microscope folders, data disks…).

    The usual way is an NFS home (or an NFS disk mounted at the same path) on every machine. ScipionWeb stores absolute paths: a node that sees a folder under another path cannot use it.

!!! warning "2. The same user, with the same uid"
    Run ScipionWeb on every node as the same Linux user, with the same **uid**. Check with `id -u` on each machine. Files written by a node must belong to the same owner as on the main node.

!!! warning "3. The nodes reach the main node"
    - The nodes need the main node's network name, for example `krios-server.cnb.csic.es`.
    - They need ports `5432` (PostgreSQL) and `6379` (Valkey) open to them. The first step below configures both.

Nothing else has to be installed on the nodes. Scipion, the plugins and their programs come from the shared disk, so a plugin installed from the web reaches every node.

!!! note "Nodes with their own local installation"
    If your nodes cannot share the installation and the Conda environment, follow [Multi-Node: Manual Setup](multi-node-cluster.md) instead. Step 2 below (`multinode enable`) is useful there too.

---

## Step 1 — Install the main node

Install ScipionWeb normally on the machine that will run the web, in a folder on the shared disk ([Guided Installation](guided-install.md), or [without Internet](offline-install.md)):

```bash
./install.sh
```

---

## Step 2 — Open the main node to the nodes

On the main node, tell ScipionWeb your nodes' network. **Preview first**: nothing changes with `--dry-run`.

```bash
~/scipionweb/scripts/scipionapi multinode enable --network 10.0.0.0/24 --dry-run
```

```text
Opening this main node (krios-server.cnb.csic.es) to the nodes of 10.0.0.0/24
  1. PostgreSQL: listen on the network and let scipion_user into scipion_db from 10.0.0.0/24 (pg_hba.conf), then restart it
  2. Valkey: listen on the network with a password (valkey.conf), then restart it. ...
  3. Firewall: if ufw is active, let 10.0.0.0/24 reach ports 5432 and 6379
  4. .env: name this machine as krios-server.cnb.csic.es instead of localhost, multi-node
  5. Restart the API and the workers of this machine with the new settings
Dry run: nothing changed.
```

If the plan looks right, run it (it asks for `sudo` once):

```bash
~/scipionweb/scripts/scipionapi multinode enable --network 10.0.0.0/24
```

What it changes:

| Where | Change |
|---|---|
| PostgreSQL | Listens on the network, and lets in **only** ScipionWeb's own user and database, **only** from your network |
| Valkey | Listens on the network and gets a password (a new one, or the one it already had) |
| Firewall | If `ufw` is active, opens ports `5432` and `6379` to your network only |
| `.env` | Uses the machine's network name instead of `localhost`, and marks the installation as multi-node |

!!! info "Everything is backed up first"
    Every file it changes (`pg_hba.conf`, `valkey.conf`, `.env`) is copied first to `scipion_home/multinode/backup-<date>/`. To undo, copy them back and restart PostgreSQL, Valkey and ScipionWeb.

!!! tip "Options"
    - `--host NAME`: the name the nodes use to reach this machine. The default is its full host name; give another if the nodes know it by a different name or IP.
    - `--yes`: no confirmation question, for scripts.

!!! warning "Valkey gets a password"
    Any other program on the main node that uses Valkey without a password stops reaching it. ScipionWeb itself is updated automatically.

At the end it checks that the database and Valkey answer on the network name, and prints the command for the nodes.

---

## Step 3 — Join each compute node

On each compute node, as the same user, run the installer with `--join` and the installation folder:

```bash
~/scipionweb/install.sh --join ~/scipionweb
```

(`install.sh` is the same file you installed with; any copy works. Inside the installation, the equivalent command is `~/scipionweb/scripts/scipionapi node join`.)

It checks the machine and starts its worker:

```text
Checking this machine (node-01) as a compute node
OK     Multi-node installation      yes
OK     Not the main node            this machine is node-01, the main node is krios-server
OK     Installation folder          /home/scipion/scipionweb
OK     SCIPION_HOME                 /home/scipion/scipionweb/scipion_home
OK     Projects folder              /home/scipion/scipionweb/scipion_home/projects
OK     Same user as the main node   uid 1001
OK     Database (from this node)    krios-server.cnb.csic.es:5432
OK     Valkey (from this node)      krios-server.cnb.csic.es:6379
OK     GPUs                         2
...
The worker of node-01 answers: it takes protocols from now on.
```

If a check fails, it says why and changes nothing. Fix it and run the same command again.

That's it: the node takes protocols from the `protocols` queue together with the main node.

| Option | What it does | Default |
|---|---|---|
| `--queue NAME` | The queue the node takes protocols from | `protocols` |
| `--concurrency N` | How many protocols it runs at a time | its number of GPUs, or 4 |
| `--host NAME` | Sends the protocols of this Scipion host to the node's queue (see below) | — |

---

## Send some protocols to some nodes (GPU nodes)

By default every node shares one queue: any protocol can run on any node. To send, for example, GPU protocols to the GPU nodes only, give those nodes their own queue and a **host** name:

```bash
# on each GPU node
~/scipionweb/install.sh --join ~/scipionweb --queue protocols-gpu --host gpu-nodes
```

This:

1. starts the node's worker on the `protocols-gpu` queue;
2. creates the Scipion host `gpu-nodes` (a copy of `localhost`), if it does not exist;
3. sends every protocol run on host `gpu-nodes` to the `protocols-gpu` queue.

Users then choose **gpu-nodes** as the host in the protocol's form (its execution settings), and the protocol runs on a GPU node. Protocols on `localhost` keep running on the other nodes.

!!! tip "Several groups of nodes"
    Repeat with other names: `--queue protocols-bigmem --host bigmem-nodes`, and so on. The mapping is kept in `scipion_home/config/celery_queue_routing.json`. Edit a host's settings in **Settings → Hosts and Queues**.

---

## Check the installation

From any node:

```bash
~/scipionweb/scripts/scipionapi node list
```

```text
Main node: krios-server
  node-01              queue protocols          concurrency 4   answering
  gpu-01               queue protocols-gpu      concurrency 2   answering
  protocols@krios-server   protocols
  plugins@krios-server     plugins
```

On a node, `scipionapi status` shows its worker, and `scipionapi doctor` checks it (database, Valkey, shared folders…).

---

## Everyday operations

| Task | Where | Command |
|---|---|---|
| Restart a node's worker | the node | `scipionapi restart` |
| Stop a node | the node | `scipionapi stop` |
| Start it again (after a reboot) | the node | `scipionapi start` |
| See a node's logs | the node | `scipionapi logs` (they are in `scipion_home/logs/nodes/<node>/`) |
| Remove a node | the node | `scipionapi node leave` |
| Update ScipionWeb | the main node | `scipionapi update`, then `scipionapi restart` on each node |
| Install a plugin | the web | it reaches every node through the shared disk; restart the nodes' workers |

On a compute node, `start`, `stop`, `restart` and `status` act on its protocol worker only: the API never starts there.

!!! tip "Start the node's worker at boot"
    Put `scipionapi start` in a `@reboot` crontab entry of the ScipionWeb user, or in a systemd unit. See [Production Deployment](deployment-systemd.md).

---

## Troubleshooting

| What you see | Why | Fix |
|---|---|---|
| `Multi-node installation: not yet` | Step 2 was not run | Run `multinode enable` on the main node |
| `Installation folder: not found here` | The shared disk is not mounted, or under another path | Mount it at the same path as on the main node |
| `Same user as the main node` fails | Another user, or another uid | Use the same user; make its uid the same on every machine (LDAP, or `usermod -u`) |
| `Database (from this node)` fails | Firewall, wrong network in step 2, or the node uses another name for the main node | Check `--network`; try `nc -zv <main> 5432`; run `multinode enable --host <name the nodes use>` |
| `Valkey (from this node)` fails | Same as above, port `6379` | Same as above |
| The node answers, but protocols never run there | They go to another queue | Check `node list`, the host chosen in the protocol's form, and `celery_queue_routing.json` |
| A protocol fails only on one node | A program or a data folder missing on that node | Check that the node sees the same data paths; see the node's `celery-protocols.log` |

---

## What runs where

| Part | Main node | Compute node |
|---|---|---|
| Web and API | ✅ | — |
| PostgreSQL, Valkey | ✅ | — |
| Plugin installation worker | ✅ | — |
| Protocols | ✅ (unless you stop its worker) | ✅ |
| Viewers, wizards, summaries | ✅ (they need the plugins and the data too) | — |
| Facility session watcher | ✅ (it needs the microscopes' folders) | — |

!!! note "Security"
    Opening the database and Valkey to a network is safe only on a trusted internal network. Keep the network in step 2 as small as possible. ➡️ [Security Notes](../operations/security.md)
