cmake_minimum_required(VERSION 2.8)

project(klav)

include(CMakeModules/Upload.cmake)

# . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
# version
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/CMakeModules/)
find_package(Hg)
if(HG_FOUND)
    message("hg found: ${HG_EXECUTABLE}")
    HG_WC_INFO(${PROJECT_SOURCE_DIR} Project)
    set(SW_VERSION "${Project_WC_REVISION}:${Project_WC_CHANGESET}")
    execute_process(COMMAND ${HG_EXECUTABLE} id -b
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
            OUTPUT_VARIABLE Project_WC_BRANCH)
    message("SW_VERSION is ${SW_VERSION}")
endif(HG_FOUND)

configure_file(${CMAKE_SOURCE_DIR}/version.cpp.in
               ${CMAKE_BINARY_DIR}/version.cpp
               @ONLY)
configure_file(${CMAKE_SOURCE_DIR}/version.txt.in
               ${CMAKE_BINARY_DIR}/version.txt
               @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/version.txt
        DESTINATION ${CMAKE_INSTALL_PREFIX}
        COMPONENT core)
add_library(PRJ_VER_LIB STATIC ${CMAKE_BINARY_DIR}/version.cpp)
set_property(TARGET PRJ_VER_LIB PROPERTY POSITION_INDEPENDENT_CODE ON)
SET(PROJECT_VERSION_LIB -Wl,--whole-archive PRJ_VER_LIB -Wl,--no-whole-archive)

# . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
# robot-data
add_subdirectory(robot-data)
include_directories(${ROBOT_INCLUDE})

# . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
# generic
get_filename_component(GENERIC_INC_DIR ./include ABSOLUTE)
get_filename_component(GENERIC_SRC_DIR ./src ABSOLUTE)
get_filename_component(SUPERVISORTOP_INC_DIR ./supervisorTop/include ABSOLUTE)
get_filename_component(SUPERVISORTOP_SRC_DIR ./supervisorTop/src ABSOLUTE)
get_filename_component(GASKET_INC_DIR ./gasket/include ABSOLUTE)
get_filename_component(GASKET_SRC_DIR ./gasket/src ABSOLUTE)
get_filename_component(MODULES_INC_DIR ./rios2-modules-klav/include ABSOLUTE)

# rios2
get_filename_component(RIOS2_INC_DIR ./rios2-core/include ABSOLUTE)
get_filename_component(RIOS2_BIN_DIR ./rios2-core/bin ABSOLUTE)
get_filename_component(RIOS2_SRC_DIR ./rios2-core/src ABSOLUTE)
#find_library(RIOS2_LIB rios ${RIOS2_BIN_DIR})
#find_library(RIOS2_LIB_DBG rios_dbg ${RIOS2_BIN_DIR})
set(RIOS2_LIB librios)
set(RIOS2_LIB_DBG librios_dbg)

# rios3
get_filename_component(RIOS3_INC_DIR ./rios3/src ABSOLUTE)

#get_filename_component(version_cpp ./src/version.cpp ABSOLUTE)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -Wno-missing-field-initializers -fpic") #
add_definitions(-D GASKET_DATA_INCLUDE="<tops-klav-data.h>" -D STZ_PARAMS_DIR='"klav-STZ/Params"' -D __cdecl="" -D typeof="__typeof__")
add_definitions(-D__STDC_LIMIT_MACROS)

include(CMakeModules/md5_module.cmake)

add_subdirectory(rios2-core)
add_subdirectory(rios3)
add_subdirectory(gasket)
add_subdirectory(supervisorTop)
add_subdirectory(rios2-modules-klav) # modules
add_subdirectory(src) # tops
add_subdirectory(klav-mns)

function(print_all_lists description)
    set(argn_var ${ARGN})
    list(LENGTH argn_var args_count)
    message("${description}: ${args_count}")
    foreach(value ${ARGN})
        if(NOT ${value} STREQUAL "")
            message(\t${value})
        else()
            message("......")
        endif()
    endforeach(value)
    message("")
endfunction(print_all_lists)

message("\n----------------------------------------------")
print_all_lists("Список топов" ${all_tops_targets})
print_all_lists("Список модулей" ${all_mods_targets})

# - - - - - - - - - - - -  - - - - - #
#           Installation part        #
# - - - - - - - - - - - -  - - - - - #


install(DIRECTORY bin
        DESTINATION ${CMAKE_INSTALL_PREFIX}
        USE_SOURCE_PERMISSIONS
        COMPONENT core)
add_to_md5_list(DIRECTORY bin DESTINATION ${CMAKE_INSTALL_PREFIX})
install(DIRECTORY klav-STZ/Params DESTINATION ${CMAKE_INSTALL_PREFIX}/klav-STZ COMPONENT core)
#add_to_md5_list(DIRECTORY klav-STZ/Params DESTINATION ${CMAKE_INSTALL_PREFIX}/Klgetavesin-STZ)
#install(DIRECTORY klav-ausio/_release/ DESTINATION ${CMAKE_INSTALL_PREFIX}/Ausio COMPONENT core)
#add_to_md5_list(DIRECTORY klav-ausio/_release/ DESTINATION ${CMAKE_INSTALL_PREFIX}/Ausio)

file(GLOB CORE_CONFIG_FILES *.csv *.conf *.cfg *.txt *.ods)
# CMakeLitst.txt shouldn't install
get_filename_component(_cmake_lists_file CMakeLists.txt ABSOLUTE)
list(REMOVE_ITEM CORE_CONFIG_FILES ${_cmake_lists_file})
install(FILES ${CORE_CONFIG_FILES}
        DESTINATION ${CMAKE_INSTALL_PREFIX}
        COMPONENT core)
add_to_md5_list(FILES ${CORE_CONFIG_FILES} DESTINATION ${CMAKE_INSTALL_PREFIX})

install(DIRECTORY conf DESTINATION ${CMAKE_INSTALL_PREFIX} COMPONENT core)
add_to_md5_list(DIRECTORY conf DESTINATION ${CMAKE_INSTALL_PREFIX})

# - - - - - - - - - - - -  - - - - - #
#           Collection part          #
# - - - - - - - - - - - -  - - - - - #

add_custom_target(_collect_core COMMAND ${CMAKE_COMMAND} -DCOMPONENT=core -P cmake_install.cmake)
add_custom_target(collect DEPENDS _collect_core)

add_custom_target(collect-clean COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_INSTALL_PREFIX})
add_custom_target(clean_all
                  COMMAND ${CMAKE_COMMAND} --build . --target clean
                  COMMAND ${CMAKE_COMMAND} --build . --target collect-clean)

# - - - - - - - - - - - -  - - - - - #
#           Uploading part           #
# - - - - - - - - - - - -  - - - - - #

CREATE_UPLOAD_TARGETS(bspu stz)
