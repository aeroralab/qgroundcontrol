find_package(Qt6 REQUIRED COMPONENTS Bluetooth Core Network Test)

# NO_SERIAL_LINK # TODO: Make this QGC_NO_SERIAL_LINK

qt_add_library(comm STATIC
	BluetoothLink.cc
	BluetoothLink.h
	LinkConfiguration.cc
	LinkConfiguration.h
	LinkInterface.cc
	LinkInterface.h
	LinkManager.cc
	LinkManager.h
	LogReplayLink.cc
	LogReplayLink.h
	MAVLinkProtocol.cc
	MAVLinkProtocol.h
	QGCMAVLink.cc
	QGCMAVLink.h
	QGCSerialPortInfo.cc
	QGCSerialPortInfo.h
	SerialLink.cc
	SerialLink.h
	TCPLink.cc
	TCPLink.h
	UdpIODevice.cc
	UdpIODevice.h
	UDPLink.cc
	UDPLink.h
)

# MockLink
target_sources(comm
	PRIVATE
		MockLink.cc
		MockLink.h
		MockLinkFTP.cc
		MockLinkFTP.h
		MockLinkMissionItemHandler.cc
		MockLinkMissionItemHandler.h
)

target_link_libraries(comm
	PRIVATE
		Qt6::Test
	PUBLIC
		qgc
		Qt6::Widgets
		Qt6::Bluetooth
		Qt6::Network
)

if(ANDROID)
    target_link_libraries(comm PUBLIC qtandroidserialport)
else()
	target_link_libraries(comm PUBLIC Qt6::SerialPort)
endif()

option(QGC_ZEROCONF_ENABLED "Enable ZeroConf Compatibility" ON)
if(QGC_ZEROCONF_ENABLED)
	target_link_libraries(comm PRIVATE qmdnsengine)
	target_include_directories(comm PRIVATE ${CMAKE_SOURCE_DIR}/libs)
        target_compile_definitions(comm PUBLIC QGC_ZEROCONF_ENABLED)


    if(WIN32)
        if(DEFINED ENV{BONJOUR_SDK_HOME} AND EXISTS $ENV{BONJOUR_SDK_HOME})
            set(BONJOUR_SDK_PREFIX $ENV{BONJOUR_SDK_HOME})
        elseif(EXISTS "C:/Program Files/Bonjour SDK")
            set(BONJOUR_SDK_PREFIX "C:/Program Files/Bonjour SDK")
        else()
            set(BONJOUR_SDK_PREFIX ${CMAKE_SOURCE_DIR}/libs/BonjourSDK)
        endif()
        if(NOT EXISTS ${BONJOUR_SDK_PREFIX})
            message(FATAL_ERROR "BONJOUR SDK not found")
        else()
            message("BONJOUR SDK found at ${BONJOUR_SDK_PREFIX}")
        endif()
        target_include_directories(comm PUBLIC ${BONJOUR_SDK_PREFIX}/Include)
        target_link_directories(comm PUBLIC ${BONJOUR_SDK_PREFIX}/Lib/x64)
        target_link_libraries(comm PUBLIC dnssd)
        ## NOMINMAX is required to disable MAX/MIN define in Windows.h which have
        ## conflicts with std::min/std::max define in <Algorithm>
        target_compile_definitions(comm PUBLIC NOMINMAX)
    endif()
endif()

target_compile_definitions(comm PUBLIC QGC_ENABLE_BLUETOOTH)

target_include_directories(comm
    PUBLIC
    	${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_SOURCE_DIR}/libs/mavlink/include/mavlink/v2.0
        ${CMAKE_SOURCE_DIR}/libs/mavlink/include/mavlink/v2.0/all
        ${CMAKE_SOURCE_DIR}/libs/mavlink/include/mavlink/v2.0/common
)
