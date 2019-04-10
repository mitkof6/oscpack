cmake_minimum_required(VERSION 2.6)
PROJECT(TestOscpack)

INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR})
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

# separate versions of NetworkingUtils.cpp and UdpSocket.cpp are provided for
# Win32 and POSIX the IpSystemTypePath selects the correct ones based on the
# current platform

IF(WIN32)
 SET(IpSystemTypePath ip/win32)
 SET(LIBS ${LIBS} Ws2_32 winmm)
ELSE(WIN32)
 SET(IpSystemTypePath ip/posix)
ENDIF(WIN32)

FILE(GLOB includes ip/*.h osc/*h)
FILE(GLOB sources ip/*.cpp ${IpSystemTypePath}/*.cpp osc/*.cpp)

SET(target oscpack)
ADD_LIBRARY(${target} ${sources} ${includes})
INSTALL(TARGETS ${target} 
  LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
INSTALL(DIRECTORY ${CMAKE_SOURCE_DIR}/osc DESTINATION ${CMAKE_INSTALL_PREFIX}/include/)
INSTALL(DIRECTORY ${CMAKE_SOURCE_DIR}/ip DESTINATION ${CMAKE_INSTALL_PREFIX}/include/)

ADD_EXECUTABLE(OscUnitTests tests/OscUnitTests.cpp)
TARGET_LINK_LIBRARIES(OscUnitTests ${target} ${LIBS})

ADD_EXECUTABLE(OscSendTests tests/OscSendTests.cpp)
TARGET_LINK_LIBRARIES(OscSendTests ${target} ${LIBS})

ADD_EXECUTABLE(OscReceiveTest tests/OscReceiveTest.cpp)
TARGET_LINK_LIBRARIES(OscReceiveTest ${target} ${LIBS})


ADD_EXECUTABLE(OscDump examples/OscDump.cpp)
TARGET_LINK_LIBRARIES(OscDump ${target} ${LIBS})

ADD_EXECUTABLE(SimpleReceive examples/SimpleReceive.cpp)
TARGET_LINK_LIBRARIES(SimpleReceive ${target} ${LIBS})

ADD_EXECUTABLE(SimpleSend examples/SimpleSend.cpp)
TARGET_LINK_LIBRARIES(SimpleSend ${target} ${LIBS})

if(MSVC)
  # Force to always compile with W4
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
  # Update if necessary
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-long-long -pedantic")
endif()
