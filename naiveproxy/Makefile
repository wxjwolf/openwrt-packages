#
# Copyright (C) 2020 Project OpenWrt
#
# This is free software, licensed under the GNU General Public License v3.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk

PKG_NAME:=naiveproxy
PKG_VERSION:=86.0.4240.75-1
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/klzgrad/naiveproxy/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=91f946e137565115649bc5787788a6889d84e0a20e9878553a832b89c0cb161f
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

PKG_LICENSE:=BSD 3-Clause
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=CN_SZTL <cnsztl@project-openwrt.eu.org>

PKG_BUILD_DEPENDS:=ninja/host python3/host
PKG_USE_MIPS16:=0
PKG_BUILD_PARALLEL:=1

ifneq ($(CONFIG_CPU_TYPE)," ")
	CPU_TYPE:=$(word 1, $(subst +," ,$(CONFIG_CPU_TYPE)))
	CPU_SUBTYPE:=$(word 2, $(subst +, ",$(CONFIG_CPU_TYPE)))
	ifeq ($(CPU_SUBTYPE),)
		CPU_SUBTYPE:=""
	endif
else
	CPU_TYPE:=""
	CPU_SUBTYPE:=""
endif

include $(INCLUDE_DIR)/package.mk

define Package/naiveproxy/config
	depends on !(arc||armeb||mips||mips64||powerpc)
endef

define Package/naiveproxy
	SECTION:=net
	CATEGORY:=Network
	SUBMENU:=Web Servers/Proxies
	URL:=https://github.com/klzgrad/naiveproxy
	TITLE:=Make a fortune quietly
	DEPENDS:=+libatomic +libnss
endef

define Package/naiveproxy/description
	NaïveProxy uses Chrome's network stack to camouflage traffic with strong
	censorship resistance and low detectability. Reusing Chrome's stack also
	ensures best practices in performance and security.
endef

ifneq ($(CONFIG_CCACHE),)
	export CCACHE_SLOPPINESS=time_macros
	export CCACHE_BASEDIR=$(PKG_BUILD_DIR)/src
	export CCACHE_CPP2=yes
	export naive_ccache_flags=cc_wrapper="ccache"
endif

define Build/Compile
( \
	cd $(PKG_BUILD_DIR) ; \
	./tools/import-upstream.sh ; \
	. ./init_env.sh "$(ARCH)" "$(BOARD)" $(CPU_TYPE) $(CPU_SUBTYPE) "$(TOOLCHAIN_DIR)" "$(DL_DIR)"; \
	export naive_flags="$$$${naive_flags} $$$${naive_ccache_flags}" ; \
	export OP_STAGING_DIR="$(STAGING_DIR)" ; \
	mkdir -p out ; \
	./gn/out/gn gen "out/Release" --args="$$$${naive_flags}" --script-executable="$(STAGING_DIR_HOSTPKG)/bin/python3" ; \
	$(STAGING_DIR_HOSTPKG)/bin/ninja -C "out/Release" naive ; \
)
endef

define Package/naiveproxy/conffiles
endef

define Package/naiveproxy/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/src/out/Release/naive $(1)/usr/bin/naive
endef

$(eval $(call BuildPackage,naiveproxy))
