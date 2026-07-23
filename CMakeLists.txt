cmake_minimum_required(VERSION 3.15)

project(papi CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if (UNIX)
    # Endstone is built against libc++ for ABI compatibility with BDS (see its
    # own CMakeLists.txt). That flag only applies inside endstone's own
    # FetchContent subdirectory scope, so it must be set here too or our own
    # _papi target falls back to the system libstdc++, which is too old to
    # even have <format> on the manylinux image and breaks the build.
    add_compile_options(-stdlib=libc++)
    add_link_options(-stdlib=libc++)
endif ()

include(FetchContent)
FetchContent_Declare(
        endstone
        GIT_REPOSITORY https://github.com/EndstoneMC/endstone.git
        GIT_TAG v0.11.6
)
FetchContent_MakeAvailable(endstone)

if (NOT CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
    add_library(endstone_papi INTERFACE)
    target_include_directories(endstone_papi INTERFACE include)
    target_link_libraries(endstone_papi INTERFACE endstone::endstone)
    return()
endif ()

execute_process(
        COMMAND "${Python_EXECUTABLE}" -m pybind11 --cmakedir
        OUTPUT_STRIP_TRAILING_WHITESPACE OUTPUT_VARIABLE pybind11_ROOT)
find_package(Python COMPONENTS Interpreter Development.Module REQUIRED)
find_package(pybind11 CONFIG REQUIRED)

pybind11_add_module(_${PROJECT_NAME} endstone_papi/_papi.cpp)
target_include_directories(_${PROJECT_NAME} PRIVATE include)
target_link_libraries(_${PROJECT_NAME} PRIVATE endstone::endstone)
install(TARGETS _${PROJECT_NAME} DESTINATION "endstone_papi" COMPONENT python)
install(DIRECTORY include/ DESTINATION "endstone_papi/include" COMPONENT python)

add_subdirectory(examples/cpp)
