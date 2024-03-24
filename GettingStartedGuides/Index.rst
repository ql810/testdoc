.. _getting_started_guide:

=====================
Getting Started Guide
=====================

This guide provides step-by-step instruction to setup host environment, get source code & build.

.. _setup_host_environment:

Set Up Host
===========

.. tabs::

   .. group-tab:: Ubuntu

      #. Requirements

         Ubuntu 22.04 LTS 64-bit.

      #. Update Host

         .. code-block:: console

            sudo apt-get update

      #. Install Git, Python, Pip, Conan, ruby, cmake, ninja & gcc

         .. code-block:: console

            cd ~
            sudo apt-get install git-all python3 python3-pip ruby cmake ninja-build
            wget https://github.com/conan-io/conan/releases/download/2.0.9/conan-ubuntu-64.deb
            sudo dpkg -i conan-ubuntu-64.deb
            sudo apt-get install build-essential gcc-12 g++-12
            sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12
            sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-12 12
            sudo update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-12 12

      #. Select gcc/g++/gcov version 12 if there are multiple versions installed in the system.

         .. code-block:: console

            sudo update-alternatives --config gcc
            sudo update-alternatives --config g++
            sudo update-alternatives --config gcov

      #. Confirm gcc/g++ version is 12

         .. code-block:: console

            gcc --version
            g++ --version
            gcov --version

      #. Install Sphinx & Doxygen

         .. code-block:: console

            sudo apt-get install python3-sphinx doxygen
            pip3 install sphinx-tabs
            pip3 install sphinx_rtd_theme
            pip3 install breathe

      #. Install gcovr

         .. code-block:: console

            pip3 install gcovr

      #. Add ``~/.local/bin`` to PATH

         Append ``export PATH="~/.local/bin:$PATH"`` to ``~/.bashrc``.

   .. group-tab:: Windows

      #. Requirements

         Windows 10 64-bit.

      #. Update Host

         Recommended to update Windows to the latest.

      #. Setup PowerShell

         .. important::

            Requires PowerShell v3+, .NET Framework 4+ and PowerShell execution policy not restricted.

         #. Run :program:`PowerShell` w/ "Run as administrator" to open a command prompt window.

         #. Confirm PowerShell mininum version.

            .. code-block:: console

               Get-Host | Select-Object Version

         #. Change PowerShell execution policy to ``RemoteSigned``.

            .. code-block:: console

               Set-ExecutionPolicy RemoteSigned

      #. Install Conan

         Download and install Conan Installer for Windows at `Conan v2.0.9 <https://github.com/conan-io/conan/releases/tag/2.0.9>`_.

      #. Install Python 3.9 or later at `https://www.python.org/downloads/ <https://www.python.org/downloads/>`_

      #. Install Sphinx & dependencies

         Open PowerShell with Admin privilege and run these commands:

         .. code-block:: console

            pip3 install sphinx
            pip3 install sphinx-tabs
            pip3 install sphinx_rtd_theme

      #. Install gcovr

         .. code-block:: console

            pip3 install gcovr

      #. Install Doxygen 1.9.7 or later

         Download and install Doxygen for Windows at `https://www.doxygen.nl/download.html <https://www.doxygen.nl/download.html>`_.

.. _install_vscode:

Install VSCode
==============

#. Download & install latest version at `https://code.visualstudio.com/ <https://code.visualstudio.com/>`_.

#. Install Extensions

   #. d-syko.syko-conan
   #. ms-vscode.cpptools
   #. marus25.cortex-debug
   #. ms-vscode-remote.remote-containers
   #. ms-vscode.vscode-serial-monitor
   #. xaver.clang-format
   #. ms-vscode.cpptools-extension-pack
   #. lextudio.restructuredtext
   #. streetsidesoftware.code-spell-checker

.. _cache_git_credential:

Conan Setup
===========

#. Open a shell terminal. For Windows, use PowerShell.

#. Install our default Conan configuration

   .. warning::
      Your Conan configuration will be overwritten. Create a backup of all your Conan configuration files (conan.conf, remotes.txt, settings.yml, profiles) if necessary.

   .. code-block:: console

      conan config install git@bitbucket.org:derivepower/conanconfiguration.git

#. Add your credential to our Conan Artifactory

   .. code-block:: console

      conan remote login -p <password> -r <remote_name> <user_name>

#. Check the setup

   #. Conan version 2.0.9 or later

      .. code-block:: console

         conan --version
   
   #. List available remote Artifactories

      .. code-block:: console

         conan remote list

      Example result:

      .. code-block:: console

         syko-dev: http://10.50.4.39:8081/artifactory/api/conan/syko-dev/ [Verify SSL: False, Enabled: False]
         conancenter: https://center.conan.io [Verify SSL: True, Enabled: True]

   #. List available profiles

      .. code-block:: console

         conan profile list

      Example result (might not have default):

      .. code-block:: console

         default
         linux.build
         linux.host
         m7-fpv5-d16.host
         win.build
         win.host

