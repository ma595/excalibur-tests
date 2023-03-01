# ExCALIBUR tests

Performance benchmarks and regression tests for the ExCALIBUR project.

These benchmarks are based on a similar project by
[StackHPC](https://github.com/stackhpc/hpc-tests).

_**Note**: at the moment the ExCALIBUR benchmarks are a work-in-progress._

## Requirements

### Spack

[Spack](https://spack.io/) is a package manager specifically designed for HPC
facilities. In some HPC facilities there may be already a central Spack installation available.
However, the version installed is most likely too old to support all the features
used by this package. Therefore we recommend you install the latest version locally, 
following the instructions below.

_**Note**: if you have already installed spack locally and you want to upgrade to 
a newer version, you might first have to clear the cache to avoid conflicts: 
`spack clean -m`_

Follow the [official instructions](https://spack.readthedocs.io/en/latest/getting_started.html) 
to install the latest version of Spack (summarised here for convenience, but not guaranteed to be 
up-to-date):
- git clone spack:
`git clone -c feature.manyFiles=true https://github.com/spack/spack.git`
- run spack setup script: `source ./spack/share/spack/setup-env.sh`
- check spack is in `$PATH`, for example `spack --version`

In order to use Spack in ReFrame, the framework we use to run the benchmarks
(see below), the directory where the `spack` program is installed needs to be in
the `PATH` environment variable. This is taken care of by the `setup-env.sh` 
script as above, and you can have your shell init script (e.g. `.bashrc`)
do that automatically in every session, by adding the following lines to it:
```sh
export SPACK_ROOT="/path/to/spack"
if [ -f "${SPACK_ROOT}/share/spack/setup-env.sh" ]; then
    source "${SPACK_ROOT}/share/spack/setup-env.sh"
fi
```
replacing `/path/to/spack` with the actual path to your Spack installation.

ReFrame requires a [Spack
Environment](https://spack.readthedocs.io/en/latest/environments.html).  We
provide Spack environments for some of the systems that are part of the
ExCALIBUR and DiRac projects.  If you want to use a different Spack environment,
set the environment variable `EXCALIBUR_SPACK_ENV` to the path of the directory
where the environment is.  If this is not set, ReFrame will try to use the
environment for the current system, if known, otherwise it will automatically
create a very basic environment.

### ReFrame

[ReFrame](https://reframe-hpc.readthedocs.io/en/stable/) is a high-level
framework for writing regression tests for HPC systems.  For our tests we
require ReFrame 3.11.0.  Follow the [official
instructions](https://reframe-hpc.readthedocs.io/en/stable/started.html) to
install this package.  Note that ReFrame requires Python 3.6: in your HPC system
you may need to load a specific module to have this version of Python available.

We provide a ReFrame configuration file with the settings of some systems that
are part of the ExCALIBUR project.  You can point ReFrame to this file by
setting the
[`RFM_CONFIG_FILE`](https://reframe-hpc.readthedocs.io/en/stable/manpage.html#envvar-RFM_CONFIG_FILE)
environment variable:

```sh
export RFM_CONFIG_FILE="${PWD}/reframe_config.py"
```

If you want to use a different ReFrame configuration file, for example because
you use a different system, you can set this environment variable to the path of
that file.

**Note**: in order to use the Spack build system in ReFrame, the `spack`
executable must be in the `PATH` also on the compute nodes of a cluster, if
you want to run your benchmarks on them. This is taken care of by adding it 
to your init file (see spack section above).

However, you will also need to set the
[`RFM_USE_LOGIN_SHELL`](https://reframe-hpc.readthedocs.io/en/stable/manpage.html#envvar-RFM_USE_LOGIN_SHELL)
environment variable (`export RFM_USE_LOGIN_SHELL="On"`) in order to make ReFrame use

```sh
!#/bin/bash -l
```

as [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line, which would load
the user's init script.

## Usage

Once you have set up Spack and ReFrame, you can execute a benchmark with

```sh
reframe -c apps/BENCH_NAME -r --performance-report
```

where `apps/BENCH_NAME` is the directory where the benchmark is.  The command
above supposes you have the program `reframe` in your PATH, if it is not the
case you can also call `reframe` with its relative or absolute path.  

For example, to run the Sombrero benchmark in the `apps/sombrero` directory you can
use

```sh
reframe -c apps/sombrero -r --performance-report
```

For benchmarks that use the Spack build system, the tests define a default Spack specification
to be installed in the environment, but users can change it when invoking ReFrame on the
command line with the
[`-S`](https://reframe-hpc.readthedocs.io/en/stable/manpage.html#cmdoption-S) option to set
the `spack_spec` variable:

```
reframe -c apps/sombrero -r --performance-report -S spack_spec='sombrero@2021-08-16%intel'
```

Note that the `-S` option can be used to set from the command line on a per-job
basis all the built-in fields of ReFrame regressions classes. One useful such field is 
[`variables`](https://reframe-hpc.readthedocs.io/en/stable/regression_test_api.html#reframe.core.pipeline.RegressionTest.variables), 
which controls the environment variables used in a job.  For example

```
reframe -c apps/sombrero -r --performance-report -S variables=OMP_PLACES:threads
```

runs the `apps/sombrero` benchmark setting the environment variable `OMP_PLACES`
to `threads`.

### Selecting system and queue access options

The provided ReFrame configuration file contains the settings for multiple systems.  If you
use it, the automatic detection of the system may fail, as some systems may use clashing
hostnames.  You can always use the flag [`--system
NAME:PARTITION`](https://reframe-hpc.readthedocs.io/en/stable/manpage.html#cmdoption-system)
to specify the system (and optionally the partition) to use.

Additionally, if submitting jobs to the compute nodes requires additional options, like for
example the resource group you belong to (for example `--account=...` for Slurm), you have
to pass the command line flag
[`--job-option=...`](https://reframe-hpc.readthedocs.io/en/stable/manpage.html#cmdoption-J)
to `reframe` (e.g., `--job-option='--account=...'`).

### Unsupported systems

The configuration provided in [`reframe_config.py`](./reframe_config.py) lets you run the
benchmarks on systems for which the configuration has been already contributed.  However you
can still use this framework on any system by choosing the "generic" system with `--system
generic`, or using your own ReFrame configuration.  Note, however, that if you use the
"generic" system, ReFrame will not know anything about the queue manager of your system, if
any, or the MPI launcher.  For the benchmarks using the Spack build system, if you choose
the "generic" system, a new empty Spack environment will be automatically created in
`spack-environments/generic`.  In any case, you can always make the benchmarks use a
different Spack environment by setting the environment variable `EXCALIBUR_SPACK_ENV`
described above.

## Contributing new systems or benchmarks

Feel free to add new benchmark apps or support new systems that are part of the
ExCALIBUR benchmarking collaboration.  Read
[`CONTRIBUTING.md`](./CONTRIBUTING.md) for more details.
