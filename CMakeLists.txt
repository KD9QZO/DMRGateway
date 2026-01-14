#
# CMakeLists.txt for DMRGateway
#



###############################################################################
# PROJECT SETUP                                                               #
###############################################################################

project(DMRGateway)

cmake_minimum_required(VERSION 3.18)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "Release")
	message(STATUS "Build type not specified; defaulting to Release.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")


include(GNUInstallDirs)

set(prefix ${CMAKE_INSTALL_PREFIX})
set(DMRGATEWAY_SHARE_DIR ${prefix}/share/DMRGateway)
set(DMRGATEWAY_ETC_DIR ${prefix}/etc)
set(SYSTEMD_DIR /etc/systemd/system)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)

set(SOURCES
	APRSWriter.cpp
	BPTC19696.cpp
	Conf.cpp
	CRC.cpp
	DMRCSBK.cpp
	DMRData.cpp
	DMRDataHeader.cpp
	DMREMB.cpp
	DMREmbeddedData.cpp
	DMRFullLC.cpp
	DMRGateway.cpp
	DMRLC.cpp
	DMRNetwork.cpp
	DMRSlotType.cpp
	DynVoice.cpp
	Golay2087.cpp
	GPSD.cpp
	Hamming.cpp
	Log.cpp
	MMDVMNetwork.cpp
	PassAllPC.cpp
	PassAllTG.cpp
	QR1676.cpp
	Reflectors.cpp
	RemoteControl.cpp
	Rewrite.cpp
	RewriteDstId.cpp
	RewriteDynTGNet.cpp
	RewriteDynTGRF.cpp
	RewritePC.cpp
	RewriteSrc.cpp
	RewriteSrcId.cpp
	RewriteTG.cpp
	RewriteType.cpp
	RS129.cpp
	SHA256.cpp
	StopWatch.cpp
	Sync.cpp
	Thread.cpp
	Timer.cpp
	UDPSocket.cpp
	Utils.cpp
	XLXVoice.cpp
)


########################################################################################################################
# GENERATE GitVersion.h FILE                                                                                           #
########################################################################################################################

include(GitVersion)

configure_file(
	${CMAKE_CURRENT_SOURCE_DIR}/GitVersion.h.in
	${CMAKE_CURRENT_BINARY_DIR}/GitVersion.h
	@ONLY
)



###############################################################################
# COMPILER SPECIFIC SETUP                                                     #
###############################################################################

if(CMAKE_COMPILER_IS_GNUCC AND NOT WIN32)
	add_definitions(-Wall)
endif()

add_definitions(-DHAVE_LOG_H)

find_package(PkgConfig REQUIRED)

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)

find_package(Threads REQUIRED)
if(NOT THREADS_FOUND)
	message(FATAL_ERROR "pthreads is required to build DAPNETGateway")
endif()


find_package(LibGPS)
option(ENABLE_GPSD "Enable support for gpsd integration" ON)
if(LIBGPS_FOUND)
	add_definitions(-DUSE_GPSD)
	set(ENABLE_GPSD ON)
	message(STATUS "libgps include directory: ${LIBGPS_INCLUDE_DIR}")
else(LIBGPS_FOUND)
	set(ENABLE_GPSD OFF)
	message(STATUS "libgps was not found")
endif(LIBGPS_FOUND)

include_directories(
	${CMAKE_SOURCE_DIR}
	${CMAKE_CURRENT_BINARY_DIR}
	${THREADS_PTHREADS_INCLUDE_DIR}
)

add_executable(DMRGateway ${SOURCES})

set_property(TARGET DMRGateway PROPERTY C_STANDARD 11)
set_property(TARGET DMRGateway PROPERTY CXX_STANDARD 14)

if(UNIX)
	target_link_libraries(DMRGateway m)
endif()

target_link_libraries(DMRGateway Threads::Threads)

if(LIBGPS_FOUND)
	include_directories(${LIBGPS_INCLUDE_DIR})
	target_link_libraries(DMRGateway ${LIBGPS_LIBRARIES})
endif()


########################################################################################################################
# INSTALLATION                                                                                                         #
########################################################################################################################

option(USE_SYSTEM_ETC "Install DMRGateway.ini in system /etc instead of in ${CMAKE_INSTALL_PREFIX}/etc directory" OFF)
option(INSTALL_LOGROTATE_CONF "Install a proper logrotate.conf file for DMRGateway in /etc/logrotate.d to automate log rotation" ON)

if(USE_SYSTEM_ETC)
	install(
		FILES ${CMAKE_SOURCE_DIR}/DMRGateway.ini
		DESTINATION "/etc"
	)
else(USE_SYSTEM_ETC)
	install(
		FILES ${CMAKE_SOURCE_DIR}/DMRGateway.ini
		DESTINATION "${DMRGATEWAY_ETC_DIR}"
	)
endif(USE_SYSTEM_ETC)

install(
	FILES ${CMAKE_SOURCE_DIR}/linux/dmrgateway.service
	DESTINATION "${SYSTEMD_DIR}"
)

if(INSTALL_LOGROTATE_CONF)
	install(
		FILES ${CMAKE_SOURCE_DIR}/linux/logrotate.d/dmrgateway
		DESTINATION "/etc/logrotate.d"
	)
endif(INSTALL_LOGROTATE_CONF)

install(
	FILES ${CMAKE_SOURCE_DIR}/XLXHosts.txt
	DESTINATION "${DMRGATEWAY_SHARE_DIR}"
)

install(
	FILES ${CMAKE_SOURCE_DIR}/XLXHostsUpdate.sh
	DESTINATION "${DMRGATEWAY_SHARE_DIR}"
)

install(
	DIRECTORY ${CMAKE_SOURCE_DIR}/Audio/
	DESTINATION "${DMRGATEWAY_SHARE_DIR}/Audio"
)

install(
	TARGETS DMRGateway
	RUNTIME DESTINATION bin
)
