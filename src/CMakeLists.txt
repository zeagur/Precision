include(FetchContent)

set(ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/..")

set(SOURCE_DIR "${ROOT_DIR}/src")
file(GLOB_RECURSE SOURCE_FILES "${SOURCE_DIR}/*.cpp" "${SOURCE_DIR}/*.h")

source_group(TREE "${ROOT_DIR}" FILES ${SOURCE_FILES})

set(SPDLOG_INSTALL ON)

FetchContent_Declare(
        glm
        GIT_REPOSITORY https://github.com/g-truc/glm.git
        GIT_TAG master
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(glm)

FetchContent_Declare(
        simpleIni
        GIT_REPOSITORY https://github.com/brofield/simpleini.git
        GIT_TAG master
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(simpleIni)
# since the default fetch content SOURCE_DIR isnt working on this one
set(simpleIni_SOURCE_DIR "${CMAKE_BINARY_DIR}/_deps/simpleini-src")
include_directories(${simpleIni_SOURCE_DIR})

FetchContent_Declare(
        tomlplusplus
        GIT_REPOSITORY https://github.com/marzer/tomlplusplus.git
        GIT_TAG master
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(tomlplusplus)

set(SKSE_SUPPORT_XBYAK ON)
FetchContent_Declare(
        xbyak
        GIT_REPOSITORY https://github.com/herumi/xbyak.git
        GIT_TAG master
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(xbyak)
include_directories(${xbyak_SOURCE_DIR})

FetchContent_Declare(
        spdlog
        GIT_REPOSITORY https://github.com/gabime/spdlog.git
        GIT_TAG v1.x
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(spdlog)
find_package(spdlog REQUIRED CONFIG)

FetchContent_Declare(
        rapidcsv
        GIT_REPOSITORY https://github.com/d99kris/rapidcsv.git
        GIT_TAG master
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(rapidcsv)
set(RAPIDCSV_INCLUDE_DIRS ${rapidcsv_SOURCE_DIR}/src)

FetchContent_Declare(
        CommonLibSSE
        GIT_REPOSITORY https://github.com/CharmedBaryon/CommonLibSSE-NG.git
        GIT_TAG main
        OVERRIDE_FIND_PACKAGE
)
FetchContent_MakeAvailable(CommonLibSSE)
include(${CommonLibSSE_SOURCE_DIR}/cmake/CommonLibSSE.cmake)

add_commonlibsse_plugin(
        "${PROJECT_NAME}"
        SOURCES
        ${SOURCE_FILES}
        "${ROOT_DIR}/.clang-format"
        "${ROOT_DIR}/.editorconfig"
)

target_compile_features(
        "${PROJECT_NAME}"
        PRIVATE
        cxx_std_23
)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    target_compile_options(
            "${PROJECT_NAME}"
            PRIVATE
            "/W3"
            "/sdl" # Enable Additional Security Checks
            "/utf-8" # Set Source and Executable character sets to UTF-8
            #			"/Zi"	# Debug Information Format

            "/await"

            "/permissive-" # Standards conformance
            "/Zc:preprocessor" # Enable preprocessor conformance mode

            "/wd4200" # nonstandard extension used : zero-sized array in struct/union

            "$<$<CONFIG:DEBUG>:/ZI>"
            "$<$<CONFIG:RELEASE>:/Zi;/Zc:inline;/JMC-;/Ob3>"
    )

    target_link_options(
            "${PROJECT_NAME}"
            PRIVATE
            "$<$<CONFIG:DEBUG>:/INCREMENTAL;/OPT:NOREF;/OPT:NOICF>"
            "$<$<CONFIG:RELEASE>:/INCREMENTAL:NO;/OPT:REF;/OPT:ICF;/DEBUG:FULL>"
    )
endif ()

target_include_directories(
        "${PROJECT_NAME}"
        PRIVATE
        "${CMAKE_CURRENT_BINARY_DIR}/src"
        "${SOURCE_DIR}"
)

target_link_libraries(
        "${PROJECT_NAME}"
        PRIVATE
        spdlog::spdlog
        glm::glm
        xbyak::xbyak
        tomlplusplus::tomlplusplus
        CommonLibSSE::CommonLibSSE
)

target_precompile_headers(
        "${PROJECT_NAME}"
        PRIVATE
        "${SOURCE_DIR}/PCH.h"
)

install(
        FILES
        "$<TARGET_FILE:${PROJECT_NAME}>"
        DESTINATION "SKSE/Plugins"
        COMPONENT "main"
)

install(
        FILES
        "$<TARGET_PDB_FILE:${PROJECT_NAME}>"
        DESTINATION "/"
        COMPONENT "pdbs"
)