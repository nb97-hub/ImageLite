# ***********************  COMMANDS ***********************
# cmake -S . -B build #creates and fills the build directory (which contains Building configs) using as CMakeLists.txt Source the one at . (here). By default it uses MSVC as compiler.
# cmake -S . -B build -G "MinGW Makefiles" -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ #uses GCC UCRT64 as compiler 
# cmake -B build --fresh -G "MinGW Makefiles" -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ #deletes and rebuilds the build direcotry using GCC as compiler
# *********************************************************

# Min CMAKE version required
cmake_minimum_required(VERSION 3.15)

# Assigns the project name
project(ImageLite
        VERSION 1.0.2
        DESCRIPTION "Simple image processing library and tool")


# BUILD AUTOMATION
# Enable testing if the option is set
option(IMAGELITE_BUILD_TESTS "Build the test suite" ON)

# BUILD AUTOMATION
# Activate CTest support
if(IMAGELITE_BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# ADVANCED
set(CMAKE_CXX_STANDARD 17)              # all target will use c++17
set(CMAKE_CXX_STANDARD_REQUIRED ON)     
set(CMAKE_CXX_EXTENSIONS OFF)

# Save the binaries in build/bin by setting this env variable CMAKE_RUNTIME_OUTPUT_DIRECTORY
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Lets create a variable for the sample_images folder
set(SAMPLE_IMAGES_DIR ${PROJECT_SOURCE_DIR}/sample_images)


# ADVANCED
# LETS CREATE A FUNCTION TO BE USED BY THE SUB-CMAKELISTS
# IT'S IMPORTANT THAT IT IS DEFINED BEFORE THE add_subdirectory. Otherwise the subdirectories wouldn't find it
#function(func_name param1 parm2 [hidden]ARGN)
function(add_imagelite_module name)
        add_library(${name} STATIC ${ARGN}) # ${ARGN} is a series of parameters passed at the end of the function. It's useful because we want to specify MULTIPLE source files (see CMakeLists.txt in libs/core).

        #message(WARNING "CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}/include")
        target_include_directories(${name}
                PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

        # ADVANCED
        # better than below "set_directory_properties" because it applies only to targets that call this
        # Add compiler-specific warning options - ONLY for this target
        target_compile_options(${name} PRIVATE
        $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra>                 #Note: $<expression>
                                                                # the ":" means $<condition:result> -> $<CXX_COMPILER_ID:GNU> checks if (CXX_COMPILER_ID == GNU)
        $<$<CXX_COMPILER_ID:Clang>:-Wall -Wextra>
        $<$<CXX_COMPILER_ID:MSVC>:/W4>
        )

        # ADVANCED
        # better than below "set_directory_properties" because it applies only to targets that call this
        # Add platform-specific definitions - ONLY for this target
        target_compile_definitions(${name} PRIVATE
        $<$<PLATFORM_ID:Windows>:IMAGELITE_WINDOWS>
        $<$<PLATFORM_ID:Linux>:IMAGELITE_LINUX>
        $<$<PLATFORM_ID:Darwin>:IMAGELITE_MACOS>
        )

        # ADVANCED
        # Add DEBUG-specific extra warnings and debug info
        target_compile_options(${name} PRIVATE
        $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:GNU>>:-Wpedantic -g>
        $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:Clang>>:-Wpedantic -g>
        $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:MSVC>>:/Zi /Od>
        )

        # ADVANCED
        # Add RELEASE-specific optimizations
        target_compile_options(${name} PRIVATE
        $<$<AND:$<CONFIG:Release>,$<OR:$<CXX_COMPILER_ID:GNU>,$<CXX_COMPILER_ID:Clang>>>:-O3>
        $<$<AND:$<CONFIG:Release>,$<CXX_COMPILER_ID:MSVC>>:/O2>
        )
endfunction()

# ADVANCED: set specific properties for current directory and subdirectories 
# Warning: These flags are not supported by MSVC.
# Only use this on projects targeting GCC or Clang.
# Since it targets everything, we'll use a target specific property setter
# set_directory_properties(PROPERTIES
# #       COMPILE_OPTIONS "-Wall;-Wextra"      # these are good but only works for gcc and clang. let's use instead generator expression
#         COMPILE_OPTIONS "$<$<OR:$<CXX_COMPILER_ID:GNU>,$<CXX_COMPILER_ID:Clang>>:-Wall;-Wextra>;$<$<CXX_COMPILER_ID:MSVC>:/W4>"
# )

# ADVANCED
# the same would have worked with the use of MACROS.
# Difference between MACRO and FUNCTION:        - variables inside the functions are local - do not affect anything outside.
#                                               - variables leak into parent scope
#                                               - NOTE: variables are the ones defined into set(...)
# macro(add_imagelite_module name use_stb_external)
#         add_library(${name} STATIC ${ARGN}) # ${ARGN} is a series of parameters passed at the end of the function. It's useful because we want to specify MULTIPLE source files (see CMakeLists.txt in libs/core).

#         target_include_directories(${name}
#                 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
        
#         if(${use_stb_external})
#                 target_include_directories(${name} PRIVATE ${PROJECT_SOURCE_DIR}/external/stb)
#         endif()
# macro()


# In this way it can findthe other CMakeLists.txt that are needed to be linked against the final executable.
add_subdirectory(libs/core)
add_subdirectory(libs/filters)

# creates an executable called "imagelite" using as SOURCES the one stated 
add_executable(imagelite apps/cli/src/main.cpp)

set_target_properties(imagelite PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/bin/debug"
        RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/bin/release"
        )       

# DEPENDENCES_INTEGRATION
# find_library helps searching for a library in the system when these libs are not managed by CMake directly
# find_library(
#       <variable>          # Variable where to store the result
#       <library_name>      # Name of the library to find
#       [PATHS paths..]     # Opt: specific paths where to search
#       [NO_DEFAULT_PATH]   # Opt: do not use default search paths
#       [REQUIRED]          # Opt: error if not found
# )
#
# find_package helps searching for a package (set of libs) in the system when these pkgs are not managed by CMake directly
# find_package(
#       <PackageName>           # Name of the package to find -> some variables are set by CMake after this command execution (ex. package=PNG => PNG_FOUND, PNG_INCLUDE_DIRS, PNG_LIBRARIES, PNG_VERSION)
#       [version]               # Opt: required version
#       [NO_DEFAULT_PATH]       # Opt: do not use default search paths
#       [REQUIRED]              # Opt: error if not found
#       [COMPONENTS comp1...]   # Opt: specific components to find
# )
#
# BUILD AUTOMATION
# Let's use Python to create an header file (python script in scripts/generate_formats.py)
find_package(Python REQUIRED COMPONENTS Interpreter)    
message(STATUS "Python executable: ${Python_EXECUTABLE}")
#Create the output directory for the generated header
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/include/imagelite)
# Generate formats.h
add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/include/imagelite/formats.h
    COMMAND ${Python_EXECUTABLE} ${CMAKE_SOURCE_DIR}/scripts/generate_formats.py
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Generating supported image formats header..."
    VERBATIM
)
# Create a target for the generated file
add_custom_target(generate_formats
    DEPENDS ${CMAKE_BINARY_DIR}/include/imagelite/formats.h
)
# Ensure the header is generated before compilation
add_dependencies(imagelite generate_formats)
#
target_include_directories(imagelite
        PRIVATE
            ${CMAKE_BINARY_DIR}/include # include path for generated header
)




