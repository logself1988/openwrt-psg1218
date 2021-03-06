
BUILDER ?= OpenWrt-ImageBuilder-15.05.1-ramips-mt7620.Linux-x86_64
SDK ?= OpenWrt-SDK-15.05.1-ramips-mt7620_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64

OPKG_PACKAGES :=  6in4 6to4 curl ethtool ip ip6tables-extra ip6tables-mod-nat \
	ipset iptables-mod-conntrack-extra iptables-mod-extra iptables-mod-ipopt iwinfo \
	iptables-mod-nat-extra kmod-crypto-des kmod-crypto-ecb kmod-crypto-hmac kmod-crypto-manager \
	kmod-crypto-md4 kmod-crypto-md5 kmod-crypto-pcompress kmod-crypto-sha1 kmod-crypto-sha256 \
	kmod-dnsresolver kmod-gre kmod-ipt-filter\
	kmod-ip6-tunnel kmod-ip6tables-extra kmod-ipip kmod-ipt-conntrack-extra kmod-ipt-extra \
	kmod-ipt-ipopt kmod-ipt-ipset kmod-ipt-nat-extra kmod-ipt-nat6 kmod-iptunnel kmod-iptunnel4 \
	kmod-iptunnel6 kmod-leds-gpio kmod-ledtrig-default-on kmod-ledtrig-netdev kmod-lib-zlib \
	kmod-ledtrig-timer  kmod-lib-textsearch kmod-macvlan kmod-mppe \
	kmod-nfnetlink kmod-pptp kmod-sit kmod-tun libcurl libdaemon \
	libevent2 liblzo libmnl libncurses libopenssl libpthread  \
	luci-lib-json luci-proto-ipv6 luci-proto-relay openvpn-openssl resolveip \
	terminfo zlib luci-i18n-base-zh-cn luci-i18n-commands-zh-cn \
	luci-i18n-diag-core-zh-cn luci-i18n-firewall-zh-cn luci-i18n-qos-zh-cn  \
	ppp-mod-pptp ppp-mod-pppol2tp kmod-nls-cp437 kmod-nls-iso8859-1 kmod-nls-utf8 \
	kmod-crypto-deflate  kmod-nf-nathelper kmod-nf-nathelper-extra \
	luci-app-ddns ddns-scripts luci-app-upnp miniupnpd luci-app-mwan3 mwan3

USB_PACKAGES := blkid fdisk kmod-fuse kmod-fs-cifs kmod-fs-ext4 kmod-fs-vfat kmod-usb-storage kmod-ledtrig-usbdev kmod-usb-ohci kmod-usb-printer \
	ntfs-3g libblkid luci-app-samba luci-i18n-samba-zh-cn samba36-server e2fsprogs mountd block-mount 

OPKG_PACKAGES_HC5661 :=
OPKG_PACKAGES_HC5761 :=
OPKG_PACKAGES_HC5861 :=
PREPARED_TARGETS = $(BUILDER) $(SDK) .check_ib .patched

# A single option for enabling all options
ifeq ($(FULL),1)
	RALINK := 1
	FEEDS := 1
	WIFI := 1
endif
# Check each option for selection of packages and dependencies
ifeq ($(RALINK),1)
	OPKG_PACKAGES_HC5761 += 8021xd uci2dat kmod-mt7610e luci-mtk-wifi
	OPKG_PACKAGES_HC5861 += 8021xd uci2dat kmod-mt76x2e luci-mtk-wifi
	PREPARED_TARGETS += .check_sdk .ralink
else
	OPKG_PACKAGES_HC5761 += kmod-mt76
	OPKG_PACKAGES_HC5861 += kmod-mt76
	PREPARED_TARGETS += .check_sdk .ralink
endif
ifeq ($(FEEDS),1)
	OPKG_PACKAGES += ipset-lists minivtun shadowsocks-libev shadowsocks-tools dnsmasq-full \
		dnspod-utils kmod-proto-bridge kmod-yavlan
	USB_PACKAGES += file-storage
	PREPARED_TARGETS += .check_sdk .feeds
endif

