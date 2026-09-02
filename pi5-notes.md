# Pi 5 Notes

- Most of the changes needed to support the Pi 5 comes from
  [Piwulf](https://github.com/middelkoopt/piwulf), although with some changes
  to make it work on real hardware.
- Two patches had to be made to allow Warewulf to support features that Piwulf
  used for network booting the Pi's:
  - Overlay templates allow you to output multiple files from a single template
    using the `{{ file }}` function. However, the implementation of the writing
    to these files didn't create the parent directories for these files if they
    didn't exist. (PR 2247)[https://github.com/warewulf/warewulf/pull/2247] to
    the Warewulf project fixes this behavior by creating parent directories of
    all output files.
  - Overlay templates also allow you to create symbolic links from template
    files using the `{{ softlink }}` function, which would typically be paired
    with the `{{ file }}` function to create multiple symlinks at the same time
    to different locations. However, this feature didn't remove existing
    symlinks, so after the first time you built the host overlay, every overlay
    build after that would throw an error about not being able to overwrite the
    existing symlink. (PR 2252)[https://github.com/warewulf/warewulf/pull/2252]
    to the Warewulf project fixes this behavior by deleting the target file
    (usually the existing symlink) before making a new one.
- Since these patches were not merged at the time, they were backported into the
  `ww-picluster.patch` next to the `/etc/warewulf.conf` changes.
- Initially, the picluster project was based on Warewulf v4.5.8, which didn't
  support the `{{ softlink }}` function in template files. Since this was needed
  for linking the kernel and initramfs to `/var/lib/tftpboot`, the project was
  bumped to Warewulf v4.6.0 which was the first version to add `{{ softlink }}`.
- The previous version of picluster didn't include `/opt/warewulf/bin` in the
  `secure_path` for sudo, so the Pi 5 fork added it to make it easier to run
  `sudo wwctl ...` as the `admin` user.
- I had issues with Warewulf prefixing itself to `/opt/warewulf` properly when
  running the `warewulf.sh` script, even when adding `PREFIX` to every `make`
  line. I was able to properly prefix Warewulf by running the `make` commands
  through my own shell with `PREFIX` set for each one. (probably user error)
- The `config.txt` used by the Pi's when booting contain two notable lines:
  - `usb_max_current_enable=1`: raises the USB current limit from 600mA -> 1.6A
  - `arm_boost=1`: slightly boosts the clock of the SoC, without triggering the
    firmware overclocking bit
- There was not a very convenient way to support automatically provisioning the
  Pi when it boots up for the first time. Some ideas included:
  - Making a script to watch the DHCP/TFTP logs and automatically setting the
    MAC address and serial number of unknown Pi's
  - Making a default `config.txt` that boots the Pi into a very slim Linux image
    that registers itself and reboots
  - Intercepting DHCP request packets
- It was determined that manually adding the MAC address and serial number of a
  Pi was easier and more reliable than any of those options. The easiest way to
  determine these two things on a Pi is to plug in a keyboard and display, and
  press the <kbd>Esc</kbd> key on the boot screen. This shows the board info
  on the first line of the screen, in the following format:
  `board: rev c04170 0015 serial 9e92eb36 2c:cf:67:04:40:a1`
  The serial number and MAC address are the last two items in this line,
  respectively. Only the last 4 bytes (8 hex characters) of the serial number
  are needed for boot configuration. Once you have these, you can apply them to
  a node with the following command:
  ```sh
  sudo wwctl node set pi-hpc-compute01 \
    --hwaddr "2c:cf:67:04:40:a1" --tagadd "Serial=9e92eb36"
  ```
- After a serial number is set for one or more nodes, you need to run
  `wwctl configure -a` to rebuild the host overlay and all of the necessary
  boot files in `/var/lib/tftpboot`
- Near the end of testing, it was found out that the TFTP server couldn't follow
  the symbolic links to the `/boot` folder of container images because it was
  chrooted to `/var/lib/tftpboot`. The solution to this was to switch to
  `dnsmasq`, which did support following the symbolic links. Warewulf has
  "experimental support" for `dnsmasq` and the configuration for it already
  existed on the version of Warewulf we were using.
- `dnsmasq` binds to loopback only by default, so a config had to be made to add
  `interface=eth0` for the on-board ethernet port
  - On boot, the head node would show `dnsmasq` as a failed service with a log
    line saying that `eth0` didn't exist, so the config was changed to bind to
    the head node's IP address instead, since it will be able to handle that IP
    roaming between any interface
- For some reason, the container image would lose the GPG key for the AltArch
  repository, so I added a line in the Dockerfile to download it before doing
  any package updates or installs
- The container image installs both the 4k and 16k pagesize kernels for raspi,
  however the install scripts for them seem to only generate the `modules.dep`
  for the currently active kernel, which would be the kernel that the head node
  is currently using. If this is the case (untested on Pi4), then mismatched
  head-compute configurations would require additional work by the user to be
  functional.
