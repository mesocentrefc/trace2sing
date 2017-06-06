# trace2sing
It traces programs execution and create [Singularity](http://singularity.lbl.gov) containers for reproducible execution.

## How it works
It can trace any program to generate a **minimal** Singularity container embedded in a resulting binary named singrun.sh. 
trace2sing doesn't require root privileges nor Singularity installation for container generation. On the other hand, the 
system executing singrun.sh will require Singularity installation.

## Demo
[![asciicast](https://asciinema.org/a/90fve3i9t0ossj06a8tptajzw.png)](https://asciinema.org/a/90fve3i9t0ossj06a8tptajzw)

## Requirements
trace2sing need the following packages:
 * strace (version >= 4.7)
 * perl (installed by default on many distributions) 
 * coreutils (installed by default on many distributions)
 * binutils (installed by default on many distributions)

It was tested with following distributions (should work on all architecture supported by strace):
 * CentOS 6.7, CentOS 7 (yum install strace)
 * Ubuntu 14.04, 16.04, 17.04 (apt-get install strace)
 * Alpine Linux 3.2 and edge (apk add strace perl)

## Limitations
 * Same as Singularity
 * Generated containers need Singularity >= 2.2 with a linux kernel >= 3.5 for execution
 * Generated containers are compatible with systems based on same CPU architecture
 * Don't use input files or binaries located in /tmp directory, they will be overriden by /tmp system partition
   where they are executed

## How to use it

### trace2sing

To trace a program and package it into an executable Singularity's container:

```bash
user@local:~$ trace2sing python -c 'print "Hello !"'
Hello !
Generating self-extracting singrun.sh archive ... please wait
```
The above command will generate a singrun.sh executable.

### singrun.sh
 * **To extract container file tree into singrun_rootfs directory:**

```bash
user@local:~$ ./singrun.sh -x
Extracting file tree into singrun_rootfs directory
```

 * **To list container files:**

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

 * **To display exported environment variables during execution:**

```bash
user@local:~$ ./singrun.sh -d
export USER="ced"
export OMP_NUM_THREADS=4
...
export MGLS_LICENSE_FILE="/home/ced/license.lic:"
export JOB="dbus"
export SHELL="/bin/sh"
```

It's possible to override an exported environment variable by prefixing the variable with SENV_, example to override OMP_NUM_THREADS
```bash
user@local:~$ export SENV_OMP_NUM_THREADS=8
user@local:~$ ./singrun.sh
```
Will set OMP_NUM_THREADS to 8 during container execution

 * **To start a limited shell in container :**

```bash
user@local:~$ ./singrun.sh -s
Run shell into container
Singularity: Invoking an interactive shell within container...

$ 
```

 * **To execute command into container:**

```bash
user@local:~$ ./singrun.sh -e python -c 'print "Goodbye !"'
Execute command into container
Goodbye !

---
**NOTE**

* Your home directory is bound to /realhome in container, so if you want to use a file located in your home directory,
you must replace $HOME by /realhome in the path

* Be careful when your share singrun.sh, some environment variables and/or files could contain some secrets

---
