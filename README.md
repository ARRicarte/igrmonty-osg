# igrmonty-osg

A simple setup for running a large number of `igrmonty` jobs on the
Open Science Grid (OSG) using HTCondor.

## Usage

This git repository is designed to be cloned to OSG connect as a run
directory for both submitting a large OSG job and gathering the
output.
To start with, one may

    ssh osg
	cd /public/<user>/
    git clone git@github.com:bhpire/igrmonty-osg.git run-01
	cd run-01

All the scripts are already placed inside `bin/`.
The empty directories `dat/`, `log/`, and `out/` will be used to stage
input GRMHD data, OSG run logs, and grmonty output.

Before submitting a job, one needs to copy a static linked `grmonty`
program to `bin/` and populate `dat/` with GRMHD data.

    cp ~/src/igrmonty/grmonty bin
	scp bh:sim-lib/sample/dump_*.h5 dat

One should also edit `bin/pargen` to generate the necessary parameter
set.
Then, one can simply submit an OSG job by

    bin/submit

`bin/submit` is a standard Condor submission script starts with a
hashbang directive to use the system `condor_submit`.
It uses `bin/pargen` to generate a list of parameter sets based on all
the GRMHD input data in `dat/` and the parameter listed inside
`bin/pargen`.
These parameter sets are then passed to `bin/wrapper` on the worker
machines as command line arguments.
`bin/wrapper` automatically generate a `grmonty` parameter file based
on the arguments, and start `grmonty` with it.
All Condor logs, `stdout`, and `stderr` will be sent to `log/`.
The parameter files, hotcross data, and spectrum output will be saved
to `out/`.
Their file names are transform according to the rules described in
`bin/submit`.
