# zfs-zabbix-userparams: Basic monitoring of ZFS storage pools

## Usage

1. Copy the userparams file (`zfs-userparams.conf`) to a directory included in your Zabbix Agent configuration, such as `/etc/zabbix/zabbix_agent2.d/`.

2. Set the owner and group of the userparams file:

   ```shell
   sudo chown root:zabbix zfs-userparams.conf
   ```

3. Set the userparams file (`zfs-userparams.conf`) permissions:

   ```shell
   sudo chmod 0440 zfs-userparams.conf
   ```

4. Restart the Zabbix Agent:

   ```shell
   sudo systemctl restart zabbix-agent2.service
   ```

5. Navigate to `Configuration → Templates → Import`, and upload the corresponding template file (`zfs-userparams-template.yaml`) to the Zabbix Server.

6. Link the `Basic ZFS by Zabbix agent active` template to the desired hosts.

7. If necessary, increase the Timeout value in the Zabbix Agent configuration file (e.g., `Timeout=30`).

## System requirements

- The `zfsutils-linux` package must be installed.

## Tested on

- Active Zabbix Agent 2 on Ubuntu LTS
- Zabbix Server 7.0 LTS

## ARC-aware memory monitoring

The ZFS ARC is allocated outside the Linux page cache, so the kernel reports it as `used` memory rather than `buff/cache`. As a result, `MemAvailable` — and therefore the stock `vm.memory.size[available]` / `vm.memory.size[pused]` items — under-report reclaimable memory on ZFS hosts, producing false "high memory" alerts.

The `zfs-userparams.arc.*` items expose the ARC size and an ARC-aware memory calculation that credits the reclaimable portion of the ARC (`size - c_min`) back as available. On ZFS hosts, alert on `zfs-userparams.arc.mem-pused` instead of `vm.memory.size[pused]`, and consider disabling or overriding the OS-template memory triggers to avoid duplicate alerting.

## Template items

- `zfs-userparams.arc.size`

   Current size of the ZFS ARC in bytes. The ARC is allocated outside the Linux page cache, so the kernel reports it as used memory rather than buff/cache.

- `zfs-userparams.arc.cmin`

   Minimum size the ARC can be reclaimed down to under memory pressure (c_min), in bytes.

- `zfs-userparams.arc.cmax`

   Maximum size the ARC is allowed to grow to (c_max), in bytes.

- `zfs-userparams.arc.reclaimable`

   Reclaimable portion of the ARC (`size - c_min`, floored at 0), in bytes. This is the memory the kernel can hand back to applications on demand.

- `zfs-userparams.arc.hit-ratio`

   ARC hit ratio as a percentage since boot. A low hit ratio may indicate the ARC is too small for the working set.

- `zfs-userparams.arc.mem-available`

   Available memory in bytes accounting for reclaimable ARC: `MemAvailable + (ARC size - c_min)`. Use this instead of `vm.memory.size[available]` on ZFS hosts.

- `zfs-userparams.arc.mem-pused`

   Used memory as a percentage, crediting reclaimable ARC as available: `100 * (1 - (MemAvailable + ARC size - c_min) / MemTotal)`. Use this instead of `vm.memory.size[pused]` for memory alerting on ZFS hosts.

- `zfs-userparams.dataset.nokey`

   Count of encrypted datasets with unavailable encryption keys. Datasets remain accessible for most operations but require key availability for full functionality.

- `zfs-userparams.pool.capacity-80`

   Count of storage pools exceeding 80% capacity utilization. Pools approaching this threshold should be monitored for capacity planning.

- `zfs-userparams.pool.capacity-90`

   Count of storage pools exceeding 90% capacity utilization. Immediate capacity action may be required.

- `zfs-userparams.pool.degraded`

   Count of storage pools in DEGRADED state. Indicates loss of redundancy that requires attention.

- `zfs-userparams.pool.errors`

   Count of storage pools reporting read, write, or checksum errors. Indicates potential hardware or data integrity issues.

- `zfs-userparams.pool.faulted`

   Count of storage pools in FAULTED state. Indicates pool is inaccessible and requires immediate intervention.

- `zfs-userparams.pool.fragmentation`

   Count of storage pools with free space fragmentation exceeding 70%. High fragmentation reduces performance.

- `zfs-userparams.pool.number`

   Total count of available ZFS storage pools.

