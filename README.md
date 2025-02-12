# QML(QT6):Demo for add sqldrivers plugin to install dir;
- author: eton;
- date: 250210

based on example from Qt;

## Background
1. use `import QtQuick.LocalStorage` in qml to operate database;
2. it works in development environment;
3. 'QSqlDatabase: QSQLITE driver not loaded' in deployment machine, which packaged by `qt_generate_deploy_qml_app_script`;
4. create this demo for fix this;

### method
1. use another install() function to do it.
```cmake
#install qt6 sqldrivers --eton@250212 --begin
find_package(Qt6 COMPONENTS Sql REQUIRED)
get_target_property(QSQLiteDriverPlugin_LOCATION Qt6::QSQLiteDriverPlugin LOCATION)
install(
    FILES ${QSQLiteDriverPlugin_LOCATION}  DESTINATION "plugins/sqldrivers"
)
#install qt6 sqldrivers --eton@250212 --begin.end
```
