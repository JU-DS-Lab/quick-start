# Accessing the Data Science Lab Cluster

## Introduction

The high-performance computing (HPC) cluster — "Apollo" — at the Data Science Lab is not a conventional computing cluster, as it comprises only two nodes: `DST-FIST-Server-01` and `DST-FIST-Server-02`. However, it still operates effectively as a functional proof of concept, with `DST-FIST-Server-02` serving as the master (as well as compute) node, and `DST-FIST-Server-01` functioning strictly as a compute node.

We use SLURM as the workload manager, given its widespread popularity and utilization in many state-of-the-art HPC systems. Additionally, we maintain an NFS-mounted directory at `/apollo`, which is accessible from both servers. This network storage consists of a RAID-5 array of HDDs with a total capacity of approximately 16 TB. We highly recommend utilizing this storage for code, datasets, virtual environments, cached files, and other pertinent data instead of the `/home` directory. This approach offers two primary advantages:

* It mitigates storage pressure on the boot disk, since `/home` and the root partition (`/`) reside on the same physical drive on both servers.
* It guarantees accessibility across all machines. Furthermore, it can be accessed directly from the Data Science Lab workstations (by opening a file manager and navigating to `/apollo`).

Ideally, `DST-FIST-Server-01` and `DST-FIST-Server-02` should possess equal computational power as members of a cluster; however, this is not the case. We recommend checking the system specifications in [the next section](#server-hardware-specifications) and deriving your own conclusions.

Consequently, you may choose to execute your programs on the server in two distinct ways:

* **The supported, recommended method**: using the SLURM scheduler
* **The unofficial, quicker method**: directly connecting to one of the compute nodes and running your code on it

### Server Hardware Specifications

| Component | Specification |
|---|---|
| **Host** | DST-FIST-Server-01 |
| **CPU** | Intel Xeon E5-2603 |
| **Cores** | 12 |
| **RAM** | 32 GB |
| **GPU** | nVidia Quadro P6000 |
| **GPU VRAM** | 24 GB |

*Hardware specifications for DST-FIST-Server-01.*

| Component | Specification |
|---|---|
| **Host** | DST-FIST-Server-02 |
| **CPU** | 2 × Intel Xeon Silver 4514Y |
| **Cores** | 2 × 32 = 64 |
| **RAM** | 512 GB |
| **GPU** | 2 × nVidia L4 |
| **GPU VRAM** | 2 × 24 GB = 48 GB |

*Hardware specifications for DST-FIST-Server-02.*

## Connecting to the Master Node

The master node, `DST-FIST-Server-02`, is assigned a static internal IP address: `172.24.56.211`. Please note that connecting to this node requires you to be on the departmental local area network (LAN). Ensure that you are either:

* stationed at a lab machine connected via Ethernet, or
* connected to the Wi-Fi network of one of the designated labs.

> [!NOTE]
> There is an active firewall on this server which explicitly permits access solely from specific labs within the department. If `ssh` fails or takes more than a few seconds, you may not have access to the server.

Once you have opened a terminal, please execute the following command:

```bash
$ ssh <username>@172.24.56.211
```

You will be prompted to enter your password. Upon successful authentication, you will be granted access to a login shell.

## Running Programs

Upon logging in, you can commence your work. In this guide, we assume that you are logged into `DST-FIST-Server-01` via `172.24.56.211`. The first step is to navigate to your workspace directory at `/apollo/<username>` before initiating any tasks.

```bash
$ cd /apollo/<username>
```

The shell prompt should update to reflect the new working directory.

> [!WARNING]
> Working within `/home/<username>` is strictly discouraged, particularly on the master server. Because `/home` and the root partition (`/`) are situated on the same physical disk, exhausting the storage in `/home` can precipitate a catastrophic system failure. The `/apollo` mount provides ample storage space; please ensure you utilize it.

Inside `/apollo/<username>`, create a new directory for your current project, or navigate into it if it has already been created.

```bash
$ mkdir <project-name> # skip if it already exists
$ cd <project-name>
```

**NOTE:** The remainder of this guide assumes you are utilizing Python.

### Virtual Environments

We recommend using [uv](https://docs.astral.sh/uv/) — a significantly fast Python package and project manager — to manage your virtual environments. If a virtual environment has already been create, you may activate it directly.

**NOTE:** Before creating a new virtual environment, please ensure you are located in `/apollo/<username>/<project-name>`. You may inspect your prompt or use the `pwd` command to verify your current working directory.

```bash
$ uv venv --python 3.12 # skip if it already exists
$ source .venv/bin/activate
$ which python # should display a new path
```

> [!NOTE]
> Please note the leftmost section of your bash prompt after activating the virtual environment; it should display the name of the activated environment. If you are utilizing an alternative shell with a customized prompt, we assume you possess the requisite knowledge to configure it to display the virtual environment name correctly.

### Using SLURM

To utilize SLURM, you must prepare a [batch script](https://slurm.schedmd.com/sbatch.html) that defines the resources required and the commands to execute. Create a new file (e.g., `job.sh`) and include the appropriate `#SBATCH` directives at the beginning. Ensure that you activate your virtual environment using its absolute path within the script.

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

# Activate the virtual environment (please use full path here)
source /apollo/<username>/<project-name>/.venv/bin/activate

python main.py
```

Once your script is ready, you can submit the job to the cluster scheduler using the `sbatch` command:

```bash
$ sbatch job.sh
```

To monitor the status of your submitted jobs, use the `squeue` command:

```bash
$ squeue -u <username>
```

### Running Directly

If you opt for the unsupported method of running your code directly on the server, first ensure that your virtual environment is activated by verifying the shell prompt or using the `which python` command.

After activating the environment, you may run your python script directly:

```bash
$ python <path-to-script>
```

#### Using Interactive Notebooks (JupyterLab)

[JupyterLab](https://jupyter.org/) provides an interactive web-based environment for executing code, visualizing data, and documenting the process via interactive "notebooks". It is highly beneficial for exploratory data analysis and iterative development.

> [!WARNING]
> Active Jupyter Notebook kernels can consume significant system memory (RAM) and video memory (VRAM) if left idling. Since these resources are shared, failing to release them may prevent other users from submitting jobs via SLURM or running their own workloads. To mitigate this, always ensure you properly shut down the Jupyter server when finished or [restart/kill the specific kernels](https://jupyterlab.readthedocs.io/en/stable/user/running.html) when they are no longer in active use.

To start a JupyterLab instance directly, execute the following command on the server you're currently logged into:

```bash
$ jupyter lab --ip='*' --IdentityProvider.token="" --PasswordIdentityProvider.hashed_password=""
```

After the server starts, open a web browser on your local machine and navigate to [http://172.24.56.211:8888/](http://172.24.56.211:8888/) (8888 is the default, but it may be different for you; please check the terminal output to confirm).

> [!WARNING]
> Starting JupyterLab with an empty token and no password presents a significant security risk, as anyone in one of the allowed labs accessing the departmental LAN could potentially access your JupyterLab instance.

For instructions on configuring proper authentication and securing your Jupyter server, please refer to the [official Jupyter Server security documentation](https://jupyter-server.readthedocs.io/en/latest/operators/security.html).

#### Using Visual Studio Code

[Visual Studio Code](https://code.visualstudio.com/) (VS Code) is a highly versatile, feature-rich code editor that has become widely popular today. We specifically mention it here for interacting with the cluster because of its beginner-friendly interface and excellent built-in support for remote development via SSH.

To connect to the cluster using VS Code, follow these steps:

1. Click the **Remote Window** icon at the absolute bottom-left corner of the VS Code interface (it looks like two brackets "`><`").
2. Select **Connect to Host...** and then **SSH** from the dropdown list.
3. If this is your first time connecting, please set up the remote host:
   1. Wait a moment for the **Remote - SSH** extension to initialize or auto-install if prompted.
   2. Select **+ Add New SSH Host...**.
   3. Enter the following connection command:
      `ssh <username>@172.24.56.211` and press `Enter`.
   4. Select your user-level SSH configuration file (typically the first option in the list).
   5. Click the Remote Window button in the bottom-left corner again to return to the selection menu.
4. Select **Connect to Host...**.
5. Select the entry for **172.24.56.211**.
6. Once the connection is established and the new remote window opens, the status bar button in the bottom-left corner will reflect the remote hostname. You may now open folders on the server and begin your work.

For more detailed assistance, please refer to the [official VS Code Remote SSH documentation](https://aka.ms/vscode-remote/ssh).
