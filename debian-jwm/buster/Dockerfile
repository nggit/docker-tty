# Copyright (c) 2022 nggit.
#
FROM debian:buster

VOLUME ["/sys/fs/cgroup", "/run", "/run/lock"]

# update system & install packages
RUN apt-get update && apt-get -y upgrade; \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install systemd-sysv jwm menu xinit xserver-xorg dbus-x11 policykit-1-gnome \
    locales libxml2-utils sudo nano vim-tiny less alsa-utils pciutils usbutils procps psmisc net-tools iproute2 iputils-ping wget curl htop

# clean up
RUN apt-get clean; \
    rm -rf /etc/machine-id /var/lib/dbus/machine-id /var/lib/apt/lists/* /tmp/* /var/tmp/*; \
    find /var/log -name "*.log" -type f -delete

# tweaks
RUN sed -i 's/^#*ForwardToConsole=[a-z]*/ForwardToConsole=yes/' /etc/systemd/journald.conf; \
    sed -i 's/^#*TTYPath=[\/0-9a-z]*/TTYPath=\/dev\/tty10/' /etc/systemd/journald.conf; \
    sed -i 's/^#*MaxLevelConsole=[a-z]*/MaxLevelConsole=info/' /etc/systemd/journald.conf; \
    sed -i 's/^#*ReadKMsg=[a-z]*/ReadKMsg=no/' /etc/systemd/journald.conf; \
    systemctl enable getty@tty10.service; \
    systemctl disable getty@tty1.service; \
    echo "dbus-launch --sh-syntax --exit-with-session jwm" | tee /etc/skel/.xinitrc /root/.xinitrc; \
    echo "\nif [ \"\$( tty )\" = /dev/tty10 ]; then exec startx; else true; fi" | tee -a /etc/skel/.profile /root/.profile; \
    echo "<?xml version=\"1.0\"?><JWM><Include>/etc/jwm/system.jwmrc</Include> \
        <StartupCommand>/usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1</StartupCommand> \
        </JWM>" | xmllint --format - | tee /etc/skel/.jwmrc /root/.jwmrc

# allow systemd (and other programs) to identify that it is executed within a container
# https://systemd.io/CONTAINER_INTERFACE/
ENV container docker

STOPSIGNAL 37

ENTRYPOINT ["/usr/bin/env", "--"]
CMD ["sh", "-c", "test -d /dev/input && chown -R root:input /dev/input/*; \
    test -d /dev/snd && chown -R root:audio /dev/snd/*; \
    test -d /dev/dri && chown -R root:video /dev/dri/*; \
    test -e /dev/fb0 && chown root:video /dev/fb0; \
    test -e /dev/video0 && chown root:video /dev/video0; \
    if msg=$( mount -o remount,rw /sys/fs/cgroup 2>&1 ); then \
        if ! grep '^cgroup /sys/fs/cgroup/systemd ' /proc/mounts > /dev/null; then \
            mkdir -p /sys/fs/cgroup/systemd; \
            mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd; \
        fi; \
        mount -o remount,ro /sys/fs/cgroup; \
        mount -o remount,ro /sys; \
    elif test -d /sys/fs/cgroup; then \
        echo \"INFO: I assume that you are running the container without --cap-add SYS_ADMIN: $msg\"; \
    fi; \
    systemd-machine-id-setup; exec /sbin/init"]
