message(STATUS "* Configuring BladeSio1")
add_library(BladeSio1 MODULE)
add_library(pcsxr::plugins::BladeSio1 ALIAS BladeSio1)
target_link_libraries(BladeSio1 PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(BladeSio1 PUBLIC 
    PACKAGE_VERSION="${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    PACKAGE_NAME="PCSXr"
    PACKAGE_STRING="PCSXr ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    ENABLE_NLS
    GETTEXT_PACKAGE="pcsxr"
    LOCALE_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/locale/"
    PSEMU_DATA_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/psemu"
    DEF_PLUGIN_DIR="${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu"    
)
if (BUILD_SIO1)
    target_compile_definitions(BladeSio1 PUBLIC ENABLE_SIO1API)
endif()

add_executable(cfgBladeSio1)
target_link_libraries(cfgBladeSio1 PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(cfgBladeSio1 PUBLIC 
    PACKAGE_VERSION="${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    PACKAGE_NAME="PCSXr"
    PACKAGE_STRING="PCSXr ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    ENABLE_NLS
    GETTEXT_PACKAGE="pcsxr"
    LOCALE_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/locale/"
    PSEMU_DATA_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/psemu"
    DEF_PLUGIN_DIR="${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu"    
)
if (BUILD_SIO1)
    target_compile_definitions(cfgBladeSio1 PUBLIC ENABLE_SIO1API)
endif()

#sources
set(PLUGIN_SRCS
    cfg.c
    sio1.c
    fifo.c
    connection.c
)

set(GUI_SRCS
    gui.c
)

#resources
set(RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/sio1.ui
)

compile_gresources(RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/bladesio1
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
                   RESOURCES ${RESOURCE_LIST})
add_custom_target(bladesio1_resource DEPENDS ${RESOURCE_FILE})
add_dependencies(cfgBladeSio1 bladesio1_resource)

target_sources(BladeSio1 PUBLIC ${PLUGIN_SRCS})
target_include_directories(BladeSio1 PUBLIC ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
target_link_libraries(BladeSio1 PUBLIC ${GTK_LIBRARIES})

target_sources(cfgBladeSio1 PRIVATE ${GUI_SRCS} ${RESOURCE_FILE})
target_include_directories(cfgBladeSio1 PRIVATE ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
target_link_libraries(cfgBladeSio1 PRIVATE ${GTK_LIBRARIES})

install(TARGETS BladeSio1 LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
install(TARGETS cfgBladeSio1 RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)