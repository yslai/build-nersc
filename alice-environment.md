# Native ALICE aliBuild environment at NERSC

## Build instruction

Run the script with
```
run-alibuild --prefix=/project/projectdirs/project1/user2/alice-haswell
```
fully builds the standard (client) [ALICE software environment](https://alisw.github.io/) installing into a project directory. While `run-alibuild` can be used without any option, installing to `$HOME/alice`, the installation is approximately 20 GB large, occupying half of the [standard NERSC home directory](http://www.nersc.gov/users/storage-and-file-systems/file-systems/) quota (which also resides on a slower mount).

A [CERN account](https://account.cern.ch/account/) is still needed to pull the [DPMJET](http://sroesler.web.cern.ch/sroesler/dpmjet3.html) event generator from a protected CERN GitLab repository ([alidist issue #833](https://github.com/alisw/alidist/issues/833)). Use [`git-credential-store` and `git-credential-cache`](https://alisw.github.io/git-advanced/#setup-a-git-credentials-cache) with the CERN account, if the build needs to complete without user interaction.

This script will apply following modification to the AliRoot code, that does not affect the function as grid client:

* Fixing memory access bugs

## Setting up login environment

Example `.bash_profile.ext` snippet for both Cori and with PDSF using CVMFS, where the user is expected to executes `alish` to load the ALICE environment (replacing `project1/user2` with your project and user ID, and redefine the two `alias alish=`&hellip; to change the loading command):

```
alice_haswell="/project/projectdirs/project1/user2/alice-haswell"
alice_analysis_data="/project/projectdirs/project1/user2/analysis-data"
alice_aliphysics_latest="AliPhysics/latest-ali-master-release"
if [[ "${NERSC_HOST}" = cori && -d "${alice_haswell}" ]]; then
    alias alish="\
export PATH=\"\${PATH}:${alice_haswell}/alibuild\"; \
export ALIBUILD_WORK_DIR=\"${alice_haswell}/sw\"; \
[ -d "${alice_analysis_data}" ] && \
export ALICE_DATA="${alice_analysis_data}"; \
eval \$(alienv load ${alice_aliphysics_latest})"
elif [[ -d /cvmfs ]]; then
    source /cvmfs/alice.cern.ch/etc/login.sh
    alias alish="eval \$(alienv load AliPhysics)"
fi
unset alice_aliphysics_latest
unset alice_analysis_data
unset alice_haswell
```

## Local OADB copy

See [ALICE GitHub advanced workflow](https://alisw.github.io/git-advanced/) how to copy the offline analysis database (OADB) directory from [CERN EOS](http://information-technology.web.cern.ch/services/eos-service) to the local file system, e.g. into the directory `$alice_analysis_data` points to above.

## Monte Carlo simulation on interactive nodes

Assuming the directory `calibration` is stored on GPFS project directory `/project/projectdirs/project1/user2/fake_cvmfs` (same for a SquashFS mount point, see [how to create this](#make-a-local-ocdb-copy)), and the run anchor is (the LHC16k) run 257209:
```
OCDB_PATH=/project/projectdirs/project1/user2/fake_cvmfs ./dpgsim.sh --run 257209 --mode ocdb
```

`LIBC_FATAL_STDERR_=1`

The (undocumented) glibc environment variable causes potential crashes to be logged via stderr.

## Monte Carlo simulation in SLURM

## Make a local OCDB copy

The OCDB (410 GB and containing 1.5 million files as of June 2018) can be transferred using a system with CVMFS mount (PDSF or shifter image on MPP systems) by:
```
rsync -ahvPSX /cvmfs/alice-ocdb.cern.ch/calibration /project/projectdirs/project1/user2/fake_cvmfs/
```
Transfer speed is 1&ndash;2 MB/s, and 2&ndash;3 days should be planned for to transfer the entire OCDB.

The SquashFS image is then created using:
```
mksquashfs /project/projectdirs/project1/user2/fake_cvmfs/calibration calibration-YYYYMMDD.sqsh -comp xz -b 1048576 -Xdict-size '100%' -no-xattrs -all-root -always-use-fragments -info
```
and can be archived to HPSS. The creation time is about 5&frac12; hours (and turning the compression completely off only shaves 1&ndash;2 hours off that).
