if(APPLE)
  # CPACK_DMG_DS_STORE_SETUP_SCRIPT was introduced only in CMake 3.5
  cmake_minimum_required(VERSION 3.5)
else()
  cmake_minimum_required(VERSION 2.8.11)
endif()

if(NOT ${CMAKE_VERSION} VERSION_LESS "3.5.0")
  # Enable PVS support
  set(CMAKE_EXPORT_COMPILE_COMMANDS ON CACHE BOOL "Enable/Disable output of compile commands during generation.")
endif()

project("Torrent File Editor")
set(EXE_NAME "torrent-file-editor")

# Fallback version. Will be used when compiling out of git repository.
set(APP_VERSION "0.3.17")

set(MACOSX_BUNDLE_EXECUTABLE_NAME "${PROJECT_NAME}")
set(MACOSX_BUNDLE_INFO_STRING "Torrent File Editor \${APP_VERSION}. This program is licensed under the GNU GPL.")
set(MACOSX_BUNDLE_ICON_FILE "application.icns")
set(MACOSX_BUNDLE_GUI_IDENTIFIER "io.github.torrent-file-editor")
set(MACOSX_BUNDLE_BUNDLE_NAME "Torrent File Editor")
set(MACOSX_BUNDLE_SHORT_VERSION_STRING "\${APP_VERSION}")
set(MACOSX_BUNDLE_BUNDLE_VERSION "\${APP_VERSION}")

include_directories(${CMAKE_SOURCE_DIR} ${CMAKE_BINARY_DIR})

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)  # to find FindSparkle.cmake file

# Workaround: On Fedora will be executed cmake not mingw32-cmake for building Windows version
add_custom_target(update_version ALL
  COMMAND ${CMAKE_COMMAND} -DWIN32=${WIN32} -DAPP_VERSION="v${APP_VERSION}" -P "${CMAKE_SOURCE_DIR}/cmake/Version.cmake"
)

# Enable C++11
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

# Enable all warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic")

