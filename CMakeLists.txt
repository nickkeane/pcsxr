cmake_minimum_required(VERSION 3.13...3.17)
cmake_policy(SET CMP0065 NEW)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} cmake)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} cmake/macros)

# Add CMake Custom Options
# ####################################################################################################################################################
set(CPM_VERSION "v0.27" CACHE STRING "Select CPM Version")
set(CONAN_VERSION "v0.15" CACHE STRING "Select Conan Version")
set(DYNAREC "auto" CACHE STRING "Build dynarec for arch.")
set_property(CACHE DYNAREC PROPERTY STRINGS auto x86_64 x86 ppc no)
set(SND_BACKEND "sdl" CACHE STRING "Sound backend to use.")
set_property(CACHE SND_BACKEND PROPERTY STRINGS oss alsa pulse sdl openal null)
option(BUILD_SIO1 "Build SIO1 plugin." ON)
option(BUILD_OPENGL "Build OpenGL plugin." ON)
option(ENABLE_CCDDA "Enables compressed CDDA support." OFF)
option(USE_LIBARCHIVE "Enables compressed data-tracks support." OFF)
option(USE_LIBCDIO "Use GNU libcdio for CD-ROM support." OFF)
# ####################################################################################################################################################

project(
    pcsxr
    DESCRIPTION "PCSXRPlus"
    VERSION 2.0.0
    LANGUAGES CXX
)

include(CheckCCompilerFlag)
include(GNUInstallDirs)
include(GlibCompileResourcesSupport)
include(Gettext)

# ISB Guard
if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
    message(FATAL_ERROR "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there.")
endif()

