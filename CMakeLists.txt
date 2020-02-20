cmake_minimum_required (VERSION 3.8)

# Change to your mod name
project ("WBDLSample")

set (CMAKE_CXX_STANDARD 17)
set (CMAKE_CXX_STANDARD_REQUIRED ON)
set (CMAKE_CXX_EXTENSIONS ON)

# Override it in CMakeSettings.json or command line option
set (MOD_SDK_PATH $ENV{BedrockServerHome})

include_directories (${MOD_SDK_PATH}/Include/MinecraftHeaders ${MOD_SDK_PATH}/Include/Public)
link_directories (${MOD_SDK_PATH}/Lib)

find_package (SQLiteCpp CONFIG REQUIRED)
find_package (Boost 1.72.0 REQUIRED)
find_package (sqlite3 CONFIG REQUIRED)
find_package (yaml-cpp CONFIG REQUIRED)

function (import_library name location)
    add_library (${name} UNKNOWN IMPORTED)
    set_target_properties (${name} PROPERTIES IMPORTED_LOCATION ${location})
endfunction ()

import_library (ModLoader ${MOD_SDK_PATH}/Lib/Chakra.lib)
import_library (BedrockServer ${MOD_SDK_PATH}/Lib/bedrock_server_mod.lib)

function (install_pdb name)
    install (FILES $<TARGET_PDB_FILE:${name}> DESTINATION Lib)
endfunction ()

function (add_mod name)
    file (GLOB_RECURSE srcs CONFIGURE_DEPENDS *.cpp)
    add_library (${name} SHARED ${srcs})
    target_compile_definitions (${name} PRIVATE MODNAME=${name})
    target_link_libraries (${name} Boost::headers yaml-cpp ModLoader BedrockServer Base ${ARGN})
    install (TARGETS ${name} RUNTIME DESTINATION Mods ARCHIVE DESTINATION Lib)
    install_pdb (${name})
    set_target_properties (${name} PROPERTIES FOLDER Mods)
endfunction ()

# Change the name or append more
add_subdirectory (SampleMod)