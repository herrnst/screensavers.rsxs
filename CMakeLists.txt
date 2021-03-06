project(visualizations.rsxs)

cmake_minimum_required(VERSION 2.6)

enable_language(CXX)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR})

find_package(Kodi REQUIRED)
find_package(OpenGL REQUIRED)
find_package(PNG REQUIRED)
find_package(ZLIB REQUIRED)
find_package(X REQUIRED)

set(rsxs_dir lib/rsxs-1.0)

include_directories(${OpenGL_INCLUDE_DIR}
                    ${PNG_INCLUDE_DIR}
                    ${KODI_INCLUDE_DIR}/.. # Hack way with "/..", need bigger Kodi cmake rework to match right include ways
                    ${X_INCLUDE_DIRS}
                    ${CMAKE_BINARY_DIR}/rsxs-prefix/src/rsxs-build
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/lib
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/cyclone
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/euphoria
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/fieldlines
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/flocks
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/flux
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/helios
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/hyperspace
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/lattice
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/plasma
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/skyrocket
                    ${PROJECT_SOURCE_DIR}/${rsxs_dir}/src/solarwinds)
                        

get_filename_component( COMPILER_FILENAME "${CMAKE_C_COMPILER}" NAME )
string( REGEX REPLACE "-[^-]+$" ""
        TOOLCHAIN_NAME "${COMPILER_FILENAME}" )

include(ExternalProject)
set(update_command "")
if(BOOTSTRAP_IN_TREE OR NOT DEFINED BOOTSTRAP_IN_TREE)
  set(update_command UPDATE_COMMAND autoreconf -vif WORKING_DIRECTORY <SOURCE_DIR>)
endif()
set(configure_start ${PROJECT_SOURCE_DIR}/${rsxs_dir}/configure)
if(DEPENDS_PATH)
  set(PNG_PATH "--with-png=${DEPENDS_PATH}")
  if(EXISTS ${DEPENDS_PATH}/share/config.site)
    set(configure_start CONFIG_SITE=${DEPENDS_PATH}/share/config.site ${configure_start})
  endif()
endif()
if(APPLE)
  set(EXTRA_FLAGS "CFLAGS=${CMAKE_C_FLAGS} -fPIC -isystem${DEPENDS_PATH}/include -isystem${KODI_INCLUDE_DIR}" "CXXFLAGS=${CMAKE_CXX_FLAGS} -fPIC -isystem${DEPENDS_PATH}/include -isystem${KODI_INCLUDE_DIR}")
else()
  set(EXTRA_FLAGS "CFLAGS=${CMAKE_C_FLAGS} -fPIC -I${KODI_INCLUDE_DIR}" "CXXFLAGS=${CMAKE_CXX_FLAGS} -fPIC -I${KODI_INCLUDE_DIR}")
endif()
externalproject_add(rsxs SOURCE_DIR ${PROJECT_SOURCE_DIR}/${rsxs_dir}
                    CONFIGURE_COMMAND ${configure_start}
                                      ac_cv_type__Bool=yes
                                      gl_cv_func_gettimeofday_clobber=no
                                      ac_cv_func_malloc_0_nonnull=yes
                                      --prefix=<INSTALL_DIR>
                                      --host=${TOOLCHAIN_NAME}
                                      --without-xscreensaver
                                      --disable-cyclone
                                      --disable-euphoria
                                      --disable-fieldlines
                                      --disable-flocks
                                      --disable-flux
                                      --disable-helios
                                      --disable-hyperspace
                                      --disable-lattice
                                      --disable-plasma
                                      --disable-skyrocket
                                      --disable-solarwinds
                                      --disable-sound
                                      --x-includes=${X_INCLUDE_DIRS}
                                      --x-libraries=${X_LIBRARIES}
                                      ${PNG_PATH}
                                      ${EXTRA_FLAGS}
                                      "CPPFLAGS=-Dpng_set_gray_1_2_4_to_8=png_set_expand_gray_1_2_4_to_8"
                                      "LIBS=${ZLIB_LIBRARIES}"
                    BUILD_COMMAND ${MAKE}
                    "${update_command}")

add_definitions(-DHAVE_CONFIG_H=1 -DHAVE_X11_XLIB_H=1 -Dpng_set_gray_1_2_4_to_8=png_set_expand_gray_1_2_4_to_8)
add_options(ALL_LANGUAGES ALL_BUILDS -fPIC)

add_library(rsxs_common STATIC ${rsxs_dir}/src/color.cc
                               ${rsxs_dir}/src/common.cc
                               ${rsxs_dir}/src/implicit.cc
                               ${rsxs_dir}/src/pngimage.cc)
add_dependencies(rsxs_common rsxs)

set(SOLARWINDS_SOURCES ${rsxs_dir}/src/solarwinds/solarwinds.cc
                       ${rsxs_dir}/src/solarwinds/wind.cc
                       src/solarwinds.cpp
                       src/addoncommon.h)

