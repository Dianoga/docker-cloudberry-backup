# Docker container for CloudBerry Backup
[![](https://images.microbadger.com/badges/image/jlesage/cloudberry-backup.svg)](http://microbadger.com/#/images/jlesage/cloudberry-backup "Get your own image badge on microbadger.com")
[![Build Status](https://travis-ci.org/jlesage/docker-cloudberry-backup.svg?branch=master)](https://travis-ci.org/jlesage/docker-cloudberry-backup)

This is a Docker container for CloudBerry Backup.  The GUI of the application is
accessed through a modern web browser (no installation or configuration needed
on client side) or via any VNC client.

---

[![CloudBerry Backup logo](https://www.cloudberrylab.com/images/logos/logo-backup.png)](https://www.cloudberrylab.com/backup/linux.aspx)
[![CloudBerry Backup](https://dummyimage.com/600x110/ffffff/575757&text=CloudBerry+Backup)](https://www.cloudberrylab.com/backup/linux.aspx)

Backup files and folders to cloud storage of your choice: Amazon S3, Azure Blob Storage, Google Cloud Storage, HP Cloud, Rackspace Cloud Files, OpenStack, DreamObjects and other.

---

## Quick Start
First create the configuration directory for CloudBerry Backup.  In this
example, `/docker/appdata/cloudberry-backup` is used.  To backup files located
under your home directory, launch the CloudBerry Backup docker container with the
following command:
```
docker run -d \
    -p 5800:5800 \
    -p 5900:5900 \
    -v /var/docker/cloudberry-backup:/config \
    -v $HOME:/storage:ro \
    jlesage/cloudberry-backup
```

Browse to `http://your-host-ip:5800` to access the CloudBerry Backup GUI.  Your
home directories and files appear under the `/storage` folder in the container.

## Usage
```
docker run [-d|--rm] \
    --name=cloudberry-backup \
    [-e <VARIABLE_NAME>=<VALUE>]... \
    [-v <HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]]... \
    [-p <HOST_PORT>:<CONTAINER_PORT>]... \
    jlesage/cloudberry-backup
```
| Parameter | Description |
|-----------|-------------|
| -d        | Run the container in background.  If not set, the container runs in foreground. |
| --rm      | Automatically remove the container when it exits. |
| -e        | Pass an environment variable to the container.  See the [Environment Variables](#environment-variables) section for more details. |
| -v        | Set a volume mapping (allows to share a folder/file between the host and the container).  See the [Data Volumes](#data-volumes) section for more details. |
| -p        | Set a network port mapping (exposes an internal container port to the host).  See the [Ports](#ports) section for more details. |

### Environment Variables

To customize some properties of the container, the following environment
variables can be passed via the `-e` parameter (one for each variable).  Value
of this parameter has the format `<VARIABLE_NAME>=<VALUE>`.

| Variable       | Description                                  | Default |
|----------------|----------------------------------------------|---------|
|`USER_ID`       | ID of the user CloudBerry Backup run as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | 1000    |
|`GROUP_ID`      | ID of the group CloudBerry Backup run as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | 1000    |
|`TZ`            | [TimeZone] of the container.  Timezone can also be set by mapping `/etc/localtime` between the host and the container. | Etc/UTC |
|`DISPLAY_WIDTH` | Width (in pixels) of the display.             | 1280    |
|`DISPLAY_HEIGHT`| Height (in pixels) of the display.            | 768     |
|`VNC_PASSWORD`  | Password needed to connect to the CloudBerry Backup GUI.  See the [VNC Pasword](#vnc-password) section for more details. | (unset) |

[TimeZone]: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

### Data Volumes

The following table describes data volumes used by the container.  The mappings
are set via the `-v` parameter.  Each mapping is specified in the following
format: `<HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]`.

| Container path  | Permissions | Description |
|-----------------|-------------|-------------|
|`/config`        | rw          | This is where CloudBerry Backup Backup stores its configuration, log and any files needing persistency. |
|`/storage`       | ro          | This is where files that need to be backup are located. |

### Ports

Here is the list of ports used by the container.  They can be mapped to the host
via the `-p <HOST_PORT>:<CONTAINER_PORT>` parameter.  The port number inside the
container cannot be changed, but you are free to use any port on the host side.

| Port | Mapping to host | Description |
|------|-----------------|-------------|
| 5800 | Mandatory  | Port used to access to the CloudBerry Backup GUI via the web interface. |
| 5900 | Mandatory  | Port used to access to the CloudBerry Backup GUI via the VNC protocol.  |

## User/Group IDs

When using data volumes (`-v` flags), permissions issues can occur between the
host and the container.  For example, the user within the container may not
exists on the host.  This could prevent the host from properly accessing files
and folders on the shared volume.

To avoid any problem, you can specify the user the application should run as.

This is done by passing the user ID and group ID to the container via the
`USER_ID` and `GROUP_ID` environment variables.

To find the right IDs to use, issue the following command on the host, with the
user owning the data volume on the host:

    id <username>

Which gives an output like this one:
```
uid=1000(myuser) gid=1000(myuser) groups=1000(myuser),4(adm),24(cdrom),27(sudo),46(plugdev),113(lpadmin)
```

The value of `uid` (user ID) and `gid` (group ID) are the ones that you should
be given the container.

## Accessing the GUI

Assuming the host is mapped to the same ports as the container, the graphical
interface of the application can be accessed via:

  * A web browser:
```
http://<HOST IP ADDR>:5800
```

  * Any VNC client:
```
<HOST IP ADDR>:5900
```

If different ports are mapped to the host, make sure they respect the
following formula:

    VNC_PORT = HTTP_PORT + 100

This is to make sure accessing the GUI with a web browser can be done without
specifying the VNC port manually.  If this is not possible, then specify
explicitly the VNC port like this:

    http://<HOST IP ADDR>:5800/?port=<VNC PORT>

## VNC Password
To restrict access to your application, a password can be specified.  This can
be done via two methods:
  * By using the `VNC_PASSWORD` environment variable.
  * By creating a `.vncpass_clear` file at the root of the `/config` volume.
  This file should contains the password (in clear).  During the container
  startup, content of the file is obfuscated and moved to `.vncpass`.

**NOTE**: This is a very basic way to restrict access to the application and it
should not be considered as secure in any way.

## Registration

### Home/Freeware Edition

If you are not able to copy-paste the registration key you received by email,
try to access CloudBerry Backup with a VNC client instead of a web browser.
If this still doesn't work, you can manually install the registration key.
Create these two files with the same information you used during the
registration:

In `/<Configuration directory on host>/etc/config/.LicenseMail`:
```
email : [PUT_YOUR_EMAIL_HERE]
```

In `/<Configuration directory on host/etc/config/.License`:
```
FREE<UserName>[PUT_YOUR_NAME_HERE]</UserName><Signature>[PUT_YOUR_REGISTRATION_KEY_HERE]</Signature>
```
