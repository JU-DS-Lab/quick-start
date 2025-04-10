# Quick Start Guide for the HPC Network at Data Science Lab

This document serves as a guide for users of a high-performance computing (HPC) facility in the form of a cluster ("Apollo") for running parallel and distributed programs.

## Accessing the Cluster

The cluster may be accessed by connecting to the **master node** via **Secure Shell (SSH)**. 

**The IP address of the master node is** `172.24.56.211`.

This IP address should be accessible from certain designated locations within the CSE department. To connect to the master node, use the following command:

```bash
$ ssh <username>@172.24.56.211
```

Replace `<username>` with the username provided to you. You will need to enter your password to log in.

## Using SLURM

Here we provide a very brief overview of the most commonly used SLURM commands. For a more detailed guide, please refer to the [official SLURM documentation](https://slurm.schedmd.com/) or use the `man` command to view the manual pages for each command.

### Scheduling Jobs

The slurm commands responsible for scheduling jobs are `srun` and `sbatch`.
- `srun`: used to run a job on the cluster interactively

    Usage: `$ srun [options] command [args]`
- `sbatch`: used to submit a batch job to the cluster

    Usage: `$ sbatch [options] job-script`

### Monitoring Jobs

The `squeue` command is used to monitor the jobs that are currently running or waiting in the queue.

Usage: `$ squeue [options]`

### Cancelling Jobs

The `scancel` command is used to cancel a job that is currently running or waiting in the queue.

Usage: `$ scancel [options] job-id`

### Managing SLURM Configuration

The `scontrol` command is used to view or modify the SLURM configuration. This includes settings up nodes, partitions, and other resources.

> [!IMPORTANT]
> Most `scontrol` commands require administrative privileges. Please consult the system administrator before making any changes to the SLURM configuration.

## Example: Running a Python Program

The Python programming language is widely used for machine learning, scientific computing and data analysis. Here we provide an example of how to run a simple python program on the cluster.

### Step 1: Set Up and Navigate to Your Working Directory

First, navigate to your designated directory inside the shared filesystem.

```bash
$ cd /apollo/<username>
```

Create a new directory for your python project. Ensure that all your files pertaining to that project are stored there:

```bash
$ mkdir <project-name>
```

Once you have uploaded everything to this directory, move to it:

```bash
$ cd <project-name>
```

### Step 2: Create a Virtual Environment

If you have not already created a virtual environment, set one up:

```bash
$ python3 -m venv myenv
```

This will create a folder named `myenv`, which will contain an isolated Python environment.

### Step 3: Activate the Virtual Environment

Activate the virtual environment before installing packages:

```bash
$ source myenv/bin/activate
```

### Step 4: Install Required Python Packages

If you have a requirements.txt, use that to install the requried packages:

```bash
$ pip install -r requirements.txt
```

Otherwise, install the required packages individually:
```bash
$ pip install <package-1> <package-2> ...
```

### Step 5: Write a SLURM Job Script

Create a new file named `job.sh` and write the following contents:

```bash
#!/bin/bash
#SBATCH --job-name=my_python_job   # Job name
#SBATCH --output=output.log        # Output file
#SBATCH --error=error.log          # Error file
#SBATCH --time=01:00:00            # Time limit (hh:mm:ss)
#SBATCH --nodes=1                  # Number of nodes
#SBATCH --ntasks=1                 # Number of tasks
#SBATCH --cpus-per-task=4          # Number of CPU cores
#SBATCH --gpus-per-node=1          # Number of GPUs
#SBATCH --mem=4G                   # Memory (adjust as needed)

# Navigate to project directory on shared filesystem
cd /apollo/<username>/<project-name>

# Activate the virtual environment
source myenv/bin/activate

# Run the Python program
python main.py
```

### Step 6: Submit the Job

Use the `sbatch` command to submit the job script to the cluster. The SLURM scheduler will allocate resources and run the job when they become available.

```bash
$ sbatch job.sh
```

### Step 7: Monitor the Job

Use the `squeue` command to monitor the status of the job:

```bash
$ squeue -u <username>
```

### Step 8: Viewing the Output

Once the job has completed, you can view the output and error logs:

```bash
$ cat output.log
```

```bash
$ cat error.log
```

