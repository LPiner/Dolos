

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




