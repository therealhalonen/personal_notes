# Disable wake from suspend, with usb device
*Got a problem in one of my laptop, where usb mouse woke up the PC, even when laptop lid was closed...*   

Create service:
```bash
sudoedit /lib/systemd/system/mouse-wakeup.service
```

```bash
# mouse-wakeup.service
[Unit]
Description = Disable USB 1-1 to wake system from suspend.

[Service]
Type=oneshot
ExecStart=/bin/bash -c "echo 'disabled' > /sys/bus/usb/devices/1-1/power/wakeup"

[Install]
WantedBy= multi-user.target
```

```bash
sudo systemctl enable mouse-wakeup 
sudo systemctl start mouse-wakeup 
sudo systemctl status mouse-wakeup 
○ mouse-wakeup.service - Disable USB 1-1 to wake system from suspend.
     Loaded: loaded (/lib/systemd/system/mouse-wakeup.service; enabled; preset: disabled)
     Active: inactive (dead) since Fri 2023-03-03 13:08:48 EET; 2min 28s ago
    Process: 3109 ExecStart=/bin/bash -c echo 'disabled' > /sys/bus/usb/devices/1-1/power/wakeup (code=exited, status=0/SUCCESS)
   Main PID: 3109 (code=exited, status=0/SUCCESS)
        CPU: 3ms
```
