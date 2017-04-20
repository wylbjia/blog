### PostgreSQL 的 postgresql.conf 配置文件详解

postgresql.conf 是 PostgreSQL 的主配置文件。包含了 PostgreSQL 服务运行所必需的所有基础配置选项。并且有一些参数选项，需要根据服务器的硬件配置来进行设置。本文档以官方 PostgreSQL 9.6 版本的配置文件为例。

#### 配置文件中的参数值单位说明

内存单位: `kB` = 千字节， `MB` = 兆字节， `GB` = 千兆字节， `TB` = 兆兆字节

时间单位: `ms` = 毫秒， `s` = 秒， `min` = 分钟， `h` = 小时， `d` = 天

#### PostgreSQL 配置文件选项列表

```
data_directory = 'ConfigDir'
```

```
hba_file = 'ConfigDir/pg_hba.conf'
```

```
ident_file = 'ConfigDir/pg_ident.conf'
```
```
external_pid_file = ''
```

```
listen_addresses = 'localhost'
```
PostgreSQL 服务器的监听 IP 地址，如果为 `*` 表示监听在本机的任何 IP 地址

```
port = 5432
```
PostgreSQL 服务器的监听端口，默认为 5432

```
max_connections = 100
```
服务器同时能够接受的最大并发连接数

```
superuser_reserved_connections = 3
```

```
unix_socket_directories = '/tmp'
```

```
unix_socket_group = ''
```

```
unix_socket_permissions = 0777
```

```
bonjour = off
```

```
bonjour_name
```

```
authentication_timeout = 1min
```

```
ssl = off
```

```
ssl_prefer_server_ciphers = on
```

```
ssl_ecdh_curve = 'prime256v1'
```

```
ssl_cert_file = 'server.crt'
```

```
ssl_key_file = 'server.key'
```

```
ssl_ca_file = ''
```

```
ssl_crl_file = ''
```

```
password_encryption = on
```

```
db_user_namespace = off
```

```
row_security = on
```

```
krb_server_keyfile = ''
```

```
krb_caseins_users = off
```

```
tcp_keepalives_idle = 0
```

```
tcp_keepalives_interval = 0
```

```
tcp_keepalives_count = 0
```

```
shared_buffers = 32MB
```
此设置定义了用于缓存最近访问过的数据页的内存区大小，所有用户会话均可共享此缓存区。此设置对查询速度有着重大影响，一般来说是越大越好，至少应该达到系统总内存的 25%，但不宜超过 8 GB，因为超过后会出现“边际收益递减”效应，即消耗的内存很多，但得到的速度提升却很少，得不偿失。修改此设置需要重启 PostgreSQL 服务。


```
huge_pages = try
```

```
temp_buffers = 8MB
```

```
max_prepared_transactions = 0
```

```
work_mem = 4MB
```
此设置指定了用于执行排序、哈希关联、表扫描等操作的最大内存量。要得到此设置的最优值需要考虑以下一些因素：数据库的使用方式，需要预留多少内存给除数据库系统外的程序，以及服务器是否专用于运行 PostgreSQL 服务等问题。如果使用场景仅仅是有很多用户并发执行简单查询，那么这个值设得很小也没问题。

```
maintenance_work_mem = 64MB
```
此设置指定可用于 vaccum 操作（即清空已标记为“被删除”状态的记录）这类系统内部维护操作的内存总量。其值不应大于 1GB。此设置的更改可动态生效，执行重新加载即可。

```
replacement_sort_tuples = 150000
```

```
autovacuum_work_mem = -1
```

```
max_stack_depth = 2MB
```

```
dynamic_shared_memory_type = posix
```

```
temp_file_limit = -1
```

```
max_files_per_process = 1000
```

```
shared_preload_libraries = ''
```

```
vacuum_cost_delay = 0
```

```
vacuum_cost_page_hit = 1
```

```
vacuum_cost_page_miss = 10
```

```
vacuum_cost_page_dirty = 20
```

```
vacuum_cost_limit = 200
```

```
bgwriter_delay = 200ms
```

```
bgwriter_lru_maxpages = 100
```

```
bgwriter_lru_multiplier = 2.0
```

```
bgwriter_flush_after = 0
```

