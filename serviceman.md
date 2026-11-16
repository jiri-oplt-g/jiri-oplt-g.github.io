# l2 tools / serviceman

The `serviceman` ensures that defined tasks are run regularly under a specified user account.

# serviceman directory tree

```text
serviceman/
├── l2_mornchecks.json
├── serviceman.fnc
├── serviceman.json
├── serviceman.sh
├── serviceman.log.browser.sh
├── shared_global_variables.inc
└── tasks/
```

- `l2_mornchecks.json` - JSON file containing individual tasks of task l2_mornchecks from l2_mornchecks_list   
- `serviceman.fnc` - bash functions
- `serviceman.json` - default JSON file for defining tasks
- `serviceman.sh` - the main script
- `serviceman.log.browser.sh` - a utility script for viewing serviceman logs
- `shared_global_variables.inc` - definitions of bash variables that are used in multiple scripts 
- `tasks/` - subdirectory containing scripts for tasks

# One task definition in json file - task list

```text
  { "task_name": "l2_mornchecks",
    "instance_name": "TEST_10.5.4",
    "server_name": ["tpifais02"],
    "os_user": ["ininfatl"],
    "hour_band": ["07"],
    "enabled": true,
    "params": {
      "l2_mornchecks_list" : [
        "run_check ipxlst.check_opt_ipx_filling '80'",
        "run_check ipxlst.regs_used_from_local_fs",
        "run_check ipcdom.check_home_ipc_file_rights",
        "run_check infa.morncheck.pgwal '75'",
        "run_check ipxlst.check_detail_log_size '10'",
        "run_check ipxlst.check_aix_etc_inittab_autostart",
        "run_check ipcdom.get_versions",
        "run_check ipxlst.get_versions",
        "run_check ipcdom.check_nas_mount_point_parameters",
        "run_check ipxlst.check_nas_mount_point_parameters",
        "run_check ipcdom.check_line_after_syslog_target",
        "run_check ipxlst.check_line_after_syslog_target",
        "run_check ipxlst.check_crs_resources",
        "run_check ipxlst.check_systemd_services",
        "run_check ipcdom.check_user_IPX_tbs_USERS_quota '10485760'",
        "run_check ipxlst.rto_db.config_status_service",
        "run_check ipxlst.check_user_IPX_grants",
        "run_check ipxlst.check_running_dtllst_processes",
        "run_check morncheck.RAC_RTO_DB_sessions_balance",
        "run_check morncheck.find_menu_processes_without_terminal",
        "run_check ipcdom.check_paused_loggers",
        "run_check ipxlst.check_olr_loc_readability",
        "run_check ogg.check_db_and_user_settings"
      ],
      "rm_sql_file_for_insert": "Yes",
      "debug_l2_mornchecks_list": "No"
    }
  },
```

- `task_name` - it is also the name of the script in tasks subdirectory
    - e.g. `task_name = "l2_mornchecks"` ---> script `tasks/l2_mornchecks.sh` is executed
- `instance_name` - this is used for identification when a task is listed multiple times in a JSON file    
- `server_name` - the server where the task is to be run
- `os_user` - the username under which the task should be run
- `hour_band` - the time slot during which the task is to be run
- `enabled` - whether the task is enabled or not
- `params` - task parameters
- `rm_sql_file_for_insert` - option for debugging purposes (task specific)
- `debug_l2_mornchecks_list` - option for debugging purposes (task specific)

# How serviceman works

ℹ️ **INFO:** The assumption is that the `serviceman` folder is located in a shared folder

1. You have an ipc user under which the tasks defined in serviceman.json are to be run
2. The crontab of the ipc user must contain the following line:
   ```
   * * * * * /srv/data/int/infa/ininfatl/remote/tools/cron/crontab.run.check_time.sh > /dev/null 2>&1
   ```
   Script `crontab.run.check_time.sh` is executed every minute
3. The `~/.ipc` file contains a line labeled `SVCMAN`, where the minute for running serviceman is defined in the fourth column. The `crontab.run.check_time.sh` script runs serviceman only at that specific minute; in all other minutes, its execution is skipped.
   ```
   ininfatl@tpifais02 (A) :~$ grep SVCMAN ~/.ipc
   SVCMAN:IPCTL:tpifais02.vs.csin.cz:19
   ininfatl@tpifais02 (A) :~$
   ```
