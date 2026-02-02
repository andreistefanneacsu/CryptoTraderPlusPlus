cmake_minimum_required(VERSION 3.26)

# FOARTE IMPORTANT: Proiectul trebuie să se numească "oop" pentru ca scripturile tale 
# de GitHub Actions să găsească fișierele (launcher.command caută acest nume).
project(oop)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Dezactivăm erorile care opresc build-ul din cauza SFML
set(WARNINGS_AS_ERRORS OFF)

include(FetchContent)
set(FETCHCONTENT_QUIET OFF)
set(FETCHCONTENT_UPDATES_DISCONNECTED ON)

# Setup SFML
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
    env_fixes.h
)

# Creăm executabilul
add_executable(${PROJECT_NAME} ${SURSE_FISIERE})

# Includem flag-urile de compilare DUPĂ ce am creat executabilul
include(cmake/CompilerFlags.cmake OPTIONAL)

###############################################################################
# Includes and Libraries
###############################################################################

target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_SOURCE_DIR}/include 
    ${CMAKE_SOURCE_DIR}
    generated/include
)

target_include_directories(${PROJECT_NAME} SYSTEM PRIVATE ${SFML_SOURCE_DIR}/include)
target_link_libraries(${PROJECT_NAME} PRIVATE sfml-graphics sfml-window sfml-system Threads::Threads)

if(UNIX AND NOT APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE X11)
endif()

###############################################################################
# Fix pentru erarea cu launcher.command (GitHub Actions)
###############################################################################

if(NOT DEFINED DESTINATION_DIR)
    set(DESTINATION_DIR "bin")
endif()

install(TARGETS ${PROJECT_NAME} DESTINATION ${DESTINATION_DIR})

if(APPLE)
    # Copiem launcher.command în folderul de build ca să îl găsească scriptul sed
    install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/launcher.command" DESTINATION ${DESTINATION_DIR})
endif()

# Creare directoare date
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