```
effective_io_concurrency = 1
```

```
max_worker_processes = 8
```

```
max_parallel_workers_per_gather = 0
```

```
old_snapshot_threshold = -1
```

```
backend_flush_after = 0
```

```
wal_level = minimal
```

```
fsync = on
```

```
synchronous_commit = on
```

```
wal_sync_method = fsync
```

```
full_page_writes = on
```

```
wal_compression = off
```

```
wal_log_hints = off
```

```
wal_buffers = -1
```

```
wal_writer_delay = 200ms
```

```
wal_writer_flush_after = 1MB
```

```
commit_delay = 0
```

```
commit_siblings = 5
```

```
checkpoint_timeout = 5min
```

```
max_wal_size = 1GB
```

```
min_wal_size = 80MB
```

```
checkpoint_completion_target = 0.5
```

```
checkpoint_flush_after = 0
```

```
checkpoint_warning = 30s
```

```
archive_mode = off
```

```
archive_command = ''
```

```
archive_timeout = 0
```

```
max_wal_senders = 0
```

```
wal_keep_segments = 0
```

```
wal_sender_timeout = 60s
```

```
max_replication_slots = 0
```

```
track_commit_timestamp = off
```

```
synchronous_standby_names = ''
```
```
vacuum_defer_cleanup_age = 0
```

```
hot_standby = off
```

```
max_standby_archive_delay = 30s
```

```
max_standby_streaming_delay = 30s
```

```
wal_receiver_status_interval = 10s
```

```
hot_standby_feedback = off
```

```
wal_receiver_timeout = 60s
```

```
wal_retrieve_retry_interval = 5s
```

```
enable_bitmapscan = on
```

```
enable_hashagg = on
```

```
enable_hashjoin = on
```

```
enable_indexscan = on
```

```
enable_indexonlyscan = on
```

```
enable_material = on
```

```
enable_mergejoin = on
```

```
enable_nestloop = on
```

```
enable_seqscan = on
```

```
enable_sort = on
```

```
enable_tidscan = on
```

```
seq_page_cost = 1.0
```

```
random_page_cost = 4.0
```

```
cpu_tuple_cost = 0.01
```

```
cpu_index_tuple_cost = 0.005
```

```
cpu_operator_cost = 0.0025
```

```
parallel_tuple_cost = 0.1
```

```
parallel_setup_cost = 1000.0
```

```
min_parallel_relation_size = 8MB
```

```
effective_cache_size = 4GB
```
此设置表示一个查询执行过程中可以使用的最大缓存，包括操作系统使用的部分以及PostgreSQL使用的部分。系统并不会根据这个值来真实地分配这么多内存，但是规划器会根据这个值来判断系统能否提供查询执行过程中所需的内存。如果将此设置设得过小，远远小于系统真实可用内存量，那么可能会给规划器造成误导，让规划器认为系统可用内存有限，从而选择不使用索引而是走全表扫描（因为使用索引虽然速度快，但需要占用更多的中间内存）。在一台专用于运行 PostgreSQL 数据库服务的服务器上，建议将 effective_cache_size 的值设为系统总内存的一半或者更多。此设置的更改可动态生效，执行重新加载即可。

```
geqo = on
```

```
geqo_threshold = 12
```

```
geqo_effort = 5
```

```
geqo_pool_size = 0
```

```
geqo_generations = 0
```

```
geqo_selection_bias = 2.0
```

```
geqo_seed = 0.0
```

```
default_statistics_target = 100
```

```
constraint_exclusion = partition
```

```
cursor_tuple_fraction = 0.1
```

```
from_collapse_limit = 8
```

```
join_collapse_limit = 8
```

```
force_parallel_mode = off
```

```
log_destination = 'stderr'
```

```
logging_collector = off
```

```
log_directory = 'pg_log'
```

```
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
```

```
log_file_mode = 0600
```

```
log_truncate_on_rotation = off
```

```
log_rotation_age = 1d
```

```
log_rotation_size = 10MB
```

```
syslog_facility = 'LOCAL0'
```

```
syslog_ident = 'postgres'
```

```
syslog_sequence_numbers = on
```

```
syslog_split_messages = on
```

```
event_source = 'PostgreSQL'
```

