# Example: A Three-Machine Cluster, Step by Step

This page builds a real small cluster from scratch, command by command. It follows the [Several Machines](multi-node.md) guide **without SLURM**: the compute nodes run the protocols on their own GPUs.

---

## The machines

| Machine | IP | Hardware | Role |
|---|---|---|---|
| `scipion-master` | `10.0.0.10` | CPU only, **no GPU** | Web, database, job queue. **Runs no protocols** |
| `gpu-01` | `10.0.0.21` | 4 GPUs | Runs protocols |
| `gpu-02` | `10.0.0.22` | 4 GPUs | Runs protocols |

The master needs no GPU: it runs no protocols, and the web's viewers only use the CPU.

When it is finished:

```text
  users ──► scipion-master (web + database + queue)     ← no GPU, no protocols
                   │
             "protocols" queue
            ┌──────┴──────┐
            ▼             ▼
         gpu-01        gpu-02          ← each takes a protocol when it has room
```

---

## Step 0 — Prepare the machines (once, usually by the system administrators)

Three things must be ready before ScipionWeb.

### a) One shared disk

The home of the user `scipion` must be the same on the three machines. That is where ScipionWeb, its Conda environment and the projects will live.

If your machines already share their homes (NFS, LDAP…), skip this. Otherwise, the usual way is NFS from the master:

```bash
# on scipion-master
sudo apt install nfs-kernel-server
echo "/home/scipion 10.0.0.0/24(rw,sync,no_subtree_check)" | sudo tee -a /etc/exports
sudo exportfs -ra
```

```bash
# on gpu-01 and on gpu-02
sudo apt install nfs-common
sudo mount scipion-master:/home/scipion /home/scipion
echo "scipion-master:/home/scipion /home/scipion nfs defaults 0 0" | sudo tee -a /etc/fstab
```

**The data too:** the microscopes' folders and the data disks your users import must also be at the same path everywhere, for example `/data/offload`.

### b) The same user, with the same uid

```bash
id -u scipion     # on the three machines: the same number
```

If it differs, make it equal (`sudo usermod -u <uid> scipion`) before going on.

### c) The GPU drivers on the GPU nodes

```bash
nvidia-smi        # on gpu-01 and gpu-02: it lists the 4 GPUs
```

### Check before going on

```bash
# on gpu-01 (and gpu-02), as scipion
ls ~                       # the same files as on the master
touch ~/test-from-gpu-01   # then, on the master: ls ~/test-from-gpu-01
```

---

## Step 1 — Install ScipionWeb on the master

On `scipion-master`, as `scipion`. Install Conda in the home (`~/miniconda3`), so that it is on the shared disk too.

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh
```

This is the normal installation, in `~/scipionweb`. When it ends, you already have a working web: open the address it prints and sign in as the administrator.

---

## Step 2 — Open the master to the nodes

Still on `scipion-master`. First look at the plan:

```bash
~/scipionweb/scripts/scipionapi multinode enable --network 10.0.0.0/24 --dry-run
```

If it looks right, do it (it asks for `sudo`):

```bash
~/scipionweb/scripts/scipionapi multinode enable --network 10.0.0.0/24
```

What changed:

- PostgreSQL and Valkey accept the machines of `10.0.0.x`, and only them, and Valkey got a password.
- The `.env` now names `scipion-master` instead of `localhost`.
- Every file was backed up first in `~/scipionweb/scipion_home/multinode/`.

---

## Step 3 — Join the GPU nodes

On `gpu-01`, as `scipion`:

```bash
~/scipionweb/install.sh --join ~/scipionweb
```

```text
Checking this machine (gpu-01) as a compute node
OK     Multi-node installation      yes
OK     Not the main node            this machine is gpu-01, the main node is scipion-master
OK     Installation folder          /home/scipion/scipionweb
OK     SCIPION_HOME                 /home/scipion/scipionweb/scipion_home
OK     Projects folder              /home/scipion/scipionweb/scipion_home/projects
OK     Same user as the main node   uid 1001
OK     Database (from this node)    scipion-master:5432
OK     Valkey (from this node)      scipion-master:6379
OK     GPUs                         4
...
The worker of gpu-01 answers: it takes protocols from now on.
```

It found the 4 GPUs, so it runs up to 4 protocols at a time. Do the same on `gpu-02`.

If a line says `FAIL`, it explains why and nothing is changed: fix it (usually step 0) and run the same command again.

---

## Step 4 — Keep the master from computing

On `scipion-master`:

```bash
~/scipionweb/scripts/scipionapi multinode compute off
```

From now on every protocol runs on `gpu-01` or `gpu-02`.

---

## Step 5 — Check that it works

```bash
~/scipionweb/scripts/scipionapi node list
```

```text
Main node: scipion-master (does not run protocols)
  gpu-01     queue protocols   concurrency 4   answering
  gpu-02     queue protocols   concurrency 4   answering
