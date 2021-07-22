# ConBee / Deconz configuration for DSM7

This document helps with the setup of a startup script for running the Deconz docker container on Synology with DSM7

## Intro

Synology DSM7 does not load, by default, drivers for external USB devices. These drivers can be loaded by the user, but a couple of tricks are necessary.

DSM7 uses `systemd` for services statup operations. Systemd allows additional scripts and complex chaining of these. To be able to use the ConBee USB dongle, three drivers are missing: `usbserial`, `ftdi_sio` and `cdc_adm`. These three must be loaded before docker daemon starts and pulls up your container.

## Usage

### Prepare the script

Login via SSH into you synology. Create/Copy/Move the startup script into `/usr/local/lib/systemd/system`

You should see the following:

```bash
ls -l /usr/local/lib/systemd/system/pkg-conbee.service 

-rw-r--r-- 1 root root 341 Jul 22 13:39 /usr/local/lib/systemd/system/pkg-conbee.service
```

### Test the script

Run the following:

```bash
sudo systemctl start pkg-conbee.service
```

This will run the script. Now you can check modules in the kernel with `lsmod` as

```bash
lsmod |grep cdc_acm

cdc_acm                18383  2 
usbcore               201223  12 etxhci_hcd,usblp,uhci_hcd,usb_storage,usbserial,ehci_hcd,ehci_pci,usbhid,ftdi_sio,cdc_acm,xhci_hcd,xhci_pci
```

This means modules are loaded.

### Enable the script

The script must run at startup, so you need to tell `systemd` to do it

```bash
systemctl enable pkg-conbee.service
```

If you now reboot, this script will run before docker and the container will find the old ACM tty.


##  Startup script

```bash
[Unit]
Description=ConBee Module loader
Requires=pkg-volume.target
Wants=
Requisite=
After=
IgnoreOnIsolate=no
OnFailure=

[Install]
WantedBy=pkg-Docker-dockerd.service

[Service]
Type=simple
ExecStart=/bin/bash -c "modprobe usbserial && modprobe ftdi_sio && modprobe cdc-acm" 
TimeoutStartSec=3600
TimeoutStopSec=3600
RemainAfterExit=true
```

## Disclaimer

USE THIS AT YOUR RISK