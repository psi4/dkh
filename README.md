# dkh
Wolf, Reiher, and Hess's Douglas-Kroll-Hess relativistic correction at 2nd–4th
order (http://www.reiher.ethz.ch/software/dkh-x2c.html) wrapped in CMake for Psi4
(https://github.com/psi4/psi4)

* A. Wolf, M. Reiher, B. A. Hess, J. Chem. Phys. 117, 9215 (2002); https://doi.org/10.1063/1.1515314
* M. Reiher, A. Wolf, J. Chem. Phys. 121, 10945 (2004); https://doi.org/10.1063/1.1818681

### Obtain

[![Anaconda-Server Badge](https://anaconda.org/conda-forge/dkh/badges/version.svg)](https://anaconda.org/conda-forge/dkh)
[![Anaconda-Server Badge](https://anaconda.org/conda-forge/dkh/badges/platforms.svg)](https://anaconda.org/conda-forge/dkh)
[![Anaconda-Server Badge](https://anaconda.org/conda-forge/dkh/badges/downloads.svg)](https://anaconda.org/conda-forge/dkh) + [![Anaconda-Server Badge](https://anaconda.org/psi4/dkh/badges/downloads.svg)](https://anaconda.org/psi4/dkh)

### History

This is the DKH project (http://www.reiher.ethz.ch/software/dkh-x2c.html) by
Prof. M. Reiher of ETH.

DKH is written in Fortran. It has discussion available at the above website.

### This Repository

DKH has been in the *ab initio* quantum chemistry package Psi4
(http://psicode.org/, https://github.com/psi4/psi4) since November 2014. In Psi4,
it builds with `cmake` and has an interface to C++ and Psi4 internals designed
by @jturney. Manual for DKH+Psi4 at http://psicode.org/psi4manual/master/dkh.html.
This repository is DKH wrapped up nicely in CMake.

#### Version

This code was received by email. The latest version number it mentions is 1.2.

#### License

By permission of Prof. Reiher, this code is licensed
under the GNU Lesser General Public License version 3
([LGPL-3.0](https://opensource.org/licenses/LGPL-3.0)).

#### Building

```bash
cmake -S. -Bobjdir
cd objdir && make
make install
```

The build is also responsive to

* static/shared toggle `BUILD_SHARED_LIBS`
* install location `CMAKE_INSTALL_PREFIX`
* of course, `CMAKE_Fortran_COMPILER`, `CMAKE_C_COMPILER`, and `CMAKE_Fortran_FLAGS`

See [CMakeLists.txt](CMakeLists.txt) for options details. All these build options should be passed as `cmake -DOPTION`.

#### Detecting

This project installs with `dkhConfig.cmake`, `dkhConfigVersion.cmake`, and `dkhTargets.cmake` files suitable for use with CMake [`find_package()`](https://cmake.org/cmake/help/v3.2/command/find_package.html) in `CONFIG` mode.

* `find_package(dkh)` - find any dkh libraries and headers
* `find_package(dkh 1.2 EXACT CONFIG REQUIRED COMPONENTS static)` - find dkh exactly version 1.2 built with static libraries or die trying

See [dkhConfig.cmake.in](cmake/dkhConfig.cmake.in) for details of how to detect the Config file and what CMake variables and targets are exported to your project.

#### Using

After `find_package(dkh ...)`,

* test if package found with `if(${dkh_FOUND})` or `if(TARGET dkh::dkh)`
* link to library (establishes dependency), including header and definitions configuration with `target_link_libraries(mytarget dkh::dkh)`
* include header files using `target_include_directories(mytarget PRIVATE $<TARGET_PROPERTY:dkh::dkh,INTERFACE_INCLUDE_DIRECTORIES>)`
* compile target applying `-DUSING_dkh` definition using `target_compile_definitions(mytarget PRIVATE $<TARGET_PROPERTY:dkh::dkh,INTERFACE_COMPILE_DEFINITIONS>)`
