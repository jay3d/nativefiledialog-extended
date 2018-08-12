# Native File Dialog Extended #

**This library is modified from (but incompatible with) Michael Labbe's Native File Dialog ([mlabbe/nativefiledialog](https://github.com/mlabbe/nativefiledialog)).  Things might break if you use both in the same project.**

Changes from original Native File Dialog:

- Friendly names for filters (e.g. `C/C++ Source files (*.c;*.cpp)` instead of `(*.c;*.cpp)`)
- Native (suffixed with `N`) and UTF-8 (suffixed with `U8`) versions of all functions (Native is UTF-16 (`wchar_t`) for Windows and UTF-8 (`char`) for Mac/Linux)
- Initialization and de-initialization of platform library (e.g. COM (Windows) / GTK (Linux)) decoupled from dialog functions, so applications can choose when to initialize/de-initialize the platform library
- C++ scoped guards for initialization and de-initialization
- Various bug fixes

Customization macros (define them *before* including `nfd.h`/`nfd.hpp`):

- `NFD_NATIVE`: Define this before including `nfd.h` to make non-suffixed function names and typedefs (e.g. `NFD_OpenDialog`) aliases for the native functions (e.g. `NFD_OpenDialogN`) instead of aliases for the UTF-8 functions (e.g. `NFD_OpenDialogU8`).  This macro does not affect `nfd.hpp`.
- `NFD_GUARD_THROWS_EXCEPTION`: Define this before including `nfd.hpp` to make `NFD::Guard` construction throw `std::runtime_error` if `NFD_Init` fails.  Otherwise, there is no way to detect failure in `NFD::Guard` construction.

Macros that might be defined by `nfd.h`:

- `NFD_DIFFERENT_NATIVE_FUNCTIONS`: Defined if the native and UTF-8 versions of functions are different (i.e. compiling for Windows); not defined otherwise.  If `NFD_DIFFERENT_NATIVE_FUNCTIONS` is not defined, then the UTF-8 versions of functions are aliases for the native versions.  This might be useful if you are writing a function that wants to provide overloads depending on whether the native functions and UTF-8 functions are the same.

**Everything below this line is about the original Native File Dialog, and not Native File Dialog Extended.  The README for Native File Dialog Extended is still under construction.**

-----

A tiny, neat C library that portably invokes native file open, folder select and save dialogs.  Write dialog code once and have it pop up native dialogs on all supported platforms.  Avoid linking large dependencies like wxWidgets and qt.

