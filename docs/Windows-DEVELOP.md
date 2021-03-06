# Windows Development Guide for rdiff-backup

Some notes for developers and other people willing to help development on
Windows platform, or simply compile rdiff-backup from source code on Windows.

See also: https://wiki.python.org/moin/WindowsCompilers

## Pre-requisites

Here the list of required component to be install to start developing
rdiff-backup on Windows platform.

### Upgrade your Windows OS
Don't overlook this step. Visual Studio required an upgraded OS.
Use Windows Update to upgrade your system. Then proceed with the installation.

### Visual Studio Build Tools

1. To install Visual Studio Build Tools, you need to install .NET Framework 4.6.  
   [Download .NET Framework 4.6](https://www.microsoft.com/en-US/download/details.aspx?id=53344)
2. Finally, download Build Tools installation.  
   [Microsoft Build Tools for Visual Studio 2019](https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019)
3. Once downloaded, the filename should be `vs_buildtools.exe` or similar, launch it to start the installation.
4. From the user interface select "C++ Build Tools"
5. In the right panel checked the following items. Everything else may be unchecked.
    * MSVCv142 - VS 2019 C++ x64/x86 build tools
    * Windows 10 SDK
    * C++ CMake Tools for Windows

### Python 3

[Download Python 3.9.4 x64](https://www.python.org/ftp/python/3.9.4/python-3.9.4-amd64.exe)

1. Check "Add Python 3.9 to PATH"
2. Choose "Customize installation" to select the installation path.

You could verify if Python is working by executing `python --version` in a new command line windows.

### Python dependencies

Once python is installed, you should have a `pip` available from command line.
Open a terminal and execut the following commands to install the dependencies required to compile and run rdiff-backup.

    pip install pywin32 pyinstaller wheel
    
You could verify if packages are properly installed using:

    python -c "import pywintypes, winnt, win32api, win32security, win32file, win32con"

### CMake

[Download CMake 3.16.2 x64](https://github.com/Kitware/CMake/releases/download/v3.16.2/cmake-3.16.2-win64-x64.msi)

1. Make sure to select *Add CMake to the system PATH for all users*.
2. Install CMake to : `C:\Program Files\CMake\`

You could verify if cmake is working. Open a new terminal and execute `cmake --version`.

### 7z

[Download 7z 19.00 x64](https://www.7-zip.org/a/7z1900-x64.exe)

Install with default settings.

## Build librsync

[Download librsync 2.2.1 sources](https://github.com/librsync/librsync/releases/download/v2.2.1/librsync-2.2.1.tar.gz)

1. Extract it content using 7z `C:\librsync-2.2.1\`.
2. Open a "x64 Native Tools Command Prompt for VS 2019" from the start menu.
3. Type `cd C:\librsync-2.2.1\` 
4. `cmake -DCMAKE_INSTALL_PREFIX=C:\librsync\ -A x64 -DBUILD_SHARED_LIBS=OFF .`
5. `cmake --build . --config Release`
6. `cmake --install . --config Release`

Note: Source directory `C:\librsync-2.2.1\` is different to the target directory `C:\librsync\`.

These steps build a 64-bit version of librsync, suitable for use in a 64-bit version of Python. If using a 32-bit version
of python, use this in place of the first `cmake` command:

1. `cmake -DCMAKE_INSTALL_PREFIX=C:\librsync\ -A Win32 -DBUILD_SHARED_LIBS=OFF .`
2. Use the "Developer Command Prompt for VS2019" instead of the x64 variant.


## Build rdiff-backup

[Download rdiff-backup tar.gz](https://github.com/rdiff-backup/rdiff-backup/releases) e.g.: rdiff-backup-1.4.0b0.tar.gz

1. Extract it content using 7z `C:\rdiff-backup\`
2. Open a Command line terminal
3. Type `cd C:\rdiff-backup\`
4. `set LIBRSYNC_DIR=C:\librsync`
4. `python setup.py build`

## Troubleshooting

### Verify if Visual studio compiler is working

You could check if the compiler `cl` is working by calling:

    cl.exe hello.c

Where the file `hello.c` contains:

    #include <stdio.h>
    int main() {
       // printf() displays the string inside quotation
       printf("Hello, World!");
       return 0;
    }
    
The expected output should be as follow:

    Compilateur d'optimisation Microsoft (R) C/C++ version 19.24.28314 pour x86
    Copyright (C) Microsoft Corporation. Tous droits réservés.
    
    hello.c
    Microsoft (R) Incremental Linker Version 14.24.28314.0
    Copyright (C) Microsoft Corporation.  All rights reserved.
    
    /out:hello.exe
    hello.obj

### unresolved external symbol

If you see link errors like these:

    _librsyncmodule.obj : error LNK2001: unresolved external symbol rs_sig_begin
    _librsyncmodule.obj : error LNK2001: unresolved external symbol rs_job_free
    _librsyncmodule.obj : error LNK2001: unresolved external symbol rs_loadsig_begin

then you have probably compiled librsync for the wrong architecture. Try both `-A Win32`/`-A x64` switches when running
cmake to build librsync.

If you see link errors like these:

    cmodule.obj : error LNK2001: unresolved external symbol __imp__Py_BuildValue
    cmodule.obj : error LNK2001: unresolved external symbol __imp__PyDict_SetItemString
    cmodule.obj : error LNK2001: unresolved external symbol __imp__PyModule_GetDict

then you are using the wrong bitness build tools. Try both "x64 Native Tools Command Prompt for VS 2019" as well as
"Developer Command Prompt for VS2019".
