# to run in terminal:
#       run all:                                ctest --test-dir build
#       run all verbose:                        ctest --test-dir build -V
#       run specific (Core in the name):        ctest --test-dir build -R Core
#       run specific (only fast opt):           ctest --test-dir build -L fast
# Find testing framework
find_package(GTest REQUIRED)

# Core tests
add_executable(core_tests
    core_tests/test_image.cpp
    core_tests/test_core.cpp
)

target_link_libraries(core_tests
    PRIVATE
        imagelite_core
        GTest::GTest
        GTest::Main
)

# Filters tests
add_executable(filters_tests
    filters_tests/test_filters.cpp
)

target_link_libraries(filters_tests
    PRIVATE
        imagelite_core
        imagelite_filter
        GTest::GTest
        GTest::Main
)

# Register the tests
add_test(NAME CoreTests COMMAND core_tests)
add_test(NAME FiltersTests COMMAND filters_tests)

# Sets a 10-second timeout for our core tests
set_tests_properties(CoreTests PROPERTIES
    TIMEOUT 10  # Fail if test takes more than 10 seconds
)

# Add labels to tests
set_tests_properties(CoreTests PROPERTIES
    LABELS "core;fast"
)

set_tests_properties(FiltersTests PROPERTIES
    LABELS "filters"
)