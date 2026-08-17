# Glycojones Group Computing Resources

Glycojones Group has access to some local Linux compute machines: **Edith**, **Ultron**, and **Jarvis**. The lab machine **Bmo** can also be used when available. University HPC is available through **[Viking](https://vikingdocs.york.ac.uk/)**. Contact your PI if you need access to Viking.

## Choosing a machine

| Machine | CPU | Memory | GPU | Connection | Recommended use |
|---|---|---|---|---|---|
| **Edith** | Intel Core i9-9900K, 8 cores | 32 GiB | RTX 2060, 8 GB | `ssh edith.its.york.ac.uk`  | code/GPU testing |
| **Ultron** | 2 × Intel Xeon Gold 6526Y, 32 cores | 128 GiB | 2 × A30, 24 GB each | `ssh ysbltest.york.ac.uk`|  GPU jobs, medium-sized parallel CPU jobs |
| **Jarvis** | AMD Ryzen Threadripper PRO 3995WX, 64 cores | 256 GiB | RTX 3090, 24 GB | `ssh jarvis.its.york.ac.uk` | CPU-parallel workloads, batch processing |
| **Bmo** | Intel Core i9-11900, 8 cores | 32 GiB | RTX 3090, 24 GB | `ssh bmo.its.york.ac.uk`|  single-GPU computation |

Connections may require the University of York network or VPN. For workloads that exceed the local machines, use **Viking**.

### Structural databases on jarvis

Jarvis maintains local copies of the following databases. They are updated every Monday:

| Database | Location |
|---|---|
| PDB mmCIF | `/vault/pdb_mirror/` |
| PDB REDO | `/vault/pdb-redo` |
| PDB mmCIF | `/y/groups/agirre_group/database/pdb` |
| PDB REDO | `/y/groups/agirre_group/database/pdb-redo` |

### Home directories and backups

The Linux home directory is stored on the same network filesystem across YSBL computers. Files saved under `/y/people/<username>/` are therefore available when you log in to another YSBL Linux machine and are protected by regular backups.

Snapshots are taken every four hours during the day, retained daily for one week, and then retained weekly for one month. Tape backups are retained for 90 days, and the complete filesystem is also copied nightly to a backup server in a separate machine room. For normal long-term data protection, keep the data on the network filesystem; a backup is not permanent storage for a file that has been deleted from the live filesystem.

If you accidentally delete or damage a file, email **Tim** as soon as possible. Include the exact file or directory path and the date or range of versions you want recovered. Files noticed missing only after the backup-retention period may be permanently unrecoverable. The network filesystem has room for most routine data, but cryo-EM data and large X-ray datasets should use their dedicated storage facilities.

### Running jobs with Slurm

[Slurm](https://slurm.schedmd.com/) is an workload manager and job scheduler. It queues jobs until the requested CPU and memory resources are available. Slurm is available on **Jarvis** and **Ultron (ysbltest)**, but not on **Edith**. Compute-intensive or long-running work on jarvis and ysbltest should be submitted through Slurm; running such jobs directly in the login shell is discouraged.

### Single-threaded job

Create a file named, for example, `single.job`:

```bash
#!/bin/bash
#SBATCH --job-name=basic_job_test
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=00:05:00
#SBATCH --output=basic_job_%j.log

echo "Working directory: $(pwd)"
echo "Running on: $(hostname)"
echo "Started at: $(date)"
echo

# Replace the following line with the command you want to run.
COMMANDS_YOU_WOULD_RUN_IN_THE_TERMINAL

echo
echo "Job completed at: $(date)"
```

### Multi-threaded job

Create a file named, for example, `threaded.job`:

```bash
#!/bin/bash
#SBATCH --job-name=threaded_job_test
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=1G
#SBATCH --time=00:05:00
#SBATCH --output=threaded_job_%j.log

# Make common threaded libraries respect the Slurm CPU allocation.
export OMP_NUM_THREADS="$SLURM_CPUS_PER_TASK"
export OPENBLAS_NUM_THREADS="$SLURM_CPUS_PER_TASK"
export MKL_NUM_THREADS="$SLURM_CPUS_PER_TASK"

echo "Working directory: $(pwd)"
echo "Running on: $(hostname)"
echo "Allocated CPUs: $SLURM_CPUS_PER_TASK"
echo "Started at: $(date)"
echo

# Replace the following line with the command you want to run.
COMMANDS_YOU_WOULD_RUN_IN_THE_TERMINAL

echo
echo "Job completed at: $(date)"
```

`--cpus-per-task=4` requests four CPU cores. Change this value to match the program and set any program-specific thread option to the same number. `--mem=1G` is the total memory requested by the job, not the memory per CPU.

### Submit a job

```bash
sbatch threaded.job
```

Slurm prints a job ID after submission. The job output is written to the log file specified by `#SBATCH --output`.

### Check job status

Show all visible jobs:

```bash
squeue
```

Show only your jobs:

```bash
squeue -u "$USER"
```

### Cancel a job

Find the job ID with `squeue`, then run:

```bash
scancel 12345
```
