# Copyright (C) 2022 The Qt Company Ltd.
# SPDX-License-Identifier: BSD-3-Clause

cmake_minimum_required(VERSION 3.16)
project(localstorage LANGUAGES CXX)

if(NOT DEFINED INSTALL_EXAMPLESDIR)
    set(INSTALL_EXAMPLESDIR "examples")
endif()
set(MyApp "localstorageexample")
set(INSTALL_EXAMPLEDIR "${INSTALL_EXAMPLESDIR}/quick/localstorage")

find_package(Qt6 REQUIRED COMPONENTS Core Gui Qml Quick)

qt_standard_project_setup(REQUIRES 6.5)

qt_add_executable(localstorageexample
    WIN32
    MACOSX_BUNDLE
    main.cpp
)

qt_add_qml_module(localstorageexample
    URI localstorage
    QML_FILES
        "Database.js"
        "Header.qml"
        "MyDelegate.qml"
        "MyModel.qml"
        "localstorage.qml"
    )

target_link_libraries(localstorageexample PRIVATE
    Qt6::Core
    Qt6::Gui
    Qt6::Qml
    Qt6::Quick
)

# install(TARGETS localstorageexample
#     RUNTIME DESTINATION "${INSTALL_EXAMPLEDIR}"
#     BUNDLE DESTINATION "${INSTALL_EXAMPLEDIR}"
#     LIBRARY DESTINATION "${INSTALL_EXAMPLEDIR}"
# )
include(GNUInstallDirs)
install(TARGETS ${MyApp}
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
qt_generate_deploy_app_script(
     TARGET ${MyApp}
     OUTPUT_SCRIPT deploy_script
     NO_UNSUPPORTED_PLATFORM_ERROR)

install(SCRIPT ${deploy_script})