- `zfs-userparams.pool.offline`

   Count of storage pools in OFFLINE state. Indicates administrator-initiated pool suspension.

- `zfs-userparams.pool.removed`

   Count of storage pools in REMOVED state. Indicates physical device removal while system was running.

- `zfs-userparams.pool.scrubbing`

   Count of storage pools actively undergoing scrub operations. Normal maintenance activity.

- `zfs-userparams.pool.unavail`

   Count of storage pools in UNAVAIL state. Indicates pool devices cannot be accessed.

- `zfs-userparams.userparams-version`

   Version number of this template file.

## Template triggers

- **ZFS: High memory utilization (ARC-aware) - 90%**

   ARC-aware memory utilization has exceeded 90% for the last 5 minutes.

   Unlike the stock memory items, this metric already discounts reclaimable ARC, so a sustained high value reflects genuine memory pressure rather than ZFS caching. Investigate application memory usage and consider capping `zfs_arc_max` or adding memory.

- **ZFS: High memory utilization (ARC-aware) - 80%**

   ARC-aware memory utilization has exceeded 80% for the last 5 minutes. Depends on the 90% trigger. This metric already discounts reclaimable ARC, so it reflects real memory pressure.

- **ZFS: Low ARC hit ratio**

   The ARC hit ratio has stayed below 50% for the last 30 minutes. Suggests the ARC is undersized for the current working set, increasing read latency and back-end disk I/O.

- **ZFS: User Parameters version mismatch**

   The deployed `zfs-userparams.conf` file is not at the expected version (4). Update the User Parameters file on the host and/or re-import the template so both are in sync.

- **ZFS: Data errors**

   One or more pools are reporting read, write, or checksum errors.

   These errors indicate potential data integrity issues or hardware problems. Review system logs, verify hardware health, and consider replacing suspect devices. Prompt investigation is essential to prevent data loss.

- **ZFS: Degraded pool**

   One or more virtual devices have failed but the pool remains operational.

   This typically occurs when a mirror or RAID-Z device loses one or more constituent devices. Fault tolerance is compromised — a single additional device failure could cause irreparable data loss. Replace failed devices immediately.

- **ZFS: Encryption key unavailable**

   One or more encrypted datasets cannot access their encryption keys.

   Affected datasets can still perform scrubbing, listing, snapshots, and replication, but require the key for read/write operations.

- **ZFS: Faulted pool**

   One or more pools are in FAULTED state and completely inaccessible.

   This indicates complete device failure preventing data I/O. If a top-level virtual device is faulted, the entire pool is inaccessible. Diagnose hardware failures immediately and restore device functionality to bring the pool back online.

- **ZFS: More than 70% free disk space fragmentation**

   One or more pools have free space fragmentation exceeding 70%.

   High fragmentation degrades performance and reduces the ability to allocate large contiguous blocks. Consider pool defragmentation through data reorganization or pool recreation to restore performance.

- **ZFS: More than 80% disk space used**

   One or more ZFS pools have exceeded 80% disk space capacity.

   Performance degradation typically occurs at this threshold. Review pool usage, consider data archival, or expand storage capacity to maintain optimal performance.

- **ZFS: More than 90% disk space used**

   One or more ZFS pools have exceeded 90% disk space capacity.

   Significant performance degradation is likely. Take immediate action to free space or expand storage to prevent potential write failures and system instability.

- **ZFS: No data pools available**

- **ZFS: Offline pool**

   One or more pools have been explicitly taken offline by an administrator.

   Offline pools are intentionally suspended and require manual intervention to return to service. Verify this state is expected and planned.

- **ZFS: Pool scrubbing**

   One or more pools are currently undergoing a scrub operation.

   Scrubbing is a normal maintenance activity that verifies data integrity and repairs correctable errors. This is expected behavior and requires no action.

- **ZFS: Removed pool**

   One or more devices were physically removed while the system was running.

   Device removal detection depends on hardware support and may not be available on all platforms. Reinstall devices or replace them to restore pool health.

- **ZFS: Unavailable pool**

   One or more pools have devices that cannot be opened or are inaccessible.

   Pools may appear in DEGRADED mode while attempting recovery. If a top-level virtual device is UNAVAIL, all data in the pool becomes inaccessible. Verify device connectivity and repair or replace failed hardware immediately.