```

Then, in the web:

1. Create a project and launch a quick protocol (an import, for example).
2. On `gpu-01` and `gpu-02`, run `~/scipionweb/scripts/scipionapi logs`: one of them shows it running.
3. The protocol finishes in the web as usual.

That's the whole cluster.

---

## Daily life

| I want to… | Where | What |
|---|---|---|
| Launch protocols | the web | As always. A free node takes each one; when both are full (4 + 4), the next waits in the queue |
| Add a third node | the new node | Mount the disk (step 0), then `install.sh --join ~/scipionweb` |
| Remove a node | that node | `~/scipionweb/scripts/scipionapi node leave` |
| Restart a node after a reboot | that node | `~/scipionweb/scripts/scipionapi start` |
| Update ScipionWeb | the master | `scipionapi update`, then `scipionapi restart` on each node |
| Install a plugin | the web | It reaches every node through the shared disk; restart the nodes' workers |

---

## The GPUs: the user chooses them

Without SLURM, nobody shares out a node's GPUs. A protocol uses the **GPU IDs** written in its form, on the node that takes it:

- GPU `0` means the first GPU **of that node**.
- `gpu-01` runs up to 4 protocols at a time. If two of them ask for GPU `0`, they share it and may run out of memory.
- So, protocols that run at the same time should use different GPU IDs: one `0`, another `1`, and so on.

If something fails, the protocol's log says why. For automatic GPU sharing, use SLURM (below).

---

## Later: machines that are not all alike

Say you add `cpu-01`, a machine **without GPUs**, for the CPU-only protocols. GPU protocols must then go only to `gpu-01` and `gpu-02`:

```bash
# on gpu-01 and gpu-02: leave, then join with a GPU queue and a host
~/scipionweb/scripts/scipionapi node leave
~/scipionweb/install.sh --join ~/scipionweb --queue protocols-gpu --host gpu-nodes

# on cpu-01: the normal queue
~/scipionweb/install.sh --join ~/scipionweb
```

Users then choose the host **gpu-nodes** in the form of a GPU protocol. Everything else runs on `cpu-01`.

---

## The same cluster with SLURM

If `gpu-01` and `gpu-02` are SLURM nodes, the cluster is built differently. Steps 2 to 4 are not needed:

1. **Step 0** as above: shared disk, same user, drivers, plus SLURM working (`sinfo`, and `sbatch` from `scipion-master`).
2. **Step 1** as above: install on `scipion-master`.
3. **Create a host with a queue** in **Settings → Hosts and Queues** (`sbatch`, `squeue`, `scancel`, a template with `--gres=gpu:%_(GPU_COUNT)s`, your partitions). ➡️ [Hosts and Queues](../user-guide/hosts-and-queues.md)
4. **Users pick that host** in the protocol's form.

SLURM then chooses the node and the GPUs, and a protocol can leave its GPU IDs at `0`. Keep the master's protocol worker **on** (no `compute off`): it submits the jobs and follows them. ➡️ [With SLURM](multi-node.md#with-slurm)
