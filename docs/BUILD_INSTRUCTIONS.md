# Build instructions

## Compiled on / General Requirements:

- 64bit system, it won't work on 32bit.
- [git 2.21.0.windows.1](https://git-scm.com/downloads) (version not important unless the interface changes)
- [cmake 3.12.2-win64-x64](https://github.com/Kitware/CMake/releases/tag/v3.12.2)
- [MITK v2018.04.2](https://github.com/MITK/MITK), more instructions on this below
- [Qt 5.11.1](https://www.qt.io/): Download and install using open-source license. This version is slightly old even now, so look for it in the archive when installing. We need the following components: "Desktop gcc (or msvc2017) 64-bit", "Qt WebEngine", and "Qt Script". On Windows install in short path, like ```C:\Qt```. On linux you can install it wherever you want, but ```/home/YOUR_USERNAME/Work/Qt``` is a good place to be reduce clutter in accordance to what happens below.

## Windows build instructions

Note: Use short paths for everything. This is a limitation of Windows. To be sure, it's better to use the exact paths listed below.

#### Windows-specific requirements

- [Visual Studio 2017](https://visualstudio.microsoft.com/vs/older-downloads/) (Community - aka free - edition is fine). This needs the 2017 c++ toolset. I wasn't able to make it work with the VS2019 generator. I think it is not supported by MITK. After/During install make sure the "Desktop development with C++" workload is installed. The default enabled checkboxes should be fine but make sure that "VC++ 2017 ... tools" and "Visual C++ tools for CMake" are installed.  
- [NSIS 2.51](https://sourceforge.net/projects/nsis/files/NSIS%202/2.51/). Make sure it's available in path, you can do this by installing nsis-2.51-setup.exe. Don't use version 3+.

#### Clone MITK and Interactive Segmentation repositories

```cmd
cd C:\
git clone https://github.com/MITK/MITK
git clone https://github.com/CBICA/InteractiveSegmentation
```

#### Checkout the v2018.04.2 tag of MITK

```cmd
cd C:\MITK
git fetch --all --tags
git checkout v2018.04.2 -b v2018.04.2-branch
```

#### Configure and generate the superbuild

Important note: Even though we are on Windows, use '/' as a path separator and not '\\' in cmake!

- Open cmake-gui.
- Put ```C:/MITK``` at "Where is the source code" (notice the forward slashes in the path)
- Put ```C:/MITK-sb``` at "Where to build the binaries" (say yes if it asks to create the folder) (notice the forward slashes in the path)
- Click configure.
- If it comes up with an "error" that it can't find Qt, find the Qt5_DIR variable and set it to "C:/Qt/5.11.1/msvc2017_64/lib/cmake/Qt5" (notice the forward slashes)
- Click configure
- Set "CMAKE_INSTALL_PREFIX" to "C:/MITK-sb/install"
- Set "MITK_USE_OpenCV" to true (check the box).
- Set "MITK_EXTENSION_DIRS" to "C:/InteractiveSegmentation" (notice the forward slashes)
- Configure (and then configure again until there are no more _red_ variables)
- Click Generate
- Click "Open Project" - that should open Visual Studio

#### Build the superbuild

- In Visual Studio there will be a combobox that says "Solution Configurations" when you hover over it and will probably say "Debug" by default. Set it to "Release". \* SEE COMMENT ON THIS BELOW
- In the "Solution Explorer" on the right, right-click "ALL_BUILD" and click "Build".

This will take 1+ hour(s) depending on the system.

\* Release means that the program will run fast, just the way we distribute it. You can choose to build the superbuild+build in "Debug" too. That way you can use a debugger to figure out issues in the code, but note that it will - for obvious reasons - run slower.

#### Configure and generate the build

Note: To distinguish between the two, the superbuild resides in "C:/MITK-sb" and the build in "C:/MITK-sb/MITK-build". It is obvious that the superbuild kind of contains the build even now without doing anything else. MITK Workbench - the GUI - will run even without doing anything in the build. But we need to enable two more plugins that work with our own.

Note: Close the "old" project in Visual Studio to not make things confusing.

- Open CMake again
- Put the "build" path in "Where to build the binaries". The path is "C:/MITK-sb/MITK-build".
- Locate "MITK_BUILD_org.mitk.gui.qt.segmentation" and set it to true
- Locate "MITK_BUILD_org.mitk.gui.qt.multilabelsegmentation" and set it to true
- They would be enabled, but for reference: our plugin is enabled/disabled by "MITK_BUILD_upenn.cbica.captk.interactivesegmentation" and MITK Workbench - the GUI - by "MITK_BUILD_APP_Workbench".
- Configure
- Generate
- Open Project

#### Build the... build

In Visual Studio, on your new "build" project and NOT the superbuild one:

- There will be a combobox that says "Solution Configurations" when you hover over it and will probably say "Debug" by default. Set it to "Release". \* SEE COMMENT ON THIS BELOW
- In the "Solution Explorer" on the right, right-click "MitkWorkbench" and click "Build".

This will take 1+ hour(s) depending on the system.

\* Release means that the program will run fast, just the way we distribute it. You can choose to build the superbuild+build in "Debug" too. That way you can use a debugger to figure out issues in the code, but note that it will - for obvious reasons - run slower.

#### Run 

You can run the program by running the bat files in ```C:\MITK-sb\MITK-build\bin```.

- ```startMitkWorkbench_release.bat``` starts the program.
- ```startMitkWorkbench_debug.bat``` starts the program in debug mode, if that was build.

#### Package

Packaging means making a zip file of the program + an installer. To do all this in the "build" project in visual studio, and only when in "Release" mode, find the "PACKAGE" project in "Solution Explorer". Right-click it and click build. That will create:

- ```C:\MITK-sb\MITK-build\_CPack_Packages\win64\ZIP\MITK-v2018.04.2-windows-x86_64.zip``` the zip package

This allows the program to run without installing. Just unzip and run bin\MitkWorkbench.exe. For some very old systems, running thirdpartyinstallers\vc_redist.x64.exe might be needed first.

- ```C:\MITK-sb\MITK-build\_CPack_Packages\win64\NSIS\MITK-v2018.04.2-windows-x86_64.exe``` the installer

This allows the program to be installed like a regular windows program. After that it will be available as MITK Workbench.

## Linux (Ubuntu) build instructions

#### Linux (Ubuntu)-specific requirements

Note: This list was compiled in Ubuntu 18.04 and it might be slightly different if you are from the future.

- Run: ```sudo apt install -y build-essential mesa-common-dev libglu1-mesa-dev libtiff5-dev libwrap0-dev libxt-dev libxi-dev g++ git gitk git-gui```

#### Clone MITK and Interactive Segmentation repositories

You are free to choose whichever paths you want, but for the instruction purposes everything will happen in ```$HOME/Work``` (```$HOME``` aka ```~``` is your home directory, i.e. /home/YOUR_USERNAME)

```cmd
mkdir -p ~/Work
cd ~/Work
git clone https://github.com/MITK/MITK
git clone https://github.com/CBICA/InteractiveSegmentation
```

#### Checkout the v2018.04.2 tag of MITK

```cmd
cd ~/Work/MITK
git fetch --all --tags
git checkout v2018.04.2 -b v2018.04.2-branch
```

#### Configure and generate the superbuild

These instructions are for a "Release" superbuild. Replace all instances of "Release" with "Debug" for a "Debug" superbuild. Release means that the program will run fast, just the way we distribute it. You can choose to build the superbuild+build in "Debug" too. That way you can use a debugger to figure out issues in the code, but note that it will - for obvious reasons - run slower.

- Open cmake-gui.
- Put ```/home/YOUR_USERNAME/Work/MITK``` at "Where is the source code"
- Put ```/home/YOUR_USERNAME/Work/MITK-sb/Release``` at "Where to build the binaries" (say yes if it asks to create the directory)
- Click configure.
- If it comes up with an "error" that it can't find Qt: If you installed Qt in ```/home/YOUR_USERNAME/Work/Qt``` then find the Qt5_DIR variable and set it to ```/home/YOUR_USERNAME/Work/Qt/5.11.1/gcc_64/lib/cmake/Qt5``` (obviously replace the first part if you installed somewhere else)
- Click configure
- Set "CMAKE_INSTALL_PREFIX" to "/home/YOUR_USERNAME/Work/MITK-sb/Release/install"
- Set "MITK_USE_OpenCV" to true (check the box).
- Set "MITK_EXTENSION_DIRS" to "/home/YOUR_USERNAME/Work/InteractiveSegmentation"
- Set CMAKE_BUILD_TYPE to "Release"
- Set CMAKE_EXPORT_COMPILE_COMMANDS to true
- Configure (and then configure again until there are no more _red_ variables)
- Click Generate

#### Build the superbuild

```terminal
cd ~/Work/MITK-sb/Release
make -j4
```

(If you do a debug build, navigate to ```~/Work/MITK-sb/Debug```)

Note: You can replace ```make -j4``` with ```make -j8``` if your system is decent and it has 16+GB of RAM. If you get a compiler error related to RAM, just run ```make``` instead.

This will take 1+ hour(s) depending on the system.

#### Configure and generate the build

Note: To distinguish between the two, the superbuild resides in "/home/YOUR_USERNAME/Work/MITK-sb/Release" and the build in "/home/YOUR_USERNAME/Work/MITK-sb/Release/MITK-build". It is obvious that the superbuild kind of contains the build even now without doing anything else. MITK Workbench - the GUI - will run even without doing anything in the build. But we need to enable two more plugins that work with our own.

Note: Change all instances of "Release" to "Debug" for a "Debug" build.

- Open CMake again
- Put the "build" path in "Where to build the binaries". The path is "/home/YOUR_USERNAME/Work/MITK-sb/Release/MITK-build".
- Locate "MITK_BUILD_org.mitk.gui.qt.segmentation" and set it to true
- Locate "MITK_BUILD_org.mitk.gui.qt.multilabelsegmentation" and set it to true
- They would be enabled, but for reference: our plugin is enabled/disabled by "MITK_BUILD_upenn.cbica.captk.interactivesegmentation" and MITK Workbench - the GUI - by "MITK_BUILD_APP_Workbench".
- Configure
- Generate

#### Build the... build

```terminal
cd ~/Work/MITK-sb/Release/MITK-build
make -j4
```

(If you do a debug build, navigate to ```~/Work/MITK-sb/Debug/MITK-build```)

Note: You can replace ```make -j4``` with ```make -j8``` if your system is decent and it has 16+GB of RAM. If you get a compiler error related to RAM, just run ```make``` instead.

#### Run 

You can run the program by running ```~/Work/MITK-sb/Release/MITK-build/bin/MitkWorkbench```.

(If you build debug, run ```~/Work/MITK-sb/Debug/MITK-build/bin/MitkWorkbench``` with your debugger)

#### Package

Release only!

```terminal
cd ~/Work/MITK-sb/Release/MITK-build
make package
```
