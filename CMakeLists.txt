cmake_minimum_required(VERSION 3.9)

project(TraceLog LANGUAGES NONE)

message(STATUS "Configuring TraceLog Adaptive Writer for ${CMAKE_SYSTEM_NAME} ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "  Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "")

# Constants
set( LIB_VERSION "1.1.0" )
set( SWIFT_VERSION "5" )

# Look for modules in the our root.
LIST(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR})

#
# Clean the package
#
add_custom_target(build-clean
        COMMAND swift package clean
        WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
        )

#
# Build xcode project Target
#
add_custom_target(xcode-project
        DEPENDS generate-source generate-tests Package.swift Package.xcconfig
        COMMAND swift package generate-xcodeproj --xcconfig-overrides Package.xcconfig --enable-code-coverage
        WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
        SOURCES Package.swift Package.xcconfig)

add_custom_target(pod-lint
        DEPENDS xcode-project
        COMMAND bundle exec pod lib lint
        WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
        )


add_custom_target(documentation
        DEPENDS xcode-project generate-source .jazzy.yaml
        COMMAND jazzy --clean --module-version ${LIB_VERSION}
        COMMAND cp ${CMAKE_SOURCE_DIR}/Sources/Documentation/Images/*.png ${CMAKE_SOURCE_DIR}/docs/img/
        WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}")

add_subdirectory(Sources)
add_subdirectory(Tests)
