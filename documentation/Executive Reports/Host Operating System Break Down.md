#Executive Reports: Host Operating System Break Down

The Host OS Break Down report indicates the number of hosts running each operating system version over a given time period.

#Grafana Name
Host OS Break Down (BR22)

##Grafana Description
This report contains a table and a chart displaying the number of hosts for each OS version for a selected cluster within a requested period.

##Grafana Query
-- Datacenter aggregation level
--------
-- BR22 - Hosts Count by OS Type versus OS Version

SELECT
    COALESCE ( v4_4_configuration_history_hosts.host_os, 'Unknown' ) AS os_type,
    COUNT ( DISTINCT v4_4_configuration_history_hosts.host_id ) AS host_count
FROM  v4_4_statistics_hosts_resources_usage_hourly AS stats_hosts
    INNER JOIN v4_4_configuration_history_hosts
        ON (
            v4_4_configuration_history_hosts.history_id =
            stats_hosts.host_configuration_version
        )
    INNER JOIN v4_4_configuration_history_hosts a
        ON (
            a.host_id =
            stats_hosts.host_id
        )
WHERE
    v4_4_configuration_history_hosts.cluster_id IN (
        SELECT v4_4_configuration_history_clusters.cluster_id
        FROM v4_4_configuration_history_clusters
        WHERE
            v4_4_configuration_history_clusters.datacenter_id = '$datacenter_id'
    )
    -- Here we filter by the cluster chosen by the user
    AND v4_4_configuration_history_hosts.cluster_id IN ( '$cluster_id' )
    AND history_datetime >= $__timeFrom()
    AND history_datetime < $__timeTo()
    -- Here we get the latest host configurations
    AND a.history_id IN (
        SELECT MAX ( b.history_id )
        FROM v4_4_configuration_history_hosts b
        GROUP BY b.host_id
    )
    AND
        CASE
            WHEN '$show_deleted' = 'No'
                THEN a.delete_date IS NULL
            ELSE
                a.delete_date IS NULL
                OR
                a.delete_date IS NOT NULL
        END
GROUP BY
    COALESCE ( v4_4_configuration_history_hosts.host_os, 'Unknown' )


-- Datacenter aggregation level
--------
-- BR22 - This query returns the daily / hourly number of hosts grouped by OS Type.

SELECT DISTINCT
    $__time(history_datetime),
    COALESCE ( os_type, MAX ( os_type ) over (partition by 1), '' ) AS os_type,
    host_count
FROM (
    SELECT
        history_datetime,
        COALESCE (
            v4_4_configuration_history_hosts.host_os,
            'Unknown'
        )
        AS os_type,
        COUNT ( DISTINCT v4_4_configuration_history_hosts.host_id )
        AS host_count
    FROM  v4_4_statistics_hosts_resources_usage_hourly AS stats_hosts
        INNER JOIN v4_4_configuration_history_hosts
            ON (
                v4_4_configuration_history_hosts.history_id =
                stats_hosts.host_configuration_version
            )
        INNER JOIN v4_4_configuration_history_hosts a
            ON (
                a.host_id =
                stats_hosts.host_id
            )
    WHERE
        v4_4_configuration_history_hosts.cluster_id IN (
            SELECT v4_4_configuration_history_clusters.cluster_id
            FROM v4_4_configuration_history_clusters
            WHERE
                v4_4_configuration_history_clusters.datacenter_id = '$datacenter_id'
        )
        AND v4_4_configuration_history_hosts.cluster_id IN ( '$cluster_id' )
        AND history_datetime >= $__timeFrom()
        AND history_datetime < $__timeTo()
        AND a.history_id IN (
            SELECT MAX ( b.history_id )
            FROM v4_4_configuration_history_hosts b
            GROUP BY b.host_id
        )
        AND
            CASE
                WHEN '$show_deleted' = 'No'
                    THEN a.delete_date IS NULL
                ELSE
                    a.delete_date IS NULL
                    OR
                    a.delete_date IS NOT NULL
            END
    GROUP BY
        history_datetime,
        COALESCE (
            v4_4_configuration_history_hosts.host_os,
            'Unknown'
        )
) AS a
GROUP BY
    history_datetime,
    os_type, host_count
ORDER BY history_datetime
