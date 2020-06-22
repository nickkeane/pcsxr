message(STATUS "* Configuring DFNet")
add_library(DFNet MODULE)
add_library(pcsxr::plugins::DFNet ALIAS DFNet)
target_link_libraries(DFNet PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(DFNet PUBLIC 
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
    target_compile_definitions(DFNet PUBLIC ENABLE_SIO1API)
endif()

add_executable(cfgDFNet)
target_link_libraries(cfgDFNet PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(cfgDFNet PUBLIC 
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
    target_compile_definitions(cfgDFNet PUBLIC ENABLE_SIO1API)
endif()

#sources
set(PLUGIN_SRCS
    cfg.c
    dfnet.c
    unix.c
)

set(GUI_SRCS
    gui.c
)

#resources
set(RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/dfnet.ui
)

compile_gresources(RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/dfnet
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
                   RESOURCES ${RESOURCE_LIST})
add_custom_target(dfnet_resource DEPENDS ${RESOURCE_FILE})

target_sources(DFNet PUBLIC ${PLUGIN_SRCS})
target_include_directories(DFNet PUBLIC ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
target_link_libraries(DFNet PUBLIC ${GTK_LIBRARIES})

target_sources(cfgDFNet PRIVATE ${GUI_SRCS} ${RESOURCE_FILE})
add_dependencies(cfgDFNet dfnet_resource)
target_include_directories(cfgDFNet PRIVATE ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore)
target_link_libraries(cfgDFNet PRIVATE ${GTK_LIBRARIES})

install(TARGETS DFNet LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
install(TARGETS cfgDFNet RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
