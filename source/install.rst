.. Copyright 2018 by Andrei Radulescu-Banu.

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at
 
     http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

*******
Install
*******

.. index::
   single: build
   single: install

.. contents:: Table of Contents

Install from source code
========================

The source code is available under Apache 2.0 license at https://github.com/bitdribble/bitdribble:

.. code-block:: none
		
   git clone git@github.com:bitdribble/bitdribble.git

All platforms require the ``expat``, ``libyaml``, ``jansson``, ``microhttpd``, ``openssl`` and ``libcurl`` development libraries installed.

.. To do: include libmicrohttpd and libmicrohttpd-devel once we need it.

Centos
------
Centos 7 has been tested. The default ``cmake`` on Centos 7 is version 2. We need cmake version 3, which we install and execute as ``cmake3``.

.. code-block:: none

   sudo yum install cmake3 expat-devel libyaml-devel jansson-devel libmicrohttpd-devel openssl-devel libcurl-devel

   cd .../bitdribble
   mkdir build && cd build && cmake3 ..
   make
   sudo make install

The *make install* command will install headers, libraries and binaries under */usr/local*. The installer can also be packaged as an ``rpm`` package.

.. code-block:: none

   cpack3 -G RPM

To install the ``rpm`` package, execute the command below. This will install headers, libraries and binaries under */usr*.

.. code-block:: none

   sudo rpm -ivh bitd-<version>-<platform>.rpm

After installing the package, enable and start the ``bitd`` service:

.. code-block:: none

   sudo systemctl enable bitd
   sudo systemctl start bitd

To uninstall the package:

.. code-block:: none

   sudo rpm -e bitd


Ubuntu
------
Ubuntu 18.04 has been tested. The default ``cmake`` on Ubuntu 18.04 has version higher than 3.1, and can be used directly.

.. code-block:: none

  sudo apt-get install libexpat-dev libyaml-dev libjansson-dev libmicrohttpd-dev libssl-dev libcurl4-openssl-dev

  cd .../bitdribble
  mkdir build && cd build && cmake ..
  make
  sudo make install

The *make install* command will install headers, libraries and binaries under */usr/local*. The installer can also be packaged as a ``deb`` package:

.. code-block:: none

   cpack -G DEB

To install the ``deb`` package, execute the command below. This will install headers, libraries and binaries under */usr*.

.. code-block:: none

   sudo dpkg -i bitd-<version>-<platform>.deb

After installing the package, enable and start the ``bitd`` service:

.. code-block:: none

   sudo systemctl enable bitd
   sudo systemctl start bitd

To uninstall the package:

.. code-block:: none

   sudo dpkg -r bitd

Raspbian
--------
Raspbian GNU/Linux 8 (jessie) and GNU/Linux 9.4 (stretch) have been tested. Raspberry Pi boards usually have a limited amount of flash. Before beginning installation, check the available flash size: ``df``. The system I tested had 14G available on the root file system, and the root file system was 33% full.

Start by upgrading all packages:

.. code-block:: none
   
   sudo apt-get update
   sudo apt-get upgrade

After upgrading all the packages, the root file system became 35% full. To compile the code, ``cmake`` needs to be installed as well, if not already installed.

.. code-block:: none

  sudo apt-get install build-essential cmake \
	libexpat-dev libyaml-dev libssl-dev libcurl4-openssl-dev

  cd .../bitdribble
  mkdir build && cd build && cmake ..
  make
  sudo make install

The *make install* command will install headers, libraries and binaries under */usr/local*. The installer can also be packaged as a ``deb`` package:

.. code-block:: none

   cpack -G DEB

To install the ``deb`` package, execute the command below. This will install headers, libraries and binaries under */usr*.

.. code-block:: none

   sudo apt-get install expat libyaml-0-2 openssl libcurl3
   sudo dpkg -i bitd-<version>-<platform>.deb

Note that on Raspbian Jessie and Stretch we need ``libcurl4-ssl-dev`` for the compilation, but ``libcurl3`` for installing the bitd Debian package. After installing the package, enable and start the ``bitd`` service:

.. code-block:: none

   sudo systemctl enable bitd
   sudo systemctl start bitd

To uninstall the package:

.. code-block:: none

   sudo dpkg -r bitd

OpenWRT
-------
Use `these instructions <https://wiki.openwrt.org/doc/howto/buildroot.exigence>`_ to install the OpenWRT SDK sources on Ubuntu. At the *make menuconfig* step, enable compilation of ``Libraries->libexpat``, ``Libraries->Languages->libyaml``, ``Libraries->SSL->libopenssl``, ``Libraries->libcurl``. These packages should either be included in the firmware image file, or should be installed with ``opkg`` after the firmware has been flashed to the device.

In this example, we build OpenWRT for ``Target System (x86)``, ``Subtarget (x86_64)``, and we enable ``Target Image->VMDK``. The resulting toolchain under ``openwrt/staging_dir`` is ``toolchain-x86_64_gcc-7.3.0_musl``, and the target is ``target-x86_64_musl``. We use these settings to create ``bitdribble/cmake/Toolchains/Toolchain-openwrt-x86_64_gcc_musl.cmake`` in the ``bitdribble`` source tree, then we build the ``bitdribble`` code:

