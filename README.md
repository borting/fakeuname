# fakeuname
A script that overrides the kernel release string output by `uname` command inside a container.

## Why fakeuname

When compiling libs/apps, which refer kernel header in their code, the configure script first invokes `uname -r` command to obtain kernel release string, and then uses that string to generate path to kernel header.
However, when compiling inside a container (like docker or singularity), the output of `uname -r` changes with the host kernel version, because `uname -r` gets kernel release string from `uname()` system call and containers share the same kernel with their host. 
This causes compiling these libs/apps gets failed due to kernel header not found.
Hence, we need to find a way to cheat the configure script.

## Usage

1. Set the `FAKE_KERNEL_RELEASE` macro in `uname` to the version of kernel header installed in the container.
For example,
```bash
$ FAKE_KERNEL_RELEASE="4.15.0-112-generic"
```
2. Create a folder in your home and copy `uname` to the folder.
It is suggested not to use a folder that is already in your command search path.
For example,
```bash
$ mkdir ~/.local/fakeuname
$ cp uname ~/.local/fakeuname/uname
```
3. Include the fakeuname folder in your PATH env when running a container.
Here is an example fot bash,
```bash
# vim ~/.bashrc
if test -f "/.dockerenv" || [[ ! -z "${SINGULARITY_NAME+x}" ]]; then
    export PATH=${HOME}/.local/fakeuname:${PATH}
fi
```
4. Mount `.bashrc` and `fakeuname` folder when executing container.
* For Docker 
```bash
$ docker run -v ${HOME}/.bashrc:/root/.bashrc:ro -v ${HOME}/.local/fakeuname/uname:/root/.local/fakeuname/uname --rm -it DOCKER_IMAGE
```
* For Singularity
```bash
$ SINGULARITY_SHELL=/bin/bash singularity shell SINGULARITY_IMAGE
```
