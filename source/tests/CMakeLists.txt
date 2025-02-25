#
# Setup test environment
#

# Check if tests are enabled
if(NOT OPTION_BUILD_TESTS)
	return()
endif()

# DISABLE CMP0037: Target names should not be reserved and should match a validity pattern
if("${CMAKE_VERSION}" VERSION_LESS "3.11" AND POLICY CMP0037)
	set_policy(CMP0037 OLD)
endif()

set(GTEST_VERSION 1.8.1)

find_package(GTest ${GTEST_VERSION})

if(NOT GTEST_FOUND)
	include(InstallGTest)

	if(NOT GTEST_FOUND)
		message(STATUS "GTest libraries not found")
		return()
	endif()

	set(GTEST_INSTALL TRUE)
endif()

# Create interface library to link against Google Test
add_library(GTest INTERFACE)

target_include_directories(GTest
	SYSTEM INTERFACE
	${GTEST_INCLUDE_DIRS}
)

target_link_libraries(GTest
	INTERFACE
	${GTEST_LIBRARIES}
)

if(GTEST_INSTALL)
	add_dependencies(GTest google-test-depends)
endif()

# Set memory check configuration
option(OPTION_TEST_MEMORYCHECK "Run tests with memory checker (valgrind)." OFF)


if(OPTION_TEST_MEMORYCHECK AND OPTION_BUILD_SANITIZER)
	message(WARNING "OPTION_TEST_MEMORYCHECK and OPTION_BUILD_SANITIZER are not compatible, disabling memcheck tests.")
endif()

if(OPTION_TEST_MEMORYCHECK AND NOT OPTION_BUILD_SANITIZER)
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --leak-check=full")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --trace-children=yes")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --show-reachable=yes")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --track-origins=yes")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --num-callers=50")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --smc-check=all-non-file") # for JITs
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --suppressions=${CMAKE_CURRENT_SOURCE_DIR}/memcheck/valgrind.supp")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --suppressions=${CMAKE_CURRENT_SOURCE_DIR}/memcheck/valgrind-python.supp")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --suppressions=${CMAKE_CURRENT_SOURCE_DIR}/memcheck/valgrind-node.supp")
	set(MEMORYCHECK_COMMAND_OPTIONS "${MEMORYCHECK_COMMAND_OPTIONS} --suppressions=${CMAKE_CURRENT_SOURCE_DIR}/memcheck/valgrind-wasm.supp")

	# TODO: Memory check does not work properly with CoreCLR
	#
	# 	Remove MEMCHECK_IGNORE label from the following tests:
	#		- cs-loader-test
	#		- metacall-test
	#		- metacall-distributable-test
	#		- metacall-inspect-test
	#		- metacall-integration-test

	add_custom_target(memcheck
		WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
		COMMAND ${CMAKE_CTEST_COMMAND}
			--label-exclude MEMCHECK_IGNORE
			--force-new-ctest-process
			--test-action memcheck
			--timeout 60
		#COMMAND cat "${CMAKE_BINARY_DIR}/Testing/Temporary/MemoryChecker.*.log"
	)
endif()

#
# Tests
#

include(CTest)

