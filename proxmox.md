## This is for AMD Ryzen and a NVIDIA CArd, Intel may need other tweaks.

### Passing Through Your GPU

1. Add amd_iommu=on to /etc/default/grub:

        root@pve:~# cat /etc/default/grub | grep GRUB_CMDLINE_LINUX_DEFAULT=
        GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on pcie_acs_override=downstream,multifunction"
    
   If you need the ACS patch now you can add it in here as well:

            root@pve:~# cat /etc/default/grub | grep GRUB_CMDLINE_LINUX_DEFAULT=
            GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on pcie_acs_override=downstream,multifunction"
    
2. Update GRUB:

        root@pve:~# update-grub
    
3. Blacklist your card in modprobe:

        root@pve:~# cat /etc/modprobe.d/blacklist.conf
        blacklist radeon
        blacklist nouveau
        blacklist nvidia
        root@pve:~#

4. Commit those changes with:

        root@pve:~# update-initramfs -u

5. Update /etc/modules:

        root@pve:~# cat /etc/modules
        vfio
        vfio_iommu_type1
        vfio_pci
        vfio_virqfd

        root@pve:~#

6. Figure out what card you want the guest to have, I'm going to use the 1080:

        root@pve:~# lspci -v | grep NVIDIA
        08:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050] (rev a1) (prog-if 00 [VGA controller])
        08:00.1 Audio device: NVIDIA Corporation Device 0fb9 (rev a1)
        0b:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050] (rev a1) (prog-if 00 [VGA controller])
        0b:00.1 Audio device: NVIDIA Corporation Device 0fb9 (rev a1)
        0c:00.0 VGA compatible controller: NVIDIA Corporation GP104 [GeForce GTX 1080] (rev a1) (prog-if 00 [VGA controller])
        0c:00.1 Audio device: NVIDIA Corporation GP104 High Definition Audio Controller (rev a1)
        root@pve:~#

7. Get the vendor ids with the PCI address, the 1080 is 0c:00 as shown above:

        root@pve:~# lspci -n -s 0c:00
        0c:00.0 0300: 10de:1b80 (rev a1)
        0c:00.1 0403: 10de:10f0 (rev a1)
        root@pve:~#
 
8. Create /etc/modprobe.d/vfio.conf and add the vendor IDs in:

        root@pve:~# cat /etc/modprobe.d/vfio.conf
        options vfio-pci ids=10de:1b80,10de:10f0
        root@pve:~#

9. Reboot the host


### Windows VM Config

1. Mount the VIRTIO ISO to the CD/DVD drive on the guest. You can download it from here. 

        https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
2. Convert the guest network adapter type to VirtIO, under the hardware tab.
3. Add a second disk, make its type SCSI and its size 1g, we will delete it in a second.
4. Boot the VM and from device manager install the missing drivers. D:\virtioscsi\win10\amd64, D:\NetKVM\win10\amd64
5. Open an elevated command prompt on the windows guest and run:

                bcdedit /set {current} safeboot minimal
 6. Turn off the VM, delete the 1gig SCSI driver you created on step 3.
 7. Detach the primary harddisk, edit it to the new SCSI type, add it back to the VM.
 8. Boot the VM, it should boot into safe mode. If it does not or it bluescreens revert the disktype back to IDE and start from set one again.
 9. Open an elevated command prompt on the windows guest and run:
 
                bcdedit /deletevalue {current} safeboot
10. Restart the VM, it should boot normally.

### Sound

Make sure Proxmox supports AC97, you can try this with hda but as of writing this it's totally borked.

    root@pve:~# kvm -soundhw ?
    Valid sound card names (comma separated):
    sb16        Creative Sound Blaster 16
    es1370      ENSONIQ AudioPCI ES1370
    ac97        Intel 82801AA AC97 Audio
    adlib       Yamaha YM3812 (OPL2)
    gus         Gravis Ultrasound GF1
    cs4231a     CS4231A
    hda         Intel HD Audio
    pcspk       PC speaker
    -soundhw all will enable all of the above
    root@pve:~#


Enable audio in Proxmox, reboot the server.

    root@pve:~# cat /etc/modprobe.d/alsa-base.conf
    options snd_hda_intel index-0
    root@pve:~#

Use alsamixer and speaker-test to make sure everything works.

    root@pve:~# alsamixer
    root@pve:~# speaker-test

Tell your QEMU to give your VM the ac97 passthrough device.

    root@pve:~# cat /etc/pve/qemu-server/110.conf | grep args
    args: -soundhw ac97
    root@pve:~#

AC97 Drivers need to be installed on your VM.

http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=14&PFid=23&Level=4&Conn=3&DownTypeID=3&GetDown=false

YOU MUST DISABLE DRIVER SIGNING TO GET THEM INSTALLED

Settings > Update & Security > Recovery in the left pane > Advanced Start-up > Restart Now

Once restarted you need to choose option 7 which disables driver signature enforcement.

Once Windows comes back go into Device Manager, Right click the multimedia device and choose 'Update Driver Software' click browse on the next page and navigate to the realtek driver you downloaded earlier.

If the driver installs and speaker-test works then you should have sound.




