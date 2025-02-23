
# passport image

Contains the necessary files to create the container with the backends for the demo application.


## Before creating the image

### update the backends

So for example:

```bash
$ cd coaguscan_image
$ mkdir -p crank/runtimes
$ rsync -avP /path/to/uncompressed/linux-raspberry-aarch64-opengles_2.0-drm-obj crank/runtimes/
```

**Note:** *linux-raspberry-aarch64-opengles_2.0-drm-obj* does not has a final '/'. This is important in order to copy the whole folder and not it's contents.

For some reason the runtimes are distributed with the binaries without the execution bit set. So the next step is to set them:

```bash
$ chmod a+x crank/runtimes/linux-raspberry-aarch64-opengles_2.0-drm-obj/bin/*
```

Finally remove the unneeded plugins.

```bash
$ rm crank/runtimes/linux-raspberry-aarch64-opengles_2.0-drm-obj/plugins/libgre-plugin-mtdev.so
```

More might be needed later on.

### Copy the GUI to coaguscan\_image/gui.

```bash
$ mkdir -p gui/
$ rsync -avP /path/to/GUI/CoaguScanGui/ gui/
```

### Copy the printer library

```bash
$ rsync -avP /path/to/sbTools/luaprint/build/linux-raspberry-aarch64-opengles_2.0-drm-obj/luaprint.so gui/scripts/
```

### Setup backends

Create the directory coaguscan\_image/backends and copy the [CoaguScanSoftware](https://github.com/HBL001/CoreScanSoftware) binaries into coaguscan\_image/backends/. The database goes in an upper level.

**WARNING:** the binaries must keep their executable flags.

```bash
$ mkdir backends
$ rsync -avP /path/to/CoreScanSoftware/build-cross/bin/ backends/
$ rsync -avP /path/to/CoreScanSoftware/build-cross/cscan.db .
```

### Create the image and push it to docker hub.

```bash
$ docker build --file Dockerfile --tag clotspot/coaguscan .
$ docker push clotspot/coaguscan
```

### SSH into the board and retrieve the image.

```bash
$ ssh torizon@192.168.6.106 # Use the IP provided by your LAN
colibri-imx8x-06899918:~$ docker pull clotspot/coaguscan
Using default tag: latest
latest: Pulling from coaguscan
1901ca797b5e: Already exists
1804d50d1842: Already exists
a6d6cd538c68: Already exists
[...]
Status: Downloaded newer image for clotspot/coaguscan:latest
clotspot/coaguscan:latest
```

The image is now ready to be deployed by [docker-compose_drm.yml](docker-compose_drm.yml).


# docker-compose.yml

This file handles all the images that must run on the board.

Copy this file to the board. Then stop all the running containers.

```bash
colibri-imx8x-06899918:~$ docker stop $(docker ps -a -q)
```

and then run:

```bash
colibri-imx8x-06899918:~$ docker-compose up --remove-orphans
```

## Enabling VNC

If enabling VNC is required add the following environment variable to the weston container:

```
- ENABLE_VNC=1
```

More information in https://developer.toradex.com/knowledge-base/remote-access-to-torizoncore-ui

# Data stored on the board

The applications should write non-volatile data to /opt/data/\<application\>. This is the path within the container. On the base board this is the same as:

    /var/lib/docker/volumes/torizon_data/_data

In order to access this path within the base OS root capabilities should be gained, for example using sudo:

```bash
colibri-imx8x-06899918:~$ sudo su -
Password:
root@colibri-imx8x-06899918:~# cd /var/lib/docker/volumes/torizon_data/_data
root@colibri-imx8x-06899918:/var/lib/docker/volumes/torizon_data/_data# ls -alh
total 24K
drwxr-xr-x 4 root root 4.0K Dec 15 15:17 .
drwxr-xr-x 3 root root 4.0K Nov 25 17:48 ..
drwxr-xr-x 2 root root 4.0K Nov 25 17:48 gui
drwxr-xr-x 4 root root 4.0K Nov 25 17:50 scan_process
root@colibri-imx8x-06899918:/var/lib/docker/volumes/torizon_data/_data#
```

# Other images

## weston image

Contains a weston image with basic modifications in order to run it in kiosk mode. Was used during the initial development phase.

```bash
$ cd weston
$ docker build --file Dockerfile --tag clotspot/weston-vivante .
$ docker push clotspot/weston-vivante
```

5. SSH into the board and retrieve the image.

```bash
$ ssh torizon@192.168.6.106
colibri-imx8x-06899918:~$ docker pull clotspot/weston-vivante
Using default tag: latest
latest: Pulling from weston-vivante
1901ca797b5e: Already exists
1804d50d1842: Already exists
a6d6cd538c68: Already exists
[...]
Status: Downloaded newer image for clotspot/weston-vivante:latest
clotspot/weston-vivante:latest
```

## demoapp_image

Contains the necessary files to create the container with the gui and scan\_process for the demo application.

Originally they where going to be two separate docker images, but in order to let the two process communicate trough [GREIO](https://support.cranksoftware.com/hc/en-us/articles/360056943692) they needed to share the same IPC namespace, which do not seemed easy to achieve in docker.

If at any point of the future it is required to split each application into it's own container then using a "Storyboard IO Over TCP" connection might be advisable.

Before creating the image:

1. Copy Storyboard's runtime to coaguscan\_image/crank/runtimes. The final layout should be:

```bash
crank/
└── runtimes
```

**NOTE:** this can be accomplished by running the script *setup_crank.sh*. In order to run it you need to install rsync and unzip:

```bash
$ sudo apt install rsync unzip
```

2. Copy the GUI to coaguscan\_image/gui.

```bash
rsync -avP /path/to/GUI/CoaguScanGui/ demoapp_image/gui/
```

3. Create the directory coaguscan\_image/scan\_process and copy the binary scan\_process into coaguscan\_image/scan\_process/.

**WARNING:** the binary must keep it's executable flags.

```bash
$ cd demoapp_image
$ rsync -avP /path/to/CoreScanSoftware/build/bin/scan_process scan_process/
```


3. Create the image and push it to the local registry.

```bash
$ cd demoapp_image
$ docker build --file Dockerfile --tag clotspot/coaguscan .
$ docker push clotspot/coaguscan
```

5. SSH into the board and retrieve the image.

```bash
$ ssh torizon@192.168.6.106
colibri-imx8x-06899918:~$ docker pull clotspot/coaguscan
Using default tag: latest
latest: Pulling from coaguscan
1901ca797b5e: Already exists
1804d50d1842: Already exists
a6d6cd538c68: Already exists
[...]
Status: Downloaded newer image for clotspot/coaguscan:latest
clotspot/coaguscan:latest
```

The image is now ready to be deployed by [docker-compose_drm.yml](docker-compose_drm.yml).
