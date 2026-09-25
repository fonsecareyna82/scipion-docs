---
hide:
  - toc
---

# Multi-Node Cluster Deployment

This guide explains how to run **ScipionWeb** across several machines: one **master** node (API + PostgreSQL + Valkey) and one or more **worker** nodes that only execute protocols or handle plugin installation.

!!! note "Scope"
    This page covers the *ScipionWeb-level* multi-node topology: which services run where, how they find each other, and how protocol execution gets routed to a specific node. It does not replace [Hosts and Queues](../user-guide/hosts-and-queues.md), which configures a scheduler (SLURM) *inside* a single node. The two are complementary and can be combined.

---

## Architecture Overview

```text
Master node
├── FastAPI / uvicorn (API + web UI)
├── PostgreSQL
├── Valkey (Celery broker)
└── Plugin Celery worker

Worker node 1                      Worker node 2                      Worker node 3
└── Protocol Celery worker         └── Protocol Celery worker         └── Protocol Celery worker
    queue: protocols                   queue: protocols                   queue: protocols-gpu
```

Every worker node points `DATABASE_URL` / `BROKER_URL` at the master's routable hostname (not `localhost`), and all nodes mount the same shared filesystem for `scipion_home/projects`.

Only the **master** serves the web UI and REST API. Worker nodes never run `uvicorn`; they only run Celery workers that pull protocol-execution (and optionally plugin-install) tasks from the shared broker.

!!! tip "Start smaller if you are unsure"
    If you only need to keep the API off the compute nodes, a single worker node already exercises this entire setup. Add the remaining nodes once the first one works end to end.

---

## Prerequisites

- A **shared filesystem** mounted at the same path on every node (NFS or equivalent), holding at least `scipion_home/projects`. Sharing `scipion_home/config` too is strongly recommended so `hosts.conf` and `celery_queue_routing.json` (see below) stay identical everywhere without manual copying.
- **PostgreSQL** and **Valkey** reachable from every worker node over the network (not `localhost`).
- The **same ScipionAPI checkout and Conda environment** installed independently on every node (code and the Python environment are *not* shared over NFS for performance reasons — only project data is).
- The plugins each worker actually needs installed **on that worker**. A shared filesystem does not make a plugin "installed" on a node that never ran its installer.

---

## Step 1 — Master Setup

Install ScipionAPI on the master exactly as in a normal single-node install (see [Guided Installation](guided-install.md) or [Manual Installation](manual-install.md)).

### 1.1 Point `.env` at routable addresses

Edit `scipion_home/.env` so `DATABASE_URL` and `BROKER_URL` use the master's **LAN hostname or IP**, not `localhost` / `127.0.0.1`:

```dotenv
DATABASE_URL=postgresql://scipion_user:<password>@master.internal:5432/scipion_db
BROKER_URL=redis://master.internal:6379/0
```

!!! warning "This is the single most common multi-node mistake"
    If these stay on `localhost`, worker nodes will start, look healthy in their own logs, and simply never receive any task — because they are talking to a broker on *themselves*, not the master.

### 1.2 Export the shared filesystem

Export `scipion_home/projects` (and ideally `scipion_home/config`) over NFS, or place them on whatever shared storage your infrastructure provides. Make sure the worker node IPs are allowed in the export.

### 1.3 Validate before starting anything

```bash
SCIPIONAPI_DEPLOYMENT_MODE=multi-node ./scripts/scipionapi doctor --strict
```