4. In practice, the serviceman is therefore run by the specified ipc user on the specified server only once per hour. 
5. When Serviceman is run under a specific ipc user on a specific server, it begins processing the tasks listed in serviceman.json, and a task is executed if:
   - `enabled` is set to `true`.
   - The current server matches one of the values in `server_name`.
   - The current ipc user matches one of the values in `os_user`.
   - The current hour matches one of the values in `hour_band`.

# Flag files

- When serviceman is launched, it creates a flag file in the flag_files subdirectory. This prevents serviceman from being accidentally launched more than once for a given time zone under a specific user on a given ipc server.
- If I want to intentionally run serviceman multiple times within a given hour — for example, to test the execution of a task — I simply need to delete the relevant flag file. It’s easy to identify the correct one — it contains the server and ipc user names in its name.
- I can tell when serviceman was last run by checking the update time of the flag file.
```
ininfatl@tpifais02 (A) :~$ ll ${IPCIPXLOGS_REMOTE}/${IPC_VERSION}/serviceman/flag_files
total 0
-rw-rw-r--. 1 dainfada ininfatt 0 Oct  8 12:04 serviceman.lastrun.tbifais02.dainfada
-rw-rw-r--. 1 dainfadb ininfatt 0 Oct  8 12:10 serviceman.lastrun.tbifais02.dainfadb
-rw-rw-r--. 1 dainfadc ininfatt 0 Feb  7  2025 serviceman.lastrun.tbifais02.dainfadc
-rw-rw-r--. 1 edinfaea ininfatt 0 Oct  8 12:16 serviceman.lastrun.tbifais02.edinfaea
-rw-rw-r--. 1 fainfatf ininfatt 0 Oct  8 12:53 serviceman.lastrun.tbifais02.fainfatf
-rw-rw-r--. 1 ininfata ininfatt 0 Oct  8 12:34 serviceman.lastrun.tbifais02.ininfata
-rw-rw-r--. 1 ininfatb ininfatt 0 Oct  8 12:40 serviceman.lastrun.tbifais02.ininfatb
-rw-rw-r--. 1 ininfatl ininfatt 0 Oct  8 12:22 serviceman.lastrun.tbifais02.ininfatl
-rw-rw-r--. 1 dainfada ininfatt 0 Oct  8 13:01 serviceman.lastrun.tpifais02.dainfada
-rw-rw-r--. 1 dainfadb ininfatt 0 Oct  8 12:07 serviceman.lastrun.tpifais02.dainfadb
-rw-rw-r--. 1 dainfadc ininfatt 0 Feb  7  2025 serviceman.lastrun.tpifais02.dainfadc
-rw-rw-r--. 1 edinfaea ininfatt 0 Oct  8 12:13 serviceman.lastrun.tpifais02.edinfaea
-rw-rw-r--. 1 fainfatf ininfatt 0 Oct  8 12:50 serviceman.lastrun.tpifais02.fainfatf
-rw-rw-r--. 1 ininfata ininfatt 0 Oct  8 12:31 serviceman.lastrun.tpifais02.ininfata
-rw-rw-r--. 1 ininfatb ininfatt 0 Oct  8 12:37 serviceman.lastrun.tpifais02.ininfatb
-rw-rw-r--. 1 ininfatl ininfatt 0 Oct  8 12:19 serviceman.lastrun.tpifais02.ininfatl
ininfatl@tpifais02 (A) :~$
```

# Logs

