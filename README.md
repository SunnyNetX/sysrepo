# Sysrepo

[![BSD license](https://img.shields.io/badge/License-BSD-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)
[![Website](https://img.shields.io/website?down_color=lightgrey&down_message=offline&up_color=blue&up_message=online&url=https%3A%2F%2Fwww.sysrepo.org%2F)](https://www.sysrepo.org/)
[![Build](https://github.com/sysrepo/sysrepo/workflows/sysrepo%20CI/badge.svg)](https://github.com/sysrepo/sysrepo/actions?query=workflow%3A%22sysrepo+CI%22)
[![Docs](https://img.shields.io/badge/docs-link-blue)](https://netopeer.liberouter.org/doc/sysrepo/)

Sysrepo is a [YANG](http://tools.ietf.org/html/rfc7950)-based configuration and operational state data store for Unix/Linux applications.

Applications can use sysrepo to store their configuration modeled by provided YANG model instead of using e.g. flat configuration files. Sysrepo will ensure data consistency of the data stored in the datastore and enforce data constraints defined by YANG model. Applications can currently use [C language API](inc/sysrepo.h) of sysrepo Client Library to access the configuration in the datastore, but the support for other programming languages is planed for later, too.

Sysrepo can be easily integrated with management agents such as [NETCONF](https://tools.ietf.org/html/rfc6241) or [RESTCONF](https://tools.ietf.org/html/rfc8040) servers, using the same client library API that applications use to access their configuration. As of now, sysrepo is integrated with the [Netopeer 2 NETCONF server](https://github.com/CESNET/Netopeer2). This means that applications that use sysrepo to store their configuration can automatically benefit from the ability to being controlled via NETCONF.

## Branches

The project uses 2 main branches `master` and `devel`. Other branches should not be cloned. In `master` there are files of the
last official *release*. Any latest improvements and changes, which were tested at least briefly are found in `devel`. On every
new *release*, `devel` is merged into `master`.

This means that when only stable official releases are to be used, either `master` can be used or specific *releases* downloaded.
If all the latest bugfixes should be applied, `devel` branch is the  one to be used. Note that whenever **a new issue is created**
and it occurs on the `master` branch, the **first response will likely be** to use `devel` before any further provided support.

## Migration from Sysrepo version 0.7.x or older

This Sysrepo is a complete rewrite of these older versions. Latest version of the `0.7` version is found in the `legacy` branch.
We tried to keep the API as similar as possible but there were some smaller or even bigger changes (mostly for the sake
of efficiency). All these changes should be mentioned in `CHANGES` text file. It is also best to look at least briefly
at the documentation where you will find information about major design changes (most importantly, no `sysrepod`).

## Provided Features

* Ability to store / retrieve YANG-modeled data elements adressed by XPath
* Startup, running, candidate, and operational datastore support
* Data consistency and constraints enforcement according to YANG models
* No single point of failure design (is just a library)
* Full transaction and concurrency support
* Notifications of subscribed applications about the changes made in the datastore
* Commit verifiers (change verification by subscribed applications)
* Operational data support (publishing of application's state/configuration data to sysrepo)
* YANG 1.1 support
* Custom RPC, Event Notifications, YANG 1.1 Actions support
* Notification store & notification replay

## Security Notes

Sysrepo does not have any master process that could enforce complex access control. So instead, it relies on and
utilizes standard file system permissions but there are some things to bear in mind.

To prevent any sensitive data from being accessible by unauthorized processes, it is imperative to **always
set correct permissions and owner** for all YANG modules being installed. The utility `sysrepoctl` can help
with both displaying all the permissions (`--list`) and modifying them (`--change <module>`) in addition
to this functionality being available in the API.

Having made certain of this, the default configuration should be suitable for a reasonably secure machine
that has no malicious running processes. Specifically, it is trivial for such a process to completely break
sysrepo by writing into shared files that must be accessible for all the processes linked with sysrepo. Also,
with some reverse engineering, it may even be possible to access data by an unathorized process when they are being
communicated in these shared files.

In order to avoid all such security issues, there are 2 `cmake` variables `SYSREPO_UMASK` and `SYSREPO_GROUP`
that should be adjusted. Generally, a new system group should be created and set for `SYSREPO_GROUP` and then
all outside access frobidden by setting `SYSREPO_UMASK` to `00007`. If then all the users executing sysrepo
processes belong to this group, none of sysrepo files and no sensitive information should be accessible to
other users.

## Requirements

### Build Requirements

* C compiler (gcc >= 4.8.4, clang >= 3.0, ...)
* cmake >= 2.8.12
* [libyang](https://github.com/CESNET/libyang)

#### Optional

* doxygen (for generating documentation)
* cmocka >= 1.0.0 (for tests only, see [Tests](#Tests))
* valgrind (for enhanced testing)

## Building

```
$ mkdir build; cd build
$ cmake ..
$ make
# make install
```

### Plugin development documentation

Another markdown document aimed at plugin developers is available in [plugin_dev_doc.md](./plugin_dev_doc.md).
The goal of the document is to provide a single place where a complete overview of information required to
start developing plugins is available.
The document describes the basics of Sysrepo plugin development and the technologies required to work with
Sysrepo like YANG, XPath, NETCONF and others.

### Useful CMake sysrepo Options

Set custom repository path:
```
-DREPO_PATH=/opt/sysrepo/my_repository
```

Set custom `sysrepo-plugind` plugins path:
```
-DPLUGINS_PATH=/opt/sysrepo-plugind/plugins
```

Set global `umask` for all sysrepo file and directory creation:
```
-DSYSREPO_UMASK=00007
```

Set system group to own all sysrepo-related files:
```
-DSYSREPO_GROUP=sysrepo
```

### Useful CMake Build Options

#### Changing Compiler

Set `CC` variable:

```
$ CC=/usr/bin/clang cmake ..
```

#### Changing Install Path

To change the prefix where the library, headers and any other files are installed,
set `CMAKE_INSTALL_PREFIX` variable:
```
$ cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
```

Default prefix is `/usr/local`.

#### Build Modes

There are two build modes:
* Release.
  This generates library for the production use without any debug information.
* Debug.
  This generates library with the debug information and disables optimization
  of the code.

The `Debug` mode is currently used as the default one. to switch to the
`Release` mode, enter at the command line:
```
$ cmake -D CMAKE_BUILD_TYPE:String="Release" ..
```

#### Code Coverage

To generate statistical information about code coverage by tests, set
`ENABLE_COVERAGE` option to `ON`:
```
$ cmake -D ENABLE_COVERAGE="ON" ..
```
and then the make's `coverage` target should be available to geenrate statistics:
```
$ make coverage
```

Note that `gcc` compiler is required for this option and additional tools are required:
* gcov
* lcov
* genhtml

## Usage

All Sysrepo functions are available via the main header:

```C
#include <sysrepo.h>
```

To compile your program with libsysrepo, it is necessary to link it using the
following linker parameters:
```
-lsysrepo
```

Note, that it may be necessary to call `ldconfig(8)` after library installation and if the
library was installed into a non-standard path, the path to libyang must be specified to the
linker. To help with setting all the compiler's options, there is `sysrepo.pc` file for
`pkg-config(1)` available in the source tree. The file is installed with the library.

### Used run-time enviromental variables

It is possible to change the repository path by setting `SYSREPO_REPOSITORY_PATH` variable.
Also, if `SYSREPO_SHM_PREFIX` is defined, it is used for all SHM files created. This way
everal *sysrepo* instances can effectively be run simultanously on one machine.

## Examples

See [examples](examples) directory, which contains an example for basic API functions.

## Bindings

There are no bindings for other languages directly in this project but they are
available separately.

* [Python](https://github.com/sysrepo/sysrepo-python/)

## Tests

libyang includes several tests built with [cmocka](https://cmocka.org/). The tests
can be found in `tests` subdirectory and they are designed for checking library
functionality after code changes.

The tests are by default built in the `Debug` build mode by running
```
$ make
```

In case of the `Release` mode, the tests are not built by default (it requires
additional dependency), but they can be enabled via cmake option:
```
$ cmake -DENABLE_TESTS=ON ..
```

Note that if the necessary [cmocka](https://cmocka.org/) headers are not present
in the system include paths, tests are not available despite the build mode or
cmake's options.

Tests can be run by the make's `test` target:
```
$ make test
```


