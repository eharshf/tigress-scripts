# TIGRESS-scripts

# Setup and Example scripts for R-based HPC at Princeton University

Contact Jonathan Olmsted (jolmsted@princeton.edu) with questions or issues.

## What?

This project comprises a set of 6 examples scripts and a setup script to help
prepare your HPC environment for R-based HPC. These scripts work *out of the
box* on both Tukey and Della. More information on these scripts is below.

## How to ... ?

### get a copy

You can create a full copy of this git repository by cloning it. To do this, run
the following at the shell prompt:

```
git clone https://github.com/olmjo/tigress-scripts
```

On the Princeton TIGRESS systems, `git` will be installed so this "just
works". The same `git` command will work on your local machine if you have `git`
installed.

### use

#### Setup
For all of the example scripts to work, the shell environment needs to be set up
correctly and particular R packages need to be installed. The helper setup
script is located in `./setup/setup.sh`. To set up your environment, simply run
the script:
```
cd setup
./setup.sh
```

You will be prompted by several questions like

```
Do you need to DL Rmpi? [y/n]
Do you need to add openmpi libs to the linker path? [y/n]
Do you need to install Rmpi? [y/n]
Do you need to misc. HPC packages? [y/n]n
```

If you've never run this before, answer "y" to each question.

To test the openmpi setup run the following:
```
cd ../test/
./test_mpi.sh
```

You should see output like:
```
Process 1 on tukey out of 3
Process 2 on tukey out of 3
Process 0 on tukey out of 3
```

If you see information like the following mixed in, that's okay.

```
librdmacm: Fatal: no RDMA devices found
--------------------------------------------------------------------------
[[20318,1],0]: A high-performance Open MPI point-to-point messaging module
was unable to find any relevant network interfaces:

Module: OpenFabrics (openib)
  Host: tukey

Another transport will be used instead, although this may result in
lower performance.
--------------------------------------------------------------------------
librdmacm: Fatal: no RDMA devices found
librdmacm: Fatal: no RDMA devices found
```

#### Examples

There are a total of six sets of example scripts in this project. Each set can
be used to submit a perfectly valid job on (at least) Della and Tukey. They are
located in the `./examples/` subdirectory. The shell scripts with a `.pbs`
suffix are PBS scripts for the resource manager. These represent a complete
description of what resources you need and what steps you need the computing to
perform for you. The scripts with a `.R` suffix are R scripts that represent our
computational jobs. The fifth example is for Matlab, and the `.m` script is our
computational job. The concepts and features demonstrated here should cover
nearly all usage.

## Example 0: Bare Bones

This is a bare bones example. It requests 1 node with 1 processor on the
node. It allows the scheduler to kill the job after 10 minutes. The script
simply generates 1,000 random numbers using the Rscript interface to R.

See `./examples/ex0/`.

## Example 1: A Reasonable Default

This PBS script represents a reasonable starting point for simple jobs. It is
more explicit about how the job should be managed than Example 0. It still
requests 1 node with 1 processor. It requests only 10 minutes of time. It uses a
custom name in the queue and has both the error log and the output log merged
into one file which begins with log.* and has a suffix determined by the job
ID. It requests emails when it begins, ends, and aborts (the email address can
be specified manually, but works by default on TIGRESS systems).

Every line beginning with # is just a PBS directive. The remainder comprises an
actual shell script. The script is verbose about where it is, when it starts,
and what resources were given to it by the scheduler.

The script ultimately generates 1,000 random numbers using the Rscript interface
to R.

See `./examples/ex1/`.

## Example 2: Example 1 + an R script

This PBS script includes all the reasonable defaults from Example 1. The only
change is that it uses Rscript to run an external R script, which is how the job
would usually be programmed.

The computational task in R is a copy of the example usage of `ideal()` from the R
package **pscl**.

See ``./examples/ex2/``.

## Example 3: Example 2 + parallel execution + passing arguments to R

This PBS script uses the sample reasonable defaults from above, but it requests
two processors on the node. We define the environmental variable `TOTALPROCS` and
make sure it is available to processes started in this script (via export). Now
Rscript has to be invoked from within the mpiexec command. The master MPI
process knows what resources were allocated from `-hostfile
$PBS_NODEFILE`. Lastly, our R script is taking a final unnamed argument of 8.

In the R script, the first set of commands parses the command line argument
passed to R (i.e. 8). The next set of commands are reading the environmental
variable `TOTALPROCS`. We then set up the MPI backend and as each of our MPI
workers to tell us where they are running.

See `./examples/ex3/`.

## Example 4: Example 2 + job arrays + passing arguments to R

This PBS script now requests an array of jobs based on the template. For jobs in
this array (indexed from 1 to 3), the shell script will run given the requested
resources. Because the log file depends on the job ID, each of the three jobs
will generated different log.* output. Because we R can read the environmental
variable `PBS_ARRAYID`, we are able to use the index on an object of interest to
us in R (e.g., a vector of names of files in a directory to be processed).

With this setup, each sub-job is requesting the same resources.

See `./examples/ex4/`.

## Example 5: single-node Matlab parallel execution

This PBS script uses the default setup (see ex1), requests 5 processors on a
single node, and runs a Matlab script. The Matlab script executes a loop
sequentially and then in parallel where each of `MC` iterations takes `MC/DUR`
seconds by construction. The parallel loop (i.e., the one using the `parfor`
construct) should be about `PROCS` times faster. This approach does not
generalize to multiple nodes.

## Example 6: "Substantive" Example

This example is less a demonstration of features available (e.g., there is no
use of job arrays or command line arguments) and, instead, shows a computational
job that provides a good template for many other embarrassingly parallel
computational tasks. Here, the goal is use the non-parametric bootstrap to
approximate sampling distribution of correlation coefficients based on samples
of size 25. The correlation of interest is between average undergraduate GPA and
average LSAT scores among students at 82 different law schools.

The output generated from the R script is just the deciles from this
distribution (without acceleration or bias-correction).
