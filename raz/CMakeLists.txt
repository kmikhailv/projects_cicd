CMAKE_MINIMUM_REQUIRED(VERSION 3.9)

OPTION(BUILD_WITH_QT "Build projects with using Qt" ON)
OPTION(BUILD_WITH_QT5 "Build with Qt5" ON)
# Ввести свои конфигурации через точку с запятой: "debug;deploy" и т.д.
set(CONFIGURATIONS_LIST "local;radio")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
 "${CMAKE_SOURCE_DIR}/CMakeModules" CACHE STRING "Modules for CMake" FORCE)

include(scripts/InstallAndConfigureFiles.cmake)
include(CMakeModules/getCurrentVersion.cmake)

ADD_DEFINITIONS(-DPrjRootPath="${CMAKE_SOURCE_DIR}")

SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)

PROJECT(raz)
include(scripts/Upload.cmake)
SET(CMAKE_BUILD_TYPE "Debug")
include(CheckCXXCompilerFlag)

function(enable_cxx_compiler_flag_if_supported flag)
    string(FIND "${CMAKE_CXX_FLAGS}" "${flag}" flag_already_set)
    if(flag_already_set EQUAL -1)
        check_cxx_compiler_flag("${flag}" flag_supported)
        if(flag_supported)
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${flag}" PARENT_SCOPE)
        endif()
    endif()
endfunction()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9)
        message(FATAL_ERROR "GCC version must be at least 4.9!")
    endif()
endif()

enable_cxx_compiler_flag_if_supported("-Wall")
enable_cxx_compiler_flag_if_supported("-Wextra")
enable_cxx_compiler_flag_if_supported("-pedantic")
enable_cxx_compiler_flag_if_supported("-std=gnu++11")
enable_cxx_compiler_flag_if_supported("-Wl,-rpath,/usr/local/lib64")

SET(DIR_BINS_EXEC ${CMAKE_SOURCE_DIR}/bins/exec)
SET(DIR_BINS_LIBS ${CMAKE_SOURCE_DIR}/bins/libs)
SET(DIR_APPS ${CMAKE_SOURCE_DIR}/apps)
SET(DIR_CFGS ${CMAKE_SOURCE_DIR}/cfgs)
SET(DIR_LIBS ${CMAKE_SOURCE_DIR}/libs)
SET(DIR_DLLS ${CMAKE_SOURCE_DIR}/dlls)
SET(DIR_TEST ${CMAKE_SOURCE_DIR}/test)
SET(DIR_DATA ${CMAKE_SOURCE_DIR}/data)

set(CONFIGS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/cfgs)
#set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR})
SET(INSTALL_BINS_EXEC ${CMAKE_INSTALL_PREFIX})
SET(INSTALL_BINS_LIBS ${CMAKE_INSTALL_PREFIX}/libs)
set(CMAKE_INSTALL_RPATH ${COLLECT_INSTALL_DIR}/libs/)

# configure a header file to pass some of the CMake SETtings
# to the source code
CONFIGURE_FILE (
	"${DIR_CFGS}/config.hpp.in"
	"${DIR_CFGS}/inc/config.hpp"
)
#${CMAKE_INCLUDE}
#RT
IF(UNIX)
        INCLUDE(CheckLibraryExists)
        CHECK_LIBRARY_EXISTS(rt clock_gettime "time.h" HAVE_CLOCK_GETTIME)

        IF(NOT HAVE_CLOCK_GETTIME)
                SET(CMAKE_EXTRA_INCLUDE_FILES time.h)
                CHECK_FUNCTION_EXISTS(clock_gettime HAVE_CLOCK_GETTIME)
                SET(CMAKE_EXTRA_INCLUDE_FILES)
        ENDIF()
ENDIF()

#Threads
IF(UNIX)
        FIND_PACKAGE(Threads REQUIRED)
        IF(NOT Threads_FOUND)
                MESSAGE(SEND_ERROR "Failed to find Threads")
                return()
        ENDIF()
ENDIF()

#OpenCV
find_package(OpenCV 2.4 COMPONENTS core highgui imgproc)
IF(OpenCV_FOUND)
    MESSAGE("OPENCV FOUND!")
    MESSAGE("OPENCV LIBS: " ${OpenCV_LIBS})
    LINK_DIRECTORIES(/usr/local/lib)
