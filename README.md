# trace2sing
Trace program execution and create [Singularity](http://singularity.lbl.gov) container for reproducible execution

## How it works
trace2sing execute a command in order to trace it with strace, it will copy files needed by
command to create a [Singularity](http://singularity.lbl.gov) container for further execution on another system with same
architecture.

## Demo
[![asciicast](https://asciinema.org/a/90fve3i9t0ossj06a8tptajzw.png)](https://asciinema.org/a/90fve3i9t0ossj06a8tptajzw)

## Requirements
trace2sing is design to be portable and require the following packages:
 * strace
 * GNU awk (installed by default on many distributions) 
 * coreutils (installed by default on many distributions)
 * binutils (installed by default on many distributions)

It was tested on:
 * CentOS 6, 7 (yum install strace)
 * Ubuntu 14.04, 16.04, 17.04 (apt-get install strace)
 * Alpine Linux 3.2 and edge (apk add strace gawk)

The system which execute singrun.sh must have Singularity installed.

## Limitations
 * Same as Singularity
 * Generated containers need Singularity >= 2.2 with a linux kernel >= 3.5
 * Generated containers are compatible with systems based on same CPU architecture

## How to use it

### trace2sing

To trace command and package into an executable Singularity container:

```bash
user@local:~$ trace2sing python -c "print 'Hello !'"
Hello !
Generating self-extracting singrun.sh archive ... please wait
```

The above command generate an executable named singrun.sh, this file
embed the container root filesystem to use with Singularity.

### singrun.sh
 * To extract container file tree into singrun_rootfs directory:
```bash
user@local:~$ ./singrun.sh -x
Extracting file tree into singrun_rootfs directory
```

 * To list container files:
```bash
user@local:~$ ./singrun.sh -l
List file tree
drwxr-xr-x ced/ced           0 2017-05-21 21:49 ./
drwxrwxrwt ced/ced           0 2017-05-21 21:49 ./tmp/
...
drwxr-xr-x ced/ced           0 2017-05-21 21:49 ./dev/
drwxr-xr-x ced/ced           0 2017-05-21 21:49 ./proc/
drwxr-xr-x ced/ced           0 2017-05-21 21:49 ./etc/
-rw-r--r-- ced/ced        2995 2016-04-15 00:09 ./etc/locale.alias
...
drwxr-xr-x ced/ced           0 2017-05-21 21:49 ./lib64/
lrwxrwxrwx ced/ced           0 2017-03-21 21:06 ./lib64/ld-linux-x86-64.so.2 -> /lib/x86_64-linux-gnu/ld-2.23.so
```

 * To display exported environment variables during execution

Be careful when your share singrun.sh, some environment variable could contain some secrets

```bash
user@local:~$ ./singrun.sh -d
export USER="ced"
export OMP_NUM_THREADS=4
...
export MGLS_LICENSE_FILE="/home/ced/license.lic:"
export JOB="dbus"
export SHELL="/bin/sh"
```

Its possible to override an exported environment variable by exporting variable with SENV_ prefix, example to override OMP_NUM_THREADS
```bash
user@local:~$ export SENV_OMP_NUM_THREADS=8
user@local:~$ ./singrun.sh
```
Will set OMP_NUM_THREADS to 8 during container execution

 * To launch a limited shell in container 
```bash
user@local:~$ ./singrun.sh -s
Run shell into container
Singularity: Invoking an interactive shell within container...

$ 
```

 * To execute command into container
```bash
user@local:~$ ./singrun.sh -e python -c "print 'Goodbye !'"
Execute command into container
Goodbye !
```
