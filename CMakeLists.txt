# Copyright (C) 2022 The Qt Company Ltd.
# SPDX-License-Identifier: BSD-3-Clause

cmake_minimum_required(VERSION 3.16)
project(qmllocalstorage LANGUAGES CXX)

if(NOT DEFINED INSTALL_EXAMPLESDIR)
    set(INSTALL_EXAMPLESDIR "examples")
endif()
set(MyApp "localstorageexample")
set(INSTALL_EXAMPLEDIR "${INSTALL_EXAMPLESDIR}/quick/localstorage")

set(QT_ENABLE_VERBOSE_DEPLOYMENT 1)
find_package(Qt6 REQUIRED COMPONENTS Core Gui Qml Quick Sql )
# Get the path of a known Qt library (e.g., Qt6Core)
get_target_property(QSQLiteDriverPlugin_LOCATION Qt6::QSQLiteDriverPlugin LOCATION)

MESSAGE(STATUS "eton.debug:qt6 plugin path=[${Qt6_FOUND},${Qt6Sql_FOUND} , ${Qt6Sql_INCLUDE_DIRS},Qt6_DIR=${Qt6_DIR}],QSQLiteDriverPlugin_LOCATION=${QSQLiteDriverPlugin_LOCATION}")
MESSAGE(STATUS "eton.debug:qt6 plugin path=[${Qt6_FOUND},${Qt6Core_FOUND} , ${Qt6Core_INCLUDE_DIRS},${Qt6Core_LIBRARIES}]")
# We want to extract /path/to/qt/install
get_filename_component(QSQLiteDriverPlugin_DIR "${QSQLiteDriverPlugin_LOCATION}" DIRECTORY)
get_filename_component(Qt6_INSTALL_ROOT "${QSQLiteDriverPlugin_DIR}" DIRECTORY)
get_filename_component(Qt6_INSTALL_ROOT "${Qt6_INSTALL_ROOT}" DIRECTORY)
get_filename_component(Qt6_INSTALL_ROOT "${Qt6_INSTALL_ROOT}" DIRECTORY)
# Construct the plugin path
set(Qt6_PLUGIN_PATH "${Qt6_INSTALL_ROOT}/plugins")
message(STATUS "eton.debug:Qt plugin path: ${Qt6_PLUGIN_PATH}")

# Output the installation root directory
message(STATUS "Qt installation root directory: ${Qt6_INSTALL_ROOT}")
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
    Qt6::QSQLiteDriverPlugin
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
#Unlike most other CMake commands provided by Qt, qt6_deploy_qml_imports{qt_deploy_qml_imports()} can only be called from a deployment script. It cannot be called directly by the project.
# qt_deploy_qml_imports(
#     TARGET ${MyApp}
#     [QML_DIR qml_dir]
#     PLUGINS_FOUND var_name4plugins
#     [NO_QT_IMPORTS]
# )

#not supported in qt6.5
#set(deploy_tool_options_arg "")
set(var_PRE_INCLUDE_REGEXES
    "plugins/sqldrivers/libqsqlite.*\\.so" # Include libraries with names starting with "libspecial"
)

set(debug_cmake_get_runtime_deps 0)
if(debug_cmake_get_runtime_deps)
    # Use file(GET_RUNTIME_DEPENDENCIES) to get the dependencies
    file(GET_RUNTIME_DEPENDENCIES
        RESOLVED_DEPENDENCIES_VAR resolved_dependencies
        UNRESOLVED_DEPENDENCIES_VAR unresolved_dependencies
        PRE_INCLUDE_REGEXES ${PRE_INCLUDE_REGEXES}
        EXECUTABLES ${MyApp}
    )

    # Print the resolved dependencies
    message(STATUS "Resolved Dependencies: ${resolved_dependencies}")

    # Print the unresolved dependencies
    message(STATUS "Unresolved Dependencies: ${unresolved_dependencies}")
endif()

#qt_generate_deploy_app_script(
qt6_generate_deploy_qml_app_script(TARGET ${MyApp}
     OUTPUT_SCRIPT deploy_script
     NO_UNSUPPORTED_PLATFORM_ERROR
     PRE_INCLUDE_REGEXES "*/plugins/sqldrivers/libqsqlite.*"
     #PRE_INCLUDE_REGEXES ${var_PRE_INCLUDE_REGEXES}
     POST_INCLUDE_FILES ${QSQLiteDriverPlugin_LOCATION}
     ADDITIONAL_QT_PLUGINS sqldrivers
     #OUTPUT_SCRIPT_FILEPATH "${CMAKE_BINARY_DIR}/deploy.sh"
     #DEPLOY_TOOL_OPTIONS ${deploy_tool_options_arg}
)
qt6_import_plugins(${MyApp}
    INCLUDE Qt::QCocoaIntegrationPlugin
         EXCLUDE Qt::QMinimalIntegrationPlugin
         INCLUDE_BY_TYPE imageformats Qt::QGifPlugin Qt::QJpegPlugin
         INCLUDE_BY_TYPE sqldrivers Qt::QSQLiteDriverPlugin
)
message(STATUS "eton.debug:after qt6_generate_deploy_qml_app_script  script-file=[${deploy_script}], Qt::QSQLiteDriverPlugin ${QSQLiteDriverPlugin} <<")

set(debug_qt_deploy_qml_imports 1)
if(debug_qt_deploy_qml_imports)
    # The following script must only be executed at install time
    set(deploy_script4sqliteDriver "${CMAKE_CURRENT_BINARY_DIR}/deploy_script4sqliteDriver.cmake")

    file(GENERATE OUTPUT ${deploy_script4sqliteDriver} CONTENT "
    include(\"${QT_DEPLOY_SUPPORT}\")

    message(STATUS \"eton-debug: QSQLiteDriverPlugin_LOCATION =[ ${QSQLiteDriverPlugin_LOCATION}]\")
    #FILES ${QSQLiteDriverPlugin_LOCATION}  DESTINATION ${QT_DEPLOY_PLUGINS_DIR}/sqldrivers
    # Deploy application runtime dependencies and runtime dependencies
    # of the found QML module plugins.
    qt_deploy_runtime_dependencies(
        EXECUTABLE $<TARGET_FILE:${MyApp}>
        ADDITIONAL_LIBRARIES ${QSQLiteDriverPlugin_LOCATION}
        ADDITIONAL_MODULES ${QSQLiteDriverPlugin_LOCATION}
        VERBOSE
    )
#Qt6::QSQLiteDriverPlugin #\${QT_DEPLOY_PLUGINS_DIR}/sqldrivers
    ")
else()
    set(deploy_script42 ${deploy_script})
endif()


# install(SCRIPT ${deploy_script}
#     FILES ${QSQLiteDriverPlugin_LOCATION}  DESTINATION ${CMAKE_INSTALL_DOCDIR}/${QT_DEPLOY_PLUGINS_DIR}/sqldrivers}
#     CODE "message(STATUS \"eton.debug:this will run after script. script-file=[\${deploy_script}]\")"
# )
install(
    FILES ${QSQLiteDriverPlugin_LOCATION}  DESTINATION "plugins/sqldrivers"
)
