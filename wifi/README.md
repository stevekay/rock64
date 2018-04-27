## Initial state - not working

Device visible at USB+syslog level but no network device.

    steve@rock64:~$ lsusb
    Bus 005 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 004 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 001 Device 002: ID 0bda:8812 Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac WLAN Adapter
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    steve@rock64:~$ sudo cat /var/log/messages|grep 'usb 2-1:'
    Apr 22 21:08:49 rock64 kernel: [    2.698000] usb 2-1: new high-speed USB device number 2 using ehci-platform
    Apr 22 21:08:49 rock64 kernel: [    2.816865] usb 2-1: New USB device found, idVendor=0bda, idProduct=8812
    Apr 22 21:08:49 rock64 kernel: [    2.821341] usb 2-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    Apr 22 21:08:49 rock64 kernel: [    2.825735] usb 2-1: Product: 802.11n NIC
    Apr 22 21:08:49 rock64 kernel: [    2.829803] usb 2-1: Manufacturer: Realtek
    Apr 22 21:08:49 rock64 kernel: [    2.833854] usb 2-1: SerialNumber: 123456
    Apr 22 21:14:13 rock64 kernel: [  316.263362] usb 2-1: USB disconnect, device number 2
    steve@rock64:~$ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 7e:63:18:24:14:fd brd ff:ff:ff:ff:ff:ff
        inet 192.168.0.28/24 brd 192.168.0.255 scope global eth0
           valid_lft forever preferred_lft forever
    steve@rock64:~$ ip link
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
        link/ether 7e:63:18:24:14:fd brd ff:ff:ff:ff:ff:ff
    steve@rock64:~$ sudo lshw -c network
      *-network
           description: Ethernet interface
           physical id: 6
           logical name: eth0
           serial: 7e:63:18:24:14:fd
           size: 1Gbit/s
           capacity: 1Gbit/s
           capabilities: ethernet physical tp aui bnc mii fibre 10bt 10bt-fd 100bt 100bt-fd 1000bt-fd autonegotiation
           configuration: autonegotiation=on broadcast=yes driver=st_gmac driverversion=March_2013 duplex=full ip=192.168.0.28 link=yes multicast=yes port=MII speed=1Gbit/s
    steve@rock64:~$

## Download + compile driver

Using notes from https://forum.pine64.org/showthread.php?tid=5768&pid=35964#pid35964.

### Download git repo

    steve@rock64:~$ git clone https://github.com/gnab/rtl8812au.git
    Cloning into 'rtl8812au'...
    remote: Counting objects: 607, done.
    remote: Total 607 (delta 0), reused 0 (delta 0), pack-reused 607
    Receiving objects: 100% (607/607), 1.80 MiB | 1.13 MiB/s, done.
    Resolving deltas: 100% (244/244), done.
    steve@rock64:~$

### Revise makefile

    steve@rock64:~$ cd rtl8812au
    steve@rock64:~/rtl8812au$ vi Makefile
    steve@rock64:~/rtl8812au$ git diff
    diff --git a/Makefile b/Makefile
    index 71d0660..1a24035 100644
    --- a/Makefile
    +++ b/Makefile
    @@ -37,7 +37,7 @@ CONFIG_SDIO_HCI = n
     CONFIG_GSPI_HCI = n
    
     CONFIG_MP_INCLUDED = y
    -CONFIG_POWER_SAVING = y
    +CONFIG_POWER_SAVING = n
     CONFIG_USB_AUTOSUSPEND = n
     CONFIG_HW_PWRP_DETECTION = n
     CONFIG_WIFI_TEST = n
    @@ -50,7 +50,8 @@ CONFIG_EXT_CLK = n
     CONFIG_FTP_PROTECT = n
     CONFIG_WOWLAN = n
    
    -CONFIG_PLATFORM_I386_PC = y
    +CONFIG_PLATFORM_I386_PC = n
    +CONFIG_PLATFORM_ARM_PC = y
     CONFIG_PLATFORM_ANDROID_X86 = n
     CONFIG_PLATFORM_JB_X86 = n
     CONFIG_PLATFORM_ARM_S3C2K4 = n
    @@ -658,6 +659,17 @@ MODDESTDIR := /lib/modules/$(KVER)/kernel/drivers/net/wireless/
     INSTALL_PREFIX :=
     endif
    
    +ifeq ($(CONFIG_PLATFORM_ARM_PC), y)
    +EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN
    +SUBARCH := arm64
    +ARCH ?= arm64
    +CROSS_COMPILE ?=
    +KVER ?= $(shell uname -r)
    +KSRC := /lib/modules/$(KVER)/build/
    +MODDESTDIR := /lib/modules/$(KVER)/kernel/drivers/net/wireless/
    +INSTALL_PREFIX :=
    +endif
    +
     ifeq ($(CONFIG_PLATFORM_ACTIONS_ATM702X), y)
     EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN -DCONFIG_PLATFORM_ANDROID -DCONFIG_PLATFORM_ACTIONS_ATM702X
     #ARCH := arm
    @@ -1049,7 +1061,7 @@ export CONFIG_RTL8812AU_8821AU = m
     all: modules
    
     modules:
    -       $(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) -C $(KSRC) M=$(shell pwd)  modules
    +       $(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) -C $(KSRC) M=$(shell pwd)
    
     strip:
            $(CROSS_COMPILE)strip $(MODULE_NAME).ko --strip-unneeded
    steve@rock64:~/rtl8812au$

