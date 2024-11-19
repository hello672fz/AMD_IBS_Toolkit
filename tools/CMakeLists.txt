
## ibs_monitor
set(IBS_MONITOR_SEC_FILES
        ${BASE_DIR}/tools/ibs_monitor/cpu_check.c
        ${BASE_DIR}/tools/ibs_monitor/ibs_monitor.c
)
add_executable(ibs_monitor ${IBS_MONITOR_SEC_FILES})
target_include_directories(ibs_monitor PUBLIC ${BASE_DIR}/include)