.. code-block:: none

   cd .../bitdribble
   mkdir build-openwrt-x86 && cd build-openwrt-x86 
   cmake -DCMAKE_TOOLCHAIN_FILE=../cmake/Toolchains/Toolchain-openwrt-x86.cmake ..
   make

For different a OpenWRT target, create a corresponding toolchain file under ``bitdribble/cmake/Toolchains``, and pass it on the *cmake* command line using ``-DCMAKE_TOOLCHAIN_FILE``.

Mac OSX
-------
The default ``openssl`` and ``curl`` libraries installed by OSX are incompatible with ``bitdribble``. Instead, install these packages using ``brew``, along with other package dependencies that are needed:

.. code-block:: none

   brew install expat libyaml openssl curl

   cd .../bitdribble
   mkdir build && cd build && cmake ..
   make

Cygwin 
------
Older Cygwin only distributes ``cmake`` version 2. You need a version of Cygwin that distributes ``cmake`` version 3. We have tested Cygwin version 2.893 (64 bit) which has cmake version 3. 

Use the Cygwin Setup program to install these packages:

- Debug, Devel categories

- expat-devel, openssl-devel, libcurl-devel. 

In a Cygwin bash terminal, do the following:

.. code-block:: none

   cd .../bitdribble

   mkdir ../cygwin && cd ../cygwin && cmake ../cygwin
   make

The install step will install the packages under ``/usr/local/bin`` and ``/usr/local/include``, in the cygwin installation tree:

.. code-block:: none

   make install

The installer package can be set up as a ``.tar.bz2`` archive with the command *cpack -G CygwinBinary*, but modern Cygwin installers use ``cygport`` instead. We do not have ``cygport`` support at this time.

Windows
-------
We use the ``mingw`` cross compilers under ``Cygwin``. Install all the Cygwin ``mingw64-i686`` and ``mingw64-x86_64`` packages. As explained in the ``Cygwin`` section, you need a version of ``Cygwin`` that distributes ``cmake`` version 3. The instructions below assume a 64 bit Cygwin installation. For 64 bit Windows builds:

.. code-block:: none

   cd .../bitdribble
   mkdir ../x86_64-w64-mingw32 && cd ../cygwin

   cmake -DCMAKE_TOOLCHAIN_FILE=../bitdribble/cmake/Toolchains/Toolchain-x86_64-w64-mingw32.cmake ../bitdribble
   make

Or this for 32 bit Windows builds:

.. code-block:: none

   cd .../bitdribble
   mkdir ../i686-w64-mingw32 && cd ../cygwin

   cmake -DCMAKE_TOOLCHAIN_FILE=../bitdribble/cmake/Toolchains/Toolchain-i686-w64-mingw32.cmake ../bitdribble
   make


The install step will install the packages under ``/usr/local/bin`` and ``/usr/local/include``, in the cygwin installation tree. Note that the ``expat``, ``libyaml``, ``ssl`` and ``curl`` libraries are dependencies and need to be manually copied in the ``PATH``.

.. code-block:: none

   make install

Run the unit tests
==================
After compiling from sources, and before ``make install``, you can optionally run the unit tests:

.. code-block:: none

   make test

You can selectively run some of the tests by executing ``ctest`` instead of ``make test``, passing a substring of the test labels using the ``-R`` argument:

.. code-block:: none

   ctest -R bitd-agent

This command will run all tests with labels containing ``bitd-agent`` as substring. To run all tests except those matchin the substring ``long`` in the test label:

.. code-block:: none

   ctest -E long

These commands will work on all platforms except on Win32 mingw builds, where you must put ``/usr/x86_64-w64-mingw32/sys-root/mingw/bin`` in the ``PATH``:

.. code-block:: none

   export PATH=/usr/x86_64-w64-mingw32/sys-root/mingw/bin:$PATH
   make test
   ctest -R bitd-agent
   ctest -E long

Test labels contain ``bitd-agent`` when the program tested is the ``bitd-agent`` itself. Executing all ``bitd-agent`` tests will cover test modules as well as the ``bitd-agent`` itself. Blocking tests that take multiple seconds to run contain ``long`` in the label, and can be skipped if a quick sanity check test run is desired.

Install precompiled packages
============================
At this point, Bitdribble packages must be manually compiled. Precompiled Bitdribble packages are not available. When an ``rpm`` or ``deb`` package has been compiled, install it with the usual ``rpm`` and ``dpkg`` commands, then enable and start the ``bitd`` service.

.. code-block:: none

   sudo systemctl enable bitd
   sudo systemctl start bitd


Building the docs
=================

Install the ``sphinx`` software and its ``sphinx_rtd_theme``. Check out the ``bitdribble-doc`` git sandbox, cd to ``bitdribble-doc``, and ``make html``. Copy the ``build/html`` folder to a web server (or, if you have key-based ssh access to your web server, customize the ``install`` make rule so that ``make install`` copies the ``build/html`` folder to your web server).