## Compile

    steve@rock64:~/rtl8812au$ make
    make ARCH=arm64 CROSS_COMPILE= -C /lib/modules/4.4.77-rockchip-ayufan-136/build/ M=/home/steve/rtl8812au
    make[1]: Entering directory '/usr/src/linux-headers-4.4.77-rockchip-ayufan-136'
      LD      /home/steve/rtl8812au/built-in.o
      CC [M]  /home/steve/rtl8812au/core/rtw_cmd.o
      CC [M]  /home/steve/rtl8812au/core/rtw_security.o
      CC [M]  /home/steve/rtl8812au/core/rtw_debug.o
      CC [M]  /home/steve/rtl8812au/core/rtw_io.o
      CC [M]  /home/steve/rtl8812au/core/rtw_ioctl_query.o
      CC [M]  /home/steve/rtl8812au/core/rtw_ioctl_set.o
      CC [M]  /home/steve/rtl8812au/core/rtw_ieee80211.o
      CC [M]  /home/steve/rtl8812au/core/rtw_mlme.o
      CC [M]  /home/steve/rtl8812au/core/rtw_mlme_ext.o
    [...]
      CC [M]  /home/steve/rtl8812au/core/rtw_mp.o
      CC [M]  /home/steve/rtl8812au/core/rtw_mp_ioctl.o
      LD [M]  /home/steve/rtl8812au/8812au.o
      Building modules, stage 2.
      MODPOST 1 modules
      CC      /home/steve/rtl8812au/8812au.mod.o
      LD [M]  /home/steve/rtl8812au/8812au.ko
    make[1]: Leaving directory '/usr/src/linux-headers-4.4.77-rockchip-ayufan-136'
    steve@rock64:~/rtl8812au$

## Load the newly compiled module, confirm NIC now detected

    steve@rock64:~/rtl8812au$ sudo insmod 8812au.ko
    steve@rock64:~/rtl8812au$ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 7e:63:18:24:14:fd brd ff:ff:ff:ff:ff:ff
        inet 192.168.0.28/24 brd 192.168.0.255 scope global eth0
           valid_lft forever preferred_lft forever
    3: enx24050f9e7faa: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
        link/ether 24:05:0f:9e:7f:aa brd ff:ff:ff:ff:ff:ff
    steve@rock64:~/rtl8812au$

## Review syslog

    steve@rock64:~/rtl8812au$ sudo cat /var/log/messages|tail -5
    Apr 23 20:42:02 rock64 kernel: [   26.392280] rk_gmac-dwmac ff540000.eth eth0: Link is Up - 1Gbps/Full - flow control rx/tx
    Apr 23 20:42:02 rock64 kernel: [   26.393217] IPv6: ADDRCONF(NETDEV_CHANGE): eth0: link becomes ready
    Apr 23 21:17:55 rock64 kernel: [ 2160.600017] usbcore: registered new interface driver rtl8812au
    Apr 23 21:17:55 rock64 kernel: [ 2160.621955] rtl8812au 1-1:1.0 enx24050f9e7faa: renamed from wlan0
    Apr 23 21:17:56 rock64 kernel: [ 2161.242391] IPv6: ADDRCONF(NETDEV_UP): enx24050f9e7faa: link is not ready
    steve@rock64:~/rtl8812au$

## Install, so loads on boot

    steve@rock64:~/rtl8812au$ sudo make install
    install -p -m 644 8812au.ko  /lib/modules/4.4.77-rockchip-ayufan-136/kernel/drivers/net/wireless/
    /sbin/depmod -a 4.4.77-rockchip-ayufan-136
    steve@rock64:~/rtl8812au$

## Reboot, to test

    steve@rock64:~/rtl8812au$ sudo reboot

## Check after reboot

    steve@rock64:~$ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 7e:63:18:24:14:fd brd ff:ff:ff:ff:ff:ff
        inet 192.168.0.28/24 brd 192.168.0.255 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::7c63:18ff:fe24:14fd/64 scope link
           valid_lft forever preferred_lft forever
    3: enx24050f9e7faa: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
        link/ether 24:05:0f:9e:7f:aa brd ff:ff:ff:ff:ff:ff
    steve@rock64:~$

## Set SSID

    steve@rock64:~$ sudo ifconfig enx24050f9e7faa up
    steve@rock64:~$ sudo iwlist enx24050f9e7faa scan | grep ESSID
                        ESSID:"MyNetwork"
    steve@rock64:~$ sudo vi /etc/network/interfaces.d/enx24050f9e7faa
    steve@rock64:~$ cat /etc/network/interfaces.d/enx24050f9e7faa
    auto enx24050f9e7faa
    iface enx24050f9e7faa inet dhcp
             wpa-ssid MyNetwork
             wpa-psk mypassword
    steve@rock64:~$ 

## TODO

- add dkms commands, so module rebuilds if a new kernel arrives
