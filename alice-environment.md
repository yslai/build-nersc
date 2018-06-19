# Native ALICE aliBuild environment at NERSC

The script `run-alibuild --prefix=/project/projectdirs/project1/user2/alice-haswell` fully builds the standard (client) ALICE software environment installing into a project directory. While `run-alibuild` can be used without any option, installing to `$HOME/alice`, the installation is approximately 20 GB large, occupying half of the standard NERSC home directory quota (which also resides on a slower mount).

A CERN account is still needed to pull DPMJET from protected CERN GitLab ([alidist issue #833](https://github.com/alisw/alidist/issues/833)).

Example `.bash_profile.ext` snippet for both Cori and with PDSF using CVMFS, where the user is expected to executes `alish` to load the ALICE environment:

```
alice_haswell="/project/projectdirs/project1/user2/alice-haswell"
alice_analysis_data="/project/projectdirs/project1/user2/analysis-data"
alice_aliphysics_latest="AliPhysics/latest-ali-master-release"
if [[ "$NERSC_HOST" = cori && -d "$alice_haswell" ]]; then
    export PATH="$PATH:$alice_haswell/alibuild"
    export ALIBUILD_WORK_DIR="$alice_haswell/sw"
    alias alish="eval \$(alienv load ${alice_aliphysics_latest})"
    [[ -d "${alice_analysis_data}" ]] && \
        export ALICE_DATA="${alice_analysis_data}"
elif [[ -d /cvmfs ]]; then
    source /cvmfs/alice.cern.ch/etc/login.sh
    alias alish="eval \$(alienv load AliPhysics)"
fi
unset alice_aliphysics_latest
unset alice_analysis_data
unset alice_haswell
```

See [ALICE GitHub advanced workflow](https://alisw.github.io/git-advanced/) how to copy the OADB directory from CERN EOS to NERSC, e.g. into the directory `$alice_analysis_data` points to above.
