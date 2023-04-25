# Network Manager Stuff
For Debian 11   

Delete a automatically created default routes, when using multiple interfaces:   
Note:    
Change INTERFACE to the actual interface, for example eth0 for ethernet connection.   

First check the interface name and default route you want to be automatically deleted:   
```bash
ip route
```

Make a script:   
```bash
sudoedit /etc/NetworkManager/dispatcher.d/99-INTERFACE-up.sh
```

Script:   
```bash
#!/bin/bash
if [ "$1" = "INTERFACE" ] && [ "$2" = "up" ]; then
    # Delete the default route when INTERFACE is brought up
    ip route del $(ip route show | grep 'dev INTERFACE'|head -1)
fi
```

Give permissions:   
```bash
sudo chmod 700 /etc/NetworkManager/dispatcher.d/99-INTERFACE-up.sh
```

Save and restart Network Manager   
```bash
sudo systemctl restart NetworkManager
```
Verify:
```bash
ip route
```

Tip:   
If you dont want to delete the route, but only change the metrics AKA priority:   
Lower metric takes priority.   
Check connections:   
```bash
nmcli connection show
```

Check the UUID of the connection you want to edit.   

Then change the metric:   
```bash
sudo nmcli connection modify THE_UUID_YOU_CHECKED ipv4.route-metric 1
```

Reconnect or restart Network Manager   
Profit!   