#. Create a default profile. A default profile is for convenience & not a requirement.

   .. tabs::

      .. group-tab:: Ubuntu

         .. code-block::

            cp ~/.conan2/profiles/<profile_filename> ~/.conan2/profiles/default

      .. group-tab:: Windows

         - Copy one of the available & suitable profiles and name it default and store it in Conan profiles folder.
         - Conan profiles folder is located in ``C:\\Users\\<username>\\.conan2\\``

#. Display profiles

   .. code-block:: console

      conan profile show -pr:b win.build -pr:h win.host

   Example result:

   .. code-block:: console

      Host profile:
      [settings]
      arch=x86_64
      build_type=Release
      compiler=gcc
      compiler.cppstd=gnu14
      compiler.libcxx=libstdc++11
      compiler.version=12
      os=Windows
      [tool_requires]
      *: mingw-w64/12.2.0, ninja/1.11.1, kconfiglib/14.1.0, ruby/3.2.2-1, doxygen/1.9.7
      [buildenv]
      SYKO_RUN_TESTS=1
      CTEST_OUTPUT_ON_FAILURE=1
      TOOLCHAIN_C_FLAGS=-coverage -ftest-coverage -fprofile-arcs
      TOOLCHAIN_LD_FLAGS=-coverage -lgcov

      Build profile:
      [settings]
      arch=x86_64
      build_type=Release
      compiler=gcc
      compiler.cppstd=gnu14
      compiler.libcxx=libstdc++11
      compiler.version=12
      os=Windows

.. _get_source_code:

Get Source Code
===============

.. parsed-literal::

   |get_source_cmd|

Build
=====

Method 0 - VSCode & Syko Conan Toolkit Extension
++++++++++++++++++++++++++++++++++++++++++++++++

   #. Open VSCode workspace

   #. Open syko-conan extension

   #. Select Conan build and host profiles

   #. Build

   #. Choose an executable to run/debug

   #. Run/debug the executable
      For debugging, **build_type** host profile needs to be **Debug**.

   .. image:: ../_images/vscode-syko-build-method.png

   .. note::
      Workspace already contains a Dev Container configuration file.
      When opening the workspace, it’ll prompt to Reopen in Container. If the prompt is disabled, open it in Remote Explorer.

      Working in a Dev Container is recommended but not required.

Method 1 - Build using Conan commands
+++++++++++++++++++++++++++++++++++++

   Use Conan to install & build the project per your configuration profile.

   .. code-block:: console

      <go to your build folder & make sure it is empty>
      conan install <path to project folder> -of <output_folder> -pr:b <build_profile> -pr:h <host_profile>
      conan build <path to project folder> -of <output_folder> -pr:b <build_profile> -pr:h <host_profile>

Method 2 - Build using CMake commands
+++++++++++++++++++++++++++++++++++++

   Use Conan install command to setup build environment for CMake to build it.

   Available build types:
      - Release
      - Debug

   .. tabs::

      .. group-tab:: Ubuntu

         .. code-block:: console

            conan install <source_folder> -of <output_folder> -pr:b <build_profile> -pr:h <host_profile>
            cd <output_folder>/build/<build_type>
            source generators/conanbuild.sh
            source generators/conanrun.sh
            cmake -G Ninja -S <source_folder> -DCMAKE_TOOLCHAIN_FILE=<output_folder>/build/<build_type>/generators/conan_toolchain.cmake -DCMAKE_POLICY_DEFAULT_CMP0091=NEW -DCMAKE_BUILD_TYPE=<build_type>
            cmake --build .
            cmake --build . --target test
            source generators/deactivate_conanrun.sh
            source generators/deactivate_conanbuild.sh

      .. group-tab:: Windows

         Open :program:`Command Prompt`

         .. code-block:: console

            conan install <source_folder> -of <output_folder> -pr:b <build_profile> -pr:h <host_profile>
            cd <output_folder>/build/<build_type>
            generators\conanbuild.bat
            generators\conanrun.bat
            cmake -G Ninja -S <source_folder> -DCMAKE_TOOLCHAIN_FILE=<output_folder>/build/<build_type>/generators/conan_toolchain.cmake -DCMAKE_POLICY_DEFAULT_CMP0091=NEW -DCMAKE_BUILD_TYPE=<build_type>
            cmake --build .
            cmake --build . --target test
            generators\deactivate_conanrun.bat
            generators\deactivate_conanbuild.bat

Syko Conan Toolkit Extension
============================

Select Profiles
+++++++++++++++

   #. Go to a desired profile
   #. Click **B** or **H** button on the profile to select it for build and/or host profile.

   .. image:: ../_images/vscode-syko-select-profiles.png

Modify Profiles
+++++++++++++++
  .. image:: ../_images/vscode-syko-modify-profiles.png

Build & Debug
+++++++++++++

  .. image:: ../_images/vscode-syko-build-method.png

  .. note::
      For debugging, **build_type** host profile needs to be **Debug**.
