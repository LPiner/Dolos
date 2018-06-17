

=====
Sound

Enable audio in Proxmox

    root@pve:~# cat /etc/modprobe.d/alsa-base.conf
    options snd_hda_intel index-0
    root@pve:~#

Tell your QEMU to give your VM the ac97 passthrough device.

    root@pve:~# cat /etc/pve/qemu-server/110.conf | grep args
    args: -soundhw ac97
    root@pve:~#

AC97 Drivers need to be installed on your VM.
http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=14&PFid=23&Level=4&Conn=3&DownTypeID=3&GetDown=false


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
