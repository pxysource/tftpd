# Tftpd64 / Tftpd32 v4.74 — CMake build (mirrors Visual Studio projects in src/)
cmake_minimum_required(VERSION 3.20)

project(tftpd64
  VERSION 4.74
  LANGUAGES C
)

if(NOT WIN32)
  message(FATAL_ERROR "This tree targets Windows (Winsock, GUI, service).")
endif()

set(TFTPD_SRC_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/src")

# Match MSVC *.vcxproj: Win32 -> libs.lib, x64 -> libs.x64.lib; exe names tftpd32.exe vs tftpd32.x64.exe
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(TFTPD_LIBS_OUTPUT_BASENAME "libs.x64")
  set(TFTPD_WIN_SUBSYS_VER "5.02")
  set(TFTPD_EXE_SUFFIX ".x64")
else()
  set(TFTPD_LIBS_OUTPUT_BASENAME "libs")
  set(TFTPD_WIN_SUBSYS_VER "5.01")
  set(TFTPD_EXE_SUFFIX "")
endif()

# Debug: /MDd, Release: /MT (matches *.vcxproj)
function(tftpd_apply_msvc_runtime target)
  set_target_properties("${target}" PROPERTIES
    MSVC_RUNTIME_LIBRARY "$<$<CONFIG:Debug>:MultiThreadedDebugDLL>$<$<NOT:$<CONFIG:Debug>>:MultiThreaded>"
  )
endfunction()

# ----- static library libs (libs.vcxproj) -----
add_library(libs STATIC
  "${TFTPD_SRC_ROOT}/_libs/GUI/centerchild.c"
  "${TFTPD_SRC_ROOT}/_libs/GUI/Cmsgbox.c"
  "${TFTPD_SRC_ROOT}/_libs/dump/dump.c"
  "${TFTPD_SRC_ROOT}/_libs/event_log/eventlog.c"
  "${TFTPD_SRC_ROOT}/_libs/lasterr/lasterr.c"
  "${TFTPD_SRC_ROOT}/_libs/linked_list/linked_list.c"
  "${TFTPD_SRC_ROOT}/_libs/GUI/listview.c"
  "${TFTPD_SRC_ROOT}/_libs/log/LogToMonitor.c"
  "${TFTPD_SRC_ROOT}/_libs/md5/md5.c"
  "${TFTPD_SRC_ROOT}/_libs/mini_tcp4u/mini_tcp4u.c"
  "${TFTPD_SRC_ROOT}/_libs/ping_api/ping_api.c"
  "${TFTPD_SRC_ROOT}/_libs/registry/registry.c"
  "${TFTPD_SRC_ROOT}/_libs/scandir/scandir.c"
  "${TFTPD_SRC_ROOT}/_libs/mini_tcp4u/tcp4u_exchg.c"
  "${TFTPD_SRC_ROOT}/_libs/mini_tcp4u/udp4u.c"
  "${TFTPD_SRC_ROOT}/_libs/GUI/window.c"
)
set_target_properties(libs PROPERTIES
  OUTPUT_NAME "${TFTPD_LIBS_OUTPUT_BASENAME}"
)
target_compile_definitions(libs PRIVATE MSVC)
target_include_directories(libs PRIVATE "${TFTPD_SRC_ROOT}")
tftpd_apply_msvc_runtime(libs)

# ----- tftpd32 standalone (tftpd32.vcxproj) -----
add_executable(tftpd32 WIN32
  "${TFTPD_SRC_ROOT}/_common/dialog_common.c"
  "${TFTPD_SRC_ROOT}/_common/setinifilename.c"
  "${TFTPD_SRC_ROOT}/_gui/about.c"
  "${TFTPD_SRC_ROOT}/_gui/browse.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_bootpd.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_bootpd_settings.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_dialog.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_dns.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_gauges.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_log.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_main.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_move_win.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_settings.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_syslogd.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_tftpd.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_tftp_dir.c"
  "${TFTPD_SRC_ROOT}/_gui/tftp_cli.c"
  "${TFTPD_SRC_ROOT}/_main/cmd_line.c"
  "${TFTPD_SRC_ROOT}/_main/main.c"
  "${TFTPD_SRC_ROOT}/_main/outofthebox.c"
  "${TFTPD_SRC_ROOT}/_main/firewall.c"
  "${TFTPD_SRC_ROOT}/_services/async_log.c"
  "${TFTPD_SRC_ROOT}/_services/bootpd.c"
  "${TFTPD_SRC_ROOT}/_services/bootpd_console.c"
  "${TFTPD_SRC_ROOT}/_services/bootpd_util.c"
  "${TFTPD_SRC_ROOT}/_services/console_actions.c"
  "${TFTPD_SRC_ROOT}/_services/dns.c"
  "${TFTPD_SRC_ROOT}/_services/ip_thread.c"
  "${TFTPD_SRC_ROOT}/_services/ip_util.c"
  "${TFTPD_SRC_ROOT}/_services/registry_thread.c"
  "${TFTPD_SRC_ROOT}/_services/service_main.c"
  "${TFTPD_SRC_ROOT}/_services/settings.c"
  "${TFTPD_SRC_ROOT}/_services/sntpd.c"
  "${TFTPD_SRC_ROOT}/_services/start_threads.c"
  "${TFTPD_SRC_ROOT}/_services/syslogd.c"
  "${TFTPD_SRC_ROOT}/_services/tftpd_main.c"
  "${TFTPD_SRC_ROOT}/_services/tftpd_thread.c"
  "${TFTPD_SRC_ROOT}/_services/tftpd_util.c"
  "${TFTPD_SRC_ROOT}/_services/tftp_dbg.c"
  "${TFTPD_SRC_ROOT}/_services/to_console.c"
  "${TFTPD_SRC_ROOT}/_gui/tftpd32.rc"
)
set_target_properties(tftpd32 PROPERTIES OUTPUT_NAME "tftpd32${TFTPD_EXE_SUFFIX}")
target_compile_definitions(tftpd32 PRIVATE MSVC TFTPD32 STANDALONE_EDITION)
target_include_directories(tftpd32 PRIVATE "${TFTPD_SRC_ROOT}" "${TFTPD_SRC_ROOT}/_common")
tftpd_apply_msvc_runtime(tftpd32)
target_link_libraries(tftpd32 PRIVATE
  libs
  shell32
  ws2_32
  comctl32
  iphlpapi
  mswsock
)
target_link_options(tftpd32 PRIVATE
  "/SUBSYSTEM:WINDOWS,${TFTPD_WIN_SUBSYS_VER}"
)

# ----- tftpd32 GUI service edition (tftpd32_gui.vcxproj) -----
add_executable(tftpd32_gui WIN32
  "${TFTPD_SRC_ROOT}/_common/dialog_common.c"
  "${TFTPD_SRC_ROOT}/_common/setinifilename.c"
  "${TFTPD_SRC_ROOT}/_gui/about.c"
  "${TFTPD_SRC_ROOT}/_gui/browse.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_bootpd.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_bootpd_settings.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_dialog.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_dns.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_gauges.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_log.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_main.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_move_win.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_settings.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_syslogd.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_tftpd.c"
  "${TFTPD_SRC_ROOT}/_gui/gui_tftp_dir.c"
  "${TFTPD_SRC_ROOT}/_gui/tftp_cli.c"
  "${TFTPD_SRC_ROOT}/_main/cmd_line.c"
  "${TFTPD_SRC_ROOT}/_main/GUI main.c"
  "${TFTPD_SRC_ROOT}/_main/service edition.c"
  "${TFTPD_SRC_ROOT}/_gui/tftpd32.rc"
)
set_target_properties(tftpd32_gui PROPERTIES OUTPUT_NAME "tftpd32_gui${TFTPD_EXE_SUFFIX}")
target_compile_definitions(tftpd32_gui PRIVATE MSVC TFTPD32 SERVICE_EDITION)
target_include_directories(tftpd32_gui PRIVATE "${TFTPD_SRC_ROOT}" "${TFTPD_SRC_ROOT}/_common")
tftpd_apply_msvc_runtime(tftpd32_gui)
target_link_libraries(tftpd32_gui PRIVATE
  libs
  shell32
  ws2_32
  comctl32
  iphlpapi
)
target_link_options(tftpd32_gui PRIVATE
  "/SUBSYSTEM:WINDOWS,${TFTPD_WIN_SUBSYS_VER}"
)

# ----- tftpd32 Windows service / console (tftpd32_svc.vcxproj) -----
add_executable(tftpd32_svc
  "${TFTPD_SRC_ROOT}/_common/dialog_common.c"
  "${TFTPD_SRC_ROOT}/_common/setinifilename.c"
  "${TFTPD_SRC_ROOT}/_main/service edition.c"
  "${TFTPD_SRC_ROOT}/_main/service stuff.c"
  "${TFTPD_SRC_ROOT}/_main/Svc main.c"
  "${TFTPD_SRC_ROOT}/_services/async_log.c"
  "${TFTPD_SRC_ROOT}/_services/bootpd.c"
  "${TFTPD_SRC_ROOT}/_services/bootpd_console.c"
  "${TFTPD_SRC_ROOT}/_services/bootpd_util.c"
  "${TFTPD_SRC_ROOT}/_services/console_actions.c"
  "${TFTPD_SRC_ROOT}/_services/dns.c"
  "${TFTPD_SRC_ROOT}/_services/ip_thread.c"
  "${TFTPD_SRC_ROOT}/_services/ip_util.c"
  "${TFTPD_SRC_ROOT}/_services/registry_thread.c"
  "${TFTPD_SRC_ROOT}/_services/service_main.c"
  "${TFTPD_SRC_ROOT}/_services/settings.c"
  "${TFTPD_SRC_ROOT}/_services/sntpd.c"
  "${TFTPD_SRC_ROOT}/_services/start_threads.c"
  "${TFTPD_SRC_ROOT}/_services/syslogd.c"
  "${TFTPD_SRC_ROOT}/_services/tftpd_main.c"
  "${TFTPD_SRC_ROOT}/_services/tftpd_thread.c"
  "${TFTPD_SRC_ROOT}/_services/tftpd_util.c"
  "${TFTPD_SRC_ROOT}/_services/tftp_dbg.c"
  "${TFTPD_SRC_ROOT}/_services/to_console.c"
  "${TFTPD_SRC_ROOT}/_services/tftpd32.rc"
)
set_target_properties(tftpd32_svc PROPERTIES OUTPUT_NAME "tftpd32_svc${TFTPD_EXE_SUFFIX}")
target_compile_definitions(tftpd32_svc PRIVATE MSVC TFTPD32 SERVICE_EDITION)
target_include_directories(tftpd32_svc PRIVATE "${TFTPD_SRC_ROOT}" "${TFTPD_SRC_ROOT}/_common")
tftpd_apply_msvc_runtime(tftpd32_svc)
target_link_libraries(tftpd32_svc PRIVATE
  libs
  shell32
  ws2_32
  comctl32
  iphlpapi
)
target_link_options(tftpd32_svc PRIVATE
  "/SUBSYSTEM:CONSOLE,${TFTPD_WIN_SUBSYS_VER}"
)
