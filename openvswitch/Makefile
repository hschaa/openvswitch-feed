# 
# Copyright (C) 2013 Julius Schulz-Zander
# Copyright (C) 2014 Helmut Schaa
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# $Id: Makefile $

include $(TOPDIR)/rules.mk

PKG_NAME:=openvswitch

PKG_RELEASE:=1
PKG_VERSION:=2.0.1
PKG_RELEASE=$(PKG_SOURCE_VERSION)

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=http://git.openvswitch.org/git/openvswitch
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_VERSION:=f9b9c7a68dc4c5eefb100b8b4d0e024b5ce87f2a
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_FIXUP=libtool

define Package/openvswitch/Default
  SECTION:=net
  CATEGORY:=Network
  URL:=http://www.openvswitchswitch.org/
  DEPENDS:=+libopenssl +librt +libpcap +libatomic
  MAINTAINER:=Julius Schulz-Zander <julius@net.t-labs.tu-berlin.de>
endef

define Package/openvswitch/Default/description
  Open vSwitch is a production quality, multilayer, software-based, Ethernet
  virtual switch. It is designed to enable massive network automation through
  programmatic extension, while still supporting standard management interfaces
  and protocols (e.g. NetFlow, sFlow, SPAN, RSPAN, CLI, LACP, 802.1ag). In
  addition, it is designed to support distribution across multiple physical
  servers similar to VMware's vNetwork distributed vswitch or Cisco's Nexus
  1000V.
endef

define Package/openvswitch-common
  $(call Package/openvswitch/Default)
  TITLE:=Open Flow Switch Userspace Package
  DEPENDS:=+libpcap +libopenssl +librt +kmod-openvswitch
endef

define Package/openvswitch-common/description
  openvswitch-common provides components required by both openvswitch-switch and
  openvswitch-controller.
endef

define Package/openvswitch-ipsec
  $(call Package/openvswitch/Default)
  TITLE:=Open Flow Switch Userspace Package
  DEPENDS:=+openvswitch-common
endef

define Package/openvswitch-ipsec/description
  The ovs-monitor-ipsec script provides support for encrypting GRE tunnels with 
  IPsec.
endef

define Package/openvswitch-benchmark
  $(call Package/openvswitch/Default)
  TITLE:=Open Flow Switch Userspace Package: ovs-benchmark
  DEPENDS:=+openvswitch-common
endef

define Package/openvswitch-benchmark/description
  ovs-benchmark
endef

define Package/openvswitch-controller
  $(call Package/openvswitch/Default)
  TITLE:=Open Flow Switch Userspace Package
  DEPENDS:=+openvswitch-common
endef

define Package/openvswitch-controller/description
  The Open vSwitch controller enables OpenFlow switches that connect to it to
  act as MAC-learning Ethernet switches.
endef

define Package/openvswitch-switch
  $(call Package/openvswitch/Default)
  TITLE:=Open Flow Switch Userspace Package
  DEPENDS:=+openvswitch-common
endef

define Package/openvswitch-switch/description
  openvswitch-switch provides the userspace components and utilities for the
  Open vSwitch kernel-based switch.
endef

define KernelPackage/openvswitch
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Support
  TITLE:=Open Flow Data Path Drivers
  KCONFIG:=CONFIG_BRIDGE
  DEPENDS:=+kmod-stp +kmod-ipv6 +kmod-gre +kmod-lib-crc32c
  FILES:= \
	$(PKG_BUILD_DIR)/datapath/linux/openvswitch.$(LINUX_KMOD_SUFFIX)
  AUTOLOAD:=$(call AutoLoad,21,openvswitch)
endef

define KernelPackage/openvswitch/description
  This package contains the Open vSwitch kernel moodule and bridge compat
  module. Furthermore, it supports OpenFlow.
endef

CONFIGURE_ARGS += --with-linux=$(LINUX_DIR) --with-rundir=/var/run
CONFIGURE_ARGS += --enable-ndebug
CONFIGURE_ARGS += --disable-ssl
CONFIGURE_ARGS += --enable-shared

define Build/Configure
	(cd $(PKG_BUILD_DIR); \
		autoreconf -v --install --force || exit 1 \
	);
	$(call Build/Configure/Default,$(CONFIGURE_ARGS))
endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) \
		$(TARGET_CONFIGURE_OPTS) \
		CFLAGS="-I$(PKG_BUILD_DIR)/lib $(TARGET_CFLAGS) -std=gnu99" \
		LDFLAGS="-L$(PKG_BUILD_DIR)/lib $(TARGET_LDFLAGS)" \
		LDFLAGS_MODULES="$(TARGET_LDFLAGS) -L$(PKG_BUILD_DIR)/lib" \
		STAGING_DIR="$(STAGING_DIR)" \
		DESTDIR="$(PKG_INSTALL_DIR)/usr" \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		ARCH="$(LINUX_KARCH)" \
		SUBDIRS="$(PKG_BUILD_DIR)/datapath/linux" \
		PATH="$(TARGET_PATH)" \
		KCC="$(KERNEL_CC)"
endef

define Package/openvswitch-ipsec/install
	$(INSTALL_DIR) $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/debian/ovs-monitor-ipsec $(1)/usr/sbin/
endef

define Package/openvswitch-benchmark/install
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-benchmark $(1)/usr/bin/
endef

define Package/openvswitch-common/install
	$(INSTALL_DIR) $(1)/etc/openvswitch

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/etc/init.d/openvswitch.init $(1)/etc/init.d/openvswitch

	$(INSTALL_DIR) $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/lib/.libs/libsflow.so*  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/lib/.libs/libopenvswitch.so*  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/ofproto/.libs/libofproto.so*  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/ovsdb/.libs/libovsdb.so*  $(1)/usr/lib/

	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-appctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-ofctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ovsdb/.libs/ovsdb-client $(1)/usr/bin/
endef

define Package/openvswitch-common/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] || /etc/init.d/openvswitch enable || true
endef

define Package/openvswitch-controller/install
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-controller $(1)/usr/bin/
endef

define Package/openvswitch-switch/install
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-dpctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-vsctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ovsdb/.libs/ovsdb-tool $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/vswitchd/.libs/ovs-vswitchd $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ovsdb/.libs/ovsdb-server $(1)/usr/sbin/

	$(INSTALL_DIR) $(1)/usr/share/openvswitch/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/vswitchd/vswitch.ovsschema $(1)/usr/share/openvswitch/
endef

$(eval $(call BuildPackage,openvswitch-ipsec))
$(eval $(call BuildPackage,openvswitch-common))
$(eval $(call BuildPackage,openvswitch-controller))
$(eval $(call BuildPackage,openvswitch-switch))
$(eval $(call BuildPackage,openvswitch-benchmark))
$(eval $(call KernelPackage,openvswitch))

