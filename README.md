SnapshotBackup by Fredrik Welander
--------
http://www.subsite.fi/pages/in-english/subsite.php

This script uses hardlinks to create time freezed incremental backups. It is a reliable solution for automatically 
backing up multiple sources with maximum diskspace and bandwidth efficiency. Run with cron for best result.


**DISCLAIMER:**
This program may not work as espected and it may destroy your data. It may stop working unexpectedly or create useless backups. It may be a security risk.
Read and understand the code, test in a safe environment, check your backups from time to time. USE AT YOUR OWN RISK.

**Syntax:**

    snapshotbackup.bash [--snapshots NUMBER] SOURCE_PATH [SOURCE_PATH ...] DESTINATION_PATH
    
**Installation:** 

    cd /usr/local/sbin
    sudo wget https://raw.github.com/fredrikwelander/snapshotbackup/master/snapshotbackup.bash
    sudo chmod 755 snapshotbackup.bash
    
Optional step, if you want error reporting and don't want to put your email into the script file:
    
    sudo echo "your.email@mailprovider.com" > /etc/scriptmail.txt

**Dependencies (standard shell commands not listed):**
- rsync
- sshfs (if you neeed push backup)
- /usr/bin/mail (if you want error reporting by mail)
- getfacl (if you need to backup permissions separately)

**Notes:**
- Run script as root to preserve file ownership and avoid permission errors, but protect your backup drive.
- Use double quotes around source dirs with spaces. Destination path cannot contain spaces.
- Source paths can be remote (user@client:/dir/dir) or local, but you cannot mix. All sources must be on the same host.
- Destination path must be on a locally mounted device.
- Backup destination must be a Linux type filesystem for the hardlinks to work, forget FAT/NTFS drives.
- SSHFS-mounts can be slow.
- SnapshotBackup has limited error handling. Killing a running script might make a mess.

**Security:**
- Make sure the backups are protected even if your main user account is compromised, restrict sudo access and use unique passwords for sudoers.
- Pull backup is recommended for security, push backup (with the destination sshfs-mounted) will make your backups available *and writable* if someone or some malware gets access to your client host. 
- Consider chmod 700 on the backup destination directory. This will make restoring a bit less smooth, but will keep the backups hidden from anybody without root access on the server.
- You could enable root login on your client sshd and use root@client:/source/path to get all read-only files (like private keys, /etc/shadow and such) but I find this unnecessary and risky, especially if the backup drive is unencrypted.


**Sample pull backup setup:**
- Create a dedicated backup user or set a password for user 'backup' if it already exists. You can use a long randomly generated password and forget it after uploading the key. 
- Make sure the backup user can log in using ssh key authentication (do ssh-keygen, ssh-copy-id backup@client, etc).
- You might need to ssh-keygen for root@server and ssh-copy-id to backup@client as well.
- Set up the backup command to run in cron (as root).

**Examples**

*Make snapshots of three directories keeping the default number of copies (SNAPSHOT_COUNT in conf section):*

    snapshotbackup.bash backup@client:/etc backup@client:/home/user /mnt/backup_drive/mybackup

*Make snapshots of local /var/www keeping 30 copies:*

    snapshotbackup.bash --snapshots 30 /var/www /var/www_backup

*Sample pull backup in /etc/cron.d*

    30 1 * * * root /usr/local/sbin/snapshotbackup.bash --snapshots 14 backup@client:/etc backup@client:/home/user /mnt/backup_drive/client/daily

**Tips** 

- SnapshotBackup has basic diskspace reporting, but you can also use my diskspace script on https://github.com/fredrikwelander/misc-scripts and run it on the backup server to get an email if the server is about to run out of space
- Want to know which files were updated in the latest snapshot? On the server, do:

        cd /path/to/backup/destination
        find snapshot.0/* -newer snapshot.1 -exec ls {} \; 

Thanks to: http://www.mikerubel.org/computers/rsync_snapshots/
