message(STATUS "* Configuring peopsxgl")
add_library(peopsxgl MODULE)
add_library(pcsxr::plugins::peopsxgl ALIAS peopsxgl)
target_link_libraries(peopsxgl PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(peopsxgl PUBLIC 
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
    target_compile_definitions(peopsxgl PUBLIC ENABLE_SIO1API)
endif()

add_executable(cfgpeopsxgl)
target_link_libraries(cfgpeopsxgl PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(cfgpeopsxgl PUBLIC 
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
    target_compile_definitions(cfgpeopsxgl PUBLIC ENABLE_SIO1API)
endif()

# GLib GTK OGL X11

include_directories(${OPENGL_INCLUDE_DIR})

#sources
set(PLUGIN_SRCS
    cfg.c
    draw.c
    fps.c
    gpu.c
    gte_accuracy.c
    key.c
    menu.c
    prim.c
    soft.c
    texture.c
    pgxp_gpu.c
)

set(GUI_SRCS
    gpucfg/main.c
)

#resources
set(RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/gpucfg/peopsxgl.ui
)

compile_gresources(RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/peopsxgl
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/gpucfg
                   RESOURCES ${RESOURCE_LIST})
add_custom_target(peopsxgl_resource DEPENDS ${RESOURCE_FILE})

target_sources(peopsxgl PUBLIC ${PLUGIN_SRCS})
target_include_directories(peopsxgl PUBLIC ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore ${X11_X11_INCLUDE_PATH} ${X11_xf86vmode_INCLUDE_PATH} ${OPENGL_INCLUDE_DIR})
target_link_libraries(peopsxgl PUBLIC ${GTK_LIBRARIES} ${X11_LIBRARIES} ${X11_Xxf86vm_LIB} ${OPENGL_gl_LIBRARY})

target_sources(cfgpeopsxgl PRIVATE ${GUI_SRCS} ${RESOURCE_FILE})
add_dependencies(cfgpeopsxgl peopsxgl_resource)
target_include_directories(cfgpeopsxgl PRIVATE ${GLib_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS} ${CMAKE_SOURCE_DIR}/libpcsxcore ${X11_X11_INCLUDE_PATH} ${X11_Xv_INCLUDE_PATH} ${X11_XShm_INCLUDE_PATH})
target_link_libraries(cfgpeopsxgl PRIVATE ${GTK_LIBRARIES})

install(TARGETS peopsxgl LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
install(TARGETS cfgpeopsxgl RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)