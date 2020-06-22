message(STATUS "* Configuring NullSio1")
add_library(NullSio1 MODULE)
add_library(pcsxr::plugins::NullSio1 ALIAS NullSio1)
target_link_libraries(NullSio1 PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(NullSio1 PUBLIC 
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
    target_compile_definitions(NullSio1 PUBLIC ENABLE_SIO1API)
endif()

target_sources(NullSio1 PUBLIC sio1.c)
target_include_directories(NullSio1 PUBLIC ${CMAKE_SOURCE_DIR}/libpcsxcore)

install(TARGETS NullSio1 LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu)
