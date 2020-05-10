#Executive Reports: Summary of Host Usage Resources

The Summary of Host Usage Resources report shows a scatter plot of average host resource utilization for a given time period in terms of CPU and memory usage.

#Grafana Name
Summary of Host Usage Resources (BR17)

#Grafana Description
The report contains a scattered chart of CPU and memory usage data within a requested period and for a selected cluster.

#Grafana Query
--BR17 - This query calculates the hosts "Average Usage Peaks"
-- By Average value of daily "cpu usage peak" vs.
-- Average values of daily "memory usage peak".

SELECT
    host_id,
    host_name,
    delete_date,
    avg_cpu_peak,
    avg_mem_peak
FROM (
    SELECT
        TBL_hourly_PEAKS.host_id,
        host_name,
        delete_date,
        -- Average value of daily cpu usage peak
        CAST (
            AVG ( TBL_hourly_PEAKS.cpu_peak )  AS int
        )
        AS avg_cpu_peak,
        -- Average values of daily memory usage peak
        CAST (
            AVG ( TBL_hourly_PEAKS.mem_peak ) AS int
        )
        AS avg_mem_peak
    FROM (
        -- Calculation of daily cpu and memory usage peaks
        SELECT
            v4_4_statistics_hosts_resources_usage_hourly.host_id,
            host_name,
            history_datetime,
            delete_date,
            MAX (
                COALESCE ( max_cpu_usage, 0 )
            )
            AS cpu_peak,
            MAX (
                COALESCE ( max_memory_usage, 0 )
            )
            AS mem_peak
        FROM v4_4_statistics_hosts_resources_usage_hourly
            INNER JOIN v4_4_configuration_history_hosts
            ON (
                v4_4_configuration_history_hosts.host_id =
                v4_4_statistics_hosts_resources_usage_hourly.host_id
            )
        WHERE
            v4_4_statistics_hosts_resources_usage_hourly.host_status = 1
            AND v4_4_configuration_history_hosts.cluster_id IN (
                SELECT cluster_id
                    FROM v4_4_configuration_history_clusters
                    WHERE datacenter_id = '$datacenter_id'
            )
            AND
                v4_4_configuration_history_hosts.cluster_id IN ( '$cluster_id' )
            AND history_datetime >= $__timeFrom()
            AND history_datetime < $__timeTo()
            -- Here we get the latest host configurations
            AND v4_4_configuration_history_hosts.history_id IN (
                SELECT MAX ( a.history_id )
                FROM v4_4_configuration_history_hosts a
                GROUP BY a.host_id
            )
            -- Here we include or remove deleted entities according to what the user
            -- chose in the "is_deleted" parameter.
            AND
                CASE
                    WHEN '$show_deleted' = 'No'
                        THEN delete_date IS NULL
                    ELSE
                        delete_date IS NULL
                        OR
                        delete_date IS NOT NULL
                END
        GROUP BY
            v4_4_statistics_hosts_resources_usage_hourly.host_id,
            host_name,
            delete_date,
            history_datetime
    )
    AS TBL_hourly_PEAKS
    GROUP BY
          TBL_hourly_PEAKS.host_id,
          host_name, delete_date

    UNION ALL

    SELECT
        TBL_daily_PEAKS.host_id,
        host_name,
        delete_date,
        -- Average value of daily cpu usage peak
        CAST (
            AVG ( TBL_daily_PEAKS.cpu_peak ) AS int
        )
        AS avg_cpu_peak,
        -- Average values of daily memory usage peak
        CAST (
            AVG ( TBL_daily_PEAKS.mem_peak ) AS int
        )
        AS avg_mem_peak
    FROM (
        -- Calculation of daily cpu and memory usage peaks
        SELECT
            v4_4_statistics_hosts_resources_usage_daily.host_id,
            host_name,
            history_datetime,
            delete_date,
            MAX (
                COALESCE ( max_cpu_usage, 0 )
            )
            AS cpu_peak,
            MAX (
                COALESCE ( max_memory_usage, 0 )
            )
            AS mem_peak
        FROM v4_4_statistics_hosts_resources_usage_daily
            INNER JOIN v4_4_configuration_history_hosts
            ON (
                v4_4_configuration_history_hosts.host_id =
                v4_4_statistics_hosts_resources_usage_daily.host_id
            )
        WHERE
            v4_4_statistics_hosts_resources_usage_daily.host_status = 1
            AND v4_4_configuration_history_hosts.cluster_id IN (
                SELECT cluster_id
                    FROM v4_4_configuration_history_clusters
                    WHERE datacenter_id = '$datacenter_id'
            )
            AND
                v4_4_configuration_history_hosts.cluster_id IN ( '$cluster_id' )
            AND history_datetime >= $__timeFrom()
            AND history_datetime < $__timeTo()
            -- Here we get the latest host configurations
            AND v4_4_configuration_history_hosts.history_id IN (
                SELECT MAX ( a.history_id )
                FROM v4_4_configuration_history_hosts a
                GROUP BY a.host_id
            )
            -- Here we include or remove deleted entities according to what the user
            -- chose in the "is_deleted" parameter.
            AND
                CASE
                    WHEN '$show_deleted' = 'No'
                        THEN delete_date IS NULL
                    ELSE
                        delete_date IS NULL
                        OR
                        delete_date IS NOT NULL
                END
        GROUP BY
            v4_4_statistics_hosts_resources_usage_daily.host_id,
            host_name,
            delete_date,
            history_datetime
    )
    AS TBL_daily_PEAKS
    GROUP BY
          TBL_daily_PEAKS.host_id,
          host_name, delete_date
) as b
ORDER BY delete_date DESC, host_id
