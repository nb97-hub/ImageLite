# DEPENDENCIES_INTEGRATION
include(FetchContent)   # Used to pull automatichally the content from repository

# DEPENDENCIES_INTEGRATION
# creates:  - "stb_SOURCE_DIR" for code download
#           - "stb_BINARY_DIR" for build output
#           - "stb_POPULATED" bool flag for population status
FetchContent_Declare(
    stb                                                 #local identifier
    GIT_REPOSITORY https://github.com/nothings/stb.git  #repo
    GIT_TAG master                                      #version
)

# DEPENDENCIES_INTEGRATION - Coarse control
FetchContent_MakeAvailable(stb) # Coarse control
add_library(stb_image INTERFACE)
target_include_directories(stb_image INTERFACE ${stb_SOURCE_DIR})

# DEPENDENCIES_INTEGRATION - Fine grain control
# FetchContent_GetProperties(stb) # Retrieve the properties without populating and fills stb_POPULATED

# if(NOT stb_POPULATED)
#     FetchContent_Populate(stb)

#     # DEPENDENCIES_INTEGRATION
#     add_library(stb_image INTERFACE)
#     target_include_directories(stb_image INTERFACE ${stb_SOURCE_DIR})
# endif()



# # Let's create a static library with source files to compile/link. 
# # add_library(LibName PUBLIC sources)
# add_library(imagelite_core STATIC 
#         src/core.cpp
#         src/image.cpp)
#
# # Let's expose the include direcotry as a PUBLIC -> everything can find it.
# # Having the include after the add_library is ok, because the CMakeLists only creates a building systems -> configures everything to then later being compiled/built.
# # target_include_directories( LibName PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/path) #CMAKE_CURRENT_SOURCE_DIR is the full pathcurrently processed by CMake an it is automatically set.
# target_include_directories(imagelite_core
#         PUBLIC  ${CMAKE_CURRENT_SOURCE_DIR}/include
#         PRIVATE ${PROJECT_SOURCE_DIR}/external/stb)


# ADVANCED
# Function call for function((add_imagelite_module) name use_stb_external)
add_imagelite_module(imagelite_core 
            src/core.cpp
            src/image.cpp
            src/platform_info.cpp)

# DEPENDENCIES_INTEGRATION
target_link_libraries(imagelite_core PRIVATE stb_image)

# ADVANCED
# enables position-independent code useful for shard libraries and set symbol visibility
set_target_properties(imagelite_core PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    CXX_VISIBILITY_PRESET hidden
)

# ADVANCED
get_target_property(VISIBILITY imagelite_core CXX_VISIBILITY_PRESET)
message("Visibility for imagelite_core: ${VISIBILITY}")


# ADVANCED
# defines IMAGELITE_DEBUG/IMAGELITE_ENABLE_LOGGING/IMAGELITE_RELEASE based on build type
target_compile_definitions(imagelite_core PRIVATE
        $<$<CONFIG:Debug>:IMAGELITE_DEBUG>
        $<$<CONFIG:Debug>:IMAGELITE_ENABLE_LOGGING>
        $<$<CONFIG:Release>:IMAGELITE_RELEASE>
)

# Set platform-specific definitions
if(WIN32)
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_WINDOWS=1)
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_NAME="Windows")
elseif(APPLE)
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_MACOS=1)
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_NAME="macOS")
elseif(UNIX)
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_LINUX=1)
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_NAME="Linux")
else()
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_UNKNOWN=1)
    target_compile_definitions(imagelite_core PUBLIC IL_PLATFORM_NAME="Unknown")
endif()