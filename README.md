# Glycojons Group Computing Resources

The Glycojons Group has access to some local Linux compute machines: **edith**, **ultron**, and **jarvis**. The lab machine **bmo** can also be used when available. University-level high-performance computing is available through **[Viking](https://vikingdocs.york.ac.uk/)**. Please contact your PI if you need access to Viking.

## Choosing a machine

| Machine | CPU | Memory | GPU | Connection | Recommended use |
|---|---|---|---|---|---|
| **edith** | Intel Core i9-9900K, 8 cores / 16 threads, up to 5.0 GHz | 32 GiB | NVIDIA GeForce RTX 2060, 8 GB | `ssh <username>@edith` (no Slurm) | Interactive work, code development, testing, and small CPU jobs |
| **ysbltest** | 2 × Intel Xeon Gold 6526Y, 32 cores / 64 threads in total | 128 GiB | 2 × NVIDIA A30, 24 GB ECC each | `ssh <username>@ysbltest`; submit jobs with `sbatch` | Long-running or reliable GPU computation, two independent GPU jobs, and medium-sized parallel CPU jobs |
| **jarvis** | AMD Ryzen Threadripper PRO 3995WX, 64 cores / 128 threads | 256 GiB | NVIDIA GeForce RTX 3090, 24 GB | `ssh <username>@jarvis`; submit jobs with `sbatch` | Large CPU-parallel workloads, batch processing, and high-throughput single-GPU computation |
| **bmo** | Intel Core i9-11900, 8 cores / 16 threads, up to 5.2 GHz | 32 GiB | NVIDIA GeForce RTX 3090, 24 GB | `ssh <username>@bmo`; shared by the whole laboratory | Interactive work and single-GPU computation when the machine is available; coordinate long jobs with other users |

Connections may require the University of York network or VPN. Ask the PI or machine administrator if you need a local account.

For short interactive tests, choose **edith**. For large numbers of CPU tasks, choose **jarvis**. For ECC GPU memory or two simultaneous GPU jobs, choose **ysbltest**. **bmo** provides an additional RTX 3090 but is shared across the whole laboratory, so availability may vary. For workloads that exceed the local machines, use **Viking**.

### Structural databases on jarvis

Jarvis maintains local copies of the following structural databases. They are updated every Monday:

| Database | Location |
|---|---|
| Standard PDB mmCIF archive | `/vault/xhpi_pdb_July2026/pdb/mmcif` |
| PDB-REDO | `/vault/xhpi_pdb_July2026/pdb-redo` |

### Home directories and backups

The Linux home directory is stored on the same network filesystem across YSBL computers. Files saved under `/y/people/<username>/` are therefore available when you log in to another YSBL Linux machine and are protected by regular backups.

Snapshots are taken every four hours during the day, retained daily for one week, and then retained weekly for one month. Tape backups are retained for 90 days, and the complete filesystem is also copied nightly to a backup server in a separate machine room. For normal long-term data protection, keep the data on the network filesystem; a backup is not permanent storage for a file that has been deleted from the live filesystem.

If you accidentally delete or damage a file, email [tim.kirk@york.ac.uk](mailto:tim.kirk@york.ac.uk) as soon as possible. Include the exact file or directory path and the date or range of versions you want recovered. Files noticed missing only after the backup-retention period may be permanently unrecoverable. The network filesystem has room for most routine data, but cryo-EM data and large X-ray datasets should use their dedicated storage facilities.

## Running jobs with Slurm

[Slurm](https://slurm.schedmd.com/) is an open-source workload manager and job scheduler. It queues jobs until the requested CPU and memory resources are available. Slurm is available on **jarvis** and **ysbltest**, but not on **edith**. Compute-intensive or long-running work on jarvis and ysbltest should be submitted through Slurm; running such jobs directly in the login shell is discouraged.

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
sbatch JOBNAME.job
```

For example:

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
scancel JOB_ID
```

For example:

```bash
scancel 12345
```

Only request the CPU cores and memory that the program can actually use. This keeps the shared machines responsive and allows other group members' jobs to run.