ENDIF()

#Armadillo
find_package(Armadillo)
IF(ARMADILLO_FOUND)
    MESSAGE("ARMADILLO FOUND!")
ENDIF()

#PROTOBUF-C see ./CMakeModules
find_package(PROTOBUF-C)
IF(PROTOBUF_FOUND)
    MESSAGE("PROTOBUF-C FOUND!")
ENDIF()

#QT
SET(CMAKE_AUTOMOC ON)
SET(CMAKE_AUTORCC ON)
SET(CMAKE_AUTOUIC ON)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

SET(COMMON_RTK_MODULES_INCLUDES
        ${DIR_LIBS}/lcommon_rtk_modules
        ${DIR_LIBS}/lcommon_rtk_modules/top_screen_widget
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/gps
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/gsn
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/inclinometer
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/mooum
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/mvr
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/solum
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/camera
        ${DIR_LIBS}/lcommon_rtk_modules/devices_modules/mp02
        ${DIR_LIBS}/lcommon_rtk_modules/bd_modules/bdgn
        ${DIR_LIBS}/lcommon_rtk_modules/bd_modules/bdgp
        ${DIR_LIBS}/lcommon_rtk_modules/bd_modules/bdpn
        ${DIR_LIBS}/lcommon_rtk_modules/stz_module
        ${DIR_LIBS}/lcommon_rtk_modules/rtk_misison_module
        ${DIR_LIBS}/lcommon_rtk_modules/func_test_module
        ${DIR_LIBS}/lcommon_rtk_modules/configurator_module
        ${DIR_LIBS}/lcommon_rtk_modules/network_screen
)

SET(CMAKE_AUTOUIC_SEARCH_PATHS ${CMAKE_AUTOUIC_SEARCH_PATHS}
    ${COMMON_RTK_MODULES_INCLUDES}
)

IF(UNIX)
    SET(CMAKE_PREFIX_PATH /usr/local/Qt-5.9.8/lib/cmake
                          /usr/local/Trolltech/Qt-4.8.7/bin)
    IF(BUILD_WITH_QT)
        ADD_DEFINITIONS("-DBUILD_WITH_QT")
        IF(BUILD_WITH_QT5)
            find_package(Qt5 5.9 COMPONENTS Core Widgets Network 3DCore 3DRender 3DExtras 3DInput
                Concurrent PrintSupport Svg OpenGL)
        ENDIF()
        IF(NOT Qt5Core_FOUND)
            find_package(Qt4 4.8.7 REQUIRED QtCore QtGui QtOpenGL QtSvg COMPONENTS)
            IF(QT4_FOUND)
                MESSAGE("QT4 FOUND!")
                SET(QT4_FOUND TRUE)
            ELSE()
                FATAL_ERROR("QT NOT FOUND!")
            ENDIF()
        ELSE()
            MESSAGE("QT5 FOUND!")
            SET(QT5_FOUND TRUE)
            enable_cxx_compiler_flag_if_supported("-Wl,-rpath,/usr/local/Qt-5.9.8/lib")
        ENDIF()

        IF(QT4_FOUND)
            MESSAGE("Using qwt-5.2.3")
            SET(QWT_DIR ${DIR_LIBS}/lqwt/qwt-5.2.3)
        ELSEIF(QT5_FOUND)
            MESSAGE("Using qwt-6.1.4")
            SET(QWT_DIR ${DIR_LIBS}/lqwt/qwt-6.1.4)
        ENDIF()

        INCLUDE_DIRECTORIES(${QWT_DIR}/src)
    ENDIF()
ENDIF()

#add subdirectories
ADD_SUBDIRECTORY(libs)
#ADD_SUBDIRECTORY(dlls)
#Applications
#CF-31
ADD_SUBDIRECTORY(apps)
#Tests
#ADD_SUBDIRECTORY(test)
#Drivers
#ADD_SUBDIRECTORY(drvs)

CREATE_UPLOAD_TARGETS(rbv1 rbv2 rbv1r rbv2r sbv1 sbv2 sbv1r sbv2r armk armr arms)
