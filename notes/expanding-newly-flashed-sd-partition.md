If you (say) are flashing an SD card with ubuntu to run on a raspberry pi, here's what you need to do:

* Plug in the card. Determine its device name. Here are a few ways to try:
    * You might be able to tell from the output of `df -h`.
    * If you're on a mac, you can determine the device name by doing a system
        report and viewing the information for the slot you plugged the SD card
        in. E. g., "Card Reader" or "USB". It will say somewhere in the
        description `/dev/disk4` or something.
    * Or, you can compare the output of `df -h` before you insert the card and
        after.
    Note, there will be a disk, and there will be partitions on the disk. If
    the disk name is `/dev/sdb`, then the partitions might also appear in the
    device list as `/dev/sdb1` and `/dev/sdb2`, eg. You can ignore the
    partitions for now.
* Unmount all partitions in Disk Utility or using `umount`.
* Copy an ubuntu `img` onto the SD card.

    ```
    dd bs=1m if=/path/to/ubuntu.img of=/dev/disk4
    ```
    You can check the status of `dd` with `Ctrl+T` on a mac, or by sending
    `pkill -SIGUSR1 dd` on linux.

    I think the `dd` command will put 2 partitions on the drive, a boot
    partition and another one. You want to expand the other one, which we will
    do below.
* Expand the partition that ubuntu installed to. If you are on a mac, you will
    likely need to mount the card into a linux box, because afaict osx won't let
    you resize partitions in the filesystem format written by the ubuntu image.

    * If using the card reader slot on OSX, you will have to mount the raw disk
      into your VM. First unmount any partitions from Disk Utility. (You will
      have to keep doing this in the following steps, otherwise you will get a
      "resource busy" or your VM won't boot.)
    * `sudo chown des4maisons:staff /deb/disk4`, otherwise virtualbox won't
        have access to that disk.
    * If using virtualbox, create a device file (???) that
        represents the physical disk:

    ```
    device_file_to_create="/Users/marguerite/VirtualBox VMs/tmp_default_1439416723556_64097/sd.vmdk"
    disk_for_device_file=/dev/disk4
    VBoxManage internalcommands createrawvmdk \
      -filename "$device_file_to_create" \
      -rawdisk "$disk_for_device_file"
    ```

    * Add this new disk to your virtual machine - power it off, click on
        settings -> storage > + hard drive, select the device file you created
    * Once you have the linux box booted with your card mounted, find out the
        device name as above.
    * Delete all non-boot partitions from the device:

        1.
            ```
            sudo fdisk /dev/sdb
            ```
        1. `p` to print partitions and find the non-boot partition
        1. `d`, then the partition number you want to delete, to delete the partition
        1. Repeat till all non-boot partitions are gone.
        1. `n` for new partition, `p` for primary.
        1. The defaults usually work for me for start and end block numbers
        1. reboot (is this cargo culting??)
     * Now that your new partition is taking up all non-boot-partition space on
         the disk, resize the filesystem on *the partition* (not the *disk*):

         ```
         sudo resize2fs /def/sdb2
         ```
         If you get "bad superblock" errors, you may be trying to resize the
         *disk* and not the *partition*.
* If you want, you can add swap space to your SD card as well. Boot the
    raspberry pi with your SD card in it, and then run

    ```
    apt-get install dphys-swapfile
    ```
    which should set everything up for you.


Resources:

- http://superuser.com/questions/373463/how-to-access-an-sd-card-from-a-virtual-machine
- https://wiki.ubuntu.com/ARM/RaspberryPi
- https://www.raspberrypi.org/documentation/installation/installing-images/mac.md
