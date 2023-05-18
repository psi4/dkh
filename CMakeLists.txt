cmake_policy(SET CMP0048 NEW)  # project_VERSION* variables populated from project(... VERSION x.x.x) string
if (CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
    # Avoid warning about DOWNLOAD_EXTRACT_TIMESTAMP in CMake 3.24:
    cmake_policy(SET CMP0135 NEW)
endif()
project(dkh
        VERSION 1.2
        LANGUAGES Fortran C)
set(dkh_AUTHORS      "Wolf, Reiher, and Hess")
set(dkh_DESCRIPTION  "Douglas-Kroll-Hess relativistic correction at 2nd–4th order")
set(dkh_URL          "http://www.reiher.ethz.ch/software/dkh-x2c.html")
set(dkh_LICENSE      "GNU Lesser General Public License, version 3")

cmake_minimum_required(VERSION 3.0)
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

################################### Options ####################################
include(psi4OptionsTools)
option_with_default(CMAKE_BUILD_TYPE "Build type" Release)
option_with_print(BUILD_SHARED_LIBS "Build final library as shared, not static" OFF)
option_with_default(BUILD_FPIC "Libraries will be compiled with position independent code" ON)
if(${BUILD_SHARED_LIBS} AND NOT ${BUILD_FPIC})
    message(FATAL_ERROR "BUILD_SHARED_LIBS ON and BUILD_FPIC OFF are incompatible, as shared library requires position independent code")
endif()
option_with_print(ENABLE_OPENMP "Enable threadsafe linking to OpenMP parallelized programs." ON)
option_with_print(ENABLE_GENERIC "Enable mostly static linking in shared library" OFF)
option_with_flags(ENABLE_XHOST "Enable processor-specific optimization" ON
                    "-xHost" "-march=native")
option_with_default(dkh_INSTALL_CMAKEDIR "Directory to which dkh CMake config files installed." share/cmake/dkh)

######################### Process & Validate Options ###########################
include(autocmake_safeguards)
include(autocmake_omp)
include(custom_static_library)

################################# Main Project #################################
include(GNUInstallDirs)
include(FortranCInterface)
include(CMakePackageConfigHelpers)


# <<<  Build  >>>

set(CMAKECONFIG_INSTALL_DIR "${dkh_INSTALL_CMAKEDIR}")

FortranCInterface_HEADER(DKH_MANGLE.h
                         MACRO_NAMESPACE "DKH_MANGLE_"
                         SYMBOL_NAMESPACE "DKH_MANGLE_")
set(raw_sources_list dkh2-dkh4_main.F90)
set(src_prefix "src/")
string(REGEX REPLACE "([^;]+)" "${src_prefix}\\1" sources_list "${raw_sources_list}")

# STATIC/SHARED on below governed by BUILD_SHARED_LIBS
add_library(dkh ${sources_list})
add_library(dkh::dkh ALIAS dkh)
set_target_properties(dkh PROPERTIES POSITION_INDEPENDENT_CODE ${BUILD_FPIC})
if(${BUILD_SHARED_LIBS})
    target_link_libraries(dkh PRIVATE ${LIBC_INTERJECT})

    if(APPLE)
        set_target_properties(dkh PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
    endif()
endif()

# BLAS
find_package(TargetLAPACK REQUIRED)
target_link_libraries (dkh PUBLIC tgt::lapack)

# <<<  Install  >>>
set(dkh ${PROJECT_NAME})  # NameSpace

install(FILES ${PROJECT_BINARY_DIR}/DKH_MANGLE.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/DKH)
install(TARGETS dkh
        EXPORT "${dkh}Targets"
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# <<<  Export Interface  >>>

target_compile_definitions(dkh INTERFACE USING_${dkh})
target_include_directories(dkh INTERFACE
                           $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

set(export_properties
  "dkh_VERSION"
  )
  set_target_properties(dkh
    PROPERTIES
      dkh_VERSION ${dkh_VERSION}
    )
set_property(TARGET dkh APPEND PROPERTY EXPORT_PROPERTIES "${export_properties}")

# <<<  Export Config  >>>

configure_package_config_file(cmake/${dkh}Config.cmake.in
                              "${CMAKE_CURRENT_BINARY_DIR}/${dkh}Config.cmake"
                              INSTALL_DESTINATION ${CMAKECONFIG_INSTALL_DIR})
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${dkh}ConfigVersion.cmake
                                 VERSION ${${dkh}_VERSION}
                                 COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${dkh}Config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${dkh}ConfigVersion.cmake
              cmake/FindTargetLAPACK.cmake
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})
install(EXPORT "${dkh}Targets"
        NAMESPACE "${dkh}::"
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})

