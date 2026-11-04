# Several Machines (`multinode`, `node`)

These commands turn an installation into a main node with compute nodes. Step by step: [Several Machines (Multi-Node)](../installation/multi-node.md).

| Command | Where | Purpose |
|---|---|---|
| `multinode enable` | main node | Open PostgreSQL and Valkey to the nodes' network and mark the installation as multi-node |
| `multinode compute off` / `on` | main node | Leave every protocol to the compute nodes, or run them on the main node again |
| `node join` | each compute node | Check the machine, save its profile and start its protocol worker |
| `node leave` | a compute node | Stop its worker and forget its profile |
| `node list` | anywhere | The compute nodes and the workers answering now |

---

## `multinode enable`

```bash
./scripts/scipionapi multinode enable --network 10.0.0.0/24 [--host NAME] [--dry-run] [--yes]
```

| Option | Meaning |
|---|---|
| `--network CIDR` | **Required.** The nodes' network; only it gets in. `0.0.0.0/0` is refused |
| `--host NAME` | The name the nodes reach this machine by (default: its full host name) |
| `--dry-run` | Print the plan and change nothing |
| `--yes`, `-y` | Do not ask for confirmation |

Steps, in order:

1. **PostgreSQL**, when it runs on this machine:
    - `listen_addresses = '*'`, set with `ALTER SYSTEM`;
    - one line in `pg_hba.conf` for ScipionWeb's user and database from the network, marked `# ScipionWeb multi-node`, with the server's own password method (`scram-sha-256` or `md5`);
    - a restart.
2. **Valkey**, when it runs on this machine:
    - `bind 0.0.0.0 -::1` and a `requirepass` in its configuration file (the old `bind` and `requirepass` lines are commented out);
    - a restart. An existing password is kept.
3. **Firewall:** if `ufw` is active, ports `5432` and `6379` are opened to the network.
4. **`.env`:**
    - `POSTGRES_HOST`, `DATABASE_URL`, `BROKER_URL` and `RESULT_BACKEND_URL` use the host name instead of `localhost`, and the Valkey URLs carry its password;
    - it adds `SCIPIONAPI_DEPLOYMENT_MODE=multi-node`, `SCIPIONAPI_MAIN_NODE` and `SCIPIONAPI_MULTINODE_NETWORK`.
5. **Restart** of the API and the workers of this machine.

After them, it checks that the database and Valkey answer on the host name.

When the database or Valkey runs on **another** machine, the plan says so and leaves it alone: open it to the network yourself.

The changed files are copied first to `scipion_home/multinode/backup-<date>-<time>/`.

---

## `multinode compute`

```bash
./scripts/scipionapi multinode compute off [--force]
./scripts/scipionapi multinode compute on
```

**`off`** stops the main node's protocol worker and writes `SCIPIONAPI_MAIN_RUNS_PROTOCOLS=0` to the `.env`. After it:

- `start`, `restart` and `update` start the API and the plugin worker only;
- `start --role protocols`, and the web's protocol-worker buttons, are refused with a message.

It refuses while protocols run on the main node, unless `--force` is given (they are killed). It warns when no compute node answers.

**`on`** writes `1` and starts the worker again.

!!! danger "Not when protocols go to SLURM"
    The main node's protocol worker is the one that submits SLURM jobs (`sbatch`) and follows them. Turn it off only when every protocol runs on compute nodes joined with `node join`.

---

## `node join`

```bash
./scripts/scipionapi node join [--queue NAME] [--concurrency N] [--host NAME]
# or, from the installer:
./install.sh --join /path/to/installation [--queue NAME] [--concurrency N] [--host NAME]
```

| Option | Meaning | Default |
|---|---|---|
| `--queue NAME` | The Celery queue the node takes protocols from | `protocols` |
| `--concurrency N` | Protocols at a time | number of GPUs, or `4` |
| `--host NAME` | With `--queue`: make the Scipion host `NAME` (a copy of `localhost` if it does not exist) and send its protocols to this queue | — |

Checks, all from this machine:

- the installation is multi-node (`multinode enable` was run), and this is not the main node;
- the installation folder, `SCIPION_HOME` and `PROJECTS_PATH` are there and writable;
- `SCIPION_HOME` belongs to this user (same uid);
- the main node's database and Valkey answer;
- the GPUs (`nvidia-smi`), for information.

If any of them fails, nothing is written. Otherwise:

1. It writes the node's profile `scipion_home/nodes/<node>.env`:

    ```text
    SCIPIONAPI_NODE_ROLE=protocols
    PROTOCOLS_CELERY_QUEUE=protocols-gpu
    PROTOCOL_WORKER_CONCURRENCY=2
    LOGS_PATH=/home/scipion/scipionweb/scipion_home/logs/nodes/gpu-01
    ```

2. With `--host`, it updates `scipion_home/config/hosts.conf` and `scipion_home/config/celery_queue_routing.json`.
3. It starts the protocol worker and waits until it answers.

The node's name is its short host name (`gpu-01.cnb.csic.es` → `gpu-01`).

---

## `node leave`

```bash
./scripts/scipionapi node leave
```

Stops the node's worker and removes its profile. Its logs stay in `scipion_home/logs/nodes/<node>/`. The queue routes it made with `--host` stay too, as other nodes may use them.

---

## `node list`

```bash
./scripts/scipionapi node list
```

Shows the main node, every compute node with its queue and concurrency, whether its worker answers now, and any other worker answering (the main node's).

---

## How the other commands behave on a node

On a machine with a node profile:

- `start`, `stop`, `restart`, `status` and `logs` without `--role` act on the node's protocol worker only. The API and the plugin worker never start on a node.
- PID files go to `.run/nodes/<node>/`, so they never mix with the main node's.
- Logs go to `scipion_home/logs/nodes/<node>/`.
- `doctor` reads the node's profile and checks the node.
