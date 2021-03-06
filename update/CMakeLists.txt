﻿#CMAKE_MINIMUM_REQUIRED(VERSION 3.1)
# CMake 最低版本号要求
PROJECT (aliyun-assist) 
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(../third_party)
include_directories(../common)
link_directories(../common)

aux_source_directory(. DIR_SRCS)
aux_source_directory(./update_check DIR_SRCS)
link_libraries(assist-common)
link_libraries(third_party)

if(WIN32)
    link_libraries(shlwapi Psapi version)
    link_libraries(ws2_32 Wldap32)
    link_libraries(libcurl.lib)
    link_libraries(libeay32)
    link_libraries(ssleay32)
    include_directories(../third_party/libcurl/windows/include)
    link_directories(../third_party/libcurl/windows/lib)
    add_definitions(-DUNICODE -D_UNICODE)
	add_executable(aliyun_assist_update ${DIR_SRCS} ../VersionResource.rc)
    SET_TARGET_PROPERTIES(aliyun_assist_update PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG   ${CMAKE_CURRENT_SOURCE_DIR}/../output/)
    SET_TARGET_PROPERTIES(aliyun_assist_update PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_SOURCE_DIR}/../output/)
else(UNIX)
    include_directories(../third_party/libcurl/linux/include)
    link_directories(../third_party/libcurl/linux/lib)
    link_libraries(curl ssl crypto)
	add_executable(aliyun_assist_update ${DIR_SRCS} ../VersionResource.rc)
    set_target_properties(aliyun_assist_update PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../output/)
	#for static link ,only success on  Ubuntu 16.04.4 LTS
	if( DEFINED ENV{USE_STATIC_LINK} ) 
		set (CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -I../third_party/glibc/include")
		set (STATIC_GLIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/glibc/lib")
		target_link_libraries(aliyun_assist_update "-nostdlib -nostartfiles -static ${STATIC_GLIB_PATH}/crt1.o ${STATIC_GLIB_PATH}/crti.o  -Wl,--start-group ${STATIC_GLIB_PATH}/libpthread.a -lstdc++  ${STATIC_GLIB_PATH}/librt.a  ${STATIC_GLIB_PATH}/libc.a  -ldl -lgcc  -lgcc_eh -Wl,--end-group  ${STATIC_GLIB_PATH}/crtn.o")
	else()
		target_link_libraries(aliyun_assist_update  -lpthread -ldl -lrt)
	endif()
endif()
