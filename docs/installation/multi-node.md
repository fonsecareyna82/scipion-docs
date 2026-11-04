# Several Machines (Multi-Node)

Run the web on one machine and the protocols on several. There are two ways. Choose by one question: **does your cluster have SLURM?**

| | **Without SLURM**: compute nodes join ScipionWeb | **With SLURM**: SLURM runs the protocols |
|---|---|---|
| Who runs a protocol | A ScipionWeb worker on a compute node, directly | A SLURM job, on the node SLURM chooses |
| Who chooses the GPUs | **The user**, with the protocol's GPU IDs | **SLURM** (`--gres`), so jobs never share a GPU |
| Queues, priorities, limits per user | No: first come, first served | Yes, SLURM's |
| What to set up | `multinode enable` on the main node, `install.sh --join` on each node | A host with a queue in **Settings → Hosts and Queues** |
| Best for | Two or three GPU machines of a lab | A real cluster or an HPC |
| Guide | this page, from [Before you start](#before-you-start-three-requirements) | [With SLURM](#with-slurm) below, and [Hosts and Queues](../user-guide/hosts-and-queues.md) |

Both can be used together: each protocol runs where its **host** says (the host is chosen in the protocol's form).

!!! example "Prefer an example?"
    [Example: A Three-Machine Cluster](multi-node-example.md) builds a master without GPUs plus two GPU nodes from scratch, command by command, and shows the same cluster with SLURM.

---

## Without SLURM: compute nodes

You install ScipionWeb **once**, on the main node. Each compute node joins with **one command**, and new nodes can be added or removed at any time.

```text
                 ┌──────────── MAIN NODE ────────────┐
   browser ──────┤  web + API · PostgreSQL · Valkey  │
                 │  (no protocols: step 4)           │
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

!!! warning "The GPUs are the user's choice"
    Without SLURM, nobody shares out a node's GPUs. A protocol uses the GPU IDs written in its form, on whichever node takes it.

    - Two protocols on the same node with the same GPU ID share that GPU, and can fail for lack of memory.
    - Users must choose different GPU IDs for protocols running at the same time.
    - Alternatively, give a node a `--concurrency` no higher than the protocols its GPUs can hold.

    If something fails, the protocol's log says so. For automatic GPU sharing, use SLURM.

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

## Step 4 — Keep the main node from computing

Until now the main node runs protocols too, which is handy while you add the nodes. Once they work, leave all the computing to them, so the web, the database and the viewers always respond fast. On the main node:

```bash
~/scipionweb/scripts/scipionapi multinode compute off
```

What it does:

- It stops the main node's protocol worker.
- It writes `SCIPIONAPI_MAIN_RUNS_PROTOCOLS=0` to the `.env`, so the worker does not start again with `start`, `restart` or `update`, and the web's worker buttons do not start it either.
- From then on, every protocol waits in the queue for a compute node.

!!! warning "Protocols running on the main node"
    If protocols are running on the main node, it refuses: stopping its worker would kill them. Wait until they finish, or add `--force`.

!!! tip "No node answering?"
    It tells you when no compute node answers. Protocols launched meanwhile are not lost: they wait in the queue until a node joins or starts.

!!! danger "Not with SLURM"
    Protocols sent to SLURM are submitted (`sbatch`) and followed by the main node's protocol worker, which only submits and waits. With `compute off` nobody submits them. If you use SLURM, leave the main node's worker on.

To let the main node compute again: `scipionapi multinode compute on`.

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

## With SLURM

When the cluster has SLURM, ScipionWeb does not need compute nodes of its own:

```text
   browser ──► main node (web, database, protocol worker)
                  │   the worker only submits and waits
                  ▼   sbatch
               SLURM ──► node-01 · node-02 · gpu-01 …   (SLURM chooses node, GPUs, memory, time)
```

1. **Install ScipionWeb on a machine that can submit to SLURM**, the one where `sbatch` and `squeue` work, in a folder on the shared disk ([Guided Installation](guided-install.md)).
2. **Check that the SLURM nodes see the same paths:** the installation, the Conda environment and the data. A SLURM job runs Scipion from there.
3. **Create a host with a queue** in **Settings → Hosts and Queues**: the submit, check and cancel commands, a submit template asking for `--gres=gpu:%_(GPU_COUNT)s`, and your partitions. Step by step: [Hosts and Queues](../user-guide/hosts-and-queues.md).
4. **Users choose that host** in the protocol's form, with its partition, time, memory and number of GPUs.

**The GPUs.**

- SLURM gives each job its own GPUs, and the job only sees those (with `gres` and cgroups configured).
- A protocol asking for GPU `0` uses the first GPU **of its job**, never another job's.
- So, with SLURM, users can leave the GPU IDs at `0` (or `0 1` for two).

!!! warning "Keep the main node's protocol worker on"
    It runs no science: it submits the job and follows it until it ends. Do not use `multinode compute off` here.

!!! tip "Mixing both ways"
    A cluster with SLURM can also have compute nodes joined to ScipionWeb, for example a GPU workstation outside SLURM. Protocols on the SLURM host go to SLURM; protocols on the node's host go to the node ([Send some protocols to some nodes](#send-some-protocols-to-some-nodes-gpu-nodes)).

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
| Protocols | only until `multinode compute off` | ✅ |
| Viewers, wizards, summaries | ✅ (they need the plugins and the data too) | — |
| Facility session watcher | ✅ (it needs the microscopes' folders) | — |

!!! note "Security"
    Opening the database and Valkey to a network is safe only on a trusted internal network. Keep the network in step 2 as small as possible. ➡️ [Security Notes](../operations/security.md)
