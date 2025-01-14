# Environment Modules

The ABCI system offers various development environments, MPIs, libraries, utilities, etc. listed in [Software](system-overview.md#software). And, users can use these software in combination as *module*s.

[Environment Modules](http://modules.sourceforge.net/) allows users to configure their environment settings, flexibly and dynamically, required to use these *module*s.

## Usage

Users can configure their environment using the `module` command:

```
$ module [options] <sub-command> [sub-command options]
```

The following is a list of sub-commands.

| Sub-command | Description |
|:--|:--|
| list | List loaded modules |
| avail | List all available modules |
| show *module* | Display the configuration of "*module*" |
| load *module* | Load a module named "*module*" into the environment |
| unload *module* | Unload a module named "*module*" from the environment |
| switch *moduleA* *moduleB* | Switch loaded "*moduleA*" with "*moduleB*" |
| purge | Unload all loaded modules (Initialize) |
| help *module* | Print the usage of "*module*" |

## Use cases

### Loading modules

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1 cudnn/7.6/7.6.5
```

### List loaded modules

```
[username@g0001 ~]$ module list
Currently Loaded Modulefiles:
  1) cuda/10.0/10.0.130.1   2) cudnn/7.6/7.6.5
```

### Display the configuration of modules

```
[username@g0001 ~]$ module show cuda/10.0/10.0.130.1
-------------------------------------------------------------------
/apps/modules/modulefiles/centos7/gpgpu/cuda/10.0/10.0.130.1:

module-whatis	 cuda 10.0.130.1
conflict	 cuda
prepend-path	 CUDA_HOME /apps/cuda/10.0.130.1
prepend-path	 CUDA_PATH /apps/cuda/10.0.130.1
prepend-path	 PATH /apps/cuda/10.0.130.1/bin
prepend-path	 LD_LIBRARY_PATH /apps/cuda/10.0.130.1/extras/CUPTI/lib64
prepend-path	 LD_LIBRARY_PATH /apps/cuda/10.0.130.1/lib64
prepend-path	 CPATH /apps/cuda/10.0.130.1/extras/CUPTI/include
prepend-path	 CPATH /apps/cuda/10.0.130.1/include
prepend-path	 LIBRARY_PATH /apps/cuda/10.0.130.1/lib64
prepend-path	 MANPATH /apps/cuda/10.0.130.1/doc/man
-------------------------------------------------------------------
```

### Unload all loaded modules (Initialize)

```
[username@g0001 ~]$ module purge
[username@g0001 ~]$ module list
No Modulefiles Currently Loaded.
```

### Load dependent modules

```
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
WARNING: cudnn/7.6/7.6.5 cannot be loaded due to missing prereq.
HINT: at least one of the following modules must be loaded first: cuda/9.0 cuda/9.2 cuda/10.0 cuda/10.1 cuda/10.2
```

Due to dependencies, you will not be able to load `cudnn/7.6/7.6.5` without loading either `cuda/9.0`, `cuda/9.2`, `cuda/10.0`, `cuda/10.1`, or `cuda/10.2` module first.

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
```

### Load exclusive modules

Modules that are in an exclusive relationship, such as modules of different versions of the same library, cannot be used at the same time.

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cuda/10.2/10.2.89
cuda/10.2/10.2.89(7):ERROR:150: Module 'cuda/10.2/10.2.89' conflicts with the currently loaded module(s) 'cuda/10.0/10.0.130.1'
```

### Switch modules

```
[username@g0001 ~]$ module load cuda/10.2/10.2.89
[username@g0001 ~]$ module switch cuda/10.2/10.2.89 cuda/11.0/11.0.3
```

Switching may not be successful if there are dependencies.

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
[username@g0001 ~]$ module switch cuda/10.0/10.0.130.1 cuda/10.2/10.2.89
[username@g0001 ~]$ env | grep LD_LIBRARY_PATH
LD_LIBRARY_PATH=/apps/cudnn/7.6.5/cuda10.0/lib64:/apps/cuda/10.2.89/lib64:/apps/cuda/10.2.89/extras/CUPTI/lib64
```

CUDA10.2 and cuDNN for CUDA10.0 are loaded.

As shown below, it is necessary to unload modules that depend on the target module in advance and reload them after switching.

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
[username@g0001 ~]$ module unload cudnn/7.6/7.6.5
[username@g0001 ~]$ module switch cuda/10.0/10.0.130.1 cuda/10.2/10.2.89
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
[username@g0001 ~]$ env | grep LD_LIBRARY_PATH
LD_LIBRARY_PATH=/apps/cudnn/7.6.5/cuda10.2/lib64:/apps/cuda/10.2.89/lib64:/apps/cuda/10.2.89/extras/CUPTI/lib64
```

## Usage in a job script

When using the `module` command in a job script for a batch job, it is necessary to add initial settings as follows.

sh, bash:

```
source /etc/profile.d/modules.sh
module load cuda/10.0/10.0.130.1
```

csh, tcsh:

```
source /etc/profile.d/modules.csh
module load cuda/10.0/10.0.130.1
```
