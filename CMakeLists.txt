cmake_minimum_required(VERSION 3.16)
project(calculator VERSION 0.1 LANGUAGES CXX)

#----Set C++ standard----
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Use the standard project setup to handle various platform-specific settings
# qt_standard_project_setup()
set(CMAKE_AUTOMOC ON) 
set(CMAKE_AUTORCC ON) 
set(CMAKE_AUTOUIC ON)

#----Include directories----
set(INCLUDE_DIRECTORIES src/include)

#----Files----
set(LIB_FILES src/lib/mathLib.cpp)
set(SRC_FILES src/main.cpp src/mainwindow.cpp)
set(UI_FILES src/mainwindow.ui)
set(TEST_FILES src/mainMathFunctionTests.cpp src/advancedMathFunctionTests.cpp)
set(IN_FILES src/include/main.h src/include/mainwindow.h src/include/ui_mainwindow.h)

#----Find Qt package----
set(Qt6_DIR "/path/to/qt6/lib/cmake/Qt6")
find_package(Qt6 COMPONENTS Widgets REQUIRED)

#----Main Executable----
qt_add_executable(calculator
    MANUAL_FINALIZATION
    ${SRC_FILES}
    ${LIB_FILES}
    ${UI_FILES}
    ${IN_FILES}
)

#----Link libraries----
target_link_libraries(calculator PRIVATE Qt6::Widgets)

#----Include directories----
target_include_directories(calculator PRIVATE ${INCLUDE_DIRECTORIES})

set_target_properties(calculator PROPERTIES
    WIN32_EXECUTABLE ON
    MACOSX_BUNDLE ON
)

#----GTESTS----
include(FetchContent)
FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/release-1.11.0.zip
)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

enable_testing()

#----Testing Executable----
add_executable(runTests ${TEST_FILES} ${LIB_FILES})
target_include_directories(runTests PRIVATE ${INCLUDE_DIRECTORIES})
target_link_libraries(runTests PRIVATE gtest_main)
include(GoogleTest)
gtest_discover_tests(runTests)

#----Profiling Executable----
add_executable(stddev src/stddev.cpp src/lib/mathLib.cpp)
target_compile_options(stddev PRIVATE -pg -O0 -g)
target_link_options(stddev PRIVATE -pg)

# ----DOXYGEN----
find_package(Doxygen REQUIRED)
if(Doxygen_FOUND)
    add_custom_target(docs ALL
        COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/src/Doxyfile
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        COMMENT "Generating API documentation with Doxygen"
        VERBATIM
    )
else()
    message(STATUS "Doxygen not found, skipping documentation")
endif()

#----Finalization for Qt----
if (QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(calculator)
endif()

# Install instructions for calculator
install(TARGETS calculator DESTINATION /usr/bin COMPONENT Calculator)
install(DIRECTORY src/ DESTINATION /opt/calculator COMPONENT Calculator)
install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/calculator.desktop" DESTINATION /usr/share/applications COMPONENT Calculator)
install(FILES src/calculator.png DESTINATION /usr/share/icons/hicolor/512x512/apps COMPONENT Calculator)

# Install instructions for stddev
install(TARGETS stddev DESTINATION /usr/bin COMPONENT StdDev)
install(DIRECTORY src/ DESTINATION /opt/stddev COMPONENT StdDev)
install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/stddev.desktop" DESTINATION /usr/share/applications COMPONENT StdDev)
install(FILES src/stddev.png DESTINATION /usr/share/icons/hicolor/512x512/apps COMPONENT StdDev)

# CPack settings
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_VENDOR "DIY")
set(CPACK_PACKAGE_CONTACT "xcomovj00@stud.fit.vutbr.cz")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Simple calculator and stddev applications")
set(CPACK_PACKAGE_VERSION "1.0-Alpha")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "${PROJECT_NAME}")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Jaroslava Comová <${CPACK_PACKAGE_CONTACT}>")
set(CPACK_GENERATOR DEB)
set(CPACK_OUTPUT_FILE_PREFIX "${CMAKE_SOURCE_DIR}/src")
set(CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/install")
set(CPACK_DEBIAN_COMPONENT_INSTALL ON)
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS YES)
set(CPACK_COMPONENTS_ALL Calculator StdDev)

include(CPack)
#----Commands for Makefile----

add_custom_target(run
    COMMAND $<TARGET_FILE:calculator>
    DEPENDS calculator
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Running the application"
)

add_custom_target(profile
    DEPENDS stddev
)

add_custom_target(clean-all
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}
    COMMENT "Removing all files in the build directory"
)


add_custom_target(bundle-repo
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/repo
    COMMAND git bundle create ${CMAKE_BINARY_DIR}/repo/repo.bundle --all
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMENT "Creating a bundle of the entire git repository including the history"
)

add_custom_target(pack 
    COMMAND ${CMAKE_COMMAND} -E remove_directory xpylypd00_xvesela00_xkremee00_xcomovj00
    COMMAND ${CMAKE_COMMAND} -E make_directory xpylypd00_xvesela00_xkremee00_xcomovj00
    COMMAND ${CMAKE_COMMAND} -E make_directory xpylypd00_xvesela00_xkremee00_xcomovj00/doc
    COMMAND ${CMAKE_COMMAND} -E make_directory xpylypd00_xvesela00_xkremee00_xcomovj00/install
    COMMAND ${CMAKE_COMMAND} -E make_directory xpylypd00_xvesela00_xkremee00_xcomovj00/repo

    # Kopírování repozitáře
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/.git xpylypd00_xvesela00_xkremee00_xcomovj00/repo/.git

    # Kopírování instalátorů
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/*.deb xpylypd00_xvesela00_xkremee00_xcomovj00/install/
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/src/preinst.sh xpylypd00_xvesela00_xkremee00_xcomovj00/install/

    # Kopírování dokumentace
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/docs/ xpylypd00_xvesela00_xkremee00_xcomovj00/doc/

    # Balení do zip
    COMMAND ${CMAKE_COMMAND} -E tar "cvf" "${CMAKE_SOURCE_DIR}/xpylypd00_xvesela00_xkremee00_xcomovj00.zip" --format=zip xpylypd00_xvesela00_xkremee00_xcomovj00
    DEPENDS bundle-repo
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Packing the project into a zip file"
)
