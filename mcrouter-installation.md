## Automatic installation for Ubuntu 14.04

To simplify dependency installation, we provided an [auto-install script](https://github.com/facebook/mcrouter/blob/master/mcrouter/scripts/install_ubuntu_14.04.sh) that's been tested on Ubuntu 14.04.

The script might also be useful for other systems - make sure to read through its source files. It contains a bunch of ["recipe" files](https://github.com/facebook/mcrouter/tree/master/mcrouter/scripts/recipes) to download and install each mcrouter dependency, including workaround for common pain points.

After you've read through the script and made sure you're fine with the commands it will run, invoke with absolute dir for self-contained install and any arguments to `make`:
```
$ git clone git@github.com:facebook/mcrouter.git
$ ./mcrouter/mcrouter/scripts/install_ubuntu_14.04.sh /home/$USER/mcrouter-install/ -j4
[sudo] password for ...:
...
$ ~/mcrouter-install/install/bin/mcrouter --help
```

## Docker

### Official Docker

We provide this [Dockerfile](https://github.com/facebook/mcrouter/blob/master/mcrouter/scripts/docker/Dockerfile) for you to build a docker image (base on ubuntu 14:04).

### Third party Dockers

* https://registry.hub.docker.com/u/jamescarr/mcrouter/
* https://registry.hub.docker.com/u/jamescarr/mcrouter-rest-api/
 
## Manual installation

### Dependencies

* GCC 4.9+. Note that Ubuntu 14 ships with GCC 4.8, so it is necessary to install and configure the build system to use GCC 4.9 (e.g., by exporting the environment variables `CC=gcc-4.9` and `CXX=g++-4.9`).  Our auto-install script for Ubuntu 14 takes care of this.
* [Boost](http://www.boost.org/) 1.51+ (boost::filesystem, boost::system, boost::regex, boost::context)
* [Ragel](http://www.complang.org/ragel/)

Here's a list of required packages for Ubuntu from the [auto-install script](https://github.com/facebook/mcrouter/blob/master/mcrouter/scripts/install_ubuntu_14.04.sh):
```Shell
sudo apt-get install -y gcc-4.9 g++-4.9 libboost1.54-dev libboost-thread1.54-dev \
    libboost-filesystem1.54-dev libboost-system1.54-dev libboost-regex1.54-dev \
    libboost-python1.54-dev libboost-context1.54-dev ragel autoconf unzip \
    git libtool python-dev cmake libssl-dev libcap-dev libevent-dev \
    libgtest-dev libsnappy-dev scons binutils-dev make \
    wget libdouble-conversion-dev libgflags-dev libgoogle-glog-dev
```

 * [folly](https://github.com/facebook/folly). Follow instructions from folly README. Also take a look at [auto install folly recipe](https://github.com/facebook/mcrouter/blob/master/mcrouter/scripts/recipes/folly.sh).

### Build mcrouter

In mcrouter folder run

```Shell
 autoreconf --install
 ./configure
 make
```

###Run unit tests (optional)

Run

```Shell
 make check
```

If you have "symbol not found" errors from gtest, build it and put
libgtest.a/libgtestmain.a into you LD_LIBRARY_PATH. To build gtest:

* load it from http://googletest.googlecode.com/files/gtest-1.6.0.zip
* in make subfolder run
```
 make
```
* Rename gtest.a to libgtest.a; gtest_main.a to libgtestmain.a
* Don't forget to add libgtest.a and libgtestmain.a to your LD_LIBRARY_PATH