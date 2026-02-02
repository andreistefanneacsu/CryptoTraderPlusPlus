cmake_minimum_required(VERSION 3.26)

project(CryptoTrader)

# C++ Standard configuration
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Include custom cmake modules if they exist
include(cmake/Options.cmake OPTIONAL)
include(cmake/CompilerFlags.cmake OPTIONAL)

###############################################################################
# External Dependencies (SFML & Threads)
###############################################################################

include(FetchContent)
set(FETCHCONTENT_QUIET OFF)
set(FETCHCONTENT_UPDATES_DISCONNECTED ON)

# SFML 2.6.1 setup
FetchContent_Declare(
    SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG        2.6.1
    GIT_SHALLOW    1
)
FetchContent_MakeAvailable(SFML)

find_package(Threads REQUIRED)
if(NOT APPLE AND UNIX)
    find_package(X11)
endif()

###############################################################################
# Source Files
###############################################################################

set(SURSE_FISIERE
    main.cpp
    # Monede
    src/Monede/Moneda.cpp
    src/Monede/MonedaDeBaza.cpp
    src/Monede/MonedaStabila.cpp
    src/Monede/MonedaMeme.cpp
    src/Monede/MonedaFiat.cpp
    src/Monede/Altcoin.cpp
    # Piata
    src/Piata/Piata.cpp
    src/Piata/AnalizatorPiata.cpp
    # Utilizator
    src/Utilizator/Utilizator.cpp
    src/Utilizator/Portofel.cpp
    src/Utilizator/ManagerUtilizatori.cpp
    # Tranzactii
    src/Tranzactii/Tranzactie.cpp
    # Utilitare
    src/Utilitare/InterfataTerminal.cpp
    src/Utilitare/ManagerFisiere.cpp
    src/Utilitare/CititorFisier.cpp
    # Tests
    tests/TestRunner.cpp
    # Extra helper from original boilerplate
    generated/src/Helper.cpp
)

add_executable(${PROJECT_NAME} ${SURSE_FISIERE})

###############################################################################
# Includes and Libraries
###############################################################################

# Include local directories
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_SOURCE_DIR}/include 
    ${CMAKE_SOURCE_DIR}
    generated/include
)

# Include SFML headers as SYSTEM to suppress external warnings
target_include_directories(${PROJECT_NAME} SYSTEM PRIVATE ${SFML_SOURCE_DIR}/include)

# Link Libraries
target_link_libraries(${PROJECT_NAME} PRIVATE 
    sfml-graphics 
    sfml-window 
    sfml-system 
    Threads::Threads
)

if(UNIX AND NOT APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE X11)
endif()

###############################################################################
# Compiler Options
###############################################################################

if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /WX /EHsc)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall -Wextra -Wpedantic -Werror -Wconversion -Wsign-conversion -Wno-unused-parameter)
endif()

###############################################################################
# Post-Build Actions (Directories & Assets)
###############################################################################

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/utilizatori
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/piata
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/tranzactii
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/logs
    COMMENT "Creating data directories in build folder..."
)

# Windows DLL copy for OpenAL (SFML Audio)
if (WIN32 AND SFML_BUILD_AUDIO)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy_if_different
         ${SFML_SOURCE_DIR}/extlibs/bin/x64/openal32.dll
         $<TARGET_FILE_DIR:${PROJECT_NAME}>
    )
endif()

# File Copying (requires cmake/CopyHelper.cmake to exist)
include(cmake/CopyHelper.cmake OPTIONAL)
if(COMMAND copy_files)
    copy_files(FILES tastatura.txt)
endif()

###############################################################################
# Testing Target
###############################################################################

add_custom_target(run-test
        COMMAND $<TARGET_FILE:${PROJECT_NAME}>
        DEPENDS ${PROJECT_NAME}
        COMMENT "Running automated tests..."
)
