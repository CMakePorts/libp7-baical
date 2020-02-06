cmake_minimum_required(VERSION 3.10)

##########################
#  Project configuration #
##########################

project(P7)
include(GenerateExportHeader)
include(CMakePackageConfigHelpers)

set(P7_MAJOR_VERSION 5)
set(P7_MINOR_VERSION 2)
set(P7_PATCH_VERSION 0)
set(P7_VERSION ${P7_MAJOR_VERSION}.${P7_MINOR_VERSION}.${P7_PATCH_VERSION})

option(BUILD_SHARED_LIBS "Build shared libs" OFF)
option(BUILD_TESTS "Build test" ON)
option(BUILD_EXAMPLES "Build examples" ON)

##########################
#  Sources configuration #
##########################

set(HEADERS "Headers/GTypes.h"
            "Headers/P7_Client.h"
            "Headers/P7_Cproxy.h"
            "Headers/P7_Extensions.h"
            "Headers/P7_Telemetry.h"
            "Headers/P7_Trace.h"
            "Headers/P7_Version.h"
)

set(SOURCES "Sources/ClBaical.cpp"
            "Sources/ClFile.cpp"
            "Sources/Client.cpp"
            "Sources/ClNull.cpp"
            "Sources/ClText.cpp"
            "Sources/CRC32.cpp"
            "Sources/Proxy.cpp"
            "Sources/Telemetry.cpp"
            "Sources/Trace.cpp"
)


set(SOURCES_TEST_SPEED "Tests/Speed/Speed.cpp")

set(SOURCES_TEST_SPEEDPARALLEL "Tests/Speed/SpeedParallel.cpp")

set(SOURCES_TEST_TRACE "Tests/Trace/Main.cpp")

if(WIN32)
  set(PLATFORM_DIR "Shared/Platforms/Windows_x86/")
  set(PLATFORM_LIBS "ws2_32")
  set(RESOURCES "Sources/P7Client.rc")
else()
  set(PLATFORM_DIR "Shared/Platforms/Linux_x86/")
  set(PLATFORM_LIBS pthread rt)
  set(RESOURCES "")
endif()

set(PLATFORM_HEADERS
  ${PLATFORM_DIR}/PAtomic.h
  ${PLATFORM_DIR}/PConsole.h
  ${PLATFORM_DIR}/PCRC32.h
  ${PLATFORM_DIR}/PFile.h
  ${PLATFORM_DIR}/PFileSystem.h
  ${PLATFORM_DIR}/PJournal.h
  ${PLATFORM_DIR}/PLock.h
  ${PLATFORM_DIR}/PMEvent.h
  ${PLATFORM_DIR}/PPerformanceInfo.h
  ${PLATFORM_DIR}/PProcess.h
  ${PLATFORM_DIR}/PShared.h
  ${PLATFORM_DIR}/PSharedLib.h
  ${PLATFORM_DIR}/PSignal.h
  ${PLATFORM_DIR}/PSocket.h
  ${PLATFORM_DIR}/PString.h
  ${PLATFORM_DIR}/PSystem.h
  ${PLATFORM_DIR}/PThreadShell.h
  ${PLATFORM_DIR}/PTime.h
  Shared/AList.h
  Shared/CRC32.h
  Shared/DnsResolver.h
  Shared/IConsole.h
  Shared/IFile.h
  Shared/IJournal.h
  Shared/IMEvent.h
  Shared/IPerformanceInfo.h
  Shared/ISignal.h
  Shared/Length.h
  Shared/Lock.h
  Shared/Predicate.h
  Shared/QStringHelper.h
  Shared/RBTree.h
  Shared/SharedLib.h
  Shared/Ticks.h
  Shared/UDP_NB.h
  Shared/UTF.h
  Shared/WString.h
)

if(NOT WIN32)
  list(APPEND PLATFORM_HEADERS 
    ${PLATFORM_DIR}/PCrashDump.h
    ${PLATFORM_DIR}/PProfiler.h
  )
endif()

##########################
#    P7 configuration    #
##########################

add_library(P7 "${SOURCES}" "${HEADERS}" "${PLATFORM_HEADERS}" "${RESOURCES}")

#Headers configuration
generate_export_header(P7
  EXPORT_FILE_NAME "P7_Export.h")

list(APPEND HEADERS "${CMAKE_CURRENT_BINARY_DIR}/P7_Export.h")

target_include_directories(P7 PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Headers>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
  $<INSTALL_INTERFACE:include/P7>
)

target_include_directories(P7 PRIVATE
  "Shared"
)

