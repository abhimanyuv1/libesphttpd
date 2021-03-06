cmake_minimum_required(VERSION 3.9)

include(CheckCCompilerFlag)

function(enable_c_compiler_flag_if_supported flag)
    string(FIND "${CMAKE_C_FLAGS}" "${flag}" flag_already_set)
    if(flag_already_set EQUAL -1)
        check_c_compiler_flag("${flag}" flag_supported)
        if(flag_supported)
            set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${flag}" PARENT_SCOPE)
        endif()
    endif()
endfunction()

enable_c_compiler_flag_if_supported("-Wall")
enable_c_compiler_flag_if_supported("-Werror")

# Enable when heatshink fallthrough bug is fixed, see https://github.com/atomicobject/heatshrink/pull/46
#enable_c_compiler_flag_if_supported("-Wextra")


add_library(esphttpd
    core/auth.c
    core/base64.c
    core/httpdespfs.c
    core/httpd.c
    core/httpd-freertos.c
    core/sha1.c
    espfs/espfs.c
    util/cgiwebsocket.c
)

target_compile_definitions(esphttpd PUBLIC "CONFIG_ESPHTTPD_MAX_CONNECTIONS=100")
target_compile_definitions(esphttpd PUBLIC "CONFIG_ESPHTTPD_SSL_SUPPORT=1")

target_include_directories(esphttpd PUBLIC "core")
target_include_directories(esphttpd PUBLIC "include")
target_include_directories(esphttpd PUBLIC "espfs")


add_executable(mkespfsimage
    espfs/mkespfsimage/main.c
    espfs/mkespfsimage/heatshrink_encoder.c
)

find_package(ZLIB REQUIRED)
target_link_libraries(mkespfsimage ${ZLIB_LIBRARIES})

target_compile_definitions(mkespfsimage PUBLIC "ESPFS_HEATSHRINK=1")
target_compile_definitions(mkespfsimage PUBLIC "ESPFS_GZIP=1")

target_include_directories(mkespfsimage PUBLIC "include")
target_include_directories(mkespfsimage PUBLIC "espfs")
target_include_directories(mkespfsimage PUBLIC "lib/heatshrink")


install(TARGETS esphttpd DESTINATION lib)
install(TARGETS mkespfsimage DESTINATION bin)
install(FILES include/libesphttpd/httpd.h DESTINATION include/libesphttpd)
install(FILES include/libesphttpd/cgiwebsocket.h DESTINATION include/libesphttpd)
install(FILES include/libesphttpd/httpdespfs.h DESTINATION include/libesphttpd)
install(FILES include/libesphttpd/linux.h DESTINATION include/libesphttpd)
install(FILES include/libesphttpd/platform.h DESTINATION include/libesphttpd)
install(FILES include/libesphttpd/espfs.h DESTINATION include/libesphttpd)
install(FILES include/libesphttpd/webpages-espfs.h DESTINATION include/libesphttpd)
