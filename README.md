# CEGUI Windows and Mac OS X dependencies package for 'default' branch (CEGUI 1.0)

This package contains source code and a custom cmake build environment for the
majority of the optional dependency libraries that can be utilised by various
parts of the 'Crazy Eddie's GUI System' (CEGUI) project.

## Available branches and versions

- The CEGUI-dependencies repository consists of multiple branches:
    - `v0-8` is meant to be used with CEGUI releases 0.x, as well as with CEGUI development branches `v0-8` and `v0`. As this branch is ABI compatible, it is possible to replace CEGUI-dependencies dynamic libraries from a specific revision in this branch with another revision in this branch, without having to rebuild the project.

    - `default` is meant to be used with the CEGUI development branch `default`. This branch may break ABI and API compatibility between any 2 revisions in this branch, or between any revision in this branch and a revision in the `v0-8` branch. That means if you switch between revisions you'll have to rebuild your project. Normally you'll only want to use this branch if you're using CEGUI branch `default`.

---

Note that none of the included source packages are in totally pristine and
unmodified form - for example, each package will contain a CEGUI-BUILD
directory that contains - at least - the custom CMakeLists.txt file.

Certain source files are also modified:

* DevIL (IL lib) files were patched to compile with the included libpng 1.4.7.
* Freetype was modified to be able to build as a DLL.
* jpeg library was modified to be able to build as a DLL.
* libtiff was modified to be able to build as a DLL.
* Some packages contain additional config header files which we copy into place
  for OS X, this is done to specify pre-configured settings and replaces an
  invocation of some ./configure script (or similar).

There may be other modifications not explicitly mentioned here, most are
identified in the original files by way of a comment containing "CEGUI"
somewhere, though it is possible - or even likely - that some modifications
are missing such identification, and this notification should suffice to
say that these libs are *not* the original code - to guarantee you're using
the original, umodified, code you should download your own copies of the
original source and perform your own build.

# Building for CEGUI 0.7.9 with Visual Studio 2015

Although the default branch is aimed at CEGUI 1.0, you can still use this
repository to produce dependency binaries that are compatible with the legacy
CEGUI 0.7.9 release on Windows with Visual Studio 2015. The process is entirely
source-based and does not require pre-built archives.

1. **Checkout the correct branch** – clone this repository and switch to the
   `v0-8` branch, which preserves the ABI used by the 0.x series of CEGUI:

   ```bash
   git clone https://github.com/cegui/cegui-dependencies.git
   cd cegui-dependencies
   git checkout v0-8
   ```

2. **Open the Visual Studio 2015 developer prompt** – launch the "VS2015
   x86/x64 Native Tools Command Prompt" that matches the architecture you want
   to build for.

3. **Configure with CMake** – create an out-of-source build directory and run
   CMake, pointing it to the checked-out sources. Visual Studio 2015 is
   automatically detected when you use the generator `Visual Studio 14 2015`
   (add `Win64` if you need 64-bit binaries):

   ```bat
   mkdir build-vs2015
   cd build-vs2015
   cmake -G "Visual Studio 14 2015" ..
   :: for 64-bit builds use:
   :: cmake -G "Visual Studio 14 2015 Win64" ..
   ```

   You can pass additional options (for example `-DCEGUI_USE_STATIC_STD_LIBS=ON`)
   to match the runtime model you require.

4. **Build the dependencies** – open the generated `cegui-dependencies.sln` in
   Visual Studio 2015 and compile the desired configuration (Debug/Release).
   You can also drive the build from the command line:

   ```bat
   cmake --build . --config Release
   ```

5. **Use the produced binaries** – after a successful build, libraries and DLLs
   are staged under `dependencies/lib` and `dependencies/bin`. Copy these
   directories into your CEGUI 0.7.9 project or adjust your linker and runtime
   paths accordingly.

Because GitHub-hosted runners no longer offer Visual Studio 2015 images, the
project cannot publish ready-made VS2015 dependency packages via GitHub Actions
alone. If you need continuous delivery, consider preparing a self-hosted runner
with VS2015 installed and reuse the steps above in a workflow.

# Cross-compiling

CEGUI dependencies can be cross-compiled from Linux to Windows. In order to do that, you need to configure a cross-compile file and invoke that using CMake. An example of such file is the following:

```CMake
# the name of the target operating system
SET(CMAKE_SYSTEM_NAME Windows)

# which compilers to use for C and C++
SET(CMAKE_C_COMPILER i686-w64-mingw32-gcc)
SET(CMAKE_CXX_COMPILER i686-w64-mingw32-g++)
SET(CMAKE_RC_COMPILER i686-w64-mingw32-windres)

# here is the target environment located
SET(CMAKE_FIND_ROOT_PATH /usr/i686-w64-mingw32 ${CMAKE_SOURCE_DIR}/install)

# adjust the default behaviour of the FIND_XXX() commands:
# search headers and libraries in the target environment, search
# programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

To invoke it you need to execute:
```bash
cmake -DCMAKE_TOOLCHAIN_FILE=cross-compile.cmake ..
```

For more details on cross-compiling with CMake visit http://www.cmake.org/Wiki/CMake_Cross_Compiling .