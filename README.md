# zfs-zabbix-userparams: Basic monitoring of ZFS data pools

## Usage

- Copy the userparams file (`zfs-userparams.conf`) to a directory included in your Zabbix Agent configuration file, such as `/etc/zabbix/zabbix_agentd.d/` or `/etc/zabbix/zabbix_agent2.d/`.
- Set the owner and group of the userparams file (`chown root:zabbix zfs-userparams.conf`).
- Set the mode of the userparams file (`chmod 0440 zfs-userparams.conf`).
- Restart the Zabbix Agent (`systemctl restart zabbix-agent.service or systemctl restart zabbix-agent2.service`).
- Import the corresponding template file (`zfs-userparams-template.yaml`) on the Zabbix Server.
- Link the 'Basic ZFS by Zabbix agent active' template to the selected hosts.
- You may need to increase the Timeout value in your Zabbix Agent configuration file, for example, `Timeout=30`.

## System requirements

- The `zfsutils-linux` package needs to be installed.

## Tested on

- Zabbix Agent 2 on Ubuntu 20.04 and Ubuntu 22.04
- Zabbix Server 6.0 LTS

## Template items

- `zfs-userparams.userparams-version`

  Return the version number of this file. This field is reserved for future use and intended for versioning within the template.

- `zfs-userparams.pool.degraded`

  Check for degraded pools.

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.faulted`

  Check for faulted pools.

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.offline`

  Check for offline pools.

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.unavail`

  Check for unavailable pools.

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.removed`

  Check for removed pools.

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.capacity-80`

  Check for pools with more than 80% disk space used (capacity).

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.capacity-90`

  Check for pools with more than 80% disk space used (capacity).

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.fragmentation`

  Check for pools with more than 70% free disk space fragmentation.

  Expected return value: an integer indicating the number of matching pools.

- `zfs-userparams.pool.number`

  Check the number of available ZFS storage pools.

  Expected return value: an integer indicating the number of available pools.

## Template triggers

- ZFS: No data pools available

- ZFS: Degraded pool

  The virtual device has encountered a failure but is still operational.

  This situation is most frequently observed when a mirror or RAID-Z device has lost one or more constituent devices. The fault tolerance of the pool may be compromised, as a subsequent fault in another device could be irreparable.

- ZFS: Faulted pool

  The device or virtual device is completely inaccessible.

  This status typically signifies a complete failure of the device, rendering it incapable of sending or receiving data to/from ZFS. If a top-level virtual device is in this state, the entire pool becomes completely inaccessible.

- ZFS: More than 70% free disk space fragmentation

  High fragmentation of free space can lead to significant performance degradation.

- ZFS: More than 80% disk space used

  Some ZFS pools are reporting more than 80% disk space used.

  Pool performance tends to degrade significantly beyond this threshold.

- ZFS: More than 90% disk space used

  Some ZFS pools are reporting more than 90% disk space used.

  Pool performance will degrade significantly beyond this threshold.

- ZFS: Offline pool

  The device has been explicitly taken offline by the administrator.

- ZFS: Removed pool

  The device was physically removed while the system was running.

  Device removal detection is dependent on the hardware and may not be supported on all platforms.

- ZFS: Unavailable pool

  The device or virtual device cannot be opened.

  In certain cases, pools with UNAVAIL devices may appear in DEGRADED mode. If a top-level virtual device is UNAVAIL, then all data in the pool becomes inaccessible.
