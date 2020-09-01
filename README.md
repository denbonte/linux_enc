# Linux Volumes Encryption

Volumes encryption on Linux using [cryptsetup](https://linux.die.net/man/8/cryptsetup)/LUKS.

Make sure cryptsetup is installed on the system, otherwise run `sudo apt install cryptsetup`.

## Setting-up a New Encrypted Volume
First thing first, check how your OS labelled the volume you want to encrypt by running `sudo fdisk -l`. As the process will clear all the data you have stored on that volume, make sure you backed up everything you need from the drive you want to encrypt (and, of course, that the drive/partition you're trying to encrypt is not the one where the OS is stored).

Let's assume we want to encrypt the volume at `/dev/sdb`. We can check the status of the volume running `sudo fdisk -l /dev/sdb`. The output should look similar to:


```
Disk /dev/sdb: 7,3 TiB, 8001563222016 bytes, 15628053168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

Device     Start         End     Sectors  Size Type
/dev/sdb1   2048 15628052479 15628050432  7,3T Microsoft basic data
```

At this stage, run the command `sudo cryptsetup -y -v luksFormat /dev/sdb` to start the encryption process. The flag `-y` makes sure the user verifies the encryption password before proceding, wjile the `-v` flag makes the process verbose. For other options, run `cryptsetup --help` or visit the cryptsetup man page (either from terminal or [online](https://linux.die.net/man/8/cryptsetup)). The output should look similar to:

```
sudo cryptsetup -y -v luksFormat /dev/sdb

WARNING!
========
This will overwrite data on /dev/sdb irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase for /dev/sdb: ********* 
Verify passphrase: *********
Command successful.
```

Everything needed to encrypt the volume has been generated.

## Opening an Encrypted Drive (and Other Useful Commands)
The next command needed to set up the encrypted volume is `sudo cryptsetup luksOpen /dev/sdb $VOLNAME`, where `$VOLNAME` is the name we want to map our encrypted drive to:

```
sudo cryptsetup luksOpen /dev/sdb sdb_enc
Enter passphrase for /dev/sdb: *********
```

The mapping is then enstablished such that `/dev/sdb` is now found at `/dev/mapper/` (as one may check by running `ll /dev/mapper`). We can verify the status of the volume by running `sudo cryptsetup -v status sdb_enc`:

```
/dev/mapper/sdb_enc is active.
  type:    LUKS1
  cipher:  aes-xts-plain64
  keysize: 256 bits
  key location: dm-crypt
  device:  /dev/sdb
  sector size:  512
  offset:  4096 sectors
  size:    15628049072 sectors
  mode:    read/write
Command successful.
```

Another very useful command that one may wants to run is `sudo cryptsetup luksDump /dev/sdb`. In fact, from `man cryptsetup`:

> LUKS  header:  If the header of a LUKS volume gets damaged, all data is permanently lost unless you have a header-backup.  If a key-slot is damaged, it can only be  restored  from  a  header-backup  or  if  another  active key-slot with known passphrase is undamaged.  Damaging the LUKS header is something people manage to do with surprising frequency. This risk is the result of a trade-off  between  security and safety, as LUKS is designed for fast and secure wiping by just overwriting header and key-slot area.

To back-up the LUKS header, run:

`sudo cryptsetup luksHeaderBackup /dev/sdb --header-backup-file /path/to/luks_backup_sdb`

Once the mapping is done, a file system needs to be initialised on it for the latter to be used. To do so, simply run `sudo mkfs.ext4 /dev/mapper/sdb_enc`
*N.B. This needs to be done only the first time, after the volume is created.*

The output should look like the following:

```
sudo mkfs.ext4 /dev/mapper/sdb_enc 
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 1953506134 4k blocks and 244191232 inodes
Filesystem UUID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
	102400000, 214990848, 512000000, 550731776, 644972544, 1934917632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

The only thing left to be done is to mount the encrypted hard drive (already opened with luks). First, let's create a mount point where we like the best, e.g., under `/mnt`: `sudo mkdir /mnt/data1`. Once that is done, just mount the drive running `sudo mount /dev/mapper/sdb_enc /mnt/data1/`.


Sources:

[Linux hard disk encryption with cryptsetup](https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/)

[Linux hard disk encryption using LUKS](https://www.tecmint.com/linux-hard-disk-encryption-using-luks/)

[How secure is an encrypted LUKS filesystem?](https://askubuntu.com/questions/97196/how-secure-is-an-encrypted-luks-filesystem)

[How to backup or restore LUKS header](https://blog.sleeplessbeastie.eu/2019/01/09/how-to-backup-or-restore-luks-header/)

[How to check volume encryption on Linux](https://unix.stackexchange.com/questions/108537/verify-if-a-hard-drive-is-encrypted-on-linux)