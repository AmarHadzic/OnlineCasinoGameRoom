cmake_minimum_required(VERSION 3.24)
project(backend)

set(CMAKE_CXX_STANDARD 17)

# Include FetchContent module
include(FetchContent)

# Fetch and populate uWebSocket library
FetchContent_Declare(
        uwsrepo
        GIT_REPOSITORY https://github.com/uNetworking/uWebSockets.git
        GIT_TAG v20.58.0
        GIT_SHALLOW true)

FetchContent_GetProperties(uwsrepo)
if(NOT uwsrepo_POPULATED)
    FetchContent_Populate(uwsrepo)

    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(UWS_LTO 0)
    else()
        set(UWS_LTO 1)
    endif()

    find_program(MAKE_EXECUTABLE NAMES gmake make mingw32-make nmake REQUIRED)
    execute_process(
            COMMAND ${CMAKE_COMMAND} -E env WITH_LTO=${UWS_LTO} LIBUS_NO_SSL=1 ${MAKE_EXECUTABLE}
            WORKING_DIRECTORY ${uwsrepo_SOURCE_DIR}/uSockets
            OUTPUT_STRIP_TRAILING_WHITESPACE)

    add_library(uwslib STATIC IMPORTED GLOBAL)
    set_target_properties(uwslib PROPERTIES IMPORTED_LOCATION ${uwsrepo_SOURCE_DIR}/uSockets/uSockets.a)
    target_include_directories(uwslib INTERFACE ${uwsrepo_SOURCE_DIR}/src ${uwsrepo_SOURCE_DIR}/uSockets/src)
    target_compile_definitions(uwslib INTERFACE LIBUS_NO_SSL UWS_WITH_PROXY UWS_HTTPRESPONSE_NO_WRITEMARK)
    target_link_libraries(uwslib INTERFACE -lz)
endif()

# Enable testing with CTest
enable_testing()

# Add Google Test submodule
add_subdirectory(external/googletest)

# Include Google Test in your include paths
include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})

# Define your test executable
add_executable(mines_testing 
    Tests/MinesTests.cpp
    Mines/Model/MinesModel.cpp
        Player/Player.cpp # Include Player.cpp as it's referenced in MinesController
)

add_executable(roulette_testing
    Tests/RouletteTests.cpp
    Roulette/Model/BettingTable.cpp
    Roulette/Model/RouletteWheel.cpp
    Roulette/View/RouletteView.cpp
        Player/Player.cpp
)

add_executable(roulette_frontend_testing
    Tests/FrontendRouletteTests.cpp
    Roulette/Model/BettingTable.cpp
    Roulette/Model/RouletteWheel.cpp
    Roulette/View/RouletteView.cpp
        Player/Player.cpp
)
# Link Google Test libraries
target_link_libraries(mines_testing gtest gtest_main)
target_link_libraries(roulette_testing gtest gtest_main)
target_link_libraries(roulette_frontend_testing gtest gtest_main)


# Add your source files
set(SOURCE_FILES
        Player/Player.cpp
        Roulette/Model/RouletteWheel.cpp
        Roulette/Model/BettingTable.cpp
        Roulette/View/RouletteView.cpp
        Mines/Model/MinesModel.cpp
        MainDriver.cpp
        CoinFlip/CoinGame.cpp)

# Add the executable
add_executable(backend ${SOURCE_FILES} MainDriver.cpp)

# Link against the uWebSocket library
target_link_libraries(backend uwslib)

# Link against zlib
find_package(ZLIB REQUIRED)
target_link_libraries(backend ZLIB::ZLIB)
