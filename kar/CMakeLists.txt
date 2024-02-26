cmake_minimum_required(VERSION 3.9.6) # Проверка версии CMake.
# Если версия установленой программы
# старее указаной, произайдёт аварийный выход.
project(kar-pa CXX) #название проекта

#SET(GCC_COVERAGE_COMPILE_FLAGS "-fsanitize=address -fsanitize=leak -g")
#SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS}")

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
endif(NOT CMAKE_BUILD_TYPE)

include(CMakeModules/Upload.cmake)

# Ввести свои конфигурации через точку с запятой: "debug;deploy" и т.д.
set(CONFIGURATIONS_LIST "debug")
include(CMakeModules/InstallAndConfigureFiles.cmake)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11 -pedantic -fPIC") #опции компилятора

option(kar_PA "tops-kar-pa" ON)
add_definitions(-Dkar_PA)

include_directories("supervisorTop/include")
include_directories("gasket/include")
include_directories("rios3/src")
include_directories("settings-file")
include_directories("settings-file/include")

add_subdirectory(robot-data)
include_directories(${ROBOT_INCLUDE})

set(SUPERVISORTOP_INC_DIR ../supervisorTop/include)
add_subdirectory(rios3)
add_subdirectory(supervisorTop)
add_subdirectory(gasket)

add_definitions(-DRIOS3_SUPPORT)

##указываем библиотеки которые линкуются
set(LIBS_TO_LINK GASKET_LIB RIOS3_LIB SUPERVISORTOP_LIB dl pthread)

add_subdirectory(settings-file)
include_directories(${SF_INCLUDE})

add_subdirectory(tops-common)
include_directories(${TOPS-COMMON_INCLUDE})

add_subdirectory(tops_kar_pa)
add_subdirectory(tops_kar_pa_dev)

add_custom_target(generate-src-modules)
add_custom_command(TARGET generate-src-modules
        COMMAND python make_modules.py out=../tops_kar_pa_dev/lib in=input/registers.ods
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/Buser2)

add_custom_target(generate-robot-data)
add_custom_command(TARGET generate-robot-data
        COMMAND python3 run.py
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/robot-framework-generator)

add_custom_target(generate-vec-modules-data)
add_custom_command(TARGET generate-vec-modules-data
        COMMAND python make_list_dev.py out=../tops_kar_pa_dev/lib in=input/registers.ods
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/Buser2)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/scripts/ DESTINATION ${CMAKE_INSTALL_PREFIX}/scripts USE_SOURCE_PERMISSIONS COMPONENT core)

CREATE_UPLOAD_TARGETS(servertop)
