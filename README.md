ghost
=====

Bash ghost utility using partclone or dd. (version 2.1.0)

- [What can be saved?](#what-can-be-saved)
- [Examples](#examples)
- [Usage](#usage)
- [Detailed Informations](#detailed-informations)
- [License](#license)


What can be saved?
------------------

Master Boot Record (512 first bytes), Amorce (446 first bytes), Partition table with `sfdisk`
the first sectors until first partitions (with a maximum of 2048 blocks),
and all the partition content with `partclone` or `dd`.


Examples
--------

- Ghost entire disk using `dd` and `partclone`:

        # ghost backup /dev/sda
        dd if='/dev/sda' of='./%2Fdev%2Fsda.start.bin' count=2048
        dd if='/dev/sda' of='./%2Fdev%2Fsda.mbr.bin' count=1 bs=512
        dd if='/dev/sda' of='./%2Fdev%2Fsda.amorce.bin' count=1 bs=446
        sfdisk -d '/dev/sda' | gzip -c > './%2Fdev%2Fsda.sfdisk.gz'
        partclone.vfat -c -s '/dev/sda1' | gzip -c > %2Fdev%2Fsda1.partclone.bin.gz
        partclone.ntfs -c -s '/dev/sda2' | gzip -c > %2Fdev%2Fsda2.partclone.bin.gz
        dd if='/dev/sda3' | gzip -c > %2Fdev%2Fsda3.dd.bin.gz
        echo -n > '%2Fdev%2Fsda5.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.mkswap'
        partclone.ext4 -c -s '/dev/sda6' | gzip -c > %2Fdev%2Fsda6.partclone.bin.gz
        partclone.ext2 -c -s '/dev/sda7' | gzip -c > %2Fdev%2Fsda7.partclone.bin.gz

        ...

- Ghost entire disk using `dd`:

        # ghost backup --dd /dev/sda
        dd if='/dev/sda' | gzip -c > './%2Fdev%2Fsda.dd.bin.gz'

        ...

- Ghost entire disk using `dd` for partitions only:

        # ghost backup --headers /dev/sda
        dd if='/dev/sda' of='./%2Fdev%2Fsda.start.bin' count=2048
        dd if='/dev/sda' of='./%2Fdev%2Fsda.mbr.bin' count=1 bs=512
        dd if='/dev/sda' of='./%2Fdev%2Fsda.amorce.bin' count=1 bs=446
        sfdisk -d '/dev/sda' | gzip -c > './%2Fdev%2Fsda.sfdisk.gz'

        ...
        # ghost backup --parts --dd /dev/sda
        dd if='/dev/sda1' | gzip -c > %2Fdev%2Fsda1.dd.bin.gz
        dd if='/dev/sda2' | gzip -c > %2Fdev%2Fsda2.dd.bin.gz
        dd if='/dev/sda3' | gzip -c > %2Fdev%2Fsda3.dd.bin.gz
        echo -n > '%2Fdev%2Fsda5.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.mkswap'
        dd if='/dev/sda6' | gzip -c > %2Fdev%2Fsda6.dd.bin.gz
        dd if='/dev/sda7' | gzip -c > %2Fdev%2Fsda7.dd.bin.gz

        ...

- Ghost entire partition using `partclone` (if available) or `dd`:

        # ghost backup /dev/sda1
        partclone.vfat -c -s '/dev/sda1' | gzip -c > %2Fdev%2Fsda1.partclone.bin.gz

        ...

- For disk and partitions restoration, see [the usage](#usage) and
  [the following detailled informations](#detailed-informations) just below.


Usage
-----

    ghost list                                # show a list of connected block devices
    ghost restore [OPTION] [FOLDER]           # restore disks/partitions from files in FOLDER
    ghost backup [OPTION] DEV [FOLDER]        # backup entire disk/part in FOLDER

    ghost restore [OPTION] [FILES_OR_FOLDERS] # restore headers/partitions from FILE
    ghost [OPTION]

- `DEV` can be either `DISK` or `PART`
- `DISK` is disk path (ex: `/dev/sda`)
- `PART` is partition path (ex: `/dev/sda1`)
- `FOLDER` is folder path (default is current working directory)
- `FILE` is file path

**OPTION:**

    -v, --version                  # print the program version and exit
    -h, --help                     # print this help and exit
        --info                     # print details about what this program does

**backup OPTION:**

    -n, --dry-run                  # just output backup commands
    -Z, --no-gzip                  # tells to not use gzip
        --headers                  # backup disk headers only
        --parts                    # backup disk partitions only
        --dd                       # use `dd` only
        --partclone                # use `partclone` only

**restore OPTION:**

    -n, --dry-run                  # just output restoration commands


Detailed informations
---------------------

For `ghost restore`, it will look for this kind of files:

    ./DEV.dd.bin[.gz]
    ./DISK.sfdisk[.gz]
    ./DISK.start.bin
    ./DISK.mbr.bin
    ./DISK.amorce.bin
    ./PART.partclone.bin[.gz]
    ./PART.[LABEL.]UUID.mkswap (which is empty)

For `ghost backup -- DISK`, it will create:

    ./DISK.start.bin
    ./DISK.mbr.bin
    ./DISK.amorce.bin
    ./DISK.sfdisk.gz
    ./PART.partclone.bin.gz (or PART.dd.bin.gz if FSTYPE is not supported)
    ./PART.[LABEL.]UUID.mkswap

For `ghost backup --headers -- DISK`, it will create:

    ./DISK.start.bin
    ./DISK.mbr.bin
    ./DISK.amorce.bin
    ./DISK.sfdisk.gz

For `ghost backup -- PART`, it will create:

    ./PART.partclone.bin.gz (or PART.dd.bin.gz if FSTYPE is not supported)
    or ./PART.[LABEL.]UUID.mkswap (for swap)

(`./` can also be `FOLDER/`.)


License
-------

> Copyright (c) 2014 Tristan Cavelier <t.cavelier@free.fr>

> This program is free software. It comes without any warranty, to
> the extent permitted by applicable law. You can redistribute it
> and/or modify it under the terms of the Do What The Fuck You Want
> To Public License, Version 2, as published by Sam Hocevar. See
> the COPYING file for more details.

