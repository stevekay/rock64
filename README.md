# rock64
ROCK64 4K60P HDR Media Board Computer

## Install

### Run the [installer](https://github.com/pine64dev/PINE64-Installer/blob/master/README.md) to write an OS image to a microSD card.

* Select 'Choose an OS'
![Install](/images/img001.jpg?raw=true "img001")

* Select 'ROCK64'
![Install](/images/img002.jpg?raw=true "img002")

* Select OS, in this case 'Community Debian Stretch Minimal (ROCK64 microSD)'
![Install](/images/img003.jpg?raw=true "img003")

* Select drive having a microSD card, to write the image to.
![Install](/images/img004.jpg?raw=true "img004")
![Install](/images/img005.jpg?raw=true "img005")

* Select 'Flash!'
![Install](/images/img006.jpg?raw=true "img006")
![Install](/images/img007.jpg?raw=true "img007")

* On completion, reports default username/password, rock64/rock64.
![Install](/images/img008.jpg?raw=true "img008")
![Install](/images/img009.jpg?raw=true "img009")

### Boot from the newly written image on microSD

* Insert microSD into ROCK64, power it on.

* Use PuTTY to logon as user 'rock64'

```shell
login as: rock64
rock64@192.168.0.20's password:
                _     __   _  _
 _ __ ___   ___| | __/ /_ | || |
| '__/ _ \ / __| |/ / '_ \| || |_
| | | (_) | (__|   <| (_) |__   _|
|_|  \___/ \___|_|\_\\___/   |_|

Linux rock64 4.4.77-rockchip-ayufan-118 #1 SMP Thu Sep 14 21:59:24 UTC 2017 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Sep 14 22:17:42 2017 from 192.168.0.12
rock64@rock64:~$
```

## Config

### Kernel version

```shell
rock64@rock64:~# uname -r
4.4.77-rockchip-ayufan-118
rock64@rock64:~# 
```

### Disk space

```shell
rock64@rock64:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           393M   16M  378M   4% /run
/dev/mmcblk1p7  7.1G  858M  5.9G  13% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mmcblk1p6  100M   41M   60M  41% /boot/efi
tmpfs           393M     0  393M   0% /run/user/1000
rock64@rock64:~$ sudo fdisk -l /dev/mmcblk1
GPT PMBR size mismatch (4458495 != 15415295) will be corrected by w(rite).
Disk /dev/mmcblk1: 7.4 GiB, 7892631552 bytes, 15415296 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 7B13C7E1-5285-472B-8602-41D21798F780

Device          Start      End  Sectors  Size Type
/dev/mmcblk1p1     64     8063     8000  3.9M Linux filesystem
/dev/mmcblk1p2   8064     8191      128   64K Linux filesystem
/dev/mmcblk1p3   8192    16383     8192    4M Linux filesystem
/dev/mmcblk1p4  16384    24575     8192    4M Linux filesystem
/dev/mmcblk1p5  24576    32767     8192    4M Linux filesystem
/dev/mmcblk1p6  32768   262143   229376  112M Microsoft basic data
/dev/mmcblk1p7 262144 15415262 15153119  7.2G Linux filesystem
rock64@rock64:~$
```

### CPU

rock64@rock64:~$ lscpu
Architecture:          aarch64
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    4
Socket(s):             1
Model:                 4
CPU max MHz:           1296.0000
CPU min MHz:           408.0000
BogoMIPS:              48.00
Flags:                 fp asimd evtstrm aes pmull sha1 sha2 crc32
rock64@rock64:~$

### Memory

```shell
rock64@rock64:~$ free -m
              total        used        free      shared  buff/cache   available
Mem:           3925          49        3722          23         153        3823
Swap:             0           0           0
rock64@rock64:~$
```

### Network

```shell
rock64@rock64:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 7e:63:18:24:14:fd brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.20/24 brd 192.168.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::7c63:18ff:fe24:14fd/64 scope link
       valid_lft forever preferred_lft forever
rock64@rock64:~$
```

### Network - wifi

```shell
rock64@rock64:~$ sudo modprobe 8188eu
rock64@rock64:~$ tail /var/log/messages
Nov 17 22:56:16 rock64 kernel: [17439.734032] =======================================================
Nov 17 22:56:16 rock64 kernel: [17439.734699] ==== Launching Wi-Fi driver! (Powered by Rockchip) ====
Nov 17 22:56:16 rock64 kernel: [17439.735429] =======================================================
Nov 17 22:56:16 rock64 kernel: [17439.736113] Realtek 8188EU USB WiFi driver (Powered by Rockchip,Ver 2.20.WFD) init.
Nov 17 22:56:16 rock64 kernel: [17439.736965] [WLAN_RFKILL]: rockchip_wifi_power: 1
Nov 17 22:56:16 rock64 kernel: [17439.737472] [WLAN_RFKILL]: rockchip_wifi_power: rfkill-wlan driver has not Successful initialized
Nov 17 22:56:16 rock64 kernel: [17439.738422] RTL871X: module init start
Nov 17 22:56:16 rock64 kernel: [17439.738830] RTL871X: rtl8188eu v4.3.24_16705.20160509
Nov 17 22:56:16 rock64 kernel: [17439.739571] usbcore: registered new interface driver rtl8188eu
Nov 17 22:56:16 rock64 kernel: [17439.740275] RTL871X: module init ret=0
rock64@rock64:~$
```

### Update

```shell
rock64@rock64:~$ sudo apt-get update
Get:1 http://ppa.launchpad.net/ayufan/rock64-ppa/ubuntu xenial InRelease [18.0 kB]
Get:3 http://security.debian.org stretch/updates InRelease [63.0 kB]
Get:5 http://deb.ayufan.eu/orgs/ayufan-rock64/releases  InRelease [1245 B]
Ign:2 http://cdn-fastly.deb.debian.org/debian stretch InRelease
Get:4 http://cdn-fastly.deb.debian.org/debian stretch-updates InRelease [91.0 kB]
...
Get:28 http://cdn-fastly.deb.debian.org/debian stretch/main Translation-en [5395 kB]
Get:29 http://cdn-fastly.deb.debian.org/debian stretch/contrib arm64 Packages [39.9 kB]
Get:30 http://cdn-fastly.deb.debian.org/debian stretch/non-free Translation-en [79.2 kB]
Fetched 20.1 MB in 15s (1332 kB/s)
Reading package lists... Done
rock64@rock64:~$ sudo apt-get upgrade
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  apt apt-transport-https apt-utils base-files curl dbus dirmngr git git-man gnupg gnupg-agent gpgv libapt-inst2.0 libapt-pkg5.0 libcurl3 libcurl3-gnutls libdb5.3 libdbus-1-3 libgnutls30
  libgssapi-krb5-2 libhogweed4 libidn2-0 libk5crypto3 libkrb5-3 libkrb5support0 libldap-2.4-2 libldap-common libncurses5 libncursesw5 libnettle6 libperl5.24 libselinux1 libssl1.0.2 libssl1.1
  libtinfo5 linux-libc-dev ncurses-base ncurses-bin ntp openssl perl perl-base perl-modules-5.24 tzdata vim vim-common vim-runtime vim-tiny wget wpasupplicant xxd
51 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 36.2 MB of archives.
After this operation, 171 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
...
Setting up git (1:2.11.0-3+deb9u2) ...
Setting up apt-transport-https (1.4.8) ...
Setting up curl (7.52.1-5+deb9u2) ...
Processing triggers for libc-bin (2.24-11+deb9u1) ...
rock64@rock64:~$
```


## Links
* (Specs)[https://www.pine64.org/?page_id=7147]


