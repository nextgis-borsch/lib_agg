################################################################################
# Project:  Lib AGG
# Purpose:  CMake build scripts
# Author:   Dmitry Baryshnikov, dmitry.baryshnikov@nexgis.com
################################################################################
# Copyright (C) 2017-2018, NextGIS <info@nextgis.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

project(agg)

if(NOT DEFINED PACKAGE_VENDOR)
    set(PACKAGE_VENDOR NextGIS)
endif()

if(NOT DEFINED PACKAGE_BUGREPORT)
    set(PACKAGE_BUGREPORT info@nextgis.com)
endif()

# Some init settings
set(CMAKE_COLOR_MAKEFILE ON)
# Set path to additional CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

if(OSX_FRAMEWORK AND (BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS))
  message(FATAL_ERROR "Only OSX_FRAMEWORK key or any or both BUILD_SHARED_LIBS
                       and BUILD_STATIC_LIBS keys are permitted")
endif()

if(OSX_FRAMEWORK)
    set(INSTALL_BIN_DIR "bin" CACHE INTERNAL "Installation directory for executables" FORCE)
    set(INSTALL_LIB_DIR "Library/Frameworks" CACHE INTERNAL "Installation directory for libraries" FORCE)
    set(INSTALL_INC_DIR "${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Headers" CACHE INTERNAL "Installation directory for headers" FORCE)
    set(INSTALL_CMAKECONF_DIR ${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Resources/CMake CACHE INTERNAL "Installation directory for cmake config files" FORCE)
    set(SKIP_INSTALL_HEADERS ON)
    set(SKIP_INSTALL_FILES ON)
    set(SKIP_INSTALL_EXPORT ON)
    set(CMAKE_MACOSX_RPATH ON)
else()
    include(GNUInstallDirs)

    set(INSTALL_BIN_DIR ${CMAKE_INSTALL_BINDIR} CACHE INTERNAL "Installation directory for executables" FORCE)
    set(INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE INTERNAL "Installation directory for libraries" FORCE)
    set(INSTALL_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE INTERNAL "Installation directory for headers" FORCE)
    set(INSTALL_DOC_DIR ${CMAKE_INSTALL_DOCDIR} CACHE INTERNAL "Share directory for data" FORCE)
    set(INSTALL_CMAKECONF_DIR ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/CMake CACHE INTERNAL "Installation directory for cmake config files" FORCE)
endif()

if(CMAKE_GENERATOR_TOOLSET MATCHES "v([0-9]+)_xp")
    add_definitions(-D_WIN32_WINNT=0x0501)
endif()

include(util)
check_version(AGG_MAJOR_VERSION AGG_MINOR_VERSION AGG_REV_VERSION)
set(VERSION ${AGG_MAJOR_VERSION}.${AGG_MINOR_VERSION}.${AGG_REV_VERSION})

if(OSX_FRAMEWORK)
    set(FRAMEWORK_VERSION "${AGG_MAJOR_VERSION}")
endif()
report_version(${PROJECT_NAME} ${VERSION})

configure_file(${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake IMMEDIATE @ONLY)

include(FindAnyProject)

include_directories ( ${CMAKE_CURRENT_SOURCE_DIR} )
include_directories ( ${CMAKE_CURRENT_BINARY_DIR} )
include_directories ( ${CMAKE_CURRENT_SOURCE_DIR}/include )

set(CMAKE_CXX_FLAGS_RELEASE "-DNDEBUG" CACHE STRING "Flags used by the compiler during release builds" FORCE)
# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-discarded-qualifiers")
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-discarded-qualifiers")

set(PUBLIC_HDRS
    include/agg_alpha_mask_u8.h
	include/agg_arc.h
	include/agg_array.h
	include/agg_arrowhead.h
	include/agg_basics.h
	include/agg_bezier_arc.h
	include/agg_bitset_iterator.h
    include/agg_blur.h
	include/agg_bounding_rect.h
    include/agg_bspline.h
    include/agg_clip_liang_barsky.h
    include/agg_color_gray.h
    include/agg_color_rgba.h
    include/agg_config.h
    include/agg_conv_adaptor_vcgen.h
    include/agg_conv_adaptor_vpgen.h
    include/agg_conv_bspline.h
    include/agg_conv_clip_polygon.h
    include/agg_conv_clip_polyline.h
    include/agg_conv_close_polygon.h
    include/agg_conv_concat.h
    include/agg_conv_contour.h
    include/agg_conv_curve.h
    include/agg_conv_dash.h
    include/agg_conv_gpc.h
    include/agg_conv_marker.h
    include/agg_conv_marker_adaptor.h
    include/agg_conv_segmentator.h
    include/agg_conv_shorten_path.h
    include/agg_conv_smooth_poly1.h
    include/agg_conv_stroke.h
    include/agg_conv_transform.h
    include/agg_conv_unclose_polygon.h
    include/agg_curves.h
    include/agg_dda_line.h
    include/agg_ellipse.h
    include/agg_ellipse_bresenham.h
    include/agg_embedded_raster_fonts.h
    include/agg_font_cache_manager.h
    include/agg_gamma_functions.h
    include/agg_gamma_lut.h
    include/agg_glyph_raster_bin.h
    include/agg_gradient_lut.h
    include/agg_gsv_text.h
    include/agg_image_accessors.h
    include/agg_image_filters.h
    include/agg_line_aa_basics.h
    include/agg_math.h
    include/agg_math_stroke.h
    include/agg_path_length.h
    include/agg_path_storage.h
    include/agg_path_storage_integer.h
    include/agg_pattern_filters_rgba.h
    include/agg_pixfmt_amask_adaptor.h
    include/agg_pixfmt_gray.h
    include/agg_pixfmt_rgb.h
    include/agg_pixfmt_rgb_packed.h
    include/agg_pixfmt_rgba.h
    include/agg_pixfmt_transposer.h
    include/agg_rasterizer_cells_aa.h
    include/agg_rasterizer_compound_aa.h
    include/agg_rasterizer_outline.h
    include/agg_rasterizer_outline_aa.h
    include/agg_rasterizer_scanline_aa.h
    include/agg_rasterizer_sl_clip.h
    include/agg_renderer_base.h
    include/agg_renderer_markers.h
    include/agg_renderer_mclip.h
    include/agg_renderer_outline_aa.h
    include/agg_renderer_outline_image.h
    include/agg_renderer_primitives.h
    include/agg_renderer_raster_text.h
    include/agg_renderer_scanline.h
    include/agg_rendering_buffer.h
    include/agg_rendering_buffer_dynarow.h
    include/agg_rounded_rect.h
    include/agg_scanline_bin.h
    include/agg_scanline_boolean_algebra.h
    include/agg_scanline_p.h
    include/agg_scanline_storage_aa.h
    include/agg_scanline_storage_bin.h
    include/agg_scanline_u.h
    include/agg_shorten_path.h
    include/agg_simul_eq.h
    include/agg_span_allocator.h
    include/agg_span_converter.h
    include/agg_span_gouraud.h
    include/agg_span_gouraud_gray.h
    include/agg_span_gouraud_rgba.h
    include/agg_span_gradient.h
    include/agg_span_gradient_alpha.h
    include/agg_span_image_filter.h
    include/agg_span_image_filter_gray.h
    include/agg_span_image_filter_rgb.h
    include/agg_span_image_filter_rgba.h
    include/agg_span_interpolator_adaptor.h
    include/agg_span_interpolator_linear.h
    include/agg_span_interpolator_persp.h
    include/agg_span_interpolator_trans.h
    include/agg_span_pattern_gray.h
    include/agg_span_pattern_rgb.h
    include/agg_span_pattern_rgba.h
    include/agg_span_solid.h
    include/agg_span_subdiv_adaptor.h
    include/agg_trans_affine.h
    include/agg_trans_bilinear.h
    include/agg_trans_double_path.h
    include/agg_trans_perspective.h
    include/agg_trans_single_path.h
    include/agg_trans_viewport.h
    include/agg_trans_warp_magnifier.h
    include/agg_vcgen_bspline.h
    include/agg_vcgen_contour.h
    include/agg_vcgen_dash.h
    include/agg_vcgen_markers_term.h
    include/agg_vcgen_smooth_poly1.h
    include/agg_vcgen_stroke.h
    include/agg_vcgen_vertex_sequence.h
    include/agg_vertex_sequence.h
    include/agg_vpgen_clip_polygon.h
    include/agg_vpgen_clip_polyline.h
    include/agg_vpgen_segmentator.h
    include/agg_pixfmt_base.h
    include/agg_rasterizer_scanline_aa_nogamma.h
)

set(CTRL_PUB_HDRS
    include/ctrl/agg_bezier_ctrl.h
    include/ctrl/agg_cbox_ctrl.h
    include/ctrl/agg_ctrl.h
    include/ctrl/agg_gamma_ctrl.h
    include/ctrl/agg_gamma_spline.h
    include/ctrl/agg_polygon_ctrl.h
    include/ctrl/agg_rbox_ctrl.h
    include/ctrl/agg_scale_ctrl.h
    include/ctrl/agg_slider_ctrl.h
    include/ctrl/agg_spline_ctrl.h
)

set(PLAT_PUB_HDRS
    include/platform/agg_platform_support.h
)

set(UTIL_PUB_HDRS
    include/util/agg_color_conv.h
    include/util/agg_color_conv_rgb16.h
    include/util/agg_color_conv_rgb8.h
)

find_anyproject(Freetype DEFAULT ON)
if(FREETYPE_FOUND)
    add_definitions(-DAGG_USE_FREETYPE)
    set(PUBLIC_HDRS ${PUBLIC_HDRS}
        font_freetype/agg_font_freetype.h
    )

    set(SRCS ${SRCS}
        font_freetype/agg_font_freetype.cpp
    )
endif()

if(WIN32)
    set(PUBLIC_HDRS ${PUBLIC_HDRS}
        font_win32_tt/agg_font_win32_tt.h
    )

    set(SRCS ${SRCS}
        font_win32_tt/agg_font_win32_tt.cpp
        src/platform/win32/agg_platform_support.cpp
        src/platform/win32/agg_win32_bmp.cpp
    )
endif()

if(OSX_FRAMEWORK)
    set(SRCS ${SRCS}
        src/platform/mac/agg_mac_pmap.cpp
        src/platform/mac/agg_platform_support.cpp
    )
endif()

set(DOCS
    COPYING
    authors
    ChangeLog
    news
    readme
    copying
)

set(SRCS
    src/agg_arc.cpp
    src/agg_arrowhead.cpp
    src/agg_bezier_arc.cpp
    src/agg_bspline.cpp
    src/agg_curves.cpp
    src/agg_color_rgba.cpp
    src/agg_embedded_raster_fonts.cpp
    src/agg_gsv_text.cpp
    src/agg_image_filters.cpp
    src/agg_line_aa_basics.cpp
    src/agg_line_profile_aa.cpp
    src/agg_rounded_rect.cpp
    src/agg_sqrt_tables.cpp
    src/agg_trans_affine.cpp
    src/agg_trans_double_path.cpp
    src/agg_trans_single_path.cpp
    src/agg_trans_warp_magnifier.cpp
    src/agg_vcgen_bspline.cpp
    src/agg_vcgen_contour.cpp
    src/agg_vcgen_dash.cpp
    src/agg_vcgen_markers_term.cpp
    src/agg_vcgen_smooth_poly1.cpp
    src/agg_vcgen_stroke.cpp
    src/agg_vpgen_clip_polygon.cpp
    src/agg_vpgen_clip_polyline.cpp
    src/agg_vpgen_segmentator.cpp
    src/ctrl/agg_cbox_ctrl.cpp
    src/ctrl/agg_gamma_ctrl.cpp
    src/ctrl/agg_gamma_spline.cpp
    src/ctrl/agg_rbox_ctrl.cpp
    src/ctrl/agg_slider_ctrl.cpp
    src/ctrl/agg_spline_ctrl.cpp
    src/ctrl/agg_scale_ctrl.cpp
    src/ctrl/agg_bezier_ctrl.cpp
    src/ctrl/agg_polygon_ctrl.cpp
)

set(LIB_NAME ${PROJECT_NAME})
if(BUILD_SHARED_LIBS)
    add_library(${LIB_NAME} SHARED ${SRCS} ${PUBLIC_HDRS} ${CTRL_PUB_HDRS} ${PLAT_PUB_HDRS} ${UTIL_PUB_HDRS})
    set_target_properties (${LIB_NAME} PROPERTIES
        VERSION ${VERSION}
        SOVERSION ${AGG_MAJOR_VERSION}
        DEFINE_SYMBOL AGG_EXPORTS)
elseif(OSX_FRAMEWORK)
    add_library(${LIB_NAME} SHARED ${SRCS} ${PUBLIC_HDRS} ${CTRL_PUB_HDRS} ${PLAT_PUB_HDRS} ${UTIL_PUB_HDRS})
    set_property(SOURCE ${CTRL_PUB_HDRS}
      PROPERTY MACOSX_PACKAGE_LOCATION Headers/ctrl
    )
    set_property(SOURCE ${PLAT_PUB_HDRS}
      PROPERTY MACOSX_PACKAGE_LOCATION Headers/platform
    )
    set_property(SOURCE ${UTIL_PUB_HDRS}
      PROPERTY MACOSX_PACKAGE_LOCATION Headers/util
    )
    set_target_properties(${LIB_NAME} PROPERTIES
      FRAMEWORK TRUE
      FRAMEWORK_VERSION ${FRAMEWORK_VERSION}
      MACOSX_FRAMEWORK_SHORT_VERSION_STRING ${VERSION}
      MACOSX_FRAMEWORK_BUNDLE_VERSION ${VERSION}
      MACOSX_FRAMEWORK_IDENTIFIER org.libagg.libagg
      XCODE_ATTRIBUTE_INSTALL_PATH "@rpath"
      PUBLIC_HEADER "${PUBLIC_HDRS}"
      VERSION ${VERSION}
      SOVERSION ${AGG_MAJOR_VERSION}
      )
else()
    add_definitions(-DJBIG_STATIC)
    add_library(${LIB_NAME} STATIC ${SRCS} ${PUBLIC_HDRS} ${CTRL_PUB_HDRS} ${PLAT_PUB_HDRS} ${UTIL_PUB_HDRS})
    #set_target_properties(${LIB_NAME} PROPERTIES DEFINE_SYMBOL JBIG_STATIC)
endif()

target_link_extlibraries(${LIB_NAME})

add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

if(NOT CYGWIN)
    set_target_properties(${LIB_NAME} PROPERTIES VERSION ${VERSION})
endif()

if(UNIX)
    # On unix-like platforms the library is almost always called lib${PROJECT_NAME}
    set_target_properties(${LIB_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
endif()

set(PACKAGE_NAME ${PROJECT_NAME})
string(TOUPPER ${PACKAGE_NAME} PACKAGE_UPPER_NAME)

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
    install(TARGETS ${LIB_NAME}
        EXPORT ${PACKAGE_UPPER_NAME}Targets
        RUNTIME DESTINATION ${INSTALL_BIN_DIR} COMPONENT applications
        ARCHIVE DESTINATION ${INSTALL_LIB_DIR} COMPONENT libraries
        LIBRARY DESTINATION ${INSTALL_LIB_DIR} COMPONENT libraries
        INCLUDES DESTINATION ${INSTALL_INC_DIR}
        FRAMEWORK DESTINATION ${INSTALL_LIB_DIR}
    )
endif()

if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL)
    install(FILES ${PUBLIC_HDRS} DESTINATION "${INSTALL_INC_DIR}/agg2")
    install(FILES ${CTRL_PUB_HDRS} DESTINATION "${INSTALL_INC_DIR}/agg2/ctrl")
    install(FILES ${PLAT_PUB_HDRS} DESTINATION "${INSTALL_INC_DIR}/agg2/platform")
    install(FILES ${UTIL_PUB_HDRS} DESTINATION "${INSTALL_INC_DIR}/agg2/util")
    install(FILES ${DOCS} DESTINATION "${INSTALL_DOC_DIR}/lib${PROJECT_NAME}")
endif()

# Export package ===============================================================

# Add path to includes to build-tree export
target_include_directories(${LIB_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
)

# Add all targets to the build-tree export set
export(TARGETS ${LIB_NAME}
    FILE ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Targets.cmake)

if(REGISTER_PACKAGE)
    # Export the package for use from the build-tree
    # (this registers the build-tree with a global CMake-registry)
    export(PACKAGE ${PACKAGE_UPPER_NAME})
endif()

# Create the ZLIBConfig.cmake file
configure_file(cmake/PackageConfig.cmake.in
    ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Config.cmake @ONLY)

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
    # Install the <Package>Config.cmake
    install(FILES
      ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Config.cmake
      DESTINATION ${INSTALL_CMAKECONF_DIR} COMPONENT dev)

    # Install the export set for use with the install-tree
    install(EXPORT ${PACKAGE_UPPER_NAME}Targets DESTINATION ${INSTALL_CMAKECONF_DIR} COMPONENT dev)
endif()

# Archiving ====================================================================

set(CPACK_PACKAGE_NAME "${PACKAGE_NAME}")
set(CPACK_PACKAGE_VENDOR "${PACKAGE_VENDOR}")
set(CPACK_PACKAGE_VERSION "${VERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${PACKAGE_NAME} Installation")
set(CPACK_PACKAGE_RELOCATABLE TRUE)
set(CPACK_ARCHIVE_COMPONENT_INSTALL ON)
set(CPACK_GENERATOR "ZIP")
set(CPACK_MONOLITHIC_INSTALL ON)
set(CPACK_STRIP_FILES TRUE)

# Get cpack zip archive name
get_cpack_filename(${VERSION} PROJECT_CPACK_FILENAME)
set(CPACK_PACKAGE_FILE_NAME ${PROJECT_CPACK_FILENAME})

include(CPack)
