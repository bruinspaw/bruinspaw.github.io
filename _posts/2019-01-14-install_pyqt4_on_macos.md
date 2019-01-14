---
layout: post
title:  "Solution to PyQt4 compiling error on macOS"
date:   2019-01-14 21:05:20 +0800
author: bruinspaw
categories: PyQt
---

When building PyQt4 from source, an error was encountered:
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

Solution: Modify 'configure.py',  the default mkspec on my machine is
'unsupported/macx-clang-libc++':
```
qt_macx_spec = 'unsupported/macx-clang-libc++'
```
