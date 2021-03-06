# Interactive usage

When you login to CSC supercomputers, you end up in one of the login nodes of the computer. These login nodes are shared by all users and they are **not** intended for heavy computing. See our [Usage policy](../overview.md#usage-policy) for details.
If you need to do heavy computing interactvely, you should use interactive batch jobs.

In an interactive batch job, a user submits a batch job that starts an interactive shell session in a computing node. For heavy interactive tasks user can also request spesific resources (time, memory, cores, disk). You can also use tools with graphical user interfaces in this interactive shell session, but in this case it is recommended that you do the initial connection to a login node of the supercomputer with [NoMachine](../../support/tutorials/nomachine-usage.md) virtual desktop.

Please notice that the interactive batch jobs run in the computing nodes, where the environment differs 
slightly from the login nodes. For example, not all the same text editors are available. Furthermore, when you log out from an interactive batch job, the session, including all the processes running in the session and data in the job specific `$TMPDIR` area, will be terminated. 

## Easy interactive work: sinteractive command

Puhti has an `interactive` partition which enables immediate access to an interactive batch job session. The easiest way to use this resource is to execute the `sinteractive` command:

```text
sinteractive --account <project_name> 
```

This command opens a shell session that runs on a compute node. You can use this
session as a normal bash shell without additional Slurm commands for starting jobs and applications.

The default sinteractive resources cover typical use cases, but you can also request more
from the command line. For example, an interactive session with 8 GiB 
of memory, 48 h runnig time and 100 GiB local scratch using project _project_2011234_
can be lauched with command:

```text
sinteractive --account project_2011234 --time 48:00:00 --mem 8000 --tmp 100
```

Available options for `sinteractive` are:

| Option        | Function                                                 | Default              |
| ------------- | -------------------------------------------------------- | -------------------- |
| -t, --time    | Run time reservation in minutes or in format d-hh:mm:ss. | 24:00:00             |
| -m, --mem     | Memory reservation in MB.                                | 1000                 |
| -j, --jobname | Job name.                                                | interactive          |
| -c, --cores   | Number of cores.                                         | 1                    |
| -A, --account | Accounting project.                                      | $CSC_PRIMARY_PROJECT |
| -d, --tmp     | Size of job specifinc $TMPDIR disk (in GiB).             | 32                   |
| -g, --gpu     | Number of GPU:s to reserve (max 4)                       | 0                    |

Note, that each user can have only one active session open in the `interactive` partition. 
In the interactive partition you can reserve in maximum 1 core, 16 GB of 
memory, 7 days of time, 160 GB of local scratch space and 0 gpus.

If your requests exceed these limits or you already have a session in the
interactive partition, `sinteractive` can submit the session request to `small` or `gpu`
partitions instead. However, in these cases your session starts queueing just like normal batch job and
you may need to wait some time before the requested resources become available and the interactive session 
starts.

All the `sinterative` sessions are executed in nodes that have [NVMe fast local disk area](/computing/running/creating-job-scripts/#local-storage) available. The environment variable `$TMPDIR` points to the local disk area of the job. This local disk area has high I/O capacity and thus it is ideal location for temporary files created by the application. Note however, that this disk area is erased when the interactive batch job session ends.

### Example: Running a Jupyter notebook server via sinteractive

You can start a Jupyter notebook server on a Puhti compute node,
and access it with your local web browser. In this case there is no
need to start NoMachine. In the Puhti terminal session, run the command:

```text
sinteractive --account <project> start-jupyter-server
```

This command will start the server, and it will then print out a web
address and a ssh command. Execute the ssh command (copy-paste) in
another linux terminal on your local machine to form a tunnel between
your machine and the compute node. Note that you need to set up
passwordless access using ssh keys to do so. After this you can access
the Jupyter server by copy-pasting the web address into your local web browser.

### Example: RStudio in sinteractive session

Open connection to Puhti with NoMachine.
In the Puhti terminal session, run commands:

```text
sinteractive --account <project> --mem 8000 --tmp 100
module load r-env 
module load rstudio
rstudio
```

## Explicit interactive shell without X11 graphics

If you don't want to use the `sinteractive` wrapper, it's possible
to use Slurm commands explicitly.
Note, as you may need to queue, it's convenient to ask for an email once the resources have been granted. 

```
srun --ntasks=1 --time=00:10:00 --mem=1G --pty \
  --account=<project> --partition=small --mail-type=BEGIN \
  --mail-user=<your email address> bash
```

Once the resource has been granted, you can work normally in the shell.
The bash prompt will show the
name of the compute node:

```bash
[csc-username@r07c02 ~]$
```

Once the requested time has passed, the shell exits automatically.

## Starting an interactive application with X11 graphics

To enable X11 graphics, add `--x11=first` to the command.
The following will start the application `myprog`: 

```
srun --ntasks=1 --time=00:10:00 --mem=1G --x11=first --pty \
  --account=<project> --partition=small --mail-type=BEGIN \
  --mail-user=<your email address> myprog
```

Note, that you can replace `myprog` with `bash`, which will launch a terminal
on the compute node. From that terminal you can launch also graphical applications.
Once the requested time has passed, the application will be
automatically shutdown.
