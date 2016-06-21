# Build instructions

NOTE: These instructions are for the master branch (Chrome 47).

There are two types of builds you can perform. You can build
CEF Python using the prebuilt CEF binaries. Or you can build both
CEF Python and CEF from sources. Building CEF is a long process that
can take hours. In the tools/ directory there is the automate.py
script that automates building CEF. However before you can run it
you must satisfy some requirements.


Table of contents:
* [Requirements](#requirements)
* [Build CEF Python using prebuilt CEF binaries](#build-cefpython-using-prebuilt-cef-binaries)
* [Build both CEF Python and CEF from sources](#build-both-cefpython-and-cef-from-sources)
* [Build CEF manually](#build-cef-manually)
* [How to patch](#how-to-patch)
* [Ninja build slows down computer](#ninja-build-slows-down-computer)


## Requirements

Below are platform specific requirements. Do these first before
following instructions in the "All platforms" section that lists
requirements common for all platforms.

__Windows__

* For Python 2.7 - VS2008 compiler is required:
  http://www.microsoft.com/en-us/download/details.aspx?id=44266
* For Python 3.4 - VS2010 compiler is required:
  https://docs.python.org/3.4/using/windows.html#compiling-python-on-windows
* For Python 3.5 - VS2015 compiler is required:
  https://docs.python.org/3.5/using/windows.html#compiling-python-on-windows
* To build CEF from sources:
    * Use Win7 x64 or later. 32-bit OS'es are not supported. For more details
     see [here](https://www.chromium.org/developers/how-tos/build-instructions-windows).
    * For CEF branch >= 2704 install VS2015 Update 2 or later
    * For CEF branch < 2704 install VS2013 Update 4 or later
    * Install [CMake](https://cmake.org/) 2.8.12.1 or newer and add cmake.exe
        to PATH
    * Install [ninja](http://martine.github.io/ninja/) and add ninja.exe
        to PATH
    * You need about 16 GB of RAM during linking. If there is an error
        just add additional virtual memory.
    * For Python 2.7 copy "cefpython/src/windows/stdint.h" to
      "%LocalAppData%\Programs\Common\Microsoft\Visual C++ for Python\9.0\VC\include\"


__Linux__

* Install required packages using apt-get or similar:
```
sudo apt-get install cmake g++ pkg-config gtk+-2.0 libgtkglext1-dev
```
* Download [ninja](http://martine.github.io/ninja/) 1.7.1 or later
  and copy it to /usr/bin and chmod 755.
* When building CEF from sources you will need to install many more packages
  using the install-build-deps.sh script - instructions are provided
  further down on this page.
* Building on Ubuntu 12.04 is supported up to branch 2526 (Chrome 47).
  For branches 2623 (Chrome 49) or later Ubuntu 14.04+ is required.
* To build on Debian 7 see
  [cef/BuildingOnDebian7.md](https://bitbucket.org/chromiumembedded/cef/wiki/BuildingOnDebian7.md) and
  [cef/#1575](https://bitbucket.org/chromiumembedded/cef/issues/1575)
* To perform a 32-bit Linux build on a 64-bit Linux system see
  [cef/#1804](https://bitbucket.org/chromiumembedded/cef/issues/1804)


__All platforms__

* Install dependencies for the automate.py tool by executing:
  `cd tools/ && pip install -r requirements.txt`. This will install
  some PyPI packages including Cython. On Windows installing Cython
  requires a VS compiler - see instructions above for Windows.


## Build CEF Python using prebuilt CEF binaries

Run the automate.py tool using the --prebuilt-cef flag:
```
cd tools/
python automate.py --prebuilt-cef
```

You should be fine by running it with the default options, but if you
need to customize the build then use the --help flag to see more.


## Build both CEF Python and CEF from sources

Run the automate.py tool using the --build-cef flag. Specify cef branch
and optionally how many parallel ninja jobs to run (by default cores/2).

```
cd tools/
python automate.py --build-cef --cef-branch 2526 --ninja-jobs 6
```

__IMPORTANT__: On Linux it will fail on first run. After the chromium
sources are downloaded, it will try to build cef projects and fail
due to missing packages. You will need to run the install-build-deps.sh
script (intended for Ubuntu systems). When the "ttf-mscorefonts-installer"
graphical installer pops up you can deny EULA and not install these fonts
(might deteriorate UX on some systems, experienced on Ubuntu 12.04 32-bit).

```
cd build/chromium/src/build/
chmod 755 install-build-deps.sh
sudo ./install-build-deps.sh --no-arm --no-chromeos-fonts --no-nacl
```

After dependencies are satisifed re-run automate.py.

You should be fine by running it with the default options, but if you
need to customize the build then use the --help flag to see more.


## Build CEF manually

CEF Python official binaries come with custom CEF binaries with
a few patches applied for our use case. These patches are in the
patches/ directory.

On Linux before running any of CEF tools apply the issue73 patch
first.

To build CEF follow the instructions on the Branches and Building
CEF wiki page:
https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding

After it is successfully built - apply patches, rebuild and remake
distribs.

Note that CEF patches must be applied in the "download_dir/chromium/src/cef/"
directory, not in the "download_dir/cef/" directory.


## How to patch

Create a patch from unstaged changes in current directory:
```
cd chromium/src/cef/
git diff --no-prefix --relative > cef.gyp.patch
```

Apply a patch in current directory:
```
cd chromium/src/cef/
git apply cef.gyp.patch
```


## Ninja build slows down computer

If ninja slows down your computer too much (6 parallel jobs by default),
build manually with this command (where -j2 means to run 2 jobs in parallel):
```
cd chromium/src
ninja -v -j2 -Cout\Release cefclient
```