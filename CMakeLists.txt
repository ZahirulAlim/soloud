cmake_minimum_required(VERSION 3.10)
project(soloud LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- SoLoud root ---
set(SOLOUD_DIR ${CMAKE_CURRENT_SOURCE_DIR})

# --- Collect SoLoud core + filter + wav + ALSA sources (.cpp + .c) ---
file(GLOB_RECURSE SOLOUD_CORE
    "${SOLOUD_DIR}/src/core/*.cpp"
)
file(GLOB_RECURSE SOLOUD_FILTER
    "${SOLOUD_DIR}/src/filter/*.cpp"
)
file(GLOB_RECURSE SOLOUD_WAV
    "${SOLOUD_DIR}/src/audiosource/wav/*.cpp"
    "${SOLOUD_DIR}/src/audiosource/wav/*.c"
)
file(GLOB_RECURSE SOLOUD_ALSA
    "${SOLOUD_DIR}/src/backend/alsa/*.cpp"
)

set(SOLOUD_SOURCES
    ${SOLOUD_CORE}
    ${SOLOUD_FILTER}
    ${SOLOUD_WAV}
    ${SOLOUD_ALSA}
)

# --- Create executable ---
add_library(soloud STATIC ${SOLOUD_SOURCES})

# --- Include SoLoud headers ---
target_include_directories(soloud PUBLIC
    "${SOLOUD_DIR}/include"
)

# --- Enable ALSA backend ---
target_compile_definitions(soloud PRIVATE WITH_ALSA=1)

# --- Link pthread/dl/ALSA ---
if(UNIX AND NOT APPLE)
    find_package(Threads REQUIRED)
    find_package(ALSA REQUIRED)
    target_link_libraries(soloud PRIVATE Threads::Threads dl ${ALSA_LIBRARIES})
    target_include_directories(soloud PRIVATE ${ALSA_INCLUDE_DIRS})
endif()

message(STATUS "Building SoLoud with ALSA backend and WAV+OGG support")
