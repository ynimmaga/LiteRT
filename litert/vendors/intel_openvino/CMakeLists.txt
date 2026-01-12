# Find OpenVINO runtime package
find_package(OpenVINO REQUIRED)

message(STATUS "Building Intel OpenVINO backend")

# Add subdirectories
add_subdirectory(compiler)
add_subdirectory(dispatch)

# Shared utilities exposed to compiler + dispatch
add_library(ov_utils INTERFACE)
target_include_directories(ov_utils INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_SOURCE_DIR}/core/model/extended_model
)

target_link_libraries(ov_utils INTERFACE
    litert_runtime_c_api_shared_lib
    #litert_extended_model
    openvino::runtime
)

# Header-only plugin C API interface
add_library(litert_compiler_plugin_c_api INTERFACE)
target_include_directories(litert_compiler_plugin_c_api INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../c
)
target_link_libraries(litert_compiler_plugin_c_api INTERFACE
    litert_runtime_c_api_shared_lib
)
