# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2023 ImmortalWrt.org

include $(TOPDIR)/rules.mk

PKG_NAME:=dae
PKG_VERSION:=stable
PKG_RELEASE:=1

PKG_VERSION_NORMALIZED := $(subst _,,$(PKG_VERSION))
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION_NORMALIZED).zip
PKG_SOURCE_URL:=https://github.com/daeuniverse/dae/releases/download/v$(PKG_VERSION_NORMALIZED)/dae-full-src.zip?
PKG_HASH:=skip

PKG_LICENSE:=AGPL-3.0-only
PKG_LICENSE_FILE:=LICENSE
PKG_MAINTAINER:=Tianling Shen <cnsztl@immortalwrt.org>

PKG_BUILD_DEPENDS:=golang/host bpf-headers
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=github.com/daeuniverse/dae
GO_PKG_LDFLAGS_X:= \
	$(GO_PKG)/cmd.Version=$(PKG_VERSION) \
	$(GO_PKG)/common/consts.MaxMatchSetLen_=1024
GO_PKG_TAGS:=trace

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/bpf.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

# Architecture mapping - Must be defined before Build/Compile
ifeq ($(ARCH),aarch64)
  GO_ARCH:=arm64
else ifeq ($(ARCH),x86_64)
  GO_ARCH:=amd64
else ifeq ($(ARCH),arm)
  GO_ARCH:=arm
else ifeq ($(ARCH),mips64el)
  GO_ARCH:=mips64le
else ifeq ($(ARCH),mips64)
  GO_ARCH:=mips64
else ifeq ($(ARCH),mipsel)
  GO_ARCH:=mipsle
else ifeq ($(ARCH),mips)
  GO_ARCH:=mips
else ifeq ($(ARCH),i386)
  GO_ARCH:=386
else ifeq ($(ARCH),riscv64)
  GO_ARCH:=riscv64
else
  GO_ARCH:=$(ARCH)
endif

UNZIP_CMD:=unzip -q -d $(PKG_BUILD_DIR) $(DL_DIR)/$(PKG_SOURCE)

define Package/dae/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  URL:=https://github.com/daeuniverse/dae
endef

define Package/dae
  $(call Package/dae/Default)
  TITLE:=A lightweight and high-performance transparent proxy solution
  # You need enable KERNEL_DEBUG_INFO_BTF and KERNEL_BPF_EVENTS
  DEPENDS:=$(GO_ARCH_DEPENDS) $(BPF_DEPENDS)  \
    +ca-bundle +kmod-sched-core +kmod-sched-bpf +kmod-xdp-sockets-diag \
    +kmod-veth
endef

define Package/dae-geoip
  $(call Package/dae/Default)
  TITLE:=geoip for dae
  DEPENDS:=+dae +v2ray-geoip
  PKGARCH:=all
endef

define Package/dae-geosite
  $(call Package/dae/Default)
  TITLE:=geosite for dae
  DEPENDS:=+dae +v2ray-geosite
  PKGARCH:=all
endef

define Package/dae/description
  dae, means goose, is a lightweight and high-performance transparent
  proxy solution.

  In order to improve the traffic diversion performance as much as possible,
  dae runs the transparent proxy and traffic diversion suite in the linux
  kernel by eBPF. Therefore, we have the opportunity to make the direct
  traffic bypass the forwarding by proxy application and achieve true direct
  traffic through. Under such a magic trick, there is almost no performance
  loss and additional resource consumption for direct traffic.
endef

define Package/dae/conffiles
/etc/dae/config.dae
/etc/config/dae
endef

DAE_CFLAGS:= \
	-O2 -Wall -Werror \
	-DMAX_MATCH_SET_LEN=1024 \
	-I$(BPF_HEADERS_DIR)/tools/lib \
	-I$(BPF_HEADERS_DIR)/arch/$(BPF_KARCH)/include/asm/mach-generic

define Build/Prepare
	# only unzip if the build dir does not already exist
	if [ ! -d "$(PKG_BUILD_DIR)" ] || [ -z "$$(ls -A $(PKG_BUILD_DIR))" ]; then \
		mkdir -p $(PKG_BUILD_DIR); \
		$(UNZIP_CMD); \
	fi
endef

define Build/Compile
	cd $(PKG_BUILD_DIR) && \
	\
	export GOOS=linux && \
	export GOARCH=$(GO_ARCH) && \
	export CGO_ENABLED=1 && \
	export GO111MODULE=on && \
	\
	$(if $(TARGET_CC),export CC=$(TARGET_CC);) \
	$(if $(TARGET_CXX),export CXX=$(TARGET_CXX);) \
	\
	export BPF_CLANG="$(CLANG)" && \
	export BPF_STRIP_FLAG="-strip=$(LLVM_STRIP)" && \
	export BPF_CFLAGS="$(DAE_CFLAGS)" && \
	export BPF_TARGET="bpfel,bpfeb" && \
	export BPF_TRACE_TARGET="$(GO_ARCH)" && \
	\
	( [ -f go.mod ] || go mod init dae ) && \
	go mod tidy -compat=1.21 && \
	\
	go mod download && \
	\
	echo ">>> Running go generate..." && \
	go generate ./... && \
	\
	echo ">>> Go build start for $(GO_ARCH) with CGO enabled..." && \
	go build -v -trimpath \
		-ldflags="$(GO_PKG_LDFLAGS_X) -s -w" \
		-tags "$(GO_PKG_TAGS)" \
		-o $(PKG_BUILD_DIR)/dae \
		. && \
	\
	echo ">>> Build completed successfully"
endef

define Package/dae/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/dae $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/etc/dae/
	$(INSTALL_CONF) $(PKG_BUILD_DIR)/example.dae $(1)/etc/dae/

	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) $(CURDIR)/files/dae.config $(1)/etc/config/dae

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(CURDIR)/files/dae.init $(1)/etc/init.d/dae
endef

define Package/dae-geoip/install
	$(INSTALL_DIR) $(1)/usr/share/dae
	$(LN) ../v2ray/geoip.dat $(1)/usr/share/dae/geoip.dat
endef

define Package/dae-geosite/install
	$(INSTALL_DIR) $(1)/usr/share/dae
	$(LN) ../v2ray/geosite.dat $(1)/usr/share/dae/geosite.dat
endef

## $(eval $(call GoBinPackage,dae))
$(eval $(call BuildPackage,dae))
$(eval $(call BuildPackage,dae-geoip))
$(eval $(call BuildPackage,dae-geosite))
