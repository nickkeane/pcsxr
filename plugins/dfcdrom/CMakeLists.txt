message(STATUS "* Configuring DFCdrom")
add_library(DFCdrom MODULE)
add_library(pcsxr::plugins::DFCdrom ALIAS DFCdrom)
target_link_libraries(DFCdrom PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(DFCdrom PUBLIC 
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
    target_compile_definitions(DFCdrom PUBLIC ENABLE_SIO1API)
endif()

add_executable(cfgDFCdrom)
target_link_libraries(cfgDFCdrom PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(cfgDFCdrom PUBLIC 
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
    target_compile_definitions(cfgDFCdrom PUBLIC ENABLE_SIO1API)
endif()

set(PLUGIN_SRCS
    cdr.c
    cdr-libcdio.c
    cdr-linux.c
    cdr-null.c
    cfg.c
    util.c
)

set(GUI_SRCS
    cdrcfg-0.1df/main.c
)

#resources
set(RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/cdrcfg-0.1df/dfcdrom.ui
)

compile_gresources(RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/dfcdrom
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/cdrcfg-0.1df
                   RESOURCES ${RESOURCE_LIST})
add_custom_target(dfcdrom_resource DEPENDS ${RESOURCE_FILE})

target_sources(DFCdrom PUBLIC ${PLUGIN_SRCS})
target_include_directories(DFCdrom PUBLIC ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
target_link_libraries(DFCdrom PUBLIC ${GTK_LIBRARIES} ${CDIO_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})

target_sources(cfgDFCdrom PRIVATE ${GUI_SRCS} ${RESOURCE_FILE})
target_include_directories(cfgDFCdrom PRIVATE ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
add_dependencies(cfgDFCdrom dfcdrom_resource)
target_link_libraries(cfgDFCdrom PRIVATE ${GTK_LIBRARIES})

if (USE_LIBCDIO)
    target_include_directories(DFCdrom PUBLIC ${CDIO_INCLUDE_DIR})
    target_compile_definitions(DFCdrom PUBLIC USE_LIBCDIO)    
    target_include_directories(cfgDFCdrom PRIVATE ${CDIO_INCLUDE_DIR})
    target_compile_definitions(cfgDFCdrom PRIVATE USE_LIBCDIO)
endif()

install(TARGETS DFCdrom LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
install(TARGETS cfgDFCdrom RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
