# zfs-zabbix-userparams: Basic monitoring of ZFS data pools

## Usage

- Copy userparams file (```zfs-userparams.conf```) to a directory included in your Zabbix Agent configuration file, e.g. ```/etc/zabbix/zabbix_agentd.d/```
- Set userparams file owner and group (```sudo chown root:zabbix zfs-userparams.conf```)
- Set userparams file mode (```sudo chmod 0440 zfs-userparams.conf```)
- Restart Zabbix Agent (```sudo systemctl restart zabbix-agent.service```)
- Import corresponding template file (```zfs-userparams-template.yaml```) on Zabbix Server
- Link ```Template zfs-userparams by Zabbix agent active``` template to selected hosts

## System requirements

- ```zfsutils-linux``` package installed

## Tested on

- Zabbix Agent 5.x on Ubuntu 20.04
- Zabbix Server 5.x

## Items

- zfs-userparams.pool.degraded

  Check for DEGRADED pools.

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.faulted

  Check for FAULTED pools.

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.offline

  Check for OFFLINE pools.

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.unavail

  Check for UNAVAIL pools.

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.removed

  Check for REMOVED pools.

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.capacity-80

  Check for pools with more than 80% disk space used (capacity).

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.capacity-90

  Check for pools with more than 90% disk space used (capacity).

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.fragmentation

  Check for pools with more than 70% free disk space fragmentation.

  Expected return value: a number indicating the number of matching pools.

- zfs-userparams.pool.number

  Check the number of available ZFS storage pools.

  Expected return value: a number indicating the number of available pools.

## Triggers

- ZFS - no data pools available

- ZFS pool - DEGRADED

  The virtual device has experienced a failure but can still function. This state is most common when a mirror or RAID-Z device has lost one or more constituent devices. The fault tolerance of the pool might be compromised, as a subsequent fault in another device might be unrecoverable.

- ZFS pool - FAULTED

  The device or virtual device is completely inaccessible. This status typically indicates total failure of the device, such that ZFS is incapable of sending data to it or receiving data from it. If a top-level virtual device is in this state, then the pool is completely inaccessible.

- ZFS pool - more than 70% free disk space fragmentation

  High free space fragmentation can cause a significant performance degradation.

- ZFS pool - more than 80% disk space used

  Some ZFS pools are reporting more than 80% disk space used. Pool performance tends to degrade significantly above this threshold.

- ZFS pool - more than 90% disk space used

  Some ZFS pools are reporting more than 90% disk space used. Pool performance will degrade significantly above this threshold.

- ZFS pool - OFFLINE

  The device has been explicitly taken offline by the administrator.

- ZFS pool - REMOVED

  The device was physically removed while the system was running. Device removal detection is hardware-dependent and might not be supported on all platforms.

- ZFS pool - UNAVAIL

  The device or virtual device cannot be opened. In some cases, pools with UNAVAIL devices appear in DEGRADED mode. If a top-level virtual device is UNAVAIL, then nothing in the pool can be accessed.
