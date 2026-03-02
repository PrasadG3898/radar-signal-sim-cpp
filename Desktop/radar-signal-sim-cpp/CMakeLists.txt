cmake_minimum_required(VERSION 3.16)

if(WIN32)
    if(DEFINED CMAKE_CXX_COMPILER AND EXISTS "${CMAKE_CXX_COMPILER}")
        get_filename_component(_bootstrap_cxx_bin "${CMAKE_CXX_COMPILER}" DIRECTORY)
    else()
        find_program(_bootstrap_cxx_exe NAMES c++.exe g++.exe)
        if(_bootstrap_cxx_exe)
            get_filename_component(_bootstrap_cxx_bin "${_bootstrap_cxx_exe}" DIRECTORY)
        endif()
    endif()

    if(DEFINED _bootstrap_cxx_bin)
        set(ENV{PATH} "${_bootstrap_cxx_bin};$ENV{PATH}")
    endif()
endif()

project(radar_signal_sim_cpp LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(radar_sim
    src/main.cpp
)

if(WIN32 AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    get_filename_component(_compiler_bin_dir "${CMAKE_CXX_COMPILER}" DIRECTORY)

    string(REPLACE ";" "\\;" _escaped_path "$ENV{PATH}")
    set(_launcher_env "PATH=${_compiler_bin_dir}\\;${_escaped_path}")
    set_property(
        TARGET radar_sim
        PROPERTY CXX_COMPILER_LAUNCHER
        "${CMAKE_COMMAND};-E;env;${_launcher_env}"
    )
    set_property(
        TARGET radar_sim
        PROPERTY CXX_LINKER_LAUNCHER
        "${CMAKE_COMMAND};-E;env;${_launcher_env}"
    )

    foreach(_dll IN ITEMS libstdc++-6.dll libgcc_s_seh-1.dll libwinpthread-1.dll)
        add_custom_command(
            TARGET radar_sim
            POST_BUILD
            COMMAND "${CMAKE_COMMAND}" -E copy_if_different
                    "${_compiler_bin_dir}/${_dll}"
                    "$<TARGET_FILE_DIR:radar_sim>/${_dll}"
            VERBATIM
        )
    endforeach()
endif()
