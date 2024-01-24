cmake_minimum_required(VERSION 3.10)
project(libide VERSION 0.0.1)

set(CMAKE_EXPORT_COMPILE_COMMANDS True)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})

find_program(GLIB_COMPILE_RESOURCES NAMES glib-compile-resources)
find_program(GLIB_GENMARSHAL NAMES glib-genmarshal)
find_package(PkgConfig REQUIRED)
pkg_check_modules(GTK4 REQUIRED gtk4)
pkg_check_modules(adwaita REQUIRED libadwaita-1)
pkg_check_modules(libdex REQUIRED libdex-1)
pkg_check_modules(libpeas REQUIRED libpeas-2)
pkg_check_modules(gtksourceview REQUIRED gtksourceview-5)
pkg_check_modules(libvte REQUIRED vte-2.91)
pkg_check_modules(libgio REQUIRED gio-2.0)
pkg_check_modules(libgiounix REQUIRED gio-unix-2.0)

include_directories(${GTK4_INCLUDE_DIRS} ${adwait_INCLUDE_DIRS} ${libpeas_INCLUDE_DIRS} ${gtksourceview_INCLUDE_DIRS} ${libgmodule_INCLUDE_DIRS} ${libdex_INCLUDE_DIRS} ${libgio_INCLUDE_DIRS} ${libgiounix_INCLUDE_DIRS})
link_directories(${GTK4_LIBRARY_DIRS} ${adwait_LIBRARY_DIRS} ${libpeas_LIBRARY_DIRS} ${gtksourceview_LIBRARY_DIRS} ${libgmodule_LIBRARY_DIRS} ${libdex_LIBRARY_DIRS} ${libgio_LIBRARY_DIRS} ${libgiounix_LIBRARY_DIRS})
add_definitions(${GTK_CFLAGS_OTHER} ${adawit_CFLAGS_OTHER} ${libpeas_CFLAGS_OTHER} ${gtksourceview_CFLAGS_OTHER} ${libgmodule_CFLAGS_OTHER} ${libdex_CFLAGS_OTHER} ${libgio_CFLAGS_OTHER} ${libgiounix_CFLAGS_OTHER})


set(UI_RESOURCE_TARGET ${CMAKE_CURRENT_SOURCE_DIR}/src/ui_resources.c)

add_custom_command(OUTPUT ${UI_RESOURCE_TARGET}
  COMMAND ${GLIB_COMPILE_RESOURCES} ${CMAKE_CURRENT_SOURCE_DIR}/src/manul.gresource.xml --target=${UI_RESOURCE_TARGET} --generate-source --sourcedir=${CMAKE_CURRENT_SOURCE_DIR}/src
  COMMENT "Generate resourcefile "
  DEPEND ${CMAKE_CURRENT_SOURCE_DIR}/src/manul.gresource.xml
)

set(IDE_CORE_MARSHAL_TARGET ${CMAKE_CURRENT_SOURCE_DIR}/src/core/ide-marshal.h)

add_custom_command(OUTPUT ${IDE_CORE_MARSHAL_TARGET}
  COMMAND ${GLIB_GENMARSHAL}  --header --prefix=ide_marshal --internal --valist-marshallers --outpu=${IDE_CORE_MARSHAL_TARGET}  ${CMAKE_CURRENT_SOURCE_DIR}/src/core/ide-marshal.defs
)


message(STATUS "${GLIB_COMPILE_RESOURCES}")
message(STATUS "${CMAKE_CURRENT_SOURCE_DIR}/src/manul.gresource.xml")
message(STATUS "${UI_RESOURCE_TARGET}")
message(STATUS "${CMAKE_CURRENT_SOURCE_DIR}/src")
message(STATUS "${CMAKE_CURRENT_BINARY_DIR}")

#######################################################
#                                                     #
#                 IDE CORE STATIC LIB                 #
#                                                     #
#                                                     #
#######################################################

set(APPLICATION_ID "org.gtk.libide")
set(PACKAGE_NAME "gtk_libide")
configure_file(config.h.in config.h)

set(BUILD_TYPE release)
set(BUILD_CHANNEL other)
set(MAJOR_VERSION 46)
set(MINOR_VERSION 1)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/core/ide-version.h.in ide-version.h)

set(ENABLE_TRACING 1)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/core/ide-debug.h.in ide-debug.h)

list(APPEND LIBIDE_CORE_SRC
  ide-action-muxer.c
  ide-context.c
  ide-global.c
  ide-gsettings-action-group.c
  ide-log.c
  ide-log-item.c
  ide-notification.c
  ide-notifications.c
  ide-object-box.c
  ide-object-notify.c
  ide-object.c
  ide-property-action-group.c
  ide-settings-flag-action.c
  ide-settings.c
  ide-transfer-manager.c
  ide-transfer.c
  gsettings-mapping.c
  ide-layered-settings.c
  ide-log-model.c
)
list(TRANSFORM LIBIDE_CORE_SRC PREPEND ${CMAKE_CURRENT_SOURCE_DIR}/src/core/)

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/core/)

add_library(libide_core STATIC ${LIBIDE_CORE_SRC} ${IDE_CORE_MARSHAL_TARGET})
target_compile_definitions(libide_core PRIVATE -DIDE_CORE_COMPILATION)





#######################################################
#                                                     #
#                 IDE THREAD STATIC LIB               #
#                                                     #
#                                                     #
#######################################################



list(APPEND LIBIDE_THREADING_SRC
  ide-cancellable.c
  ide-environment-variable.c
  ide-environment.c
  ide-search-path.c
  ide-gtask.c
  ide-subprocess-launcher.c
  ide-subprocess-supervisor.c
  ide-subprocess.c
  ide-task.c
  ide-thread-pool.c
  ide-unix-fd-map.c
  ide-flatpak-subprocess.c
  ide-simple-subprocess.c
)
list(TRANSFORM LIBIDE_THREADING_SRC PREPEND ${CMAKE_CURRENT_SOURCE_DIR}/src/threading/)

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/core/)

# _GNU_SOURCE  access some unportable api like pipe2
add_library(libide_threading STATIC ${LIBIDE_THREADING_SRC})
target_compile_definitions(libide_threading PRIVATE -DIDE_THREADING_COMPILATION -D_GNU_SOURCE)


#add_executable(manul ${MANUL_SRC})
#target_link_libraries(manul ${GTK4_LIBRARIES} ${adwaita_LIBRARIES} ${libpeas_LIBRARIES} ${gtksourceview_LIBRARIES})
