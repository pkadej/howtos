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
