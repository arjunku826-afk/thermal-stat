cmake_minimum_required(VERSION 3.22)
project(thermal_states LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)

FetchContent_Declare(
    sweetloader_android
    GIT_REPOSITORY https://github.com/ChimeraAnt-DEV/preloader-android.git
    GIT_TAG 0.2.2)
FetchContent_MakeAvailable(sweetloader_android)

# Produces libthermal_states.so, which manifest.json's "entry" points at.
add_library(thermal_states SHARED src/main.cpp)
target_link_libraries(thermal_states PRIVATE sweetloader dl)
