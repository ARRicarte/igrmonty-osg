# igrmonty-osg

A simple setup for running a large number of `igrmonty` jobs on the
Open Science Grid (OSG) using HTCondor.

## Usage

This git repository is designed to be cloned to OSG connect as a
run/work directory for both submitting a large OSG job and gathering
the output.

It is recommended to place the output of your OSG jobs at your OSG
connect home `/home/<user>`.
Therefore, to start,

    ssh <user>@login<ID>.osgconnect.net
    mkdir -p runs
    cd runs
    git clone git@github.com:bhpire/igrmonty-osg.git Ma+0.94_w5_SED
    cd Ma+0.94_w5_SED

All the scripts are placed inside `bin/`.
The empty directories `log/`, and `out/` will be used to stage OSG run
logs and grmonty output.

It is recommended to place the (large) input of your OSG jobs at the
public directory `/public/<user>`.
I.e.,

    ssh <user>@login<ID>.osgconnect.net
    cd /public/<user>
    mkdir -p eht/sgra/{bias,md5,rho0,Ma+0.94_w5}
    cd eht/sgra
    # Populate information tables in `bias/`, `md5/`, and `rho0/`
    rsync -rav <supercomputer>:/GRMHD/snapshots/ Ma+0.94_w5/

Before submitting a job, one needs to copy a static linked `grmonty`
binary to `bin/`

    ssh <user>@login<ID>.osgconnect.net
    mkdir -p src
    cd src
    git clone https://github.com/AFD-Illinois/igrmonty.git
    cd igrmonty
    # Change the default N_THBINS in `/model/iharm/model.h` to a larger number, e.g.,
    # #define N_THBINS 18
    # Edit `make` so that the CFLAGS line contains `-static`; e.g.,
    # CFLAGS = -static -std=gnu99 -O3 -fopenmp -funroll-loops -Wall -Wextra
    module load gsl hdf5
    make

    cd ~/runs/Ma+0.94_w5_SED
    cp ~/src/igrmonty/grmonty bin

Also make sure that you update the md5sum of grmonty in `bin/wrapper`, e.g.,

     grmd5="22691ef253e109166acb1eb5d5ac1084"

You may also need to edit `bin/pargen` to generate the necessary
parameter sets.
Then, simply submit an OSG job by

    bin/batches

`bin/batches` will create a table of input parameters of Condor and
put it in `par/BATCH.ALL`.  However, because at the EHT we are running
a large number of jobs, they may exceed the maximum allowed jobs on
your queue.  Therefore, `bin/batches` also split `par/BATCH.ALL` into
smaller files `par/batch.p00`, `par/batch.p01`, ... and try to submit
each of them individually.  Once a job is submitted successfully, it
will be renamed as `par/BATCH.D00`.  You may simply rerun
`bin/batches` multiple times.  Every time it will try to submit the
last unsubmitted `par/batch.pXX` file.

`bin/batches` uses `bin/submit` under the hood, which is a standard
Condor submission script starts with a hashbang directive to use the
system `condor_submit`.

It uses `bin/parget` to get the list of parameters from the
corresponding file in `par/`.
These parameter sets are then passed to `bin/wrapper` on the worker
machines as command line arguments.
`bin/wrapper` will automatically generate a `grmonty` parameter file
based on the arguments, and start `grmonty` with it.

All Condor logs, `stdout`, and `stderr` will be sent to `log/`.
The parameter files, hotcross data, and spectrum output will be saved
to `out/`.
Their file names are transform according to the rules described in
`bin/submit`.