add_custom_command(
        OUTPUT "${CMAKE_BINARY_DIR}/build_info.txt"    # Generate a file called build_info.txt in build/
        COMMAND ${CMAKE_COMMAND} -E echo "ImageLite Build Information" >> "${CMAKE_BINARY_DIR}/build_info.txt" # Write into build_info.txt
                                #-E stands for execute
        COMMAND ${CMAKE_COMMAND} -E echo "CMake Version: ${CMAKE_VERSION}" >> "${CMAKE_BINARY_DIR}/build_info.txt"
        COMMAND ${CMAKE_COMMAND} -E echo "C++ Compiler: ${CMAKE_CXX_COMPILER_VERSION}" >> "${CMAKE_BINARY_DIR}/build_info.txt"
        COMMAND ${CMAKE_COMMAND} -E echo "System: ${CMAKE_SYSTEM_NAME} ${CMAKE_SYSTEM_VERSION}" >> "${CMAKE_BINARY_DIR}/build_info.txt"
        COMMAND ${CMAKE_COMMAND} -E echo "Project Version: ${PROJECT_VERSION}" >> "${CMAKE_BINARY_DIR}/build_info.txt"
        COMMAND ${CMAKE_COMMAND} -E echo "Project Description: ${PROJECT_DESCRIPTION}" >> "${CMAKE_BINARY_DIR}/build_info.txt"
    
        COMMENT "Creating detailed build information file"
        VERBATIM        # Ensures Portability of commands
)

