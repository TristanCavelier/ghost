# ghost

Bash ghost utilities using tar, partclone or dd.

## Usage

Usage:

    $0 ACTION NAME DEV             # backup/restore partition table and amorce
    $0 ACTION NAME PART            # backup/restore partition (partclone + gz)

    $0 ACTION-table NAME DEV       # backup/restore partition table
    $0 ACTION-amorce NAME DEV      # backup/restore amorce

    $0 ACTION-partclone NAME PART  # backup/restore partition (partclone + gz)
    $0 ACTION-tar NAME PART        # backup/restore partition content (tar + gz)

    $0 ACTION-ddgz NAME {DEV|PART} # backup/restore device (dd + gz)


ACTION is 'backup' or 'restore'.

NAME is the name of the backup. Ex: "Saved_Data"

DEV is the device which contains partition table. Ex: sdb

PART is the partition device. Ex: sdb1

It uses the files which name is in format:

    ./NAME.DEV.mbr.bin
    ./NAME.DEV.amorce.bin
    ./NAME.DEV.sfdisk

for the partition table and amorce backup/restore. And:

    ./NAME.PART.ntfs.UUID.partclone.gz

for the partition backup/restore.

Commands used to backup partition table and amorce:

    dd if=/dev/DEV of=NAME.DEV.mbr.bin bs=512 count=1
    dd if=/dev/DEV of=NAME.DEV.amorce.bin bs=446 count=1
    sfdisk -d /dev/DEV > NAME.DEV.sfdisk

Command used to backup partition:

    partclone.TYPE -c -s /dev/PART | gzip -c > NAME.PART.TYPE.UUID.partclone.gz
    or
    tar czf NAME.PART.TYPE.UUID.tar.gz -X MOUNTDIRS -C MOUNTPOINT FILES
    or
    dd if=/dev/DEV | gzip -c > NAME.DEV.dd.gz

Commands used to restore partition table and amorce:

    dd if=NAME.DEV.mbr.bin of=/dev/DEV
    dd if=NAME.DEV.amorce.bin of=/dev/DEV
    sfdisk --force /dev/DEV < NAME.DEV.sfdisk

Command used to restore partition:

    zcat NAME.PART.TYPE.UUID.partclone.gz | partclone.restore -o /dev/PART
    or
    tar xf NAME.PART.TYPE.UUID.tar.gz -C MOUNTPOINT
    or
    zcat NAME.DEV.dd.gz | dd of=/dev/DEV
