cmake_minimum_required(VERSION 3.13)

if (NOT DEFINED VERSION)
    set(VERSION 1.2.2)
endif ()

project(deepin_deb_installer_all)
option(DMAN_RELEAE OFF "Install dman resources to system or not")
add_subdirectory(src/AptInstallDepend)
add_subdirectory(src/deb_installer)