add_custom_target(build_info
        ALL # ensures that build_info (this target) happens automatically every time you build the project
        DEPENDS ${CMAKE_BINARY_DIR}/build_info.txt)

# With this addition we will be able to specify --target run when building the project and the program imagelite.exe will be executed with the specified parameters
# How to call it: cmake --build build --target run
add_custom_target(run
        COMMAND imagelite.exe blur ${SAMPLE_IMAGES_DIR}/input.jpg ${SAMPLE_IMAGES_DIR}/output.jpg 5
        DEPENDS imagelite                                       # We use the DEPENDS to make sure the cli (imagelite.exe) is built before executing the COMMAND
        WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}     # WORKING_DIRECTORY specifies from where we execute the code
        COMMENT "Running the CLI tool with sample arguments"    # A comment printed out before the COMMAND is executed
)


# PRE-build command -> runs before imagelite is built
add_custom_command(
    TARGET imagelite
    PRE_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/README.md" "${CMAKE_BINARY_DIR}/bin/README.md"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/temp"

    COMMENT "Setting up build environment"
)

# POST-build command -> runs after imagelite is built

add_custom_command(
    TARGET imagelite
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E rm -rf "${CMAKE_BINARY_DIR}/temp"
    COMMENT "Post build clean up"
)

# ADVANCED
# CLI-specific compile options - ONLY affects this executable
target_compile_options(imagelite PRIVATE
    # Extra strict flags for our application code
    $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra -Wpedantic -Wconversion -Wshadow>
    $<$<CXX_COMPILER_ID:Clang>:-Wall -Wextra -Wpedantic -Wconversion -Wshadow>
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX>
)

# ADVANCED
# Add debug-specific options
target_compile_options(imagelite PRIVATE
    $<$<AND:$<CONFIG:Debug>,$<OR:$<CXX_COMPILER_ID:GNU>,$<CXX_COMPILER_ID:Clang>>>:-g -O0>
    $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:MSVC>>:/Zi /Od /RTC1>
)

# ADVANCED
# Add release-specific options
target_compile_options(imagelite PRIVATE
    $<$<AND:$<CONFIG:Release>,$<OR:$<CXX_COMPILER_ID:GNU>,$<CXX_COMPILER_ID:Clang>>>:-O3>
    $<$<AND:$<CONFIG:Release>,$<CXX_COMPILER_ID:MSVC>>:/O2>
)

# Tells CMake that imagelite.exe depends on those libraries
target_link_libraries(imagelite
        PRIVATE
            imagelite_core
            imagelite_filter
)

# NOT NEEDED, the PUBLIC specifier in the libraries already provides the paths.
#
#
# gives imagelite the path of where the headers are. It's different from the one in the subdirectory since that was needed to compile the library itself (example: core.cpp uses include imagelite/core.h)
# target_include_directories(imagelite
#         PRIVATE
#             libs/core/include
#             libs/filters/include)

# Platform detection
message(STATUS "Detecting platform...")
if(WIN32)
    message(STATUS "Platform: Windows")
elseif(APPLE)
    message(STATUS "Platform: macOS, iOS, tvOS, visionOS or watchOS")
elseif(UNIX AND NOT APPLE)
    message(STATUS "Platform: Linux")
else()
    message(STATUS "Platform: Unknown")
endif()

# BUILD AUTOMATION
# Set default build type if not specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose build type: Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif()
# BUILD AUTOMATION
# Print the current build type
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")