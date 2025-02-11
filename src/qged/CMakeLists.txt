include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${BRLCAD_BINARY_DIR}/${INCLUDE_DIR}
  ${BRLCAD_SOURCE_DIR}/include
  ${QTCAD_INCLUDE_DIRS}
  ${GED_INCLUDE_DIRS}
  ${RT_INCLUDE_DIRS}
  ${BU_INCLUDE_DIRS}
  )

# NOTE: We can't use the CMake global values for automoc et. al.
# because most of BRL-CAD's targets are not Qt targets.  We don't
# want to run the automatic moc logic for all of them.

set(qged_srcs
  main_window.cpp
  app.cpp
  attributes.cpp
  event_filter.cpp
  fbserv.cpp
  import.cpp
  palettes.cpp
  )

set(moc_headers
  main_window.h
  app.h
  attributes.h
  event_filter.h
  fbserv.h
  import.h
  palettes.h
  )

if (BRLCAD_ENABLE_QT)

  # Let main_window know what the platform's extension is so it knows
  # what plugin filenames to look for
  set_property(SOURCE main_window.cpp APPEND PROPERTY
    COMPILE_DEFINITIONS "QGED_PLUGIN_SUFFIX=\"${CMAKE_SHARED_LIBRARY_SUFFIX}\"")

  QT5_WRAP_CPP(moc_srcs ${moc_headers})

  # https://github.com/Alexhuszagh/BreezeStyleSheets
  QT5_ADD_RESOURCES(theme_qrc theme/breeze.qrc)

  add_executable(qged main.cpp ${qged_srcs} ${moc_srcs} ${theme_qrc})
  # Note - Network is needed for async embedded raytracing display
  target_link_libraries(qged libqtcad libged librt libbu Qt5::Widgets Qt5::Network)
  target_compile_definitions(qged PRIVATE BRLCADBUILD HAVE_CONFIG_H)
  install(TARGETS qged
    RUNTIME DESTINATION ${BIN_DIR}
    LIBRARY DESTINATION ${LIB_DIR}
    ARCHIVE DESTINATION ${LIB_DIR})

endif (BRLCAD_ENABLE_QT)

# Core application is defined, now handle plugins
add_subdirectory(plugins)

set(qged_ignore_srcs
  ${qged_srcs}
  ${moc_headers}
  display/AxesRenderer.cpp
  display/AxesRenderer.h
  display/Camera.h
  display/Display.cpp
  display/Display.h
  display/DisplayManager.cpp
  display/DisplayManager.h
  display/GeometryRenderer.cpp
  display/GeometryRenderer.h
  display/OrthographicCamera.cpp
  display/OrthographicCamera.h
  display/README.txt
  display/Renderable.h
  fonts/Inconsolata-LICENSE.txt
  fonts/Inconsolata.otf
  CMakeLists.txt
  qdm_tests.txt
  TODO
  )

CMAKEFILES(${qged_ignore_srcs})

# Local Variables:
# tab-width: 8
# mode: cmake
# indent-tabs-mode: t
# End:
# ex: shiftwidth=2 tabstop=8

