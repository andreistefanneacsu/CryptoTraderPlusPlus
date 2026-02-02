cmake_minimum_required(VERSION 3.26)
project(CryptoTrader)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 1. Setup Options first (before targets)
include(cmake/Options.cmake OPTIONAL)

###############################################################################
# Dependencies
###############################################################################

include(FetchContent)
set(FETCHCONTENT_QUIET OFF)
set(FETCHCONTENT_UPDATES_DISCONNECTED ON)

FetchContent_Declare(
    SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG        2.6.1
    GIT_SHALLOW    1
)

# Move this up! Targets sfml-graphics, etc., are created here.
FetchContent_MakeAvailable(SFML)

find_package(Threads REQUIRED)
if(NOT APPLE AND UNIX)
    find_package(X11)
endif()

###############################################################################
# Create the Main Target
###############################################################################

set(SURSE_FISIERE
    main.cpp
    src/Monede/Moneda.cpp
    src/Monede/MonedaDeBaza.cpp
    src/Monede/MonedaStabila.cpp
    src/Monede/MonedaMeme.cpp
    src/Monede/MonedaFiat.cpp
    src/Monede/Altcoin.cpp
    src/Piata/Piata.cpp
    src/Piata/AnalizatorPiata.cpp
    src/Utilizator/Utilizator.cpp
    src/Utilizator/Portofel.cpp
    src/Utilizator/ManagerUtilizatori.cpp
    src/Tranzactii/Tranzactie.cpp
    src/Utilitare/InterfataTerminal.cpp
    src/Utilitare/ManagerFisiere.cpp
    src/Utilitare/CititorFisier.cpp
    tests/TestRunner.cpp
    generated/src/Helper.cpp
)

# Now the target "CryptoTrader" exists!
add_executable(${PROJECT_NAME} ${SURSE_FISIERE})

###############################################################################
# Apply Flags and Includes
###############################################################################

# 2. NOW include the flags, because the targets (CryptoTrader & SFML) exist
include(cmake/CompilerFlags.cmake OPTIONAL)

target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_SOURCE_DIR}/include 
    ${CMAKE_SOURCE_DIR}
    generated/include
)

target_include_directories(${PROJECT_NAME} SYSTEM PRIVATE ${SFML_SOURCE_DIR}/include)

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
# Post-Build & Assets
###############################################################################

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/utilizatori
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/piata
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/tranzactii
    COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${PROJECT_NAME}>/date/logs
)

include(cmake/CopyHelper.cmake OPTIONAL)
if(COMMAND copy_files)
    copy_files(FILES tastatura.txt)
endif()

add_custom_target(run-test
        COMMAND $<TARGET_FILE:${PROJECT_NAME}>
        DEPENDS ${PROJECT_NAME}
)
