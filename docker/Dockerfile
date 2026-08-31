# syntax=docker/dockerfile:1
FROM ghcr.io/warewulf/warewulf-rockylinux:9
COPY yum.repos.d/* /etc/yum.repos.d/
RUN dnf update -y
RUN dnf install -y --setopt=install_weak_deps=False rocky-release-rpi kernel-rpi-4k-core kernel-rpi-firmware rpi-firmware-bluez rpi-firmware-nonfree
RUN dnf install -y "https://github.com/warewulf/warewulf/releases/download/v4.6.0/warewulf-dracut-4.6.0-1.el9.noarch.rpm"
RUN dracut --verbose --force --no-hostonly --add wwinit --regenerate-all
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/chrony chrony; echo
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/slurm-compute slurm-slurmd munge; echo
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/ipa-client ipa-client; echo
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/glusterfs-client glusterfs-fuse; echo