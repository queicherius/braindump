# Free backup service for ESXi

This is a guide for setting up a free backup service for ESXi, using GhettoVBC and a NFS server as the backup storage.

### Connecting the NFS server

> If you are on OVH or SYS, find out the data to connect to your NFS storage server [here](http://docs.ovh.ca/en/services-backup-storage.html#nfs).

As the first step, mount the NFS server via the vSphere Client:

1. Configuration
2. Storage
3. Add Storage
4. Network File System
5. Server: Your IP Address
6. Folder: The path on your NFS server (on OVH something like `/export/ftpbackup/...`)
7. Datastore Name: `Ghetto`

---

### Install ghettoVCB

Since you will need SSH access to the ESXi host server, you have to enable that via the following steps:

1. Configuration
2. Security Profile
3. Properties (Top Right)
4. SSH -> Options -> Start

Now you will be able to SSH into your host using your IP, username and password. When you are there, create a directory for ghettoVCB to live in:

```bash
mkdir ghettoVCB
```

Now on your personal machine clone ghettoVCB from Github and then copy it over via SCP.

```bash
git clone https://github.com/lamw/ghettoVCB
cd ghettoVCB
scp * root@youresxiip:/ghettoVCB
```

---

### Configure ghettoVCB

SSH back into your ESXi server and execute the following commands to finish the configuration:

```bash
cd /ghettoVCB

# Fix the file permissions
chmod +x ghettoVCB.sh
chmod +x ghettoVCB-restore.sh

# Add the names of vms to backup, one per line
# They have to match what you see in the vSphere Client
vi vms_to_backup

# Make sure the folder the backups get saved in exists
mkdir -p /vmfs/volumes/Ghetto/ghettovms
```

**`ghettoVCB.conf`**

```
VM_BACKUP_VOLUME=/vmfs/volumes/Ghetto/ghettovms
DISK_BACKUP_FORMAT=thin
VM_BACKUP_ROTATION_COUNT=3
POWER_VM_DOWN_BEFORE_BACKUP=0
ENABLE_HARD_POWER_OFF=0
ITER_TO_WAIT_SHUTDOWN=3
POWER_DOWN_TIMEOUT=5
ENABLE_COMPRESSION=0
VM_SNAPSHOT_MEMORY=0
VM_SNAPSHOT_QUIESCE=0
ALLOW_VMS_WITH_SNAPSHOTS_TO_BE_BACKEDUP=0
ENABLE_NON_PERSISTENT_NFS=0
UNMOUNT_NFS=0
NFS_SERVER=<your nfs server>
NFS_VERSION=nfs
NFS_MOUNT=<your nfs folder>
NFS_LOCAL_NAME=<your nfs name, in this example "Ghetto">
NFS_VM_BACKUP_DIR=ghettovms
SNAPSHOT_TIMEOUT=15
EMAIL_ALERT=0
EMAIL_LOG=0
EMAIL_SERVER=auroa.primp-industries.com
EMAIL_SERVER_PORT=25
EMAIL_DELAY_INTERVAL=1
EMAIL_TO=auroa@primp-industries.com
EMAIL_ERRORS_TO=
EMAIL_FROM=root@ghettoVCB
WORKDIR_DEBUG=0
VM_SHUTDOWN_ORDER=
VM_STARTUP_ORDER=
```

After the configuration is done, we can check if the backup is theoretically working using a "dryrun" command, which should exit with some sort of success message.

```bash
/ghettoVCB/ghettoVCB.sh -f /ghettoVCB/vms_to_backup -g /ghettoVCB/ghettoVCB.conf -l /ghettoVCB/ghettoVCB.log -d dryrun
```

---

### Setup the backup interval

We'll add the command to backup in the crontab equivalent of ESXi, so it runs automatically every sunday.

**`/var/spool/cron/crontabs/root`**

```crontab
27   1    *   *   0   /ghettoVCB/ghettoVCB.sh -f /ghettoVCB/vms_to_backup -g /ghettoVCB/ghettoVCB.conf -l /ghettoVCB/ghettoVCB.log
```

Next, we have to restart the currently running cron service to add our newly added crontab:

```bash
# Get the process id
cat /var/run/crond.pid

# Kill the process by process id
kill whateverpid

# Start the process again
/bin/crond
```

Since we want to keep the crontab when restarting the host server, we'll have to add the commands to a shell file which will get executed on restart:

**`/etc/rc.local.d/local.sh`**

```
/bin/kill $(cat /var/run/crond.pid)
/bin/echo '27   1    *   *   0   /ghettoVCB/ghettoVCB.sh -f /ghettoVCB/vms_to_backup -g /ghettoVCB/ghettoVCB.conf -l /ghettoVCB/ghettoVCB.log' >> /var/spool/cron/crontabs/root
/bin/crond
```