set(PLASMA_SOURCES ${rsxs_dir}/src/plasma/plasma.cc
                   src/plasma.cpp
                   src/addoncommon.h)

set(EUPHORIA_SOURCES ${rsxs_dir}/src/euphoria/euphoria.cc
                     ${rsxs_dir}/src/euphoria/wisp.cc
                     src/euphoria.cpp
                     src/addoncommon.h)

set(FLOCKS_SOURCES ${rsxs_dir}/src/flocks/bug.cc
                   ${rsxs_dir}/src/flocks/flocks.cc
                   src/flocks.cpp
                   src/addoncommon.h)

set(CYCLONE_SOURCES ${rsxs_dir}/src/cyclone/blend.cc
                    ${rsxs_dir}/src/cyclone/cyclone.cc
                    ${rsxs_dir}/src/cyclone/particle.cc
                    src/cyclone.cpp
                    src/addoncommon.h)

set(FLUX_SOURCES ${rsxs_dir}/src/flux/flux.cc
                 ${rsxs_dir}/src/flux/trail.cc
                 src/flux.cpp
                 src/addoncommon.h)

set(FIELDLINES_SOURCES ${rsxs_dir}/src/fieldlines/fieldlines.cc
                       src/fieldlines.cpp
                       src/addoncommon.h)

set(HELIOS_SOURCES ${rsxs_dir}/src/helios/helios.cc
                   ${rsxs_dir}/src/helios/particle.cc
                   src/helios.cpp
                   src/addoncommon.h)

set(HYPERSPACE_SOURCES ${rsxs_dir}/src/hyperspace/caustic.cc
                       ${rsxs_dir}/src/hyperspace/cubemaps.cc
                       ${rsxs_dir}/src/hyperspace/extensions.cc
                       ${rsxs_dir}/src/hyperspace/flares.cc
                       ${rsxs_dir}/src/hyperspace/goo.cc
                       ${rsxs_dir}/src/hyperspace/hyperspace.cc
                       ${rsxs_dir}/src/hyperspace/nebula.cc
                       ${rsxs_dir}/src/hyperspace/particle.cc
                       ${rsxs_dir}/src/hyperspace/shaders.cc
                       ${rsxs_dir}/src/hyperspace/spline.cc
                       ${rsxs_dir}/src/hyperspace/starburst.cc
                       ${rsxs_dir}/src/hyperspace/tunnel.cc
                       src/hyperspace.cpp
                       src/addoncommon.h)

set(LATTICE_SOURCES ${rsxs_dir}/src/lattice/lattice.cc
                    ${rsxs_dir}/src/lattice/resources.cc
                    src/lattice.cpp
                    src/addoncommon.h)

set(SKYROCKET_SOURCES ${rsxs_dir}/src/skyrocket/bee.cc
                      ${rsxs_dir}/src/skyrocket/bigmama.cc
                      ${rsxs_dir}/src/skyrocket/bomb.cc
                      ${rsxs_dir}/src/skyrocket/explosion.cc
                      ${rsxs_dir}/src/skyrocket/flares.cc
                      ${rsxs_dir}/src/skyrocket/fountain.cc
                      ${rsxs_dir}/src/skyrocket/meteor.cc
                      ${rsxs_dir}/src/skyrocket/overlay.cc
                      ${rsxs_dir}/src/skyrocket/resources.cc
                      ${rsxs_dir}/src/skyrocket/rocket.cc
                      ${rsxs_dir}/src/skyrocket/shockwave.cc
                      ${rsxs_dir}/src/skyrocket/skyrocket.cc
                      ${rsxs_dir}/src/skyrocket/smoke.cc
                      ${rsxs_dir}/src/skyrocket/spinner.cc
                      ${rsxs_dir}/src/skyrocket/star.cc
                      ${rsxs_dir}/src/skyrocket/streamer.cc
                      ${rsxs_dir}/src/skyrocket/stretcher.cc
                      ${rsxs_dir}/src/skyrocket/sucker.cc
                      ${rsxs_dir}/src/skyrocket/world.cc
                      src/skyrocket.cpp
                      src/addoncommon.h)

set(DEPLIBS ${OPENGL_LIBRARIES}
            ${PNG_LIBRARIES}
            ${CMAKE_BINARY_DIR}/rsxs-prefix/src/rsxs-build/lib/libmisc.a
            rsxs_common)

foreach(addon solarwinds plasma euphoria flocks flux cyclone fieldlines helios
              hyperspace lattice skyrocket)
  string(TOUPPER ${addon} ADDON)
  option(BUILD_${ADDON} "Build screensaver.rsxs.${addon}?" ON)
  if(BUILD_${ADDON})
    build_addon(screensaver.rsxs.${addon} ${ADDON} DEPLIBS)
    add_dependencies(screensaver.rsxs.${addon} rsxs_common)
  endif()
endforeach()

include(CPack)
