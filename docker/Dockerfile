# syntax=docker/dockerfile:1
FROM ghcr.io/warewulf/warewulf-rockylinux:9
COPY yum.repos.d/* /etc/yum.repos.d/
RUN wget https://download.rockylinux.org/pub/sig/9/altarch/aarch64/altarch-common/RPM-GPG-KEY-Rocky-SIG-AltArch -O /etc/pki/rpm-gpg/RPM-GPG-KEY-Rocky-SIG-AltArch
RUN dnf update -y
RUN dnf install -y --setopt=install_weak_deps=False rocky-release-rpi kernel-rpi-4k-core kernel-rpi-firmware rpi-firmware-bluez rpi-firmware-nonfree
RUN dnf install -y "https://github.com/warewulf/warewulf/releases/download/v4.6.0/warewulf-dracut-4.6.0-1.el9.noarch.rpm"
RUN dracut --verbose --force --no-hostonly --add wwinit --regenerate-all
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/chrony chrony; echo
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/slurm-compute slurm-slurmd munge; echo
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/ipa-client ipa-client; echo
RUN dnf install -y --downloadonly --downloaddir=/apps/pkgs/glusterfs-client glusterfs-fuse; echo