#Executive Reports: Cluster Capacity Vs Usage

The Cluster Capacity Vs Usage report shows the relationship between system capacity and usage (workload) over a given time period. Capacity is expressed in terms of CPU cores and physical memory, while usage is expressed as vCPUs and virtual machine memory.

#Grafana Query
SELECT DISTINCT
    host_cpu_cores,
    vm_cpu_cores_total,
    host_mem_avg,
    vm_mem_total
FROM (
    SELECT
        t.history_datetime AS time,
        host_cpu_cores,
        host_mem_avg,
        vm_cpu_cores_total,
        vm_mem_total
    FROM (
        SELECT
            history_datetime,
            SUM ( coalesce ( v4_4_configuration_history_hosts.number_of_cores , 0 ) * minutes_in_status ) / SUM ( minutes_in_status ) as host_cpu_cores,
            SUM ( coalesce ( v4_4_configuration_history_hosts.memory_size_mb , 0 ) * minutes_in_status ) / SUM ( minutes_in_status ) as host_mem_avg
        FROM v4_4_statistics_hosts_resources_usage_samples
            INNER JOIN v4_4_configuration_history_hosts
                ON (
                    v4_4_configuration_history_hosts.history_id = v4_4_statistics_hosts_resources_usage_samples.host_configuration_version
                )
                    INNER JOIN v4_4_configuration_history_hosts latest
                        ON (
                            latest.host_id = v4_4_statistics_hosts_resources_usage_samples.host_id
                        )
        WHERE v4_4_statistics_hosts_resources_usage_samples.host_status = 1
            AND v4_4_configuration_history_hosts.cluster_id in (
                SELECT v4_4_configuration_history_clusters.cluster_id
                FROM v4_4_configuration_history_clusters
                WHERE v4_4_configuration_history_clusters.datacenter_id = '$datacenter_id'
            )
            AND v4_4_configuration_history_hosts.cluster_id IN ( '$cluster_id' )
            AND history_datetime >= $__timeFrom()
            AND history_datetime < $__timeTo()
            AND latest.history_id in (
                SELECT max ( b.history_id )
                FROM v4_4_configuration_history_hosts b
                GROUP BY b.host_id
            )
            AND
                CASE
                    WHEN '$show_deleted' = 'No'
                        THEN latest.delete_date IS NULL
                    ELSE
                        latest.delete_date IS NULL
                        OR
                        latest.delete_date IS NOT NULL
                END
        GROUP BY history_datetime
    ) as t
    INNER JOIN (
        SELECT
            nested_query.history_datetime,
            SUM ( nested_query.cpu_cores ) as vm_cpu_cores_total,
            SUM ( nested_query.mem_total ) as vm_mem_total
        FROM (
            SELECT
                history_datetime,
                v4_4_configuration_history_vms.vm_id,
                SUM ( coalesce ( v4_4_configuration_history_vms.cpu_per_socket , 0 ) * coalesce ( v4_4_configuration_history_vms.number_of_sockets , 0 ) * minutes_in_status ) / SUM ( minutes_in_status ) as cpu_cores,
                SUM ( coalesce ( v4_4_configuration_history_vms.memory_size_mb , 0 ) * minutes_in_status ) / SUM ( minutes_in_status ) as mem_total
            FROM v4_4_statistics_vms_resources_usage_samples
                INNER JOIN v4_4_configuration_history_vms
                    ON (
                        v4_4_configuration_history_vms.history_id = v4_4_statistics_vms_resources_usage_samples.vm_configuration_version
                    )
                    INNER JOIN v4_4_configuration_history_vms latest
                        ON (
                            latest.vm_id = v4_4_statistics_vms_resources_usage_samples.vm_id
                        )
            WHERE v4_4_statistics_vms_resources_usage_samples.vm_status = 1
                AND v4_4_configuration_history_vms.cluster_id in (
                    SELECT v4_4_configuration_history_clusters.cluster_id
                    FROM v4_4_configuration_history_clusters
                    WHERE v4_4_configuration_history_clusters.datacenter_id = '$datacenter_id'
                )
                AND v4_4_configuration_history_vms.cluster_id IN ( '$cluster_id' )
                AND history_datetime >= $__timeFrom()
                AND history_datetime < $__timeTo()
                AND latest.history_id in (
                    SELECT max ( b.history_id )
                    FROM v4_4_configuration_history_vms b
                    GROUP BY b.vm_id
                )
                AND
                    CASE
                        WHEN '$show_deleted' = 'No'
                            THEN latest.delete_date IS NULL
                        ELSE
                            latest.delete_date IS NULL
                            OR
                            latest.delete_date IS NOT NULL
                    END
            GROUP BY history_datetime, v4_4_configuration_history_vms.vm_id
        )
        as nested_query
        GROUP BY nested_query.history_datetime
    ) as h
        ON (
            h.history_datetime =
            t.history_datetime
        )
)
as query_run
 

