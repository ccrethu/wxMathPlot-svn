# Main wxMathPlot CMakeLists.txt
# Manage project
#
# Author: Davide Rondini
# Maintainer: Lars Easior
# Last Update: 2022-04-15
# License: wxWindows license

# Set CMake flags to enable compatibility both with 2.4 and 2.6
cmake_minimum_required(VERSION 2.6...3.13)

project(wxMathPlot
LANGUAGES CXX
HOMEPAGE_URL https://github.invalid/easior/wxMathPlot
DESCRIPTION "wxMathPlot is a framework for mathematical graph plotting in wxWindows.")
set(wxMathPlot_VERSION_MAJOR 0)
set(wxMathPlot_VERSION_MINOR 2)
set(wxMathPlot_VERSION_BUILD 0)

include_directories(include)

# Create options to be chosen by the user:
if(UNIX)
    set(LINUX_64_32_CROSSCOMPILE OFF CACHE BOOL "Cross compile from Linux x86_64 to Linux x86?")
    set(GDB_DEBUG OFF CACHE BOOL "Build with gdb debugger support?")
else(UNIX)
    # Windows option to build using UNICODE
    set(MATHPLOT_UNICODE ON CACHE BOOL "Build using Unicode wxWidgets build?")
endif(UNIX)
# Option for any platform
set(MATHPLOT_SHARED ON CACHE BOOL "Create wxMathPlot as a shared library?")
set(MATHPLOT_DO_LOGGING OFF CACHE BOOL "Build with verbose debugging messages?")

# Important note on Unicode:
# On Windows, at least with Visual C++, it needs to be set macro UNICODE and _UNICODE
# in order to link correctly the code, o a lot of undefined symbols will come out.
# Sometimes the macro are not defined correctly through CMake and must be set manually
# in the resultin Visual Studio project (i hope to fix it in the future)
if (MATHPLOT_UNICODE)
    set(wxWidgets_CONFIGURATION mswu)
endif(MATHPLOT_UNICODE)

if(LINUX_64_32_CROSSCOMPILE)
    message(STATUS "Cross compiling from Linux x86_64 to Linux x86")
    set(CMAKE_LIBRARY_PATH "/usr/lib")
    set(CMAKE_SYSTEM_LIBRARY_PATH "/usr/lib")
    set(CMAKE_SYSTEM_PREFIX_PATH "/usr")
else(LINUX_64_32_CROSSCOMPILE)
    message(STATUS "Native build")
endif(LINUX_64_32_CROSSCOMPILE)
# message(STATUS "CMAKE_LIBRARY_PATH: " ${CMAKE_LIBRARY_PATH})

if(UNIX)
    include(GNUInstallDirs)
endif(UNIX)

#set(wxWidgets_USE_LIBS base core)
find_package(wxWidgets COMPONENTS base core)
if(wxWidgets_FOUND)
    message(STATUS ${wxWidgets_LIBRARIES})
    include(${wxWidgets_USE_FILE})
    if(MATHPLOT_SHARED)
        add_library(mathplot SHARED src/mathplot.cpp include/wx/mathplot.h)
        set_target_properties(mathplot PROPERTIES
	  VERSION ${wxMathPlot_VERSION_MAJOR}.${wxMathPlot_VERSION_MINOR}.${wxMathPlot_VERSION_BUILD}
	  SOVERSION ${wxMathPlot_VERSION_MAJOR})
    else(MATHPLOT_SHARED)
        add_library(mathplot STATIC src/mathplot.cpp include/wx/mathplot.h)
    endif(MATHPLOT_SHARED)
    if(UNIX)
        if(LINUX_64_32_CROSSCOMPILE)
            set_source_files_properties(src/mathplot.cpp PROPERTIES COMPILE_FLAGS "-m32 -DwxSIZE_T_IS_UINT" )
            set_target_properties(mathplot PROPERTIES LINK_FLAGS "-m32 ") # -L${CMAKE_LIBRARY_PATH}
        endif(LINUX_64_32_CROSSCOMPILE)
        if(GDB_DEBUG)
            set_source_files_properties(src/mathplot.cpp PROPERTIES COMPILE_FLAGS "-g -ggdb -Wall -pg -O0" )
            set_target_properties(mathplot PROPERTIES LINK_FLAGS "-g -ggdb -Wall -pg -O0")
            if (MATHPLOT_DO_LOGGING)
                set_source_files_properties(src/mathplot.cpp PROPERTIES COMPILE_FLAGS "-g -ggdb -Wall  -pg -O0 -DMATHPLOT_DO_LOGGING" )
            endif(MATHPLOT_DO_LOGGING)
        endif(GDB_DEBUG)
        if(LINUX_64_32_CROSSCOMPILE AND GDB_DEBUG)
            set_source_files_properties(src/mathplot.cpp PROPERTIES COMPILE_FLAGS "-m32 -g -ggdb -pg -O0 -DwxSIZE_T_IS_UINT" )
            set_target_properties(mathplot PROPERTIES LINK_FLAGS "-m32  -g -ggdb -pg -O0")
            if (MATHPLOT_DO_LOGGING)
                set_source_files_properties(src/mathplot.cpp PROPERTIES COMPILE_FLAGS "-m32 -g -ggdb -Wall -pg -O0 -DMATHPLOT_DO_LOGGING" )
            endif(MATHPLOT_DO_LOGGING)
        endif(LINUX_64_32_CROSSCOMPILE AND GDB_DEBUG)
    else(UNIX)
        if (MATHPLOT_UNICODE)
            set_source_files_properties(src/mathplot.cpp PROPERTIES COMPILE_FLAGS "/DUNICODE /D_UNICODE" )
            set_target_properties(mathplot PROPERTIES LINK_FLAGS "/DUNICODE /D_UNICODE")
        endif(MATHPLOT_UNICODE)
    endif(UNIX)
    target_link_libraries(mathplot ${wxWidgets_LIBRARIES})
    
    # generate pc file for pkg-config
    set(MATHPLOT mathplot)
    configure_file(${PROJECT_NAME}.pc.in lib/pkgconfig/${PROJECT_NAME}.pc @ONLY)

    # Compile samples?
    set(WXMATHPLOT_BUILD_EXAMPLES OFF CACHE BOOL "Build examples?")
    if(WXMATHPLOT_BUILD_EXAMPLES)
        add_subdirectory(samples)
    endif(WXMATHPLOT_BUILD_EXAMPLES)
    
    # library installation
#     if(UNIX)
        # if(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
        #    set(LIBRARY_APPEND_PATH lib64)
        # else(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
        #    set(LIBRARY_APPEND_PATH lib)
        # endif(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
        install(TARGETS mathplot
#            RUNTIME DESTINATION bin
           LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
           ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )
        #set(WXMATHPLOT_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/share/wxMathPlot/)
        install(FILES include/wx/mathplot.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/wx)
        install(FILES "${CMAKE_BINARY_DIR}/lib/pkgconfig/${PROJECT_NAME}.pc" DESTINATION lib/pkgconfig)
        install(FILES samples/sample1/mp1.cpp samples/sample1/CMakeLists.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/samples/sample1)
        install(FILES samples/sample2/mp2.cpp samples/sample2/CMakeLists.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/samples/sample2)
        install(FILES samples/sample3/mp3.cpp samples/sample3/CMakeLists.txt samples/sample3/gridmap.png DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/samples/sample3)
#     else(UNIX)
#     endif(UNIX)

else(wxWidgets_FOUND)
    message(STATUS "wxWidgets not found!")
endif(wxWidgets_FOUND)
