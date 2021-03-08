# mysql_sys.x-user_summary

SELECT 
    IF((`performance_schema`.`accounts`.`USER` IS NULL),
        'background',
        `performance_schema`.`accounts`.`USER`) AS `user`,
    SUM(`sys`.`stmt`.`total`) AS `statements`,
    SUM(`sys`.`stmt`.`total_latency`) AS `statement_latency`,
    IFNULL((SUM(`sys`.`stmt`.`total_latency`) / NULLIF(SUM(`sys`.`stmt`.`total`), 0)),
            0) AS `statement_avg_latency`,
    SUM(`sys`.`stmt`.`full_scans`) AS `table_scans`,
    SUM(`sys`.`io`.`ios`) AS `file_ios`,
    SUM(`sys`.`io`.`io_latency`) AS `file_io_latency`,
    SUM(`performance_schema`.`accounts`.`CURRENT_CONNECTIONS`) AS `current_connections`,
    SUM(`performance_schema`.`accounts`.`TOTAL_CONNECTIONS`) AS `total_connections`,
    COUNT(DISTINCT `performance_schema`.`accounts`.`HOST`) AS `unique_hosts`,
    SUM(`sys`.`mem`.`current_allocated`) AS `current_memory`,
    SUM(`sys`.`mem`.`total_allocated`) AS `total_memory_allocated`
FROM
    (((`performance_schema`.`accounts`
    LEFT JOIN `sys`.`x$user_summary_by_statement_latency` `stmt` ON ((IF((`performance_schema`.`accounts`.`USER` IS NULL), 'background', `performance_schema`.`accounts`.`USER`) = `sys`.`stmt`.`user`)))
    LEFT JOIN `sys`.`x$user_summary_by_file_io` `io` ON ((IF((`performance_schema`.`accounts`.`USER` IS NULL), 'background', `performance_schema`.`accounts`.`USER`) = `sys`.`io`.`user`)))
    LEFT JOIN `sys`.`x$memory_by_user_by_current_bytes` `mem` ON ((IF((`performance_schema`.`accounts`.`USER` IS NULL), 'background', `performance_schema`.`accounts`.`USER`) = `sys`.`mem`.`user`)))
GROUP BY IF((`performance_schema`.`accounts`.`USER` IS NULL),
    'background',
    `performance_schema`.`accounts`.`USER`)
ORDER BY SUM(`sys`.`stmt`.`total_latency`) DESC
