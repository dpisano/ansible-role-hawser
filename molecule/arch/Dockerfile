# Arch Linux with systemd and Python, for the Molecule matrix. Mirrors what the
# geerlingguy/docker-*-ansible images provide for the Debian-family hosts.
FROM archlinux:latest

RUN pacman -Syu --noconfirm python sudo systemd \
    && pacman -Scc --noconfirm

STOPSIGNAL SIGRTMIN+3
VOLUME ["/sys/fs/cgroup"]
CMD ["/usr/lib/systemd/systemd"]
