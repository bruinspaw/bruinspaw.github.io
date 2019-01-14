---
layout: post
title:  "Solution to PyQt4 compiling error on macOS"
date:   2019-01-14 21:05:20 +0800
author: bruinspaw
categories: PyQt
---

# Parameters

* Qt Dir: /usr/local/Qt/4.8.7/bin
* SIP: sip-4.19.13
* PyQt: PyQt4_gpl_mac-4.12.3
* macOS: 10.14

# Reference

[Building PyQt4](http://pyqt.sourceforge.net/Docs/PyQt4/installation.html#building-pyqt4)

# Steps
## Step 1: build sip for PyQt4
```
$ python3 configure.py --sip-module PyQt4.sip --no-dist-info --no-tools
his is SIP 4.19.13 for Python 3.7.1 on darwin.
The PyQt4.sip module will be installed in
/usr/local/Cellar/python/3.7.1/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/PyQt4.
The sip.pyi stub file will be installed in
/usr/local/Cellar/python/3.7.1/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/PyQt4.
The default directory to install .sip files in is
/usr/local/Cellar/python/3.7.1/Frameworks/Python.framework/Versions/3.7/share/sip.
Creating sipconfig.py...
Creating top level Makefile...
Creating sip code generator Makefile...
Creating sip module Makefile...
$ make && make install
```

## Step 2:
```
$ export PATH=/usr/local/Qt/4.8.7/bin:$PATH
$ export QMAKESPEC=unsupported/macx-clang-libc++
```

## Step 3:
* Copy sipconfig.py generated above to PyQt4 source directory
* Modify 'configure.py' in PyQt4 source directory,  the default mkspec of Qt4
on my machine is 'unsupported/macx-clang-libc++':
```
qt_macx_spec = 'unsupported/macx-clang-libc++'
```
then
```
$ python3 configure.py
$ make && make install
```

# Troubleshooting
## Set: qt_macx_spec = 'unsupported/macx-clang-libc++'
```
$ python3 configure.py --qmake /usr/local/Qt/4.8.7/bin/qmake
Determining the layout of your Qt installation...
Error: Failed to determine the layout of your Qt installation. Try again using
the --verbose flag to see more detail about the problem.
```
```
$ python3 configure.py --qmake /usr/local/Qt/4.8.7/bin/qmake --verbose
Determining the layout of your Qt installation...
/usr/local/Qt/4.8.7/bin/qmake -spec macx-g++ -o qtdirs.mk qtdirs.pro
make -f qtdirs.mk
g++ -c -pipe -O2 -arch x86_64 -Xarch_x86_64 -mmacosx-version-min=10.5 -Wall -W -DQT_NO_DEBUG -DQT_CORE_LIB -DQT_SHARED -I/usr/local/Qt/4.8.7/mkspecs/macx-g++ -I. -I/usr/local/Qt/4.8.7/lib/QtCore.framework/Versions/4/Headers -I/usr/local/Qt/4.8.7/include/QtCore -I/usr/local/Qt/4.8.7/include -I. -F/usr/local/Qt/4.8.7/lib -o qtdirs.o qtdirs.cpp
warning: include path for stdlibc++ headers not found; pass '-std=libc++' on the command line to use the libc++ standard library instead [-Wstdlibcxx-not-found]
In file included from qtdirs.cpp:1:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Versions/4/Headers/QCoreApplication:1:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Versions/4/Headers/qcoreapplication.h:45:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Headers/qobject.h:47:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Headers/qobjectdefs.h:45:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Headers/qnamespace.h:45:
/usr/local/Qt/4.8.7/lib/QtCore.framework/Headers/qglobal.h:68:10: fatal error: 'algorithm' file not found
#include <algorithm>
         ^~~~~~~~~~~
1 warning and 1 error generated.
make: *** [qtdirs.o] Error 1
Error: Failed to determine the layout of your Qt installation. Try again using
the --verbose flag to see more detail about the problem.
```

## Set: QMAKESPEC=unsupported/macx-clang-libc++
```
g++ -c -pipe -fno-strict-aliasing -O2 -arch x86_64 -Xarch_x86_64 -mmacosx-version-min=10.5 -Wall -W -fPIC -DQT_DISABLE_DEPRECATED_BEFORE=0x04ffff -DQT_NO_DEBUG -DQT_GUI_LIB -DQT_CORE_LIB -DQT_SHARED -I/usr/local/Qt/4.8.7/mkspecs/macx-g++ -I. -I/usr/local/Qt/4.8.7/lib/QtCore.framework/Versions/4/Headers -I/usr/local/Qt/4.8.7/include/QtCore -I/usr/local/Qt/4.8.7/lib/QtGui.framework/Versions/4/Headers -I/usr/local/Qt/4.8.7/include/QtGui -I/usr/local/Qt/4.8.7/include -I/usr/local/Cellar/python/3.7.1/Frameworks/Python.framework/Versions/3.7/include/python3.7m -I../../QtCore -I. -I. -F/usr/local/Qt/4.8.7/lib -o qpycore_chimera.o qpycore_chimera.cpp
warning: include path for stdlibc++ headers not found; pass '-std=libc++' on the
      command line to use the libc++ standard library instead
      [-Wstdlibcxx-not-found]
In file included from qpycore_chimera.cpp:23:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Versions/4/Headers/QByteArray:1:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Versions/4/Headers/qbytearray.h:45:
In file included from /usr/local/Qt/4.8.7/lib/QtCore.framework/Headers/qatomic.h:45:
/usr/local/Qt/4.8.7/lib/QtCore.framework/Headers/qglobal.h:68:10: fatal error:
      'algorithm' file not found
#include <algorithm>
         ^~~~~~~~~~~
1 warning and 1 error generated.
make[2]: *** [qpycore_chimera.o] Error 1
make[1]: *** [all] Error 2
make: *** [all] Error 2
```

## Build sip as a private module: --sip-module PyQt4.sip 
```
ModuleNotFoundError: No module named 'PyQt4.sip'
```