```
client_min_messages = notice
```

```
log_min_messages = warning
```

```
log_min_error_statement = error
```

```
log_min_duration_statement = -1
```

```
debug_print_parse = off
```

```
debug_print_rewritten = off
```

```
debug_print_plan = off
```

```
debug_pretty_print = on
```

```
log_checkpoints = off
```

```
log_connections = off
```

```
log_disconnections = off
```

```
log_duration = off
```

```
log_error_verbosity = default
```

```
log_hostname = off
```

```
log_line_prefix = ''
```

```
log_lock_waits = off
```

```
log_statement = 'none'
```

```
log_replication_commands = off
```

```
log_temp_files = -1
```

```
log_timezone = 'GMT'
```

```
cluster_name = ''
```

```
update_process_title = on
```

```
track_activities = on
```

```
track_counts = on
```

```
track_io_timing = off
```

```
track_functions = none
```

```
track_activity_query_size = 1024
```

```
stats_temp_directory = 'pg_stat_tmp'
```

```
log_parser_stats = off
```

```
log_planner_stats = off
```

```
log_executor_stats = off
```

```
log_statement_stats = off
```

```
autovacuum = on
```

```
log_autovacuum_min_duration = -1
```

```
autovacuum_max_workers = 3
```

```
autovacuum_naptime = 1min
```

```
autovacuum_vacuum_threshold = 50
```

```
autovacuum_analyze_threshold = 50
```

```
autovacuum_vacuum_scale_factor = 0.2
```

```
autovacuum_analyze_scale_factor = 0.1
```

```
autovacuum_freeze_max_age = 200000000
```

```
autovacuum_multixact_freeze_max_age = 400000000
```

```
autovacuum_vacuum_cost_delay = 20ms
```

```
autovacuum_vacuum_cost_limit = -1
```

```
search_path = '"$user", public'
```

```
default_tablespace = ''
```

```
temp_tablespaces = ''
```

```
check_function_bodies = on
```

```
default_transaction_isolation = 'read committed'
```

```
default_transaction_read_only = off
```

```
default_transaction_deferrable = off
```

```
session_replication_role = 'origin'
```

```
statement_timeout = 0
```

```
lock_timeout = 0
```

```
idle_in_transaction_session_timeout = 0
```

```
vacuum_freeze_min_age = 50000000
```

```
vacuum_freeze_table_age = 150000000
```

```
vacuum_multixact_freeze_min_age = 5000000
```

```
vacuum_multixact_freeze_table_age = 150000000
```

```
bytea_output = 'hex'
```

```
xmlbinary = 'base64'
```

```
xmloption = 'content'
```

```
gin_fuzzy_search_limit = 0
```

```
gin_pending_list_limit = 4MB
```

```
datestyle = 'iso, mdy'
```

```
intervalstyle = 'postgres'
```

```
timezone = 'GMT'
```

```
timezone_abbreviations = 'Default'
```

```
extra_float_digits = 0
```

```
client_encoding = sql_ascii
```

```
lc_messages = 'C'
```

```
lc_monetary = 'C'
```

```
lc_numeric = 'C'
```

```
lc_time = 'C'
```

```
default_text_search_config = 'pg_catalog.simple'
```

```
dynamic_library_path = '$libdir'
```

```
local_preload_libraries = ''
```

```
session_preload_libraries = ''
```

```
deadlock_timeout = 1s
```

```
max_locks_per_transaction = 64
```

```
max_pred_locks_per_transaction = 64
```

```
array_nulls = on
```

```
backslash_quote = safe_encoding
```

```
default_with_oids = off
```

```
escape_string_warning = on
```

```
lo_compat_privileges = off
```

```
operator_precedence_warning = off
```

```
quote_all_identifiers = off
```

```
sql_inheritance = on
```

```
standard_conforming_strings = on
```

```
synchronize_seqscans = on
```

```
transform_null_equals = off
```

```
exit_on_error = off
```

```
restart_after_crash = on
```

```
include_dir = 'conf.d'
```

```
include_if_exists = 'exists.conf'
```

```
include = 'special.conf'
```

---------------------------------

Author: typefo <typefo@qq.com> 本文档使用 CC-BY 4.0 协议 ![by](../by.png)
