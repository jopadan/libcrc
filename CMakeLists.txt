cmake_minimum_required(VERSION 3.0)
project(crc LANGUAGES C)

set(PROJECT_VERSION 2.1)
set(PROJECT_DESCRIPTION "Multi platform MIT licensed CRC library in C")

include(GNUInstallDirs)

if(WIN32 AND (MSYS OR MINGW))
	execute_process(COMMAND cygpath -ma /usr OUTPUT_VARIABLE CMAKE_INSTALL_PREFIX)
	string(STRIP ${CMAKE_INSTALL_PREFIX} CMAKE_INSTALL_PREFIX)
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(TAB_FILES
        tab/gentab32.inc
        tab/gentab64.inc
)

set(SOURCE_FILES
	src/crcccitt32.c
	src/crcccitt.c
	src/crc8.c
	src/crc16.c
	src/crc32.c
	src/crc64.c
	src/crcsick.c
	src/crckrmit.c
	src/crcdnp.c
	src/nmea-chk.c
	${TAB_FILES}
)

set(INCLUDE_FILES
	include/checksum.h
)

set(TAB32_FILES
	tab/gentab32.inc
   )
set(TAB64_FILES
	tab/gentab64.inc
   )

set(EXAMPLE_SOURCE_FILES
	examples/tstcrc.c
)

set(TEST_SOURCE_FILES
	test/testall.c
	test/testall.h
	test/testcrc.c
	test/testnmea.c
)

set(PRECALC_SOURCE_FILES
	precalc/crc32_table.c
	precalc/crc64_table.c
	precalc/precalc.c
	precalc/precalc.h
)

# prc executable
add_executable( prc ${PRECALC_SOURCE_FILES} )
target_include_directories(prc PUBLIC
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/precalc>
			$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
			)
add_custom_command( OUTPUT ${TAB32_FILES} COMMAND prc --crc32 ${TAB32_FILES} DEPENDS prc ) 
add_custom_command( OUTPUT ${TAB64_FILES} COMMAND prc --crc64 ${TAB64_FILES} DEPENDS prc )

# libcrc library
add_library( ${PROJECT_NAME} SHARED ${SOURCE_FILES} ${INCLUDE_FILES} ${TAB_FILES} )
target_include_directories(${PROJECT_NAME} PUBLIC
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
			$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
			)
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER ${INCLUDE_FILES} ) 

# tstcrc example
add_executable( tstcrc ${EXAMPLE_SOURCE_FILES} ${INCLUDE_FILES} )
target_link_libraries( tstcrc PRIVATE ${PROJECT_NAME} )
set_target_properties( tstcrc PROPERTIES RUNTIME_OUTPUT_DIRECTORY
	${CMAKE_BINARY_DIR} )
add_test(tstcrc tstcrc)

# testall test
add_executable( testall ${TEST_SOURCE_FILES} ${INCLUDE_FILES} )
target_link_libraries( testall PRIVATE ${PROJECT_NAME} )
set_target_properties( testall PROPERTIES RUNTIME_OUTPUT_DIRECTORY
	${CMAKE_BINARY_DIR} )
add_test(testall testall)

set(install_prefix "${CMAKE_INSTALL_PREFIX}")
set(install_libdir "${CMAKE_INSTALL_LIBDIR}")
set(install_includedir "${CMAKE_INSTALL_INCLUDEDIR}")

configure_file(lib${PROJECT_NAME}.pc.in ${PROJECT_BINARY_DIR}/lib${PROJECT_NAME}.pc @ONLY)

set(ENV{PKG_CONFIG_PATH} "${PROJECT_BINARY_DIR}:$ENV{PKG_CONFIG_PATH}")

install(TARGETS prc crc tstcrc testall EXPORT ${PROJECT_NAME}Config
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	PUBLIC_HEADER DESTINATION
	${CMAKE_INSTALL_INCLUDEDIR}
	)

install(FILES lib${PROJECT_NAME}.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/)
