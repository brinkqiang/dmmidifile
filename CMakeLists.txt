cmake_minimum_required(VERSION 3.21)

project(midifile C CXX)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(cmake/ModuleImport.cmake)
include(cmake/ModuleCompileOptions.cmake)

ModuleSetCompileOptions()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(CheckIncludeFiles)
include(GNUInstallDirs)

include_directories(include)

check_include_files(unistd.h HAVE_UNISTD_H)
check_include_files(humdrum.h HAVE_HUMDRUM_H)
check_include_files(sys/io.h HAVE_SYS_IO_H)

option(BUILD_MIDILIBRARY_ONLY "Build only the midifile library" ON)

##############################
##
## Operating-system specific settings:
##

if(APPLE)
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.9" CACHE STRING "Minimum OS X deployment version")
endif()

if(MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)

    option(STATIC_CRT "Use static CRT libraries" OFF)

    # Rewrite command line flags to use /MT if necessary
    if(STATIC_CRT)
        foreach(flag_var
                CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
                CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
            if(${flag_var} MATCHES "/MD")
                string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
            endif(${flag_var} MATCHES "/MD")
        endforeach(flag_var)
    endif()
endif()


##############################
##
## Library:
##

set(SRCS
    src/Options.cpp
    src/Binasc.cpp
    src/MidiEvent.cpp
    src/MidiEventList.cpp
    src/MidiFile.cpp
    src/MidiMessage.cpp
)

set(HDRS
    include/Binasc.h
    include/MidiEvent.h
    include/MidiEventList.h
    include/MidiFile.h
    include/MidiMessage.h
    include/Options.h
)

add_library(midifile STATIC ${SRCS} ${HDRS})

##############################
##
## Programs:
##

if(NOT BUILD_MIDILIBRARY_ONLY)
    ExeImport("tools" "midifile")  
endif()

if(HAVE_UNISTD_H AND HAVE_SYS_IO_H)
    ExeImport("linux-tools" "midifile")  
endif()

# The following programs require headers from humextra repository.
# <https://github.com/humdrum-tools/humextra>
if(HAVE_HUMDRUM_H)
    add_executable(henonfile tools/henonfile.cpp)
    add_executable(mid2hum tools/mid2hum.cpp)
    add_executable(midiexcerpt tools/midiexcerpt.cpp)
    add_executable(peep2midi tools/peep2midi.cpp)

    target_link_libraries(henonfile midifile)
    target_link_libraries(mid2hum midifile)
    target_link_libraries(midiexcerpt midifile)
    target_link_libraries(peep2midi midifile)
endif()

install(TARGETS midifile
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(FILES ${HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/midifile)