# If building a release version, make most warnings into errors
if(CMAKE_BUILD_TYPE STREQUAL Release OR CMAKE_BUILD_TYPE STREQUAL RelWithDbInfo)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror")
  
  # Earlier versions of GCC might not recognise certain pragma directives in the C++ code
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-error=pragmas")

  # #warning is not an error
  if(${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang" OR ${CMAKE_CXX_COMPILER_ID} STREQUAL "AppleClang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-#warnings")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-error=cpp")
  endif()
endif()

# Disable automatic conversions from 8-bit strings (char *) to unicode QStrings
add_definitions(-DQT_NO_CAST_FROM_ASCII)

option(ENABLE_DEBUG_LOG "Show debugging messages" OFF)
if(ENABLE_DEBUG_LOG)
  add_definitions(-DDEBUG)
endif()

if(APPLE)
  # Force Qt5 build on Mac OS X. Qt4 is not supported
  set(QT5_BUILD ON)
else()
  if(QT_QMAKE_EXECUTABLE AND EXISTS ${QT_QMAKE_EXECUTABLE})
    execute_process(COMMAND ${QT_QMAKE_EXECUTABLE} -query QT_VERSION OUTPUT_VARIABLE QT_VERSION)
  endif()

  if(QT_VERSION)
    if(QT_VERSION MATCHES "^5\\.")
      set(QT5_BUILD_DEFAULT ON)
    else()
      set(QT5_BUILD_DEFAULT OFF)
    endif()
  elseif(WIN32)
    set(QT5_BUILD_DEFAULT OFF)
  else()
    set(QT5_BUILD_DEFAULT ON)
  endif()

  option(QT5_BUILD "Force Qt5 build" ${QT5_BUILD_DEFAULT})
endif()

option(DISABLE_DONATION "Do not show donation text in About dialog" OFF)
if(DISABLE_DONATION)
  add_definitions(-DNO_DONATION)
endif()

if(UNIX)
  option(ENABLE_NVWA "Check for memory leaks" OFF)
else()
  # Not working correctly on Windows. See https://stackoverflow.com/questions/46025819/global-overload-operator-new-delete-in-mingw
  set(ENABLE_NVWA OFF)

  # For Windows by default build single binary without any dependencies
  option(BUILD_SHARED "Link with shared libraries" OFF)
endif()

if(ENABLE_NVWA)
  add_definitions(
    -DENABLE_NVWA
    -D_DEBUG_NEW_REDEFINE_NEW=0
    -D_DEBUG_NEW_ERROR_ACTION=
  )
endif()

if(QT5_BUILD)
  find_package(Qt5Core REQUIRED)
  find_package(Qt5Gui REQUIRED)
  find_package(Qt5Widgets REQUIRED)
  find_package(Qt5LinguistTools REQUIRED)

  add_definitions(-DHAVE_QT5)

  macro(QT4_ADD_TRANSLATION)
    qt5_add_translation(${ARGN})
  endmacro()

  macro(qt4_add_resources)
    qt5_add_resources(${ARGN})
  endmacro()

  macro(qt4_wrap_cpp)
    qt5_wrap_cpp(${ARGN})
  endmacro()

  macro(qt4_wrap_ui)
    qt5_wrap_ui(${ARGN})
  endmacro()
else()
  find_package(Qt4 REQUIRED)
  find_package(QJSON 0.8.0 REQUIRED)
  add_definitions("'-DQStringLiteral(str)=QString::fromUtf8(\"\" str \"\", sizeof(str) - 1)'")
  include_directories(${QJSON_INCLUDE_DIR})

  if(WIN32 AND (NOT BUILD_SHARED))
    string(REPLACE "-DQT_DLL" "" QT_DEFINITIONS "${QT_DEFINITIONS}")
    set(QT_DEFINITIONS ${QT_DEFINITIONS} "-DQT_NODLL")
    add_definitions(-DQJSON_STATIC)
  endif()

  include(${QT_USE_FILE})
endif()

if(WIN32 AND (NOT BUILD_SHARED))
  add_definitions(-DBUILD_STATIC)
endif()

macro(qt_add_translation ARG1 ARG2)
  if(QT5_BUILD)
    qt5_add_translation(${ARG1} ${ARG2})
  else()
    qt4_add_translation(${ARG1} ${ARG2})
  endif()
endmacro()

configure_file(translations.qrc.in translations.qrc COPYONLY)
if(WIN32)
  configure_file(qt_translations.qrc.in qt_translations.qrc COPYONLY)
endif()

# NOTE: make changes in qt_translations.qrc.in and translations.qrc.in also
set(LANGS af ar bn cs de en es fi fr he ko hu id it ja nl pl pt ro ru tr uk vi zh_CN zh_TW)

foreach(l ${LANGS})
  list(APPEND TRANSLATIONS ${CMAKE_SOURCE_DIR}/translations/torrentfileeditor_${l}.ts)
endforeach()

if(WIN32)
  if(QT5_BUILD)
    get_target_property(QT_QMAKE_EXECUTABLE Qt5::qmake IMPORTED_LOCATION)
    execute_process(COMMAND ${QT_QMAKE_EXECUTABLE} -query QT_INSTALL_TRANSLATIONS
                    OUTPUT_VARIABLE QT_TRANSLATIONS_DIR
                    OUTPUT_STRIP_TRAILING_WHITESPACE
    )
  endif()
  foreach(l ${LANGS})
    if(EXISTS "${QT_TRANSLATIONS_DIR}/qt_${l}.qm")
      configure_file("${QT_TRANSLATIONS_DIR}/qt_${l}.qm" "qt_${l}.qm" COPYONLY)
    endif()
  endforeach()
endif()

set(HEADERS
  ${CMAKE_SOURCE_DIR}/application.h
  ${CMAKE_SOURCE_DIR}/mainwindow.h
  ${CMAKE_SOURCE_DIR}/datewidget.h
  ${CMAKE_SOURCE_DIR}/lineeditwidget.h
  ${CMAKE_SOURCE_DIR}/urledit.h
  ${CMAKE_SOURCE_DIR}/folderedit.h
  ${CMAKE_SOURCE_DIR}/aboutdlg.h
  ${CMAKE_SOURCE_DIR}/bencodemodel.h
  ${CMAKE_SOURCE_DIR}/bencodedelegate.h
  ${CMAKE_SOURCE_DIR}/tableview.h
  ${CMAKE_SOURCE_DIR}/treeview.h
  ${CMAKE_SOURCE_DIR}/combobox.h
  ${CMAKE_SOURCE_DIR}/searchdlg.h
  ${CMAKE_SOURCE_DIR}/plaintextedit.h
)

if(WIN32)
  list(APPEND HEADERS
    ${CMAKE_SOURCE_DIR}/checkupdate.h
  )
endif()

set(PLAIN_HEADERS
  ${CMAKE_SOURCE_DIR}/bencode.h
  ${CMAKE_SOURCE_DIR}/abstracttreemodel.h
  ${CMAKE_SOURCE_DIR}/abstracttreenode.h
  ${CMAKE_SOURCE_DIR}/proxystyle.h
  ${CMAKE_BINARY_DIR}/config.h
)

# config.h is a generated file
set_source_files_properties(${CMAKE_BINARY_DIR}/config.h
  PROPERTIES GENERATED TRUE
  HEADER_FILE_ONLY TRUE
)

set(FORMS
  ${CMAKE_SOURCE_DIR}/mainwindow.ui
  ${CMAKE_SOURCE_DIR}/aboutdlg.ui
  ${CMAKE_SOURCE_DIR}/searchdlg.ui
)

set(SOURCES
  ${CMAKE_SOURCE_DIR}/application.cpp
  ${CMAKE_SOURCE_DIR}/main.cpp
  ${CMAKE_SOURCE_DIR}/mainwindow.cpp
  ${CMAKE_SOURCE_DIR}/bencode.cpp
  ${CMAKE_SOURCE_DIR}/datewidget.cpp
  ${CMAKE_SOURCE_DIR}/lineeditwidget.cpp
  ${CMAKE_SOURCE_DIR}/urledit.cpp
  ${CMAKE_SOURCE_DIR}/folderedit.cpp
  ${CMAKE_SOURCE_DIR}/aboutdlg.cpp
  ${CMAKE_SOURCE_DIR}/bencodemodel.cpp
  ${CMAKE_SOURCE_DIR}/bencodedelegate.cpp
  ${CMAKE_SOURCE_DIR}/proxystyle.cpp
  ${CMAKE_SOURCE_DIR}/tableview.cpp
  ${CMAKE_SOURCE_DIR}/treeview.cpp
  ${CMAKE_SOURCE_DIR}/combobox.cpp
  ${CMAKE_SOURCE_DIR}/searchdlg.cpp
  ${CMAKE_SOURCE_DIR}/plaintextedit.cpp
)

if(WIN32)
  list(APPEND SOURCES
    ${CMAKE_SOURCE_DIR}/checkupdate.cpp
  )
endif()

if(APPLE)
  list(APPEND SOURCES
    ${CMAKE_SOURCE_DIR}/cocoainitializer.mm
    ${CMAKE_SOURCE_DIR}/sparkleautoupdater.mm
  )

  find_package(Sparkle REQUIRED)
  include_directories(${SPARKLE_INCLUDE_DIR})
endif()

set(RESOURCES
  ${CMAKE_BINARY_DIR}/translations.qrc
  resources.qrc
)

if(WIN32)
  list(APPEND RESOURCES "${CMAKE_BINARY_DIR}/qt_translations.qrc")
endif()

QT4_ADD_TRANSLATION(QM ${TRANSLATIONS})
qt4_add_resources(QRC_SOURCES ${RESOURCES})
qt4_wrap_cpp(MOC_SOURCES ${HEADERS})
qt4_wrap_ui(UI_SOURCES ${FORMS})

if(WIN32)
  if(NOT BUILD_SHARED)
    set(START_STATIC -Wl,-Bstatic)
    set(END_STATIC z jpeg png tiff stdc++ -lwinpthread -Wl,-Bdynamic)
    set(QJSON_LIBRARIES qjson.a)
    if(QT5_BUILD)
      find_package(PkgConfig REQUIRED)
      pkg_check_modules(QT5_PKGCONFIG Qt5Widgets)

      set(EXTRA_LIBS
          Qt5::QWindowsIntegrationPlugin
          Qt5::QICOPlugin
          Qt5::QJpegPlugin
          Qt5::QGifPlugin
          Qt5::QWindowsVistaStylePlugin
          ${QT5_PKGCONFIG_LDFLAGS}
          Qt5AccessibilitySupport
          Qt5EventDispatcherSupport
          Qt5FontDatabaseSupport
          Qt5ThemeSupport
          jpeg
      )
    endif()
  endif()

  list(APPEND EXTRA_LIBS ws2_32 winmm imm32 wininet)

  # resource compilation for MinGW
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/app.o
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/app.rc
    COMMENT "Building RC object app.o"
    COMMAND ${CMAKE_RC_COMPILER} -I${CMAKE_CURRENT_SOURCE_DIR} -i${CMAKE_CURRENT_BINARY_DIR}/app.rc -o ${CMAKE_CURRENT_BINARY_DIR}/app.o
  )
  set(SOURCES ${SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/app.o)
endif()

if(ENABLE_NVWA)
  add_subdirectory(nvwa)
  set(NWVA_TARGET $<TARGET_OBJECTS:nvwa>)
  if(UNIX AND NOT APPLE)
    set(NWVA_LIBS -ldl)
  elseif(WIN32)
    set(NWVA_LIBS -ldbghelp)
  endif()
endif()
add_executable(${EXE_NAME} WIN32 MACOSX_BUNDLE ${QM} ${HEADERS} ${PLAIN_HEADERS} ${SOURCES} ${MOC_SOURCES} ${QRC_SOURCES} ${UI_SOURCES} ${NWVA_TARGET})

add_dependencies(${EXE_NAME} update_version)

if(QT5_BUILD)
  target_link_libraries(${EXE_NAME} ${START_STATIC} Qt5::Core Qt5::Gui Qt5::Widgets ${END_STATIC} ${EXTRA_LIBS} ${NWVA_LIBS})
else()
  target_link_libraries(${EXE_NAME} ${START_STATIC} ${QJSON_LIBRARIES} ${START_STATIC} ${QT_LIBRARIES} ${END_STATIC} ${EXTRA_LIBS} ${NWVA_LIBS})
endif()

if(APPLE)
  target_link_libraries(${EXE_NAME} ${SPARKLE_LIBRARY} "-framework AppKit")
endif()

add_subdirectory(translations)

if(UNIX AND NOT APPLE)
  install(TARGETS ${EXE_NAME} DESTINATION bin)
  install(FILES torrent-file-editor.desktop DESTINATION share/applications)
  install(FILES torrent-file-editor.appdata.xml DESTINATION share/appdata)
  install(FILES icons/app_16.png DESTINATION share/icons/hicolor/16x16/apps RENAME torrent-file-editor.png)
  install(FILES icons/app_32.png DESTINATION share/icons/hicolor/32x32/apps RENAME torrent-file-editor.png)
  install(FILES icons/app_48.png DESTINATION share/icons/hicolor/48x48/apps RENAME torrent-file-editor.png)
  install(FILES icons/app_64.png DESTINATION share/icons/hicolor/64x64/apps RENAME torrent-file-editor.png)
  install(FILES icons/app_128.png DESTINATION share/icons/hicolor/128x128/apps RENAME torrent-file-editor.png)
  install(FILES icons/app_256.png DESTINATION share/icons/hicolor/256x256/apps RENAME torrent-file-editor.png)
endif()

if(APPLE)
  install(TARGETS ${EXE_NAME} BUNDLE DESTINATION . COMPONENT Runtime)
  set_target_properties(torrent-file-editor PROPERTIES
    OUTPUT_NAME "${PROJECT_NAME}"
  )
  configure_file(MacOSXBundleInfo.plist.in "${CMAKE_BINARY_DIR}/MacOSXBundleInfo.plist.in")

  # Install app icon
  install(FILES icons/application.icns DESTINATION "${PROJECT_NAME}.app/Contents/Resources" COMPONENT Runtime)

  # Install needed Qt plugins by copying each pluging separately
  find_package(Qt5PrintSupport REQUIRED)
  set(PLATFORM_PLUGINS Qt5::QCocoaIntegrationPlugin)
  set(IMAGE_PLUGINS Qt5::QICNSPlugin)
  set(PRINTSUPPORT_PLUGINS Qt5::QCocoaPrinterSupportPlugin)
  set(PLUGINS_DEST_PATH "${PROJECT_NAME}.app/Contents/PlugIns")
  set(TRANSLATIONS_DEST_PATH "${PROJECT_NAME}.app/Contents/Resources/translations")

  foreach(plugin ${PLATFORM_PLUGINS})
    get_target_property(_loc ${plugin} LOCATION)
    install(FILES "${_loc}" DESTINATION ${PLUGINS_DEST_PATH}/platforms/ COMPONENT Runtime)
  endforeach()

  foreach(plugin ${IMAGE_PLUGINS})
    get_target_property(_loc ${plugin} LOCATION)
    install(FILES "${_loc}" DESTINATION ${PLUGINS_DEST_PATH}/imageformats/ COMPONENT Runtime)
  endforeach()

  foreach(plugin ${PRINTSUPPORT_PLUGINS})
    get_target_property(_loc ${plugin} LOCATION)
    install(FILES "${_loc}" DESTINATION ${PLUGINS_DEST_PATH}/printsupport/ COMPONENT Runtime)
  endforeach()

  if(TARGET Qt5::QMacStylePlugin)
    get_target_property(_loc Qt5::QMacStylePlugin LOCATION)
    install(FILES "${_loc}" DESTINATION ${PLUGINS_DEST_PATH}/styles/ COMPONENT Runtime)
  endif()

  # Install public key for Sparkle Updater
  install(FILES "dsa_pub.pem" DESTINATION "${PROJECT_NAME}.app/Contents/Resources" COMPONENT Runtime)

  # Install qt.conf file
  # Qt can't find plugins in standard path on case-sensetive file system
  # Write correct case-sensetive folder name in qt.conf file
  install(FILES "qt.conf" DESTINATION "${PROJECT_NAME}.app/Contents/Resources" COMPONENT Runtime)

  # Now the work of copying dependencies into the bundle/package
  # The quotes are escaped and variables to use at install time have their $ escaped
  # An alternative is the do a configure_file() on a script and use install(SCRIPT  ...).
  # Note that the image plugins depend on QtSvg and QtXml, and it got those copied
  # over.
  get_target_property(QT_LIBRARY_DIR Qt5::Core LOCATION)
  while(NOT ${QT_LIBRARY_DIR} MATCHES "lib$")
    get_filename_component(QT_LIBRARY_DIR ${QT_LIBRARY_DIR} DIRECTORY)
  endwhile()

  foreach(l ${LANGS})
    # In Qt lacks some languages
    if(EXISTS "${QT_LIBRARY_DIR}/../translations/qtbase_${l}.qm")
      install(FILES "${QT_LIBRARY_DIR}/../translations/qtbase_${l}.qm" DESTINATION ${TRANSLATIONS_DEST_PATH} COMPONENT Runtime)
    endif()
  endforeach()

  SET(APPS "\${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}.app")
  install(CODE "
    file(GLOB_RECURSE QTPLUGINS
      \"\${CMAKE_INSTALL_PREFIX}/${PLUGINS_DEST_PATH}/*${CMAKE_SHARED_LIBRARY_SUFFIX}\"
    )
    include(BundleUtilities)
    fixup_bundle(\"${APPS}\" \"\${QTPLUGINS}\" \"${QT_LIBRARY_DIR};${SPARKLE_LIBRARY}\")
    file(GLOB BINARIES
      \"${APPS}/Contents/Frameworks/*.framework\"
      \"${APPS}/Contents/Frameworks/Sparkle.framework/Resources/Autoupdate.app\"
      \"${APPS}/Contents/MacOS/*.dylib\"
      \"${APPS}/Contents/PlugIns/*.dylib\"
      \"${APPS}/Contents/PlugIns/*/*.dylib\"
    )
    execute_process(COMMAND codesign --deep -f -s \"Ivan Romanov\" \${BINARIES} \"${APPS}\" WORKING_DIRECTORY \"${APPS}\")
    " COMPONENT Runtime
  )

  # To Create a package, one can run "cpack -G DragNDrop CPackConfig.cmake" on Mac OS X
  # where CPackConfig.cmake is created by including CPack
  # And then there's ways to customize this as well
  set(CPACK_GENERATOR "DragNDrop")
  set(CPACK_DMG_VOLUME_NAME ${PROJECT_NAME})
  set(CPACK_DMG_DS_STORE_SETUP_SCRIPT "${CMAKE_SOURCE_DIR}/CMakeDMGSetup.scpt")
  set(CPACK_DMG_BACKGROUND_IMAGE "${CMAKE_SOURCE_DIR}/DS_Background.png")
  include(CPack)

  add_custom_target("dmg"
    COMMAND ${CMAKE_COMMAND} -DPROJECT_NAME=${PROJECT_NAME} -DAPP_NAME=${EXE_NAME} -P "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Dmg.cmake"
    DEPENDS "${EXE_NAME}"
    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}"
  )
endif()

# collect_info_files function
# Takes collect_info_files - list of non-source files to look for
# Returns INFO_FILES with all files found from the input list.
# Based on macro posted here
# http://lists.qt-project.org/pipermail/qt-creator/2012-August/001191.html [^]
function(collect_info_files)
	list(APPEND _all_found)
	foreach(_it ${ARGN})
		if(NOT IS_DIRECTORY ${_it})
			get_filename_component(_path ${_it} ABSOLUTE)
			if(EXISTS ${_path})
				list(APPEND _all_found ${_it})
				if(NOT ${_it} MATCHES "^/\\\\..*$;~$")
					set_source_files_properties(${_it} PROPERTIES HEADER_FILE_ONLY TRUE)
				endif()
			endif()
		endif()
	endforeach()
	set(INFO_FILES ${_all_found} PARENT_SCOPE)
endfunction()

# List all non-source code files (documents, configuration files, etc.)
set(OTHER_FILES
  .gitignore
  .travis.yml
  README.md
  cmake/Dmg.cmake
  appcast.xml.in
  CMakeDMGSetup.scpt
  cmake/Version.cmake
  build.sh
  PVS-Studio.cfg
  Plog-Converter.cfg
  translations.qrc.in
  qt_translations.qrc.in
  appcast.json.in
  build.sh
  update-translations.sh
)

# Collect all existing files and configure as non-sources.
collect_info_files(${OTHER_FILES})

# Create dummy target/project for an IDE with documents attached
add_custom_target(docs SOURCES ${INFO_FILES})

if(CMAKE_EXPORT_COMPILE_COMMANDS)
  if(NOT PVS_ANALYZER)
    find_program(PVS_ANALYZER pvs-studio-analyzer)
    find_program(PVS_CONVERTER plog-converter)

    if(PVS_ANALYZER AND PVS_CONVERTER)
      message(STATUS "Found PVS-Studio: ${PVS_ANALYZER}")
    endif()
  endif()

  mark_as_advanced(PVS_ANALYZER PVS_CONVERTER)
  if(PVS_ANALYZER AND PVS_CONVERTER)
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
      set(PVS_PREPROCESSOR "clang")
    else()
      set(PVS_PREPROCESSOR "gcc")
    endif()

    include(ProcessorCount)
    ProcessorCount(N)

    add_custom_target(analyze
                      COMMAND ${PVS_ANALYZER} analyze
                              -o PVS-Studio.log.raw
                              --preprocessor=${PVS_PREPROCESSOR}
                              -e ${CMAKE_BINARY_DIR}
                              -c ${CMAKE_SOURCE_DIR}/PVS-Studio.cfg
                              -j ${N}
                      COMMAND ${PVS_CONVERTER} -t errorfile PVS-Studio.log.raw -o PVS-Studio.log -s ${CMAKE_SOURCE_DIR}/Plog-Converter.cfg
                      COMMAND rm -f PVS-Studio.log.raw
                      COMMAND sed -i "'/^www\\.viva64\\.com/d'" PVS-Studio.log
                      COMMAND cat PVS-Studio.log 1>&2
                      COMMENT "Analyze ${EXE_NAME}"
                      DEPENDS ${EXE_NAME}
    )
  endif()
endif()
