# add_library(imagelite_filter STATIC src/filters.cpp)

# target_include_directories(imagelite_filter 
#         PUBLIC  ${CMAKE_CURRENT_SOURCE_DIR}/include)

add_imagelite_module(imagelite_filter src/filters.cpp)

# Tells CMake that filters.cpp needs imagelite_core lib. This is due to #include "imagelite/image.h" in filters.h
target_link_libraries(imagelite_filter 
        PUBLIC imagelite_core)
