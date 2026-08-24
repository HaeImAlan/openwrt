
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=qca-ssdk
PKG_VERSION:=12.5.0.6
PKG_RELEASE:=2

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://git.codelinaro.org/clo/qsdk/oss/lklm/qca-ssdk
PKG_SOURCE_VERSION:=a71cfe83d988116769eb5ca6847f8f6bc176e302
PKG_MIRROR_HASH:=skip

LOCAL_VARIANT=$(patsubst qca-ssdk-%,%,$(patsubst qca-ssdk-%,%,$(BUILD_VARIANT)))
include $(INCLUDE_DIR)/package.mk

define KernelPackage/qca-ssdk/default-nohnat
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Devices
  TITLE:=Kernel driver for QCA SSDK
  FILES:=$(PKG_BUILD_DIR)/build/bin/qca-ssdk.ko
  AUTOLOAD:=$(call AutoLoad,30,qca-ssdk)
  PROVIDES:=qca-ssdk
endef

define KernelPackage/qca-ssdk-nohnat
$(call KernelPackage/qca-ssdk/default-nohnat)
  DEPENDS:=@(TARGET_ar71xx||TARGET_ipq50xx||TARGET_ipq60xx||TARGET_ipq_ipq806x||TARGET_ipq807x)
  VARIANT:=nohnat
endef

define KernelPackage/qca-ssdk-nohnat/Description
This package contains a qca-ssdk driver for QCA chipset
endef

define KernelPackage/qca-ssdk-hnat
$(call KernelPackage/qca-ssdk/default-nohnat)
  DEPENDS:=@(TARGET_ar71xx||TARGET_ipq40xx||TARGET_ipq806x) \
	+kmod-ipt-extra +kmod-ipt-filter +kmod-ppp \
	+(TARGET_ipq40xx||TARGET_ipq806x):kmod-qca-rfs
  TITLE+= (hnat)
  VARIANT:=hnat
endef

define KernelPackage/qca-ssdk-hnat/Description
This package contains a qca-ssdk-hnat driver for QCA chipset
endef

ifeq ($(CONFIG_TARGET_ipq),y)
subtarget:=$(SUBTARGET)
else
subtarget:=$(CONFIG_TARGET_BOARD)
endif

GCC_VERSION=$(shell echo "$(CONFIG_GCC_VERSION)" | sed 's/[^0-9.]*\([0-9.]*\).*/\1/')
ifeq ($(GCC_VERSION), 4.8)
GCC_VERSION=4.8.3
endif

ifdef CONFIG_TOOLCHAIN_BIN_PATH
TOOLCHAIN_BIN_PATH=$(CONFIG_TOOLCHAIN_BIN_PATH)
else
TOOLCHAIN_BIN_PATH=$(TOOLCHAIN_DIR)/bin
endif

ifdef CONFIG_TARGET_NAME
QCASSDK_CONFIG_OPTS+= \
		      TARGET_NAME=$(CONFIG_TARGET_NAME)
endif

QCASSDK_CONFIG_OPTS+= TOOL_PATH=$(TOOLCHAIN_BIN_PATH) \
                SYS_PATH=$(LINUX_DIR) \
                TOOLPREFIX=$(TARGET_CROSS) \
                KVER=$(LINUX_VERSION) \
                ARCH=$(LINUX_KARCH) \
		TARGET_SUFFIX=$(CONFIG_TARGET_SUFFIX) \
		GCC_VERSION=$(GCC_VERSION) \
		CFLAGS=-I$(STAGING_DIR)/usr/include

ifeq ($(LOCAL_VARIANT),hnat)
    QCASSDK_CONFIG_OPTS+= HNAT_FEATURE=enable
ifeq ($(CONFIG_TARGET_ar71xx),)
    QCASSDK_CONFIG_OPTS+= RFS_FEATURE=enable
endif
endif

ifneq (, $(findstring $(subtarget), "ipq60xx" "ipq60xx_64" "ipq807x" "ipq807x_64"))
    QCASSDK_CONFIG_OPTS+= PTP_FEATURE=enable
endif

# ifneq (, $(findstring $(CONFIG_KERNEL_IPQ_MEM_PROFILE), 256 512)$(CONFIG_LOWMEM_FLASH))
    QCASSDK_CONFIG_OPTS+= MINI_SSDK=enable
ifneq (, $(findstring $(subtarget), "ipq807x" "ipq807x_64"))
    QCASSDK_CONFIG_OPTS+= CHIP_TYPE=HPPE
else ifneq (, $(findstring $(subtarget), "ipq60xx" "ipq60xx_64"))
    QCASSDK_CONFIG_OPTS+= CHIP_TYPE=CPPE