With `SCIPIONAPI_DEPLOYMENT_MODE=multi-node`, `doctor` escalates three checks from warnings to hard failures: **Broker reachability**, **Database reachability**, and **Projects filesystem** (rejecting local-only filesystem types such as `ext4` when it cannot confirm the path is shared). See [`doctor` — Multi-Node Deployment Checks](../cli/doctor.md#multi-node-deployment-checks) for what each check does.

Fix everything `doctor --strict` reports before moving to the worker nodes.

### 1.4 Start only the master's services

```bash
./scripts/scipionapi runtime start --role api
./scripts/scipionapi runtime start --role plugins
```

Protocol execution is normally **not** run on the master, so the machine serving the web UI stays responsive. If the master also has spare capacity, `--role protocols` (or `--role all`) is a valid choice too. See [`start` / `stop` / `restart` / `status` / `logs` — `--role`](../cli/runtime.md#-role-per-node-service-selection).

---

## Step 2 — Each Worker Node

Repeat this for every worker node.

### 2.1 Mount the shared filesystem

Mount the master's NFS export at the exact same path used on the master (for example `scipion_home/projects`).

### 2.2 Install ScipionAPI locally

Install the same ScipionAPI version with its own Conda environment on this node ([Manual Installation](manual-install.md) or [Provision](provision.md) are the usual paths for a worker-only node — you do not need to run `provision`'s admin-user bootstrap again if the database already exists).

### 2.3 Install the plugins this node should run

Install (via the plugin manager or `scipionapi_cli`) whatever plugins the protocols routed to this node actually need. GPU-heavy plugins typically only make sense on GPU-equipped nodes.

### 2.4 Point `.env` at the master

```dotenv
DATABASE_URL=postgresql://scipion_user:<password>@master.internal:5432/scipion_db
BROKER_URL=redis://master.internal:6379/0
```

### 2.5 Validate

```bash
SCIPIONAPI_DEPLOYMENT_MODE=multi-node ./scripts/scipionapi doctor --strict
```

### 2.6 Start only the protocol worker

```bash
./scripts/scipionapi runtime start --role protocols
```

For a node that should only receive a specific class of protocol (for example, a GPU node), give it its own Celery queue name instead of the shared default:

```bash
./scripts/scipionapi runtime start --role protocols --queue protocols-gpu
```

!!! tip "Make the queue name persistent"
    A `--queue` passed on the command line only applies to that one `start`/`restart` invocation. To have the same queue name survive future restarts — including ones triggered from the web UI's worker restart button — set it in that node's `.env` instead:

    ```dotenv
    PROTOCOLS_CELERY_QUEUE=protocols-gpu
    ```

---

## Step 3 — Route Protocols to the Right Node

By default every protocol dispatch goes to the shared `protocols` queue, and any protocol worker listening on it can pick it up — this is unchanged and safe even if you never touch queue routing.

To send specific protocols to a specific node, reuse the **Host** name you already assign per protocol in [Hosts and Queues](../user-guide/hosts-and-queues.md) (the same value used for SLURM partition selection). Create or edit:

```text
scipion_home/config/celery_queue_routing.json
```

```json
{
  "gpu-cluster": "protocols-gpu"
}
```

`"gpu-cluster"` here is a Host name as configured in **Settings > Host** / `hosts.conf`, not a Celery worker name. Any protocol launched with that host now dispatches straight to the `protocols-gpu` queue — which only the node started with `--queue protocols-gpu` consumes. Hosts with no entry in this file keep going to the plain `protocols` queue, exactly like a single-node deployment.

Because `scipion_home/config` is on the shared filesystem, this file only needs to exist once and every node/master sees the same mapping immediately.

---

## Step 4 — Verify the Cluster

### 4.1 Confirm every worker answers

From the master (or any node with network access to Valkey):

```bash
python -m celery -A app.workers.task_queue inspect ping
```

You should see one `protocols@<hostname>` entry per worker node, plus `plugins@<master-hostname>`.

### 4.2 Check the dashboard

Open **Settings > Jobs** in the web UI (admin only). Each worker shows up as its own card. Click **Refresh nodes** to broadcast a capability report to every node and confirm, per node, which GPUs and plugins are actually installed there — this is the fastest way to catch "I forgot to install this plugin on worker 2" before a protocol launch fails on it.

### 4.3 Launch a test protocol on the routed queue

Launch a protocol whose Host maps to `protocols-gpu` in `celery_queue_routing.json`, and confirm in **Settings > Jobs > Active jobs** that its `worker` column shows the GPU node's hostname, not one of the plain `protocols` workers.

---

## Networking Checklist

- [ ] `DATABASE_URL` / `BROKER_URL` on every worker point to the master's routable hostname/IP, not `localhost`
- [ ] PostgreSQL (`5432`) and Valkey (`6379`) accept connections from every worker's IP (firewall / security group / `pg_hba.conf`)
- [ ] The NFS (or equivalent) export allows every worker's IP
- [ ] `SCIPIONAPI_DEPLOYMENT_MODE=multi-node ./scripts/scipionapi doctor --strict` passes on the master and on every worker
- [ ] Each worker node has the plugins it needs installed locally
- [ ] `scipion_home/config/celery_queue_routing.json` (if used) is reachable from every node through the shared filesystem

---

## Common Issues

!!! warning "Worker looks 'ready' in its own log but never runs anything"
    Almost always `DATABASE_URL`/`BROKER_URL` still pointing at `localhost` on that node. Re-run `doctor --strict` with `SCIPIONAPI_DEPLOYMENT_MODE=multi-node` — this is exactly the case it is designed to catch.

!!! warning "Protocol stays in Scheduled/Launched forever"
    Confirm the Celery broker queue actually has a consumer: `python -m celery -A app.workers.task_queue inspect active_queues`. If a worker was started and then the broker connection was interrupted (network blip, VPN drop), the process can stay alive without actually consuming tasks. Restarting that one worker (`./scripts/scipionapi runtime restart --role protocols`) recovers it; this is an operational Celery/Redis quirk, not something specific to this deployment mode.

!!! warning "A protocol fails immediately with an import/plugin error on one node but not another"
    The plugin it needs is not installed on the node that picked it up. Use **Settings > Jobs > Refresh nodes** to confirm per-node plugin inventory before assuming code is broken, and route that protocol's Host to a queue only consumed by nodes that have the plugin.

!!! warning "`celery_queue_routing.json` changes do not seem to take effect"
    The mapping is read fresh on every dispatch, so no restart is needed — but it is read from `SCIPION_HOME/config/`, so double-check the file actually landed on the shared mount and not on a single node's local disk.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../deployment-systemd/" style="text-decoration:none; display:inline-block;">
    ← Previous: Production Deployment
  </a>
  <a href="../../cli/doctor/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: doctor command →
  </a>
</div>