#Build configuration
target_include_directories(P7 PRIVATE ${PLATFORM_DIR})
if(BUILD_SHARED_LIBS)
  target_link_libraries(P7 PRIVATE ${PLATFORM_LIBS})
else()
  target_link_libraries(P7 PUBLIC ${PLATFORM_LIBS})
endif()

if(WIN32)
  set_target_properties(P7 PROPERTIES PREFIX "")
  if(CMAKE_SIZEOF_VOID_P EQUAL 8)   
    set_target_properties(P7 PROPERTIES DEBUG_POSTFIX "x64d")
    set_target_properties(P7 PROPERTIES RELEASE_POSTFIX "x64")
    set_target_properties(P7 PROPERTIES RELWITHDEBINFO_POSTFIX "x64")
    set_target_properties(P7 PROPERTIES MINSIZEREL_POSTFIX "x64")
  else()
    set_target_properties(P7 PROPERTIES DEBUG_POSTFIX "x32d")
    set_target_properties(P7 PROPERTIES RELEASE_POSTFIX "x32")
    set_target_properties(P7 PROPERTIES RELWITHDEBINFO_POSTFIX "x32")
    set_target_properties(P7 PROPERTIES MINSIZEREL_POSTFIX "x32")
  endif()
else()
  set_target_properties(P7 PROPERTIES PREFIX "lib")
  set_target_properties(P7 PROPERTIES POSTFIX "")
  set_target_properties(P7 PROPERTIES DEBUG_POSTFIX "d")
endif()

#Cmake config generation
configure_package_config_file(
  "CMake/libp7-baical-config.cmake.in"
  "libp7-baical-config.cmake"
  INSTALL_DESTINATION "lib/libp7-baical"
)

write_basic_package_version_file(
  "libp7-baical-config-version.cmake"
  VERSION ${P7_VERSION}
  COMPATIBILITY AnyNewerVersion)

install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/libp7-baical-config.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/libp7-baical-config-version.cmake"
  DESTINATION "lib/libp7-baical"
)

##########################
#         Tests          #
##########################

if(BUILD_TESTS)

  add_executable(P7-Test-Speed ${SOURCES_TEST_SPEED})
  #add_executable(P7-Test-SpeedParallel ${SOURCES_TEST_SPEEDPARALLEL})
  add_executable(P7-Test-Trace ${SOURCES_TEST_TRACE})

  target_link_libraries(P7-Test-Speed P7)
  #target_link_libraries(P7-Test-SpeedParallel P7)
  target_link_libraries(P7-Test-Trace P7)

  target_include_directories(P7-Test-Speed PRIVATE "Shared")
  #target_include_directories(P7-Test-SpeedParallel PRIVATE "Shared")
  target_include_directories(P7-Test-Trace PRIVATE "Shared")

  if(MSVC)
    target_compile_definitions(P7-Test-Speed PRIVATE "-D_CRT_SECURE_NO_WARNINGS")
    #target_compile_definitions(P7-Test-SpeedParallel "-D_CRT_SECURE_NO_WARNINGS")
    target_compile_definitions(P7-Test-Trace PRIVATE "-D_CRT_SECURE_NO_WARNINGS")
  endif()

  if(WIN32)
    target_include_directories(P7-Test-Speed PRIVATE "Shared/Platforms/Windows_x86/")
    #target_include_directories(P7-Test-SpeedParallel PRIVATE "Shared/Platforms/Windows_x86/")
    target_include_directories(P7-Test-Trace PRIVATE "Shared/Platforms/Windows_x86/")
  else()
    target_include_directories(P7-Test-Speed PRIVATE "Shared/Platforms/Linux_x86/")
    #target_include_directories(P7-Test-SpeedParallel PRIVATE "Shared/Platforms/Linux_x86/")
    target_include_directories(P7-Test-Trace PRIVATE "Shared/Platforms/Linux_x86/")
  endif()
endif()

if(BUILD_EXAMPLES)
   add_subdirectory(Examples)
endif(BUILD_EXAMPLES)

##########################
#      Installation      #
##########################

install(
  TARGETS P7
  EXPORT libp7-baical-targets
  RUNTIME DESTINATION "bin"
  ARCHIVE DESTINATION "lib"
  LIBRARY DESTINATION "lib"
)

install(
  EXPORT libp7-baical-targets 
  NAMESPACE libp7-baical::
  DESTINATION "lib/libp7-baical"
)

install(
  FILES ${HEADERS}
  DESTINATION "include/P7"
)

if(BUILD_TESTS)
  install(
    TARGETS 
      P7-Test-Speed
     #P7-Test-SpeedParallel
      P7-Test-Trace
    RUNTIME DESTINATION "bin"
  )
endif()
