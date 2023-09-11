# debian-jwm
Debian image with a minimalist Window Manager "JWM". The display will be on `tty10` (***Ctrl-Alt-F10***), coexist with the host.

## Requirements
Make sure `tty10` is empty / not in use. Check with **Ctrl-Alt-F10**, make sure you don't see getty's *login prompt* there.

It usually doesn't matter because most distros only use 1 - 6, and 7 for display managers (lightdm, sddm, etc).

## How-to
**1. Installation (creating a container)**
```
docker create -it --name debian_jwm \
    --cap-add SYS_ADMIN --cap-add SYS_TTY_CONFIG --tmpfs /run --tmpfs /run/lock -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    --device /dev/tty --device /dev/tty0 --device /dev/tty7 --device /dev/tty10 \
    --device /dev/input --device /dev/psaux \
    --device /dev/bus/usb --device /dev/usb \
    --device /dev/snd \
    --device /dev/dri --device /dev/fb0 --device /dev/video0 --device /dev/vga_arbiter \
    -v /run/udev/control:/run/udev/control:ro -v /run/udev/data:/run/udev/data:ro \
    -v /etc/localtime:/etc/localtime:ro \
    nggit/debian-jwm:bookworm
```
At this point, you can add other volume bindings if you want to share data between the host and the container, using the ` -v` or ` -volume` flags. Because partitions like **/dev/sda1**, etc. are by default not exposed to containers for security reasons.

If the container cannot be created in the above way, for example there is a different device path on your device, you can customize it yourself. Or more simply use *privileged* mode to access all devices. But this also means ignoring the security aspect.
```
docker create -it --name debian_jwm \
    --privileged --tmpfs /run --tmpfs /run/lock -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    -v /run/udev/control:/run/udev/control:ro -v /run/udev/data:/run/udev/data:ro \
    -v /etc/localtime:/etc/localtime:ro \
    nggit/debian-jwm:bookworm
```

**2. Run the container and check the logs**
```
docker start debian_jwm
docker logs debian_jwm
```
You should see a line like the following at the end of the logs if the container is running properly:
```
Debian GNU/Linux 12 246eb7415c97 console
```
If you want to make the container always run automatically even if the host is restarted, you can change the restart policy on the container to *always* or *unless-stopped*:
```
docker update --restart unless-stopped debian_jwm
```
**3. Change the root password**
```
docker exec -it debian_jwm passwd
```
**4. Login**

You can login JWM in `tty10` (**Ctrl-Alt-F10**).

or console mode
```
docker attach debian_jwm
```
Press *Enter* or *Ctrl-C* if the login prompt has not appeared / blocked by the logs.

Done. For emergency you can enter using:
```
docker exec -it debian_jwm bash
```
## Login without root

**1. Create a group "mygroup"**
```
docker exec -it debian_jwm groupadd -g 1000 mygroup
```
**2. Create user "myuser"**
```
docker exec -it debian_jwm useradd -mu 1000 -g 1000 -G audio,video,input,tty,sudo,systemd-journal -s /bin/bash myuser
```
**3. Change the password of user "myuser"**
```
docker exec -it debian_jwm passwd myuser
```
Done. You can login with user "myuser" and your password.

## Installing the Display Manager in the container
Display Managers like lightdm default to `tty7`, and when first running will display via `tty0`. This means it will follow wherever you currently are, e.g. `tty1`. You can get `tty1` back with **Ctrl-Alt-F1** if it gets overwritten by lightdm.

The main problem is `tty7`. Make sure your host is not using lightdm on that `tty7` as well.
## Stopping and deleting the container
```
docker stop debian_jwm
docker rm debian_jwm
```

## License
MIT