- The log directory contains logs for every combination of ipc user and ipc server.
- The logs are in JSON format.
- The logs are managed by the log_shift task. The parameters for log_shift are the maximum size of a single log and the number of logs. When the size of the current log exceeds its maximum size, a log shift occurs and the contents of the oldest log are discarded.
```
ininfatl@tpifais02 (A) :~$ ll ${IPCIPXLOGS_REMOTE}/${IPC_VERSION}/serviceman/logs
total 159044
-rw-rw-r--. 1 dainfada ininfatt 2145281 Oct  8 13:04 serviceman.tbifais02.dainfada.log
-rw-rw-r--. 1 dainfada ininfatt 3008884 Mar 17  2026 serviceman.tbifais02.dainfada.log.1
-rw-rw-r--. 1 dainfada ininfatt 3007484 Jun  2  2025 serviceman.tbifais02.dainfada.log.2
-rw-rw-r--. 1 dainfada ininfatt 3009401 Aug 19  2024 serviceman.tbifais02.dainfada.log.3
-rw-rw-r--. 1 dainfadb ininfatt 2197201 Oct  8 13:10 serviceman.tbifais02.dainfadb.log
-rw-rw-r--. 1 dainfadb ininfatt 3004574 Mar 13  2026 serviceman.tbifais02.dainfadb.log.1
-rw-rw-r--. 1 dainfadb ininfatt 3002724 May 31  2025 serviceman.tbifais02.dainfadb.log.2
-rw-rw-r--. 1 dainfadb ininfatt 3010341 Aug 19  2024 serviceman.tbifais02.dainfadb.log.3
-rw-rw-r--. 1 dainfadc ininfatt 1563038 Feb  7  2025 serviceman.tbifais02.dainfadc.log
-rw-rw-r--. 1 dainfadc ininfatt 3008741 Aug 29  2024 serviceman.tbifais02.dainfadc.log.1
-rw-rw-r--. 1 edinfaea ininfatt 4410428 Oct  8 13:16 serviceman.tbifais02.edinfaea.log
-rw-rw-r--. 1 edinfaea ininfatt 3010341 Jun  2  2025 serviceman.tbifais02.edinfaea.log.1
-rw-rw-r--. 1 edinfaea ininfatt 3009794 Aug 19  2024 serviceman.tbifais02.edinfaea.log.2
-rw-rw-r--. 1 fainfatf ininfatt 2717072 Oct  8 12:53 serviceman.tbifais02.fainfatf.log
-rw-rw-r--. 1 fainfatf ininfatt 3007788 Feb  4  2026 serviceman.tbifais02.fainfatf.log.1
-rw-rw-r--. 1 fainfatf ininfatt 3006687 May  7  2025 serviceman.tbifais02.fainfatf.log.2
-rw-rw-r--. 1 fainfatf ininfatt 3001118 Aug  8  2024 serviceman.tbifais02.fainfatf.log.3
-rw-rw-r--. 1 ininfata ininfatt 2178862 Oct  8 13:34 serviceman.tbifais02.ininfata.log
-rw-rw-r--. 1 ininfata ininfatt 3004316 Mar 14  2026 serviceman.tbifais02.ininfata.log.1
-rw-rw-r--. 1 ininfata ininfatt 3005137 Jun  1  2025 serviceman.tbifais02.ininfata.log.2
-rw-rw-r--. 1 ininfata ininfatt 3006746 Aug 19  2024 serviceman.tbifais02.ininfata.log.3
-rw-rw-r--. 1 ininfatb ininfatt 2716448 Oct  8 13:40 serviceman.tbifais02.ininfatb.log
-rw-rw-r--. 1 ininfatb ininfatt 3005492 Feb  4  2026 serviceman.tbifais02.ininfatb.log.1
-rw-rw-r--. 1 ininfatb ininfatt 3001564 May  8  2025 serviceman.tbifais02.ininfatb.log.2
-rw-rw-r--. 1 ininfatb ininfatt 3010392 Aug  9  2024 serviceman.tbifais02.ininfatb.log.3
-rw-rw-r--. 1 ininfatl ininfatt  496359 Oct  8 13:22 serviceman.tbifais02.ininfatl.log
-rw-rw-r--. 1 ininfatl ininfatt 3010909 Aug 27 19:22 serviceman.tbifais02.ininfatl.log.1
-rw-rw-r--. 1 ininfatl ininfatt 3012117 Dec 14  2025 serviceman.tbifais02.ininfatl.log.2
-rw-rw-r--. 1 ininfatl ininfatt 3009492 Apr  3  2025 serviceman.tbifais02.ininfatl.log.3
-rw-rw-r--. 1 dainfada ininfatt  414880 Oct  8 13:01 serviceman.tpifais02.dainfada.log
-rw-rw-r--. 1 dainfada ininfatt 3009564 Sep  2 19:01 serviceman.tpifais02.dainfada.log.1
-rw-rw-r--. 1 dainfada ininfatt 3010628 Dec 15  2025 serviceman.tpifais02.dainfada.log.2
-rw-rw-r--. 1 dainfada ininfatt 3002295 Mar 29  2025 serviceman.tpifais02.dainfada.log.3
-rw-rw-r--. 1 dainfadb ininfatt  426469 Oct  8 13:07 serviceman.tpifais02.dainfadb.log
-rw-rw-r--. 1 dainfadb ininfatt 3009244 Sep  1 19:07 serviceman.tpifais02.dainfadb.log.1
-rw-rw-r--. 1 dainfadb ininfatt 3000689 Dec 14  2025 serviceman.tpifais02.dainfadb.log.2
-rw-rw-r--. 1 dainfadb ininfatt 3002295 Mar 29  2025 serviceman.tpifais02.dainfadb.log.3
-rw-rw-r--. 1 dainfadc ininfatt 1286724 Feb  7  2025 serviceman.tpifais02.dainfadc.log
-rw-rw-r--. 1 dainfadc ininfatt 3002400 Sep  8  2024 serviceman.tpifais02.dainfadc.log.1
-rw-rw-r--. 1 edinfaea ininfatt 2307578 Oct  8 13:13 serviceman.tpifais02.edinfaea.log
-rw-rw-r--. 1 edinfaea ininfatt 3007844 Dec 20  2025 serviceman.tpifais02.edinfaea.log.1
-rw-rw-r--. 1 edinfaea ininfatt 3009391 Apr  3  2025 serviceman.tpifais02.edinfaea.log.2
-rw-rw-r--. 1 edinfaea ininfatt 3002207 Jul 17  2024 serviceman.tpifais02.edinfaea.log.3
-rw-rw-r--. 1 fainfatf ininfatt 2876542 Oct  8 13:50 serviceman.tpifais02.fainfatf.log
-rw-rw-r--. 1 fainfatf ininfatt 3002040 Jan 19  2026 serviceman.tpifais02.fainfatf.log.1
-rw-rw-r--. 1 fainfatf ininfatt 3004119 Apr 21  2025 serviceman.tpifais02.fainfatf.log.2
-rw-rw-r--. 1 fainfatf ininfatt 3005755 Jul 23  2024 serviceman.tpifais02.fainfatf.log.3
-rw-rw-r--. 1 ininfata ininfatt 1846639 Oct  8 13:31 serviceman.tpifais02.ininfata.log
-rw-rw-r--. 1 ininfata ininfatt 3002456 Apr 30 19:31 serviceman.tpifais02.ininfata.log.1
-rw-rw-r--. 1 ininfata ininfatt 3009365 Sep  1  2025 serviceman.tpifais02.ininfata.log.2
-rw-rw-r--. 1 ininfata ininfatt 3008671 Jan 31  2025 serviceman.tpifais02.ininfata.log.3
-rw-rw-r--. 1 ininfatb ininfatt 2876222 Oct  8 13:37 serviceman.tpifais02.ininfatb.log
-rw-rw-r--. 1 ininfatb ininfatt 3002360 Jan 19  2026 serviceman.tpifais02.ininfatb.log.1
-rw-rw-r--. 1 ininfatb ininfatt 3006168 Apr 21  2025 serviceman.tpifais02.ininfatb.log.2
-rw-rw-r--. 1 ininfatb ininfatt 3006988 Jul 23  2024 serviceman.tpifais02.ininfatb.log.3
-rw-rw-r--. 1 ininfatl ininfatt 2308040 Oct  8 13:19 serviceman.tpifais02.ininfatl.log
-rw-rw-r--. 1 ininfatl ininfatt 3004827 May 14 19:19 serviceman.tpifais02.ininfatl.log.1
-rw-rw-r--. 1 ininfatl ininfatt 3008789 Oct 30  2025 serviceman.tpifais02.ininfatl.log.2
-rw-rw-r--. 1 ininfatl ininfatt 3011570 Feb 28  2025 serviceman.tpifais02.ininfatl.log.3
ininfatl@tpifais02 (A) :~$
```