add_subdirectory(preprocessor_test)
add_subdirectory(environment_test)
add_subdirectory(log_test)
add_subdirectory(log_custom_test)
add_subdirectory(adt_set_test)
add_subdirectory(adt_trie_test)
add_subdirectory(adt_vector_test)
add_subdirectory(adt_map_test)
add_subdirectory(reflect_value_cast_test)
add_subdirectory(reflect_function_test)
add_subdirectory(reflect_object_class_test)
add_subdirectory(reflect_scope_test)
add_subdirectory(reflect_metadata_test)
add_subdirectory(dynlink_test)
add_subdirectory(detour_test)
add_subdirectory(serial_test)
add_subdirectory(configuration_test)
add_subdirectory(rb_loader_parser_test)
add_subdirectory(portability_path_test)
add_subdirectory(metacall_logs_test)
add_subdirectory(metacall_load_memory_test)
add_subdirectory(metacall_load_configuration_test)
add_subdirectory(metacall_load_configuration_relative_test)
add_subdirectory(metacall_load_configuration_python_node_test)
add_subdirectory(metacall_duplicated_handle_test)
add_subdirectory(metacall_duplicated_symbols_test)
add_subdirectory(metacall_handle_export_test)
add_subdirectory(metacall_handle_get_test)
add_subdirectory(metacall_test)
add_subdirectory(metacall_node_test)
add_subdirectory(metacall_node_event_loop_test)
add_subdirectory(metacall_node_event_loop_signal_test)
#add_subdirectory(metacall_node_default_export_test) # TODO: This is a feature in order to export by default all functions if there is no module.exports (bootstrap.js)
add_subdirectory(metacall_node_call_test)
add_subdirectory(metacall_node_inline_test)
add_subdirectory(metacall_node_async_test)
add_subdirectory(metacall_node_reentrant_test)
add_subdirectory(metacall_node_port_test)
add_subdirectory(metacall_node_port_await_test)
# add_subdirectory(metacall_node_python_port_mock_test) # TODO: Recursion bug in Python Port (disabled for readability of tests)
# add_subdirectory(metacall_node_python_port_ruby_test) # TODO: Recursion bug in Python Port (disabled for readability of tests)
add_subdirectory(metacall_node_callback_test)
add_subdirectory(metacall_node_fail_test)
add_subdirectory(metacall_node_fail_env_var_test)
add_subdirectory(metacall_node_fail_load_leak_test)
add_subdirectory(metacall_node_typescript_test)
add_subdirectory(metacall_node_python_async_after_destroy_test)
add_subdirectory(metacall_node_python_await_test)
add_subdirectory(metacall_node_clear_mem_test)
add_subdirectory(metacall_node_async_resources_test)
add_subdirectory(metacall_node_await_chain_test)
add_subdirectory(metacall_distributable_test)
add_subdirectory(metacall_cast_test)
add_subdirectory(metacall_init_fini_test)
add_subdirectory(metacall_ducktype_test)
add_subdirectory(metacall_inspect_test)
add_subdirectory(metacall_integration_test)
add_subdirectory(metacall_depends_test)
add_subdirectory(metacall_configuration_exec_path_test)
add_subdirectory(metacall_clear_test)
add_subdirectory(metacall_python_test)
add_subdirectory(metacall_python_object_class_test)
add_subdirectory(metacall_python_gc_test)
add_subdirectory(metacall_python_open_test)
add_subdirectory(metacall_python_dict_test)
add_subdirectory(metacall_python_model_test)
add_subdirectory(metacall_python_pointer_test)
add_subdirectory(metacall_python_reentrant_test)
add_subdirectory(metacall_python_varargs_test)
add_subdirectory(metacall_python_loader_port_test)
add_subdirectory(metacall_python_port_test)
add_subdirectory(metacall_python_port_https_test)
add_subdirectory(metacall_python_port_callback_test)
add_subdirectory(metacall_python_callback_test)
add_subdirectory(metacall_python_fail_test)
add_subdirectory(metacall_python_relative_path_test)
add_subdirectory(metacall_python_without_functions_test)
add_subdirectory(metacall_python_builtins_test)
add_subdirectory(metacall_python_async_test)
add_subdirectory(metacall_python_await_test)
add_subdirectory(metacall_python_node_await_test)
add_subdirectory(metacall_map_test)
add_subdirectory(metacall_map_await_test)
add_subdirectory(metacall_initialize_test)
add_subdirectory(metacall_initialize_ex_test)
add_subdirectory(metacall_reinitialize_test)
add_subdirectory(metacall_initialize_destroy_multiple_test)
add_subdirectory(metacall_initialize_destroy_multiple_node_test)
add_subdirectory(metacall_reload_functions_test)
add_subdirectory(metacall_invalid_loader_test)
add_subdirectory(metacall_fork_test)
add_subdirectory(metacall_return_monad_test)
add_subdirectory(metacall_callback_complex_test)
add_subdirectory(metacall_ruby_fail_test)
add_subdirectory(metacall_ruby_fail_empty_test)
add_subdirectory(metacall_ruby_object_class_test)
add_subdirectory(metacall_ruby_parser_integration_test)
# add_subdirectory(metacall_ruby_rails_integration_test) # TODO
add_subdirectory(metacall_function_test)
add_subdirectory(metacall_cobol_test)
add_subdirectory(metacall_file_test)
add_subdirectory(metacall_file_fail_test)
add_subdirectory(metacall_typescript_test)
add_subdirectory(metacall_typescript_node_test)
add_subdirectory(metacall_typescript_call_map_test)
add_subdirectory(metacall_typescript_tsx_test)
add_subdirectory(metacall_typescript_tsx_loop_fail_test)
add_subdirectory(metacall_typescript_require_test)
add_subdirectory(metacall_typescript_jsx_default_test)
add_subdirectory(metacall_lua_test)
add_subdirectory(metacall_rpc_test)
#add_subdirectory(metacall_csharp_function_test) # TODO: C# 9.0 seems not to work so top level expressions do not work
add_subdirectory(metacall_csharp_static_class_test)
add_subdirectory(metacall_llvm_test)
add_subdirectory(metacall_ruby_test)
add_subdirectory(metacall_cs_test)
add_subdirectory(metacall_julia_test)
add_subdirectory(metacall_java_test)
add_subdirectory(metacall_wasm_test)
add_subdirectory(metacall_wasm_python_port_test)
add_subdirectory(metacall_rust_test)
add_subdirectory(metacall_c_test)
add_subdirectory(metacall_version_test)
add_subdirectory(metacall_dynlink_path_test)
add_subdirectory(metacall_library_path_without_env_vars_test)