Features:

 - Lean C API, static library -- no ObjC, no C++, no STL.
 - Zlib licensed.
 - Consistent UTF-8 support on all platforms.
 - Simple universal file filter syntax.
 - Paid support available.
 - Multiple file selection support.
 - 64-bit and 32-bit friendly.
 - GCC, Clang, Xcode, Mingw and Visual Studio supported.
 - No third party dependencies for building or linking.
 - Support for Vista's modern `IFileDialog` on Windows.
 - Support for non-deprecated Cocoa APIs on OS X.
 - GTK+3 dialog on Linux.
 - Optional Zenity support on Linux to avoid linking GTK+.
 - Tested, works alongside [http://www.libsdl.org](SDL2) on all platforms, for the game developers out there.

# Example Usage #

```C
#include <nfd.h>
#include <stdio.h>
#include <stdlib.h>

int main( void )
{
    nfdchar_t *outPath = NULL;
    nfdresult_t result = NFD_OpenDialog( NULL, NULL, &outPath );
        
    if ( result == NFD_OKAY ) {
        puts("Success!");
        puts(outPath);
        free(outPath);
    }
    else if ( result == NFD_CANCEL ) {
        puts("User pressed cancel.");
    }
    else {
        printf("Error: %s\n", NFD_GetError() );
    }

    return 0;
}
```

See [NFD.h](src/include/nfd.h) for more options.

# Screenshots #

![Windows 8 rendering an IFileOpenDialog](screens/open_win8.png?raw=true)
![GTK3 on Linux](screens/open_gtk3.png?raw=true)
![Cocoa on Yosemite](screens/open_cocoa.jpg?raw=true)

## Changelog ##

release | what's new                  | date
--------|-----------------------------|---------
1.0.0   | initial                     | oct 2014
1.1.0   | premake5; scons deprecated  | aug 2016
1.1.1   | mingw support, build fixes  | aug 2016
1.1.2   | test_pickfolder() added     | aug 2016
1.1.3   | zenity linux backend added  | nov 2017
1.1.3   | fix char type in decls      | nov 2017

## Building ##

NFD uses [Premake5](https://premake.github.io/download.html) generated Makefiles and IDE project files.  The generated project files are checked in under `build/` so you don't have to download and use Premake in most cases.

If you need to run Premake5 directly, further [build documentation](docs/build.md) is available.

Previously, NFD used SCons to build.  It still works, but is now deprecated; updates to it are discouraged.  Opt to use the native build system where possible.

`nfd.a` will be built for release builds, and `nfd_d.a` will be built for debug builds.

### Makefiles ###

The makefile offers five options, with `release_x64` as the default.

    make config=release_x86
    make config=release_x64
    make config=debug_x86
    make config=debug_x64

### Compiling Your Programs ###

 1. Add `src/include` to your include search path.
 2. Add `nfd.lib` or `nfd_d.lib` to the list of list of static libraries to link against (for release or debug, respectively).
 3. Add `build/<debug|release>/<arch>` to the library search path.

#### Linux ####
On Linux, you have the option of compiling and linking against GTK+.  If you use it, the recommended way to compile is to include the arguments of `pkg-config --cflags --libs gtk+-3.0`.

Alternatively, you can use the Zenity backend by running the Makefile in `build/gmake_linux_zenity`.  Zenity runs the dialog in its own address space, but requires the user to have Zenity correctly installed and configured on their system.

#### MacOS ####
On Mac OS, add `AppKit` to the list of frameworks.

#### Windows ####
On Windows, ensure you are building against `comctl32.lib`.

## Usage ##

See `NFD.h` for API calls.  See `tests/*.c` for example code.

After compiling, `build/bin` contains compiled test programs.

## File Filter Syntax ##

There is a form of file filtering in every file dialog API, but no consistent means of supporting it.  NFD provides support for filtering files by groups of extensions, providing its own descriptions (where applicable) for the extensions.

A wildcard filter is always added to every dialog.

### Separators ###

 - `;` Begin a new filter.
 - `,` Add a separate type to the filter.

#### Examples ####

`txt` The default filter is for text files.  There is a wildcard option in a dropdown.

`png,jpg;psd` The default filter is for png and jpg files.  A second filter is available for psd files.  There is a wildcard option in a dropdown.

`NULL` Wildcard only.

## Iterating Over PathSets ##

See [test_opendialogmultiple.c](test/test_opendialogmultiple.c).

# Known Limitations #

I accept quality code patches, or will resolve these and other matters through support.  See [submitting pull requests](docs/submitting_pull_requests.md) for details.

 - No support for Windows XP's legacy dialogs such as `GetOpenFileName`.
 - No support for file filter names -- ex: "Image Files" (*.png, *.jpg).  Nameless filters are supported, however.

# Copyright and Credit #

Copyright &copy; 2014-2017 [Frogtoss Games](http://www.frogtoss.com), Inc.
File [LICENSE](LICENSE) covers all files in this repo.

Native File Dialog by Michael Labbe
<mike@frogtoss.com>

Tomasz Konojacki for [microutf8](http://puszcza.gnu.org.ua/software/microutf8/)

[Denis Kolodin](https://github.com/DenisKolodin) for mingw support.

[Tom Mason](https://github.com/wheybags) for Zenity support.

## Support ##

Directed support for this work is available from the original author under a paid agreement.

[Contact Frogtoss Games](http://www.frogtoss.com/pages/contact.html).
