# Create version.h
configure_file(${BR_SHARE_DIR}/version.h.in version.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# Collect source files
set(SRC openbr.cpp
        openbr_plugin.cpp)
aux_source_directory(core BR_CORE)
include(plugins/plugins.cmake)

# Optional GUI module
if(NOT BR_EMBEDDED)
  aux_source_directory(gui BR_GUI)
  qt_add_resources(BR_ICONS ../share/openbr/icons.qrc)
  file(GLOB HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/gui/*.h)
  install(FILES ${HEADERS} DESTINATION include/openbr/gui)
endif()

# Plotting library
qt_add_resources(BR_PLOTTING ../share/openbr/plotting.qrc)

# Compile third party resources
if(BR_THIRDPARTY_RESOURCES)
  qt_add_resources(THIRDPARTY_RESOURCES ${BR_THIRDPARTY_RESOURCES})
endif()

add_library(openbr SHARED ${SRC} ${BR_CORE} ${BR_JANUS} ${BR_GUI} ${BR_ICONS} ${BR_PLOTTING} ${BR_THIRDPARTY_SRC} ${BR_RESOURCES} ${NATURALSTRINGCOMPARE_SRC} ${THIRDPARTY_RESOURCES})

foreach(QT_DEPENDENCY IN LISTS QT_DEPENDENCIES)
    target_link_libraries(openbr "Qt6::${QT_DEPENDENCY}")
endforeach()

set_target_properties(openbr PROPERTIES
                             DEFINE_SYMBOL BR_LIBRARY
                             VERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}
                             SOVERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR})
target_link_libraries(openbr PRIVATE ${BR_THIRDPARTY_LIBS})
add_cppcheck(openbr)

if(BR_WITH_DLIB)
  add_dependencies(openbr dlib)
endif()

# Install
install(TARGETS openbr
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

if(NOT BR_EMBEDDED)
  file(GLOB HEADERS *.h)
  install(FILES ${HEADERS} DESTINATION include/openbr)
endif()

include(InstallDependencies)
install_compiler_libraries()
install_qt_libraries(QT_DEPENDENCIES)
if(NOT BR_EMBEDDED)
  install_qt_imageformats()
  install_qt_platforms()
endif()
install_qt_misc()
install_opencv_libraries(OPENCV_DEPENDENCIES)
install_ffmpeg()
install_qt_ssl()

set(BR_INSTALL_ICU ON CACHE BOOL "Install Qt ICU libraries")
if(BR_INSTALL_ICU)
  install_qt_icu()
endif()
