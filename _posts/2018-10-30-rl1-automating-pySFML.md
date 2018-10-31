---
title: "RL 1: Automating pySFML"
categories: 
  - Python
  - Relearning Python
tags:
  - best practice
  - build
---

[SFML](https://www.sfml-dev.org/index.php) is a great multi-media library that is written in C++. [pySFML](https://github.com/Sonkun/python-sfml) is
a Cython binding for SFML and it essentially allows Python code to utilize SFML. At the current time, pySFML has kind of been abandoned
and there isn't much documentation on how to properly build the project. PySFML is also a dependency of my oldish Python game framework -- built in 2012.

# Goal

It would be great if we could have Linux, OSX and Windows Python Wheels available to download. This pySFML project is particularly tricky because it has C++
dependencies. So whatever depends on pySFML will also be dependent on the C++ SFML library and also dependent on SFML's dependencies.

# Tools

## TravisCI

TravisCI is a continuous integration tool that can integrate with Github in an event-driven manner. You may configure Github to notify TravisCI each time someone pushes to some Github repository.
The notification will then trigger TravisCI to read the .travis.yml file in your project's root directory. From there TravisCI will spawn a virtual machine and execute an automated build.

## Cython

PySFML is written using .cpp/.c, .hpp/.h and .pyx files. These are all compiled using Cython and some C compiler (based on your OS). The compilation eventually results with a 
.dll or .so file that can be imported and used by a regular Python script -- this can be run with the default Python interpreter I believe.

This entire process is extremely prone to error though. That is because SFML, pySFML and Python all need to be compiled by the same C compiler in order to work correctly at runtime. In addition to that, it may be difficult to gain support for 
multiple distributions due to the difference in packages and other variances. 

Tip: [You can determine what C compiler version your Python interpreter was built with using sys.version in a script!](https://stackoverflow.com/questions/15350780/how-to-check-which-compiler-was-used-to-build-python)

## SFML

SFML is the entire reason for pySFML. SFML is the multi-media library that pySFML exposes to Python scripts. At the time this article was written, 
pySFML was dependent on SFML-2.3.2 specifically. So that'll be the version that should be used during compilation and runtime of pySFML.

# Experiment 1: Regular Linux Build

My first step was to simply make [TravisCI](https://travis-ci.org/) download [pySFML](https://github.com/jawaff/python-sfml) from my Github account and build pySFML in a Linux environment.
This entire experiment was put into a pull request and can be viewed [here](https://github.com/jawaff/python-sfml/pull/1). The following subsections will
show the steps of the experiment.

## Build Matrix

The first problem that I wanted to solve was that we needed a way to run multiple Linux builds for different
Python versions. For that, TravisCI provides build matrices to allow multiple vms to be deployed and also with different configurations.

.travis.yml:
```yaml
matrix:
  include:
  - os: linux
    language: python
    python: "2.7"
  - os: linux
    language: python
    python: "3.7-dev"
```

## Download SFML

During TravisCI's before_install phase, we need to download SFML-2.3.2 from their [download page](https://www.sfml-dev.org/download/sfml/2.3.2/) using 'wget'.
It's important that the SFML that we download should be compiled against the same C compiler as Cython and the intended Python interpreter version.
Thankfully, since we're in the Linux world we can just utilize the gcc C compiler.
After successfully downloading the correct SFML binaries, 'tar' can be used to extract SFML into the extlibs/ subdirectory in the pySFML project.
Then the SFML-2.3.2/lib/ and SFML-2.3.2/include/ directories are added to gcc's environment variables. When Cython goes to utilize gcc, gcc will now automatically
be linked with the downloaded SFML library.

scripts/sfml_install.sh:
```bash
#!/usr/bin/env bash

# Retrieves the directory of this script.
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"
CUR_DIR="$SCRIPT_DIR/.."
SFML_VERSION="2.3.2"
EXT_LIBS_DIR="$CUR_DIR/extlibs"
SFML_DIR="$EXT_LIBS_DIR/SFML-$SFML_VERSION"

echo "Downloading SFML-$SFML_VERSION:"

wget https://www.sfml-dev.org/files/SFML-$SFML_VERSION-linux-gcc-64-bit.tar.gz

echo "Extracting SFML-$SFML_VERSION into $EXT_LIBS_DIR:"

# "SFML-$SFML_VERSION" is the directory that is placed into extLibs/.
# The SFML dir contains an include/, lib/ and share/ directory.
tar -xvzf SFML-$SFML_VERSION-linux-gcc-64-bit.tar.gz -C $EXT_LIBS_DIR

ls $EXT_LIBS_DIR
ls $SFML_DIR

echo "Setting up environment variables for SFML:"

# Compile time environment variables for gcc.
export CPPFLAGS="-I$SFML_DIR/include"
export LIBRARY_PATH="$SFML_DIR/lib"

echo "CPPFLAGS=$CPPFLAGS"
echo "LIBRARY_PATH=$LIBRARY_PATH"
```

## Run the Build

After getting SFML downloaded and linked up with gcc, there's not much else to do. Cython needs to be installed by pip.
Then 'python setup.py sdist bdist_wheel' will create the source and binary wheel distributions of the pySFML project. 
Nothing else needs to be done for building the project, but there's still the deployment to pypi that will be discussed later.

# Experiment 2: MultiOS Build

My next experiment was to expand upon the first and figure out how to make also make Windows build. At the time this article was written,
TravisCI has very experimental support for Windows, because it was added within a month ago. (I don't believe that my MultiOS build works
anymore with the current TravisCI Windows environments, because choco is probably corrupted.) The pull request containing the work for this 
experiment can be found [here](https://github.com/jawaff/python-sfml/pull/2). The following steps can be found below.

## Build Matrices Revisited

Build matrices were utilized in the first experiment to allow multiple Python builds. It can also allow for multiOS builds.
Each configuration in the build matrix will deploy a separate vm.

```yaml
matrix:
  include:
  - os: linux
    language: python
    python: "2.7"
  - os: linux
    language: python
    python: "3.7-dev"
  - os: windows
    # Travis' Windows Env doesn't support a Python language.
    language: shell
```

## Download SFML

In the Windows environment we're still going to be needing to download SFML, but it will be an SFML that was built using Visual Studios 14 (2015).
Just like in the last experiment, the SFML-2.3.2 binaries can be downloaded [here](https://www.sfml-dev.org/download/sfml/2.3.2/).
There isn't a convenient SFML-2.3.2 build for Visual Studios 15 (2017), so we are going to in turn lose support for Python 3.7 -- which was built using Visual Studios 15.
That means that we're going to be restricted to Python 3.6.6 with the Windows build of pySFML, but that's not too bad and is a good reason to update
the pySFML binding to SFML-2.5.X.

The scripts/sfml_install.sh file didn't have to change too much. It receives the current OS and SFML version and downloads/unzips the appropriate
binaries. However, since we'll be using Visual Studios 14 instead of gcc, we have to link the SFML binaries a different way. That will be discussed 
in a later step.

scripts/sfml_install.sh:
```bash
#!/usr/bin/env bash

# Argumets
CUR_OS="$1"
SFML_VERSION="$2"

# Retrieves the directory of this script.
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"

# Install Constants
CUR_DIR="$SCRIPT_DIR/.."
EXT_LIBS_DIR="$CUR_DIR/extlibs"
SFML_DIR="$EXT_LIBS_DIR/SFML-$SFML_VERSION"
SFML_DOWNLOAD_PAGE="https://www.sfml-dev.org/files/"

echo "Current OS: $CUR_OS"
if [ "$CUR_OS" = "linux" ]
then
  TAR_FILENAME=SFML-$SFML_VERSION-linux-gcc-64-bit.tar.gz
elif [ "$CUR_OS" = "windows" ]
then
  TAR_FILENAME=SFML-$SFML_VERSION-windows-vc14-64-bit.zip
elif [ "$CUR_OS" = "osx" ]
then
  TAR_FILENAME=SFML-$SFML_VERSION-osx-clang-universal.tar.gz
else
  echo ERROR: Unsupported OS Detected: $CUR_OS
  exit 1
fi

echo "Downloading: $TAR_FILENAME"
wget "$SFML_DOWNLOAD_PAGE$TAR_FILENAME"

echo "Extracting SFML-$SFML_VERSION into $EXT_LIBS_DIR:"
if [ "$CUR_OS" = "windows" ]
then
  unzip $TAR_FILENAME -d $EXT_LIBS_DIR
else
  tar -xvzf $TAR_FILENAME -C $EXT_LIBS_DIR
fi

echo "Setting up environment variables for SFML:"
if [ "$CUR_OS" = "linux" ]
then
  # Compile time environment variables for gcc.
  export CPPFLAGS="-I$SFML_DIR/include"
  export LIBRARY_PATH="$SFML_DIR/lib"
  echo "CPPFLAGS=$CPPFLAGS"
  echo "LIBRARY_PATH=$LIBRARY_PATH"
elif [ "$CUR_OS" = "osx" ]
then
  echo "TODO"
fi
```

## Setup Windows Environment

Since we decided to utilize Python 3.6.6 and Visual Studios 14 (2015), they have to be installed in the Windows environment.
This is all due to the fact that TravisCI's Windows support is experimental and they don't support many environments yet.
However, we can setup our own environment using Chocolatey (the Windows package manage). Below you can see the powershell script that I
wrote to setup the ideal Windows environment with Chocolatey.

scripts/windows_env_setup.ps1:
```powershell
param(
  [string]$PYTHON_VERSION,
  [string]$PYTHON_HOME
)

echo "PythonHome=$PYTHON_HOME"

# Variables describing expected environment
$PYTHON_SCRIPTS="$PYTHON_HOME\Scripts"
$PYTHON_EXE="$PYTHON_HOME\python.exe"
$SCRIPTS_DIR=$PSScriptRoot

# Installs Python into $PYTHON_HOME
# We specifically need a version of Python that is built with VS14
& choco install python --version $PYTHON_VERSION --allow-downgrade
# Downloads pip into $PYTHON_SCRIPTS
& $PYTHON_EXE "$SCRIPTS_DIR\get-pip.py"

# Installs the Visual Studio 14 (2015) compiler
& choco install vcbuildtools


# The VS2017 direrctory needs to be deleted so that it doesn't interfere with the VS2015 build.
Remove-Item "C:\Program Files (x86)\Microsoft Visual Studio\2017" -Force -Recurse
```

## SFML Linking

SFML was really easy to link in the Linux environment because we had the standard gcc environment variables, but that is not the case in the Windows environment.
Cython must supply the linking arguments to the Visual Studio command line tool during compilation. Therefore, the linking must be explicitly configured within
pySFML's setup.py file -- that's where Cython is configured.

Configuring the SFML linking was a pretty simple edit to the existing pySFML setup.py file. We just had to add the extlibs/SFML-2.3.2/include/ and extlibs/SFML-2.3.2/lib/ 
directories (of the downloaded SFML) to the extensions that were already defined by pySFML. Doing so means that Cython now passes that linking information
to Visual Studios during compilation.

setup.py:
```python
extension = lambda name, files, libs: Extension(
    name='sfml.' + name,
    sources= [os.path.join('src', 'sfml', name, filename) for filename in files],
    # Here is where the extlibs/SFML-2.3.2/include/ and extlibs/SFML-2.3.2/lib/ directories are linked.
    include_dirs=[os.path.join('include', 'Includes'), os.path.join('extlibs', 'SFML-2.3.2', 'include')],
    library_dirs=[os.path.join('extlibs', 'SFML-2.3.2', 'lib'), os.path.join('extlibs', 'libs-msvc-universal', arch)] if sys.hexversion >= 0x03050000 else [os.path.join('extlibs', 'SFML-2.3.2', 'lib')],
    language='c++',
    libraries=libs,
    define_macros=[('SFML_STATIC', '1')] if platform.system() == 'Windows' else [])

if platform.system() == 'Windows':
    system_libs      = ['winmm', 'sfml-system-s']
    window_libs      = ['user32', 'advapi32', 'winmm', 'sfml-system-s', 'gdi32', 'opengl32', 'sfml-window-s']
    graphics_libs    = ['user32', 'advapi32', 'winmm', 'sfml-system-s', 'gdi32', 'opengl32', 'sfml-window-s', 'freetype', 'jpeg', 'sfml-graphics-s']
    audio_libs       = ['winmm', 'sfml-system-s', 'flac', 'vorbisenc', 'vorbisfile', 'vorbis', 'ogg', 'openal32', 'sfml-audio-s']
    network_libs     = ['ws2_32', 'sfml-system-s', 'sfml-network-s']
else:
    system_libs      = ['sfml-system']
    window_libs      = ['sfml-system', 'sfml-window']
    graphics_libs    = ['sfml-system', 'sfml-window', 'sfml-graphics']
    audio_libs       = ['sfml-system', 'sfml-audio']
    network_libs     = ['sfml-system', 'sfml-network']

system = extension(
    'system',
    ['system.pyx', 'error.cpp', 'hacks.cpp', 'NumericObject.cpp'],
    system_libs)

window = extension(
    'window',
    ['window.pyx', 'DerivableWindow.cpp'],
    window_libs)

graphics = extension(
    'graphics',
    ['graphics.pyx', 'DerivableRenderWindow.cpp', 'DerivableDrawable.cpp', 'NumericObject.cpp'],
    graphics_libs)

audio = extension(
    'audio',
    ['audio.pyx', 'DerivableSoundRecorder.cpp', 'DerivableSoundStream.cpp'],
    audio_libs)

network = extension(
    'network',
    ['network.pyx'],
    network_libs)
```

# Conclusion

We were able to figure out an automated way to build pySFML in a Linux and Windows environment. However, TravisCI's early Windows support is terrible.
Instead, I'd probably recommend [AppVeyor](https://www.appveyor.com/) for Windows builds. I haven't used it yet, but it seems to offerr the same thing
that TravisCI offers, however AppVeyor focuses on Windows support. Maybe a combination of both tools would be good.