# Linux Backup

A set of helper scripts to automatically backup your file system. It uses Duplicity ([http://duplicity.nongnu.org](http://duplicity.nongnu.org)) to create full and incremental backups.


## Prerequisites

- An existing mount point (external harddrive, network storage, etc.)
- Duplicity (can be installed with `apt-get install duplicity`)
- Appropriate privileges to read all files to be backed up
- Optional: A working mail system to send status emails after a backup


## Install & Configure

Clone the repository to a location of your choice (in the following, we will assume it is /opt/backup).

Copy the default config file (/opt/backup/cfg/default) to a location of your choice (e.g. /opt/backup/cfg/myconfig) and configure the following settings:

- `logfile`: The filename of the logfile. It should contain the date and time of each backup so you can verify if the backup was successful.
- `backupdir`: The location where the backup should be stored (e.g. /mnt/backup). Make sure the user executing the backup has write permission on this directory.
- `exclude`: A list of directories that should not be backed up. By default, directories such as /dev, /proc, etc. are excluded. Extend this list by file paths you want to exclude from the backup.
- `keep_backups`: The number of full backups that should be kept. Comment or remove this line if you want to keep all backups.
- `email`: An email address to which the status report after each backup is sent. It is disabled by default. Enabling the status email requires a working mail service on your system.

Since the goal of this backup script is to backup most of the file system, it makes sense to run it as a privileged user (i.e. root). In any case, make sure the config file is executable for the user running the backup.


## Manual backups

    /opt/backup/bin/backup config_file [ -f ]

The `-f` option forces the creation of a full backup.

Note: If another backup process is still running, no new backup can be created and `/opt/backup/bin/backup` will abort.

**Example:** Create a full backup with our configuration file:

    /opt/backup/bin/backup /opt/backup/cfg/myconfig -f

**Example:** Create an incremental backup with our configuration file:

    /opt/backup/bin/backup /opt/backup/cfg/myconfig

Note: If this is the first backup, a full backup will be created even without the `-f` option.

## Automatic backups

Use cron jobs to periodically create backups.

**Example:** Create a full backup on the first day of each month at 02:30 and an incremental backup every day at 03:00:

Add the following two lines to your crontab (`crontab -e`):

    30 2 1 * * /opt/backup/bin/backup /opt/backup/cfg/myconfig -f
    0 3 * * * /opt/backup/bin/backup /opt/backup/cfg/myconfig

## Verify backups

The logfiles created with each backup, as well as the status email (if enabled), tell you if everything went smoothly or if there was a problem. A list of all files in the backup can be shown as follows:

    /opt/backup/bin/list config_file time

`time` can be specified in the following formats: 

- Current time: `now`
- Relative time in seconds (`s`), minutes (`m`), hours (`h`), days (`D`), weeks (`W`), months (`M`), years (`Y`) (e.g. `5s`, `12m`, `5h`, `2D`, `3W`, `1M`, `1Y`)
- Midnight (according to local time zone) at a given date: `YYYY-MM-DD` (e.g. `2018-02-21`)
- W3 time format: `YYYY-MM-DDThh:mm:ssTZD` (e.g. `2018-02-21T14:30:00+01:00`)

**Example:** Show all files in the most current backup:

    /opt/backup/bin/list /opt/backup/cfg/myconfig now

**Example:** Show all files in the newest backup before 1 December 2017, 06:45 (UTC+1):

    /opt/backup/bin/list /opt/backup/cfg/myconfig 2017-12-01T06:45:00+01:00


## Restore single files

Files can be restored from any existing backup.

    /opt/backup/bin/restore-single config_file time file target [ -f ]

Specify the path to the file or folder you want to restore relative to the root directory, without `/` (e.g. `home/foo/importantdocument.txt` instead of `/home/foo/importantdocument.txt`). The specified file or folder is retrieved from the newest backup before `time` and copied to the location specified in `target`. It is advisible to restore files into a new directory (e.g. /tmp/restore) and then move them to the correct location. If the target file already exists, duplicity will not override it. Add the `-f` parameter to force files to be overridden by the restored file from the backup.

**Example:** Restore the file /home/foo/importantdocument.txt from two days ago into /tmp/restore/importantdocument.txt:

    /opt/backup/bin/restore-single /opt/backup/cfg/myconfig 2D home/foo/importantdocument.txt /tmp/restore/importantdocument.txt

**Example:** Restore the directory /home/foo from two days ago into /tmp/restore/foo:

    /opt/backup/bin/restore-single /opt/backup/cfg/myconfig 2D home/foo /tmp/restore/foo

**Example:** Restore the directory /home/foo from two days ago directly into /home/foo:

    /opt/backup/bin/restore-single /opt/backup/cfg/myconfig 2D home/foo /home/foo -f

Note: This will override any existing files inside /home/foo.


## Restore entire file system

Similar to the recovery of single files, you can also choose to restore all files from an existing backup. 

    /opt/backup/bin/restore-all config_file time target [ -f ]

**Example:** Restore all files from two days ago into /tmp/restore:

    /opt/backup/bin/restore-all /opt/backup/cfg/myconfig 2D /tmp/restore

**Example:** Restore all files from two days ago directly to their respective location:

*Attention: Directly replacing certain files from the backup may lead to errors and mess up your system. Use this option only if you know exactly what you are doing.*

    /opt/backup/bin/restore-all /opt/backup/cfg/myconfig 2D / -f




