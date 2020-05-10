#Executive reports: Active Virtual Machines by OS over time

The Active Virtual Machines by OS report shows a summary of the number of active virtual machines in a given time period, broken down by operating system. 

## Grafana Query

SELECT DISTINCT
    $__time(time),
    COALESCE (
        os_type,
        'RHEL'
    )
    AS os_type,
    vm_count
FROM (
    SELECT DISTINCT
        CASE
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) = 'OTHER OS'
                THEN 'Other OS'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%WINDOWS%'
                THEN 'Windows'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'RHEL%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'RED HAT%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'OTHER L%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'LINUX'
                THEN 'Other Linux'
            WHEN UPPER (COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%UBUNTU%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%SUSE%'
                THEN 'Other Linux'
            ELSE 'Other OS'
        END
        AS os_type,
        history_datetime AS time,
        COUNT ( DISTINCT v4_4_configuration_history_vms.vm_id ) AS vm_count
    FROM v4_4_statistics_vms_resources_usage_samples
        INNER JOIN v4_4_configuration_history_vms
            ON (
                v4_4_configuration_history_vms.history_id =
                v4_4_statistics_vms_resources_usage_samples.vm_configuration_version
            )
        INNER JOIN enum_translator enum_os_type
            ON (
                enum_os_type.enum_key =
                v4_4_configuration_history_vms.operating_system
                AND enum_os_type.enum_type = 'OS_TYPE'
                AND language_code = 'en_US'
            )
        INNER JOIN v4_4_configuration_history_vms latest_config
            ON (
                latest_config.vm_id =
                v4_4_statistics_vms_resources_usage_samples.vm_id
            )
    WHERE
        -- Here we filter only the vms that are in active status
        v4_4_statistics_vms_resources_usage_samples.vm_status = 1
        -- Filter vms list according to the datacenter that was chosen by the user
        AND v4_4_configuration_history_vms.cluster_id IN
            (
                SELECT cluster_id
                FROM v4_4_configuration_history_clusters
                WHERE datacenter_id = '$datacenter_id'
            )
        -- Filter vms list according to the cluster that was chosen by the user
        AND v4_4_configuration_history_vms.cluster_id IN ( '$cluster_id' )
        -- Here we get the vm latest configuration
        AND latest_config.history_id IN
            (
                SELECT MAX ( b.history_id )
                FROM v4_4_configuration_history_vms b
                GROUP BY b.vm_id
            )
       AND history_datetime >= $__timeFrom()
       AND history_datetime < $__timeTo()
       AND
            -- This will determine where deleted entities will be included in the report,
            -- according to the user selection for "is_deleted" parameter
            CASE
                WHEN '$show_deleted' = 'No'
                    THEN latest_config.delete_date IS NULL
                ELSE
                    latest_config.delete_date IS NULL
                    OR
                    latest_config.delete_date IS NOT NULL
            END
    GROUP BY
        CASE
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) = 'OTHER OS'
                THEN 'Other OS'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%WINDOWS%'
                THEN 'Windows'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'RHEL%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'RED HAT%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'OTHER L%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'LINUX'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%UBUNTU%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%SUSE%'
                THEN 'Other Linux'
            ELSE 'Other OS'
        END,
        History_datetime