define BeforeBuildImage
	mkdir -p $(BUILDER)/dl
	cp -f repo-base.conf $(BUILDER)/repositories.conf
	@[ -n "$(SDK)" -a -f "$(SDK)"/bin/ramips/packages/Packages ] && \
		echo "src ralink file:$(shell cd $(SDK)/bin/ramips/packages; pwd)" >> $(BUILDER)/repositories.conf || :
	mkdir -p $(BUILDER)/target/linux/ramips/base-files/etc
	cp -f opkg.conf $(BUILDER)/target/linux/ramips/base-files/etc/opkg.conf
endef

define EnableWireless
	@[ -n "$(WIFI)" ] || exit 0; \
		F=$(shell echo $(1)); openwrt-repack.sh $$F -w -o $$F.out && mv $$F.out $$F
endef

all: HC5661 HC5761 HC5861 XIAOMI-MIWIFI-MINI

HC5661: $(PREPARED_TARGETS)
	$(call BeforeBuildImage)
	make -C $(BUILDER) image PROFILE=HC5661 \
		FILES="$(shell cd $(BUILDER); pwd)/target/linux/ramips/base-files" \
		PACKAGES="$(OPKG_PACKAGES) $(USB_PACKAGES) $(OPKG_PACKAGES_HC5661)"
	$(call EnableWireless,$(shell echo $(BUILDER)/bin/ramips/openwrt-*-hc5661-*-sysupgrade.bin))

HC5761: $(PREPARED_TARGETS)
	$(call BeforeBuildImage)
	make -C $(BUILDER) image PROFILE=HC5761 \
		FILES="$(shell cd $(BUILDER); pwd)/target/linux/ramips/base-files" \
		PACKAGES="$(OPKG_PACKAGES) $(USB_PACKAGES) $(OPKG_PACKAGES_HC5761)"
	$(call EnableWireless,$(shell echo $(BUILDER)/bin/ramips/openwrt-*-hc5761-*-sysupgrade.bin))

HC5861: $(PREPARED_TARGETS)
	$(call BeforeBuildImage)
	make -C $(BUILDER) image PROFILE=HC5861 \
		FILES="$(shell cd $(BUILDER); pwd)/target/linux/ramips/base-files" \
		PACKAGES="$(OPKG_PACKAGES) $(USB_PACKAGES) $(OPKG_PACKAGES_HC5861)"
	$(call EnableWireless,$(shell echo $(BUILDER)/bin/ramips/openwrt-*-hc5861-*-sysupgrade.bin))

XIAOMI-MIWIFI-MINI: $(PREPARED_TARGETS)
	$(call BeforeBuildImage)
	make -C $(BUILDER) image PROFILE=XIAOMI-MIWIFI-MINI \
		FILES="$(shell cd $(BUILDER); pwd)/target/linux/ramips/base-files" \
		PACKAGES="$(OPKG_PACKAGES) $(OPKG_PACKAGES_HC5761)"
	$(call EnableWireless,$(shell echo $(BUILDER)/bin/ramips/openwrt-*-xiaomi-miwifi-mini-*-sysupgrade.bin))

PSG1218: $(PREPARED_TARGETS)
	$(call BeforeBuildImage)
	cp ../patches/PSG1218.dts  OpenWrt-ImageBuilder-15.05.1-ramips-mt7620.Linux-x86_64/target/linux/ramips/dts/
	make -C $(BUILDER) image PROFILE=PSG1218 \
		FILES="$(shell cd $(BUILDER); pwd)/target/linux/ramips/base-files" \
		PACKAGES="$(OPKG_PACKAGES) $(OPKG_PACKAGES_HC5761)"
	$(call EnableWireless,$(shell echo $(BUILDER)/bin/ramips/openwrt-*-gehua-*-sysupgrade.bin))

GEHUA: $(PREPARED_TARGETS)
	$(call BeforeBuildImage)
	cp ../patches/GEHUA.dts  OpenWrt-ImageBuilder-15.05.1-ramips-mt7620.Linux-x86_64/target/linux/ramips/dts/
	make -C $(BUILDER) image PROFILE=GEHUA \
		FILES="$(shell cd $(BUILDER); pwd)/target/linux/ramips/base-files" \
		PACKAGES="$(OPKG_PACKAGES) $(USB_PACKAGES) $(OPKG_PACKAGES_HC5661)"
	$(call EnableWireless,$(shell echo $(BUILDER)/bin/ramips/openwrt-*-hc5661-*-sysupgrade.bin))

