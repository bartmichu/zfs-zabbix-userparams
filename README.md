# zfs-zabbix-userparams: Basic monitoring of ZFS storage pools

## Usage

1. Copy the userparams file (`zfs-userparams.conf`) to a directory included in your Zabbix Agent configuration, such as `/etc/zabbix/zabbix_agentd.d/` or `/etc/zabbix/zabbix_agent2.d/`.

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

   Or:

   ```shell
   sudo systemctl restart zabbix-agent.service
   ```

5. Navigate to `Configuration → Templates → Import`, and upload the corresponding template file (`zfs-userparams-template.yaml`) to the Zabbix Server.

6. Link the `Basic ZFS by Zabbix agent active` template to the desired hosts.

7. If necessary, increase the Timeout value in the Zabbix Agent configuration file (e.g., `Timeout=30`).

## System requirements

- The `zfsutils-linux` package must be installed.

## Tested on

- Zabbix Agent 2 running on Ubuntu 22.04 and 24.04
- Zabbix Server 6.0 LTS

## Template items

- `zfs-userparams.userparams-version`

   Return the version number of this file. Reserved for future use and versioning within the template.

- `zfs-userparams.pool.degraded`

   Check for degraded pools.

   Expected return value: an integer indicating the number of degraded pools.

- `zfs-userparams.pool.faulted`

   Check for faulted pools.

   Expected return value: an integer indicating the number of faulted pools.

- `zfs-userparams.pool.offline`

   Check for offline pools.

   Expected return value: an integer indicating the number of offline pools.

- `zfs-userparams.pool.unavail`

   Check for unavailable pools.

   Expected return value: an integer indicating the number of unavailable pools.

- `zfs-userparams.pool.removed`

   Check for removed pools.

   Expected return value: an integer indicating the number of removed pools.

- `zfs-userparams.pool.capacity-80`

   Check for pools with more than 80% disk space used.

   Expected return value: an integer indicating the number of pools exceeding 80% usage.

- `zfs-userparams.pool.capacity-90`

   Check for pools with more than 90% disk space used.

   Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.fragmentation`

   Check for pools with more than 70% free disk space fragmentation.

   Expected return value: an integer indicating the number of pools with fragmentation above 70%.

- `zfs-userparams.pool.number`

   Check the total number of available ZFS storage pools.

   Expected return value: an integer indicating the total number of available pools.

## Template triggers

- **ZFS: No data pools available**

- **ZFS: Degraded pool**

  The virtual device has encountered a failure but remains operational.

  This situation most commonly occurs when a mirror or RAID-Z device has lost one or more constituent devices. The fault tolerance of the pool may be compromised, as a subsequent fault in another device could lead to irreparable damage.

- **ZFS: Faulted pool**

  The device or virtual device is completely inaccessible.

  This status typically indicates a complete failure of the device, rendering it incapable of sending or receiving data to/from ZFS. If a top-level virtual device is in this state, the entire pool becomes inaccessible.

- **ZFS: More than 70% free disk space fragmentation**

  High fragmentation of free space can lead to significant performance degradation.

- **ZFS: More than 80% disk space used**

  Some ZFS pools are reporting more than 80% disk space usage.

  Pool performance tends to degrade significantly beyond this threshold.

- **ZFS: More than 90% disk space used**

  Some ZFS pools are reporting more than 90% disk space usage.

  Pool performance will degrade significantly beyond this threshold.

- **ZFS: Offline pool**

  The device has been explicitly taken offline by the administrator.

- **ZFS: Removed pool**

  The device was physically removed while the system was running.

  Device removal detection depends on the hardware and may not be supported on all platforms.

- **ZFS: Unavailable pool**

  The device or virtual device cannot be opened.

  In some cases, pools with UNAVAIL devices may appear in DEGRADED mode. If a top-level virtual device is UNAVAIL, all data in the pool becomes inaccessible.
