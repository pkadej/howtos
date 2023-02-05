# howtos
## docker - mount new volume to existing container

1. Go to the container directory: 
`cd /var/lib/docker/containers/<container id>` 
2. Normalize config.v2.json with jq: 
`jq . config.v2.json > tmp.json`
also create backup of config
`cp config.v2.json config.v2.orig` 
3. Stop container, stop docker
`docker stop <container>`
`service docker stop`
4. edit tmp.json, you’ll need to scroll down to find “MountPoints,” which contains the configuration for all the volumes and bind mounts. You can add a new one here, which should be in the following format:
````json
  "MountPoints": {
    "/mnt": {
      "Source": "/dir/on/host",
      "Destination": "/dir/in/container",
      "RW": true,
      "Name": "",
      "Driver": "",
      "Type": "bind",
      "Propagation": "rprivate",
      "Spec": {
        "Type": "bind",
        "Source": "/dir/on/host",
        "Target": "/dir/in/container"
      },
      "SkipMountpointCreation": false
    }
  }
````
5. Minify config back: `jq -c . tmp.json > config.v2.json`
6. start docker `service docker start`

## meld as mergetool in git (Windows)
```bash
git config --global merge.tool meld
git config --global mergetool.meld.cmd "C:\\Program\ Files\ \(x86\)\\Meld\\meld\\meld.exe \"$LOCAL\" \"$BASE\" \"$REMOTE\""
```

## systemd+kde - 90s shutdown/reboot wait bug

Turns out the other problem was caused by the kwin_x11 process failing to stop on shutdown, this meant that the system waited for it to stop until 90 seconds had passed (the default systemd process wait time). We can solve this by creating a service that automatically kills this process when we shutdown the computer.

To do that, first we need a script that kills the kwin process, open a terminal and change directory to /usr/lib/systemd/system-shutdown.

`cd /usr/lib/systemd/system-shutdown`

Then, we have to create a script (to create a file and edit it within /usr you will need root privileges, so for the next commands sudo will be used).

`sudo touch kill_kwin.shutdown`

Then, open the file with nano (if you get an error saying that the command nano is unknown, download nano), or the terminal text editor of your choice.

`sudo nano kill_kwin.shutdown`

Then add this code in the file.

```
#!/bin/sh

# Kill KWin immediately to prevent stalled shutdowns/reboots
pkill -KILL kwin_x11

```

Afterwards, ctrl+o to write the changes to the file and ctrl+x to exit nano.

Then, we need to make this file executable.

`chmod +x kill_kwin.shutdown`

Afterwards, we need to make the service that will automatically run this script on shutdown, to do this.

Change directory to /etc/systemd/system.

`cd /etc/systemd/system`

Then, create a new file called kill_kwin.service (to create a file and edit it within /etc you will need root privileges, so for the next commands sudo will be used).

`sudo touch kill_kwin.service`

Then, open the file with nano (if you get an error saying that the command nano is unknown, download nano), or the terminal text editor of your choice.

`sudo nano kill_kwin.service`

Afterwards, put this code in the file.

```
[Unit]
Description=Kill KWin at shutdown/reboot

[Service]
Type=oneshot
ExecStart=/bin/true
ExecStop=/bin/sh /usr/lib/systemd/system-shutdown/kill_kwin.shutdown
RemainAfterExit=true

[Install]
WantedBy=multi-user.target

```

Afterwards, ctrl+o to write the changes to the file and ctrl+x to exit nano.

Then, make the file executable (IDK if this step is necessary but I did do it).

`sudo chmod +x kill_kwin.service`

Then enable the service.

`sudo systemctl enable kill_kwin.service`

This will make it so the service is automatically started when you boot up the computer, if you want it to also be active on your current session, then also input.

`sudo systemctl start kill_kwin.service`

This fix has actually worked for me for the better part of a month, so it should work for you guys too, any doubts, feel free to respond. And please forgive me for forgetting to update the post after finding the real fix.
