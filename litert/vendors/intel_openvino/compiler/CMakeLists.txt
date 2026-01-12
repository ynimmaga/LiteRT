#
# Compiler plugin shared library
#
add_library(openvino_compiler_plugin SHARED
    openvino_compiler_plugin.cc
)

set(INTEL_OPENVINO_DIR $ENV{INTEL_OPENVINO_DIR})

target_include_directories(openvino_compiler_plugin PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_SOURCE_DIR}/path/to/tensor_buffer
    ${CMAKE_SOURCE_DIR}/path/to/schema_fbs
    ${CMAKE_SOURCE_DIR}/core/model/extended_model
)

#target_include_directories(openvino_compiler_plugin PUBLIC
#    ${CMAKE_CURRENT_SOURCE_DIR}
#)

target_compile_options(openvino_compiler_plugin PRIVATE
    -fexceptions
)

target_link_libraries(openvino_compiler_plugin PRIVATE
    graph_iterator
    decoder
    litert_runtime_c_api_shared_lib
    litert_logging
    absl::str_format
    ov_utils
    openvino::runtime
    ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
)

set_target_properties(openvino_compiler_plugin PROPERTIES
    OUTPUT_NAME "LiteRTCompilerPlugin_IntelOpenVINO"
)

#
# Standalone compiler plugin executable
#
add_library(LiteRtCompilerPlugin
    openvino_compiler_plugin.cc
)

target_link_libraries(LiteRtCompilerPlugin PRIVATE
    graph_iterator
    decoder
    litert_runtime_c_api_shared_lib
    litert_logging
    #litert_tensor_buffer
    absl::str_format
    ov_utils
    openvino::runtime
    ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
)

#
# graph_iterator
#
add_library(graph_iterator STATIC
    graph_iterator.cc
    graph_iterator.h
)

target_compile_options(graph_iterator PRIVATE -fexceptions)

target_link_libraries(graph_iterator PRIVATE
    decoder
    litert_runtime_c_api_shared_lib
    litert_logging
    ov_utils
    openvino::runtime
    ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
)

#
# decoder
#
add_library(decoder STATIC
    decoder.cc
    decoder.h
)

target_compile_options(decoder PRIVATE -fexceptions)

target_link_libraries(decoder PRIVATE
    litert_runtime_c_api_shared_lib
    litert_logging
    ov_utils
    #schema_fbs
    openvino::runtime
    ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
)