.patched:
	cat ../patches/01-hiwifi-hc5x61.patch ../patches/12-disable-pdnsd.patch > .patching
	patch -d $(BUILDER) -p0 < .patching
	mv .patching .patched

.ralink:
	@cd $(SDK); [ ! -L dl -a -d /var/dl ] && { rmdir dl && ln -s /var/dl; } || :
	@cd $(SDK)/package; [ -d ralink ] || ln -sv $(shell cd ..; pwd)/packages/ralink
	make package/8021xd/compile V=s -C "$(SDK)"
	make package/uci2dat/compile V=s -C "$(SDK)"
	make package/mt7610e/compile V=s -C "$(SDK)"
	make package/mt76x2e/compile V=s -C "$(SDK)"
	make package/luci-mtk-wifi/compile V=s -C "$(SDK)"
	cd "$(SDK)/bin/ramips/packages" && ../../../scripts/ipkg-make-index.sh . > Packages && gzip -9c Packages > Packages.gz

.feeds:
	@cd $(SDK); [ ! -L dl -a -d /var/dl ] && { rmdir dl && ln -s /var/dl; } || :
	@cd $(SDK)/package; [ -d network-feeds ] && { cd network-feeds; git pull; } || git clone https://github.com/boytm/network-feeds.git
	make package/ipset-lists/compile V=s -C "$(SDK)"
	make package/shadowsocks-libev/compile V=s -C "$(SDK)"
	make package/shadowsocks-tools/compile V=s -C "$(SDK)"
	make package/minivtun-tools/compile V=s -C "$(SDK)"
	make package/file-storage/compile V=s -C "$(SDK)"
	make package/dnspod-utils/compile V=s -C "$(SDK)"
	make package/proto-bridge/compile V=s -C "$(SDK)"
	make package/pdnsd/compile V=s -C "$(SDK)"
	#make package/mproxy/compile V=s -C "$(SDK)"
	cd "$(SDK)/bin/ramips/packages" && ../../../scripts/ipkg-make-index.sh . > Packages && gzip -9c Packages > Packages.gz

.check_ib:
	@if ! [ -n "$(BUILDER)" -a -d "$(BUILDER)" ]; then \
		echo "Please specify a valid ImageBuilder directory by adding \"BUILDER=...\"."; \
		echo "Type \"make help\" for more details."; \
		exit 1; \
	fi
.check_sdk:
	@if ! [ -n "$(SDK)" -a -d "$(SDK)/package" ]; then \
		echo "Please specify a valid OpenWrt SDK directory by adding \"SDK=...\"."; \
		echo "Type \"make help\" for more details."; \
		exit 1; \
	fi

# Try extracting ImageBuilder & SDK to current directory
$(BUILDER):
	tar jxvf /var/dl/$(BUILDER).tar.bz2
$(SDK):
	tar jxvf /var/dl/$(SDK).tar.bz2

help:
	@echo "Usage:"
	@echo "  make BUILDER=.... [RALINK=1] [FEEDS=1]     build OpenWrt firmware for this board"
	@echo "Options:"
	@echo "  BUILDER=<directory>        specify a valid ImageBuilder directory"
	@echo "  SDK=<directory>            specify a valid OpenWrt SDK directory"
	@echo "  RALINK=1                   build and install Ralink 5G drivers"
	@echo "  FEEDS=1                    build and install Shadowsocks, minivtun, kmod-proto-bridge, kmod-yavlan"
	@echo "  FULL=1                     enable both the options above"

clean: .check_ib
	make clean -C $(BUILDER)
	@if [ -e .patched ]; then \
		patch -R -d $(BUILDER) -p0 < .patched; \
		rm -vf .patched; \
	fi
	[ -n "$(SDK)" -a -d "$(SDK)"/bin/ramips ] && rm -rf "$(SDK)"/bin/ramips/* || :