UNION ALL

    SELECT DISTINCT
        CASE
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) = 'OTHER OS'
                THEN 'Other OS'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%WINDOWS%'
                THEN 'Windows'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'RHEL%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'RED HAT%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'OTHER L%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'LINUX'
                THEN 'Other Linux'
            WHEN UPPER (COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%UBUNTU%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%SUSE%'
                THEN 'Other Linux'
            ELSE 'Other OS'
        END
        AS os_type,
        history_datetime AS time,
        COUNT ( DISTINCT v4_4_configuration_history_vms.vm_id ) AS vm_count
    FROM v4_4_statistics_vms_resources_usage_hourly
        INNER JOIN v4_4_configuration_history_vms
            ON (
                v4_4_configuration_history_vms.history_id =
                v4_4_statistics_vms_resources_usage_hourly.vm_configuration_version
            )
        INNER JOIN enum_translator enum_os_type
            ON (
                enum_os_type.enum_key =
                v4_4_configuration_history_vms.operating_system
                AND enum_os_type.enum_type = 'OS_TYPE'
                AND language_code = 'en_US'
            )
        INNER JOIN v4_4_configuration_history_vms latest_config
            ON (
                latest_config.vm_id =
                v4_4_statistics_vms_resources_usage_hourly.vm_id
            )
    WHERE
        -- Here we filter only the vms that are in active status
        v4_4_statistics_vms_resources_usage_hourly.vm_status = 1
        -- Filter vms list according to the datacenter that was chosen by the user
        AND v4_4_configuration_history_vms.cluster_id IN
            (
                SELECT cluster_id
                FROM v4_4_configuration_history_clusters
                WHERE datacenter_id = '$datacenter_id'
            )
        -- Filter vms list according to the cluster that was chosen by the user
        AND v4_4_configuration_history_vms.cluster_id IN ( '$cluster_id' )
        -- Here we get the vm latest configuration
        AND latest_config.history_id IN
            (
                SELECT MAX ( b.history_id )
                FROM v4_4_configuration_history_vms b
                GROUP BY b.vm_id
            )
       AND history_datetime >= $__timeFrom()
       AND history_datetime < $__timeTo()
       AND
            -- This will determine where deleted entities will be included in the report,
            -- according to the user selection for "is_deleted" parameter
            CASE
                WHEN '$show_deleted' = 'No'
                    THEN latest_config.delete_date IS NULL
                ELSE
                    latest_config.delete_date IS NULL
                    OR
                    latest_config.delete_date IS NOT NULL
            END
    GROUP BY
        CASE
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) = 'OTHER OS'
                THEN 'Other OS'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%WINDOWS%'
                THEN 'Windows'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'RHEL%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'RED HAT%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'OTHER L%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'LINUX'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%UBUNTU%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%SUSE%'
                THEN 'Other Linux'
            ELSE 'Other OS'
        END,
        History_datetime

UNION ALL

    SELECT DISTINCT
        CASE
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) = 'OTHER OS'
                THEN 'Other OS'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%WINDOWS%'
                THEN 'Windows'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'RHEL%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'RED HAT%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'OTHER L%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE 'LINUX'
                THEN 'Other Linux'
            WHEN UPPER (COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%UBUNTU%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE( enum_os_type.value,'Other OS' ) ) LIKE '%SUSE%'
                THEN 'Other Linux'
            ELSE 'Other OS'
        END
        AS os_type,
        history_datetime AS time,
        COUNT ( DISTINCT v4_4_configuration_history_vms.vm_id ) AS vm_count
    FROM v4_4_statistics_vms_resources_usage_daily
        INNER JOIN v4_4_configuration_history_vms
            ON (
                v4_4_configuration_history_vms.history_id =
                v4_4_statistics_vms_resources_usage_daily.vm_configuration_version
            )
        INNER JOIN enum_translator enum_os_type
            ON (
                enum_os_type.enum_key =
                v4_4_configuration_history_vms.operating_system
                AND enum_os_type.enum_type = 'OS_TYPE'
                AND language_code = 'en_US'
            )
        INNER JOIN v4_4_configuration_history_vms latest_config
            ON (
                latest_config.vm_id =
                v4_4_statistics_vms_resources_usage_daily.vm_id
            )
    WHERE
        -- Here we filter only the vms that are in active status
        v4_4_statistics_vms_resources_usage_daily.vm_status = 1
        -- Filter vms list according to the datacenter that was chosen by the user
        AND v4_4_configuration_history_vms.cluster_id IN
            (
                SELECT cluster_id
                FROM v4_4_configuration_history_clusters
                WHERE datacenter_id = '$datacenter_id'
            )
        -- Filter vms list according to the cluster that was chosen by the user
        AND v4_4_configuration_history_vms.cluster_id IN ( '$cluster_id' )
        -- Here we get the vm latest configuration
        AND latest_config.history_id IN
            (
                SELECT MAX ( b.history_id )
                FROM v4_4_configuration_history_vms b
                GROUP BY b.vm_id
            )
       AND history_datetime >= $__timeFrom()
       AND history_datetime < $__timeTo()
       AND
            -- This will determine where deleted entities will be included in the report,
            -- according to the user selection for "is_deleted" parameter
            CASE
                WHEN '$show_deleted' = 'No'
                    THEN latest_config.delete_date IS NULL
                ELSE
                    latest_config.delete_date IS NULL
                    OR
                    latest_config.delete_date IS NOT NULL
            END
    GROUP BY
        CASE
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) = 'OTHER OS'
                THEN 'Other OS'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%WINDOWS%'
                THEN 'Windows'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'RHEL%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'RED HAT%'
                THEN 'RHEL'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'OTHER L%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like 'LINUX'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%UBUNTU%'
                THEN 'Other Linux'
            WHEN UPPER ( COALESCE ( enum_os_type.value,'Other OS' ) ) like '%SUSE%'
                THEN 'Other Linux'
            ELSE 'Other OS'
        END,
        history_datetime
)
AS all_query
ORDER BY os_type
