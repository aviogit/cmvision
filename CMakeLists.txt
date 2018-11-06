cmake_minimum_required(VERSION 2.8.3)
project(cmvision)

if(NOT WIN32)
  string(ASCII 27 Esc)
  set(Reset       "${Esc}[m")
  set(Bold        "${Esc}[1m")
  set(Red         "${Esc}[31m")
  set(Green       "${Esc}[32m")
  set(Yellow      "${Esc}[33m")
  set(Blue        "${Esc}[34m")
  set(Magenta     "${Esc}[35m")
  set(Cyan        "${Esc}[36m")
  set(White       "${Esc}[37m")
  set(BoldRed     "${Esc}[1;31m")
  set(BoldGreen   "${Esc}[1;32m")
  set(BoldYellow  "${Esc}[1;33m")
  set(BoldBlue    "${Esc}[1;34m")
  set(BoldMagenta "${Esc}[1;35m")
  set(BoldCyan    "${Esc}[1;36m")
  set(BoldWhite   "${Esc}[1;37m")
endif()

MESSAGE("")
MESSAGE("-= ${BoldYellow}CMAKE ${PROJECT_NAME} BEGIN${Reset} =-")
MESSAGE("")


find_package(catkin REQUIRED 
  COMPONENTS 
  cv_bridge
  message_generation
  roscpp
  sensor_msgs
  std_msgs
)

add_message_files(
  FILES
  Blob.msg
  Blobs.msg
)

# Generate added messages and services with any dependencies listed here
generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(CATKIN_DEPENDS message_runtime std_msgs)

include_directories(${catkin_INCLUDE_DIRS})

message("Finding wxWidgets libraries, if not found, please do: ${BoldBlue}sudo apt install libwxgtk3.0-dev${Reset}")
message("")


execute_process(COMMAND bash -c "grep -a5 \"# Check if a specfic version was requested by find_package\" /usr/share/cmake-3.10/Modules/FindwxWidgets.cmake | grep \"unset(_filename)\"" OUTPUT_VARIABLE GREP_OUTPUT RESULT_VARIABLE RETVAL)
#string(REGEX REPLACE "\n" "" GREP_OUTPUT ${GREP_OUTPUT})
#message("${GREP_OUTPUT} ${RETVAL}")

if (${RETVAL} EQUAL 1)

message("According to what I see inside /usr/share/cmake-3.10/Modules/FindwxWidgets.cmake, you should now encounter an error.")
message("--------------------------------------------------------------------------------------------------------------------")
message("If you see this error (https://github.com/Kitware/CMake/pull/320):")
message("")
message("${BoldRed}CMake Error at /usr/share/cmake-3.10/Modules/FindwxWidgets.cmake:861 (file):")
message("  file failed to open for reading (No such file or directory):${Reset}")
message("")
message("${BoldYellow}Please apply the provided patch in the following way (remove the --dry-run to actually perform the changes when sure):${Reset}")
message("")
message("")
message("${BoldGreen}#> roscd ${PROJECT_NAME} ; sudo patch -d/usr/share/cmake-3.10/Modules -p0 --verbose --dry-run < cmake-findwx-widgets-patch/patch-usr-share-cmake-3.10-Modules-FindwxWidgets-cmake-file-failed-to-open-for-reading-No-such-file-or-directory-20180529-151452.txt${Reset}")
message("Hmm...  Looks like a unified diff to me...")
message("The text leading up to this was:")
message("--------------------------")
message("|--- FindwxWidgets.cmake	2016-03-24 15:12:21.000000000 +0100")
message("|+++ /usr/share/cmake-3.10/Modules/FindwxWidgets.cmake	2018-04-04 16:17:30.651275499 +0200")
message("--------------------------")
message("File FindwxWidgets.cmake is read-only; trying to patch anyway")
message("checking file FindwxWidgets.cmake")
message("Using Plan A...")
message("Hunk #1 succeeded at 851.")
message("done")
message("--------------------------------------------------------------------------------------------------------------------")
message("Ok, now you should have the real error, to solve it, apply the provided patch.")
message("")
message("")

endif()


find_package(wxWidgets REQUIRED)
include(${wxWidgets_USE_FILE})
include_directories(${wxWidgets_INCLUDE_DIRS})

set(CMAKE_CXX_FLAGS         "${CMAKE_CXX_FLAGS}         -I/usr/lib/x86_64-linux-gnu/wx/include/gtk2-unicode-3.0 -I/usr/include/wx-3.0 -D_FILE_OFFSET_BITS=64 -DWXUSINGDLL -D__WXGTK__ -pthread")

find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

add_executable(cmvision
	src/cmvision.cc
	src/conversions.c
	src/cmvision_color_blob_finder.cpp
	src/main.cpp
)
add_dependencies(cmvision
  cmvision_gencpp
)
target_link_libraries(cmvision 
  ${catkin_LIBRARIES}
  ${OpenCV_LIBRARIES}
)

add_executable(colorgui
	src/cmvision.cc
	src/conversions.c
	src/color_gui.cpp
)
target_link_libraries(colorgui 
  fltk
  ${catkin_LIBRARIES}
  ${OpenCV_LIBRARIES}
  ${wxWidgets_LIBRARIES}
)

# Mark executables and/or libraries for installation
install(TARGETS cmvision colorgui
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(FILES cmvision.launch colors.txt
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

MESSAGE("-= ${BoldYellow}CMAKE ${PROJECT_NAME} END${Reset} =-")
MESSAGE("")

