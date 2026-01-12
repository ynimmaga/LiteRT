set(DISPATCH_SRC
    device_context.cc
    dispatch_api.cc
    invocation_context.cc
)

set(DISPATCH_HDRS
    device_context.h
    invocation_context.h
    openvino_shared_core.h
    remote_tensor_buffer.h
)

set(INTEL_OPENVINO_DIR $ENV{INTEL_OPENVINO_DIR})

#
# Shared library: LiteRtDispatch_IntelOpenvino
#
add_library(litert_intel_openvino_dispatch SHARED
    ${DISPATCH_SRC}
    ${DISPATCH_HDRS}
)

target_compile_options(litert_intel_openvino_dispatch PRIVATE
    -Os
    -fexceptions
    -fno-unwind-tables
    -fno-asynchronous-unwind-tables
    -ffunction-sections
    -fdata-sections
)

target_include_directories(litert_intel_openvino_dispatch PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

# Only actual libraries, remove header-only targets
target_link_libraries(litert_intel_openvino_dispatch PRIVATE
    litert_runtime_c_api_shared_lib
    litert_logging
    #tensor_type_util
    ov_utils
    absl::cleanup
    openvino::runtime
    ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
)

set_target_properties(litert_intel_openvino_dispatch PROPERTIES
    OUTPUT_NAME "LiteRtDispatch_IntelOpenvino"
    SOVERSION 1
)

#
# Executable: LiteRtDispatch
#
add_library(LiteRtDispatch
    device_context.cc
    dispatch_api.cc
    invocation_context.cc
    openvino_shared_core.cc
    remote_tensor_buffer.cc
)

target_compile_options(LiteRtDispatch PRIVATE -fexceptions)

target_include_directories(LiteRtDispatch PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

target_link_libraries(LiteRtDispatch PRIVATE
    litert_intel_openvino_dispatch
    litert_runtime_c_api_shared_lib
    #litert_tensor_buffer          # this one is a proper static lib
    litert_logging
    #tensor_type_util
    ov_utils
    absl::cleanup
    openvino::runtime
    ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
)

#
# Header-only API library
#
add_library(litert_dispatch_api INTERFACE)

target_include_directories(litert_dispatch_api INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

target_link_libraries(litert_dispatch_api INTERFACE
    litert_runtime_c_api_shared_lib
    #tensor_type_util
    ov_utils
    absl::cleanup
    openvino::runtime
    ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
)

#
# Tests
#
if (LITERT_BUILD_TESTS)
    add_executable(dispatch_api_openvino_test
        dispatch_api_openvino_test.cc
    )

    target_compile_options(dispatch_api_openvino_test PRIVATE -fexceptions)

    # Only link actual libraries, remove interface-only header libraries
    target_link_libraries(dispatch_api_openvino_test PRIVATE
        litert_intel_openvino_dispatch
        litert_runtime_c_api_shared_lib
        litert_logging
        litert_any
        litert_expected
        litert_environment
        litert_options
        filesystem_testonly
        litert_test_common
        litert_test_matchers
        simple_model_npu
        absl::span
        gtest_main
        openvino::runtime
        ${INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino_tensorflow_lite_frontend.so
    )

    add_test(NAME dispatch_api_openvino_test COMMAND dispatch_api_openvino_test)
endif()