- Find today's run of task `l2_mornchecks` :
```
ininfatl@tpifais02 (A) :~$ ${IPCIPXTOOLS_REMOTE}/bin/jq 'select((.task_name | contains("l2_mornchecks")?) and (.date | contains("2026-10-08")?))' ${IPCIPXLOGS_REMOTE}/${IPC_VERSION}/serviceman/logs/serviceman.tpifais02.ininfatl.log
{
  "date": "2026-10-08 07:19:02.614824324",
  "server_name": "tpifais02",
  "os_user": "ininfatl",
  "event": "run_task",
  "serviceman_run_id": "20261008071902340934849",
  "task_name": "/srv/data/int/infa/ininfatl/remote/tools/serviceman/tasks/l2_mornchecks.sh",
  "instance_name": "",
  "task_parameters": {
    "l2_mornchecks_list": [
      "run_check ipxlst.check_opt_ipx_filling '80'",
      "run_check ipxlst.regs_used_from_local_fs",
      "run_check ipcdom.check_home_ipc_file_rights",
      "run_check infa.morncheck.pgwal '75'",
      "run_check ipxlst.check_detail_log_size '10'",
      "run_check ipxlst.check_aix_etc_inittab_autostart",
      "run_check ipcdom.get_versions",
      "run_check ipxlst.get_versions",
      "run_check ipcdom.check_nas_mount_point_parameters",
      "run_check ipxlst.check_nas_mount_point_parameters",
      "run_check ipcdom.check_line_after_syslog_target",
      "run_check ipxlst.check_line_after_syslog_target",
      "run_check ipxlst.check_crs_resources",
      "run_check ipxlst.check_systemd_services",
      "run_check ipcdom.check_user_IPX_tbs_USERS_quota '10485760'",
      "run_check ipxlst.rto_db.config_status_service",
      "run_check ipxlst.check_user_IPX_grants",
      "run_check ipxlst.check_running_dtllst_processes",
      "run_check morncheck.RAC_RTO_DB_sessions_balance",
      "run_check morncheck.find_menu_processes_without_terminal",
      "run_check ipcdom.check_paused_loggers",
      "run_check ipxlst.check_olr_loc_readability",
      "run_check ogg.check_db_and_user_settings"
    ],
    "rm_sql_file_for_insert": "Yes",
    "debug_l2_mornchecks_list": "No"
  }
}
{
  "date": "2026-10-08 07:19:02.662953674",
  "server_name": "tpifais02",
  "os_user": "ininfatl",
  "event": "task_begin",
  "serviceman_run_id": "20261008071902340934849",
  "task_name": "/srv/data/int/infa/ininfatl/remote/tools/serviceman/tasks/l2_mornchecks.sh"
}
{
  "date": "2026-10-08 07:43:19.801750645",
  "server_name": "tpifais02",
  "os_user": "ininfatl",
  "event": "task_end",
  "serviceman_run_id": "20261008071902340934849",
  "task_name": "/srv/data/int/infa/ininfatl/remote/tools/serviceman/tasks/l2_mornchecks.sh"
}
{
  "date": "2026-10-08 07:43:19.898612690",
  "server_name": "tpifais02",
  "os_user": "ininfatl",
  "event": "run_task",
  "serviceman_run_id": "20261008071902340934849",
  "task_name": "/srv/data/int/infa/ininfatl/remote/tools/serviceman/tasks/l2_mornchecks.sh",
  "instance_name": "",
  "task_parameters": {
    "ipc_version": "10.5.8",
    "l2_mornchecks_list": [
      "run_check ipxlst.regs_used_from_local_fs",
      "run_check ipxlst.check_detail_log_size",
      "run_check ipxlst.check_aix_etc_inittab_autostart",
      "run_check ipcdom.get_versions",
      "run_check ipxlst.get_versions",
      "run_check ipxlst.check_nas_mount_point_parameters",
      "run_check ipxlst.check_line_after_syslog_target",
      "run_check ipxlst.check_systemd_services",
      "run_check ipcdom.check_user_IPX_tbs_USERS_quota",
      "run_check ipxlst.check_running_dtllst_processes",
      "run_check morncheck.find_menu_processes_without_terminal",
      "run_check ipcdom.check_paused_loggers"
    ]
  }
}
{
  "date": "2026-10-08 07:43:19.938518734",
  "server_name": "tpifais02",
  "os_user": "ininfatl",
  "event": "task_begin",
  "serviceman_run_id": "20261008071902340934849",
  "task_name": "/srv/data/int/infa/ininfatl/remote/tools/serviceman/tasks/l2_mornchecks.sh"
}
{
  "date": "2026-10-08 07:47:23.918902458",
  "server_name": "tpifais02",
  "os_user": "ininfatl",
  "event": "task_end",
  "serviceman_run_id": "20261008071902340934849",
  "task_name": "/srv/data/int/infa/ininfatl/remote/tools/serviceman/tasks/l2_mornchecks.sh"
}
ininfatl@tpifais02 (A) :~$
```