else ifneq (, $(findstring $(subtarget), "ipq50xx" "ipq50xx_64"))
    QCASSDK_CONFIG_OPTS+= ISISC_ENABLE=enable
    QCASSDK_CONFIG_OPTS+= CHIP_TYPE=MP
# ssdk 12.5's sfp_phy.c does not build on kernel 4.4: SFP_PHY_FEATURES puts an
# #ifndef/#endif inside a macro line-continuation (illegal, only noticed once
# the macro is expanded), and sfp_phy_phydev_adv_update() passes
# phydev->advertising to a linkmode helper, which is a u32 not a bitmap on 4.4.
# This board has no SFP cage; hsl_phy.c guards its SFP table entry with
# #ifdef IN_SFP_PHY and falls back to NULL, and hsl_port_is_sfp() lives in
# hsl_phy.c, so the driver can simply not be built.
    QCASSDK_CONFIG_OPTS+= IN_SFP_PHY=FALSE IN_SFP=FALSE
# MHT is the QCA8084 "Manhattan" switch, which this board does not have. Its
# ssdk_mht_pinctrl.c wants PIN_CONFIG_OUTPUT_ENABLE, a pinconf-generic enum
# value added in v4.20. ssdk's own config only enables MHT for
# ipq53xx/ipq95xx/ipq60xx, so turning it off here matches upstream intent.
    QCASSDK_CONFIG_OPTS+= MHT_ENABLE=disable
# ifeq (, $(CONFIG_LOWMEM_FLASH))
# ifeq (, $(findstring $(CONFIG_KERNEL_IPQ_MEM_PROFILE), 256))
#     QCASSDK_CONFIG_OPTS+= MINI_SSDK=disable
# endif
# endif
endif
    QCASSDK_CONFIG_OPTS+= PTP_FEATURE=disable
# endif

# ifeq ($(CONFIG_TARGET_ipq806x_QSDK_Standard),)
#     QCASSDK_CONFIG_OPTS+= HK_CHIP=enable
# endif

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include/qca-ssdk
	$(INSTALL_DIR) $(1)/usr/include/qca-ssdk/api
	$(INSTALL_DIR) $(1)/usr/include/qca-ssdk/ref
	$(INSTALL_DIR) $(1)/usr/include/qca-ssdk/fal
	$(INSTALL_DIR) $(1)/usr/include/qca-ssdk/sal
	$(INSTALL_DIR) $(1)/usr/include/qca-ssdk/init
	$(CP) -rf $(PKG_BUILD_DIR)/include/api/sw_ioctl.h $(1)/usr/include/qca-ssdk/api
	if [ -f $(PKG_BUILD_DIR)/include/ref/ref_vsi.h ]; then \
	$(CP) -rf $(PKG_BUILD_DIR)/include/ref/ref_vsi.h $(1)/usr/include/qca-ssdk/ref/; \
	fi
	if [ -f $(PKG_BUILD_DIR)/include/ref/ref_fdb.h ]; then \
	$(CP) -rf $(PKG_BUILD_DIR)/include/ref/ref_fdb.h $(1)/usr/include/qca-ssdk/ref/; \
	fi
	if [ -f $(PKG_BUILD_DIR)/include/ref/ref_port_ctrl.h ]; then \
	$(CP) -rf $(PKG_BUILD_DIR)/include/ref/ref_port_ctrl.h $(1)/usr/include/qca-ssdk/ref/; \
	fi
	if [ -f $(PKG_BUILD_DIR)/include/init/ssdk_init.h ]; then \
	$(CP) -rf $(PKG_BUILD_DIR)/include/init/ssdk_init.h $(1)/usr/include/qca-ssdk/init/; \
	fi
	$(CP) -rf $(PKG_BUILD_DIR)/include/fal $(1)/usr/include/qca-ssdk
	$(CP) -rf $(PKG_BUILD_DIR)/include/common/*.h $(1)/usr/include/qca-ssdk
	$(CP) -rf $(PKG_BUILD_DIR)/include/sal/os/linux/*.h $(1)/usr/include/qca-ssdk
	$(CP) -rf $(PKG_BUILD_DIR)/include/sal/os/*.h $(1)/usr/include/qca-ssdk

endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) $(strip $(QCASSDK_CONFIG_OPTS))
endef

define KernelPackage/qca-ssdk-nohnat/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/qca-ssdk $(1)/etc/init.d/qca-ssdk
endef

define KernelPackage/qca-ssdk-hnat/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/qca-ssdk $(1)/etc/init.d/qca-ssdk
endef

$(eval $(call KernelPackage,qca-ssdk-nohnat))
$(eval $(call KernelPackage,qca-ssdk-hnat))