# Identify Debug or Release
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(WARNING "Build type not specified. Defaulting to 'Release' type")
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS |"Debug" "Release" "RelWithDebInfo" "MinSizeRel")
    set(CMAKE_BUILD_TYPE
        "Release"
        CACHE STRING "CMake Build Type: Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif()

# Export Compile Commands
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# TODO: IPQ Check
# option(ENABLE_IPO "Enable Interprocedural Optimization i.e. Link Time Optimization" ON)
# if(ENABLE_IPO)
#     include(CheckIPOSupported) # check_ipo_supported()
#     check_ipo_supported(
#         RESULT
#         result
#         OUTPUT
#         output)
#     if(result)
#         message(STATUS "IPO Enabled")
#         set(IPO_ENABLED TRUE)
#     else()
#         message(FATAL_ERROR "IPO is not supported: ${output}")
#     endif()
# endif()

# Grab Package Managers for CPM and Conan
if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/pkgman/CPM.cmake")
    message(STATUS "Downloading CPM.cmake ${CPM_VERSION} from https://github.com/TheLartians/CPM.cmake")
    file(DOWNLOAD "https://github.com/TheLartians/CPM.cmake/raw/${CPM_VERSION}/cmake/CPM.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/pkgman/CPM.cmake")
endif()
if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/pkgman/conan.cmake")
    message(STATUS "Downloading conan.cmake ${CONAN_VERSION} from https://github.com/conan-io/cmake-conan")
    file(DOWNLOAD "https://github.com/conan-io/cmake-conan/raw/${CONAN_VERSION}/conan.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/pkgman/conan.cmake")
endif()
include(${CMAKE_CURRENT_BINARY_DIR}/pkgman/CPM.cmake) # CPMAddPackage, CPMFindPackage
include(${CMAKE_CURRENT_BINARY_DIR}/pkgman/conan.cmake) # conan_cmake_run

# Find 3rd Party Packages
# TODO: Replace some of these with CPM / Conan
find_package(Threads REQUIRED)
find_package(ZLIB REQUIRED)
find_library(DL_LIB dl REQUIRED)
find_library(RT_LIB rt REQUIRED)
find_package(X11 REQUIRED)
find_package(GLib REQUIRED)
if (USE_LIBARCHIVE)
  find_package(LibArchive REQUIRED)
endif()
if (USE_LIBCDIO)
  find_package(Cdio REQUIRED)
endif()
if (ENABLE_CCDDA)
    find_package(FFMPEG REQUIRED)
endif()
if(NOT GLib_FOUND)
    message(FATAL_ERROR "GLIB2 library not found")
endif()
find_package(GTK3 3.14.0 REQUIRED)
if(NOT GTK3_FOUND)
    message(FATAL_ERROR "GTK3 library not found")
endif()
set(GTK_LIBRARIES ${GTK3_LIBRARY} ${GDK3_LIBRARY} ${GDKPixbuf_LIBRARY} ${Pango_LIBRARY} ${Cairo_LIBRARY} ${GObject_LIBRARY} ${GLib_LIBRARY} ${GIO_LIBRARY})
if (BUILD_OPENGL)
    find_package(OpenGL REQUIRED)
    if(NOT OPENGL_FOUND)
        message(FATAL_ERROR "GL library not found")
    endif()
endif()
set(USE_SDL2 0)
find_package(SDL2)
if(NOT SDL2_FOUND)
    find_package(SDL)
    if(NOT SDL_FOUND)
        message(FATAL_ERROR "SDL or SDL2 library not found")
    else()
        message(STATUS "Using SDL")
        set(SDL_USE_V1 TRUE)
    endif()
else()
    set(USE_SDL2 1)
    message(STATUS "Using SDL2")
    set(SDL_USE_V2 TRUE)
endif()
if (${SND_BACKEND} STREQUAL "oss")
    find_package(OSS REQUIRED)
elseif (${SND_BACKEND} STREQUAL "alsa")
    find_package(ALSA REQUIRED)
elseif (${SND_BACKEND} STREQUAL "pulse")
    find_package(PulseAudio REQUIRED)
elseif (${SND_BACKEND} STREQUAL "openal")
    find_package(OpenAL REQUIRED)
endif()

# Create Inheritable Interface Targets for all subtargets
add_library(IProjectOptions INTERFACE)
add_library(IProject::Options ALIAS IProjectOptions)
target_compile_features(IProjectOptions INTERFACE cxx_std_20)
target_compile_options(IProjectOptions INTERFACE -s)

add_library(IProjectLinkerOpts INTERFACE)
add_library(IProject::LinkerOpts ALIAS IProjectLinkerOpts)
#WARNING! this is required for dynarec to work!
list(APPEND COMPILER_IDS "GNU" "Clang")
target_link_options(IProjectLinkerOpts INTERFACE -fno-pie)
if(CMAKE_C_COMPILER_ID IN_LIST COMPILER_IDS)
    CHECK_C_COMPILER_FLAG("-no-pie" NO_PIE_UPSTREAM)
    if(NO_PIE_UPSTREAM)
        target_link_options(IProjectLinkerOpts INTERFACE -no-pie)
    endif()
    CHECK_C_COMPILER_FLAG("-nopie" NO_PIE_PATCHED)
    if(NO_PIE_PATCHED)
        target_link_options(IProjectLinkerOpts INTERFACE -nopie)
    endif()
endif()

###############################################################
# pcsxcore
###############################################################
message(STATUS "* Configuring Core")
add_library(pcsxcore STATIC)
add_library(pcsxr::core ALIAS pcsxcore)
target_link_libraries(pcsxcore PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(pcsxcore PUBLIC 
    PACKAGE_VERSION="${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    PACKAGE_NAME="PCSXr"
    PACKAGE_STRING="PCSXr ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    ENABLE_NLS
    GETTEXT_PACKAGE="pcsxr"
)
if (BUILD_SIO1)
    target_compile_definitions(pcsxcore PUBLIC ENABLE_SIO1API)
endif()
target_include_directories(pcsxcore ${CMAKE_SOURCE_DIR})

# TODO: pcsxcore specific
if (ENABLE_CCDDA)
  target_include_directories(pcsxcore PUBLIC ${FFMPEG_INCLUDE_DIRS})
  target_compile_definitions(pcsxcore PUBLIC ENABLE_CCDDA)
endif()

if (USE_LIBARCHIVE)
  target_include_directories(pcsxcore PUBLIC ${LibArchive_INCLUDE_DIRS})
  target_compile_definitions(pcsxcore PUBLIC HAVE_LIBARCHIVE)
endif()

message(STATUS "Building PCSXr on arch " ${CMAKE_SYSTEM_PROCESSOR})
include(TargetArch)
target_architecture(PCSXR_TARGET_ARCH)
message(STATUS "Targeting arch " ${PCSXR_TARGET_ARCH})
if (${PCSXR_TARGET_ARCH} MATCHES "ppc")
    set(_ARCH_PPC 1)
elseif(${PCSXR_TARGET_ARCH} MATCHES "i386")
    set(_ARCH_32 1)
elseif(${PCSXR_TARGET_ARCH} MATCHES "x86_64")
    set(_ARCH_64 1)
else()
    message(STATUS "Unsupported arch. Will not build dynarec")
    target_compile_definitions(pcsxcore PUBLIC NOPSXREC)
endif()
set(CMAKE_POSITION_INDEPENDENT_CODE OFF) #for x86

if (${DYNAREC} STREQUAL "auto")
    if (_ARCH_PPC)
#if anyone ever fixes ppc dynarec
#        set(DYNAREC_PPC 1)
#        message(STATUS "Autodetected PPC dynarec.")
        message(STATUS "Autodetected PPC dynarec is broken, sorry.")
        target_compile_definitions(pcsxcore PUBLIC NOPSXREC)
    elseif(_ARCH_64)
        set(DYNAREC_64 1)
        message(STATUS "Autodetected x86_64 dynarec.")
    elseif(_ARCH_32)
        set(DYNAREC_32 1)
        message(STATUS "Autodetected x86 dynarec.")
    endif()
elseif (${DYNAREC} STREQUAL "ppc")
#if anyone ever fixes ppc dynarec
#    set(DYNAREC_PPC 1)
#    message(STATUS "User selected PPC dynarec")
    message(STATUS "User selected PPC dynarec is broken, sorry.")
    target_compile_definitions(pcsxcore PUBLIC NOPSXREC)
elseif (${DYNAREC} STREQUAL "x86_64")
    set(DYNAREC_64 1)
    message(STATUS "User selected x86_64 dynarec.")
elseif (${DYNAREC} STREQUAL "x86")
    set(DYNAREC_32 1)
    message(STATUS "User selected x86 dynarec.")
elseif (${DYNAREC} STREQUAL "no")
    message(STATUS "User selected to not build dynarec.")
    target_compile_definitions(pcsxcore PUBLIC NOPSXREC)
endif()


set(
    pcsxcore_SOURCES
    libpcsxcore/psxbios.c
    libpcsxcore/cdrom.c
    libpcsxcore/psxcounters.c
    libpcsxcore/psxdma.c
    libpcsxcore/disr3000a.c
    libpcsxcore/gpu.c
    libpcsxcore/spu.c
    libpcsxcore/sio.c
    libpcsxcore/psxhw.c
    libpcsxcore/mdec.c
    libpcsxcore/psxmem.c
    libpcsxcore/misc.c
    libpcsxcore/plugins.c
    libpcsxcore/decode_xa.c
    libpcsxcore/r3000a.c
    libpcsxcore/psxinterpreter.c
    libpcsxcore/gte.c
    libpcsxcore/psxhle.c
    libpcsxcore/debug.c
    libpcsxcore/psxcommon.c
    libpcsxcore/cdriso.c
    libpcsxcore/cheat.c
    libpcsxcore/socket.c
    libpcsxcore/ppf.c
    libpcsxcore/pgxp_cpu.c
    libpcsxcore/pgxp_debug.c
    libpcsxcore/pgxp_gte.c
    libpcsxcore/pgxp_mem.c
    libpcsxcore/pgxp_value.c
)

if(DYNAREC_64)
    file(GLOB_RECURSE DYNAREC_SRC  libpcsxcore/ix86_64/*.c)
elseif(DYNAREC_32)
    file(GLOB_RECURSE DYNAREC_SRC  libpcsxcore/ix86/*.c)
elseif(DYNAREC_PPC)
    enable_language(ASM-ATT)
    SET(CMAKE_ASM-ATT_SOURCE_FILE_EXTENSIONS nasm;nas;asm;s)
    file(GLOB_RECURSE DYNAREC_SRC  libpcsxcore/ppc/*.c)
    set(DYNAREC_SRC ${DYNAREC_SRC} libpcsxcore/ppc/pasm.s)
endif()
set(pcsxcore_SOURCES ${pcsxcore_SOURCES} ${DYNAREC_SRC})

target_sources(pcsxcore PUBLIC ${pcsxcore_SOURCES})
target_link_libraries(pcsxcore ${FFMPEG_LIBRARIES} ${LibArchive_LIBRARIES} -lm)

#####################################################
# pcsxr GUI
#####################################################
message(STATUS "* Configuring UI")
add_executable(pcsxr PRIVATE)
target_link_libraries(pcsxr PUBLIC IProject::Options IProject::LinkerOpts)
target_compile_definitions(pcsxr PUBLIC 
    PACKAGE_VERSION="${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    PACKAGE_NAME="PCSXr"
    PACKAGE_STRING="PCSXr ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
    ENABLE_NLS
    GETTEXT_PACKAGE="pcsxr"
)
if (BUILD_SIO1)
    target_compile_definitions(pcsxr PUBLIC ENABLE_SIO1API)
endif()
target_include_directories(pcsxr ${CMAKE_SOURCE_DIR})
target_include_directories(pcsxr PRIVATE ${GTK3_INCLUDE_DIRS})
target_include_directories(pcsxr PRIVATE ${ZLIB_INCLUDE_DIRS})
target_include_directories(pcsxr PRIVATE ${X11_XTest_INCLUDE_PATH})

if (${DYNAREC} STREQUAL "no")
    message(STATUS "User selected to not build dynarec")
    target_compile_definitions(pcsxr PRIVATE NOPSXREC)
endif()
target_compile_definitions(pcsxr PRIVATE
    LOCALE_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/locale/"
    PSEMU_DATA_DIR="${CMAKE_INSTALL_FULL_DATAROOTDIR}/psemu"
    DEF_PLUGIN_DIR="${CMAKE_INSTALL_FULL_LIBDIR}/games/psemu"    
)

set(pcsxr_SOURCES
    gui/AboutDlg.c
    gui/Cheat.c
    gui/ConfDlg.c
    gui/Config.c
    gui/DebugMemory.c
    gui/GtkGui.c
    gui/LnxMain.c
    gui/MemcardDlg.c
    gui/Plugin.c
    gui/resources.c    
)

#resources
set(pcsxr_RESOURCE_LIST
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pcsxr.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-icon.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-cd-open.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-plugin-cdr.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-plugin-gpu.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-iso-open.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-iso-reopen.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-memcard.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-plugin-pad.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-play.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-plugin-sio1.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-run-bios.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-save-state.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-load-state.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-settings.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-plugin-netplay.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-cpu.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-shutdown.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-reset.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-run-exe.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-plugin-spu.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-cheats.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-cheats-search.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-cheats-browse.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-exit.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-about.png
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/data/pixmaps/pcsxr-memory-dump.png
)


compile_gresources(pcsxr_RESOURCE_FILE
                   XML_OUT
                   TYPE EMBED_C
                   PREFIX /org/pcsxr/gui
                   SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/gui/data
                   RESOURCES ${pcsxr_RESOURCE_LIST})

add_custom_target(resource DEPENDS ${pcsxr_RESOURCE_FILE})
target_sources(pcsxr PRIVATE ${pcsxr_SOURCES} ${pcsxr_RESOURCE_FILE})
add_dependencies(pcsxr resource)
target_link_libraries(pcsxr pcsxr::core ${GTK_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${ZLIB_LIBRARIES} ${RT_LIB} ${DL_LIB} ${X11_XTest_LIB})

install(TARGETS pcsxr RUNTIME DESTINATION bin)
install(FILES gui/data/pcsxr.desktop DESTINATION share/applications)
install(FILES gui/data/pixmaps/pcsxr-icon.png DESTINATION share/icons/hicolor/48x48/apps)

message(STATUS "* Configuring translations")
#find_package(LibArchive REQUIRED)
GETTEXT_CREATE_TRANSLATIONS(gui/po/pcsxr.pot ALL gui/po/es gui/po/fr gui/po/hu gui/po/it gui/po/ko_KR gui/po/pt_BR gui/po/ru gui/po/zh_CN gui/po/zh_TW)

####################################################
# Plugins
####################################################
add_subdirectory(plugins)

####################################################
# Docs
####################################################
install(FILES doc/pcsxr.1 DESTINATION ${CMAKE_INSTALL_PREFIX}/share/man/man1)
install(FILES doc/keys.txt DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/pcsxr)
install(FILES doc/tweaks.txt DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/pcsxr)

install(FILES AUTHORS DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/pcsxr)
install(FILES COPYING DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/pcsxr)
install(FILES README DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/pcsxr)



