include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/version.mk

PKG_NAME:=ipq-wifi
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL=$(PROJECT_GIT)/project/firmware/qca-wireless.git
PKG_SOURCE_DATE:=2025-05-30
PKG_SOURCE_VERSION:=fde3d2a7ce59be389224304049a4a5b9ca49a45e
PKG_MIRROR_HASH:=2e149366118192d4976baf42ac23d22ef32e477feb111d2756a2411278b6cf46
PKG_FLAGS:=nonshared

include $(INCLUDE_DIR)/package.mk

define Build/Compile
endef

# Use ath10k-bdencoder from https://github.com/qca/qca-swiss-army-knife.git
# to generate the board-* files here.
#
# This is intended to be used on an interim basis until device-specific
# board data for new devices is available through the upstream compilation
#
# Please send a mail with your device-specific board files upstream.
# You can find instructions and examples on the linux-wireless wiki:
# <https://wireless.wiki.kernel.org/en/users/drivers/ath10k/boardfiles>

ALLWIFIBOARDS:= \
	8devices_mango \
	arcadyan_aw1000 \
	asus_rt-ax89x \
	buffalo_wxr-5950ax12 \
	cambiumnetworks_xe34 \
	cmcc_rm2-6 \
	compex_wpq873 \
	dynalink_dl-wrx36 \
	edgecore_eap102 \
	edimax_cax1800 \
	linksys_mx4200 \
	linksys_mx5300 \
	linksys_mx8500 \
	linksys_whw03 \
	netgear_lbr20 \
	netgear_rax120v2 \
	netgear_sxk80 \
	netgear_wax214 \
	netgear_wax218 \
	netgear_wax620 \
	netgear_wax630 \
	prpl_haze \
	qnap_301w \
	redmi_ax6 \
	skspruce_wia3300-20 \
	spectrum_sax1v1k \
	tplink_eap660hd-v1 \
	tplink_archer-c6-v2 \
	wallys_dr40x9 \
	xiaomi_ax3600 \
	xiaomi_ax9000 \
	yyets_le1 \
	yuncore_ax880 \
	yuncore_fap650 \
	zbtlink_zbt-z800ax \
	zte_mf269 \
	zte_mf287 \
	zte_mf287plus \
	zyxel_nbg7815

ALLWIFIPACKAGES:=$(foreach BOARD,$(ALLWIFIBOARDS),ipq-wifi-$(BOARD))

define Package/ipq-wifi-default
  SUBMENU:=ath10k Board-Specific Overrides
  SECTION:=firmware
  CATEGORY:=Firmware
  DEPENDS:=@(TARGET_ath79||TARGET_ipq40xx||TARGET_ipq806x||TARGET_qualcommax)
  TITLE:=Custom Board
endef

define ipq-wifi-install-one-to
  $(INSTALL_DIR)  $(2)/lib/firmware/ath10k/$(3)/
  $(INSTALL_DATA) $(1) $(2)/lib/firmware/ath10k/$(3)/board-2.bin
endef

define ipq-wifi-install-ath11-one-to
  $(INSTALL_DIR)  $(2)/lib/firmware/ath11k/$(3)/
  $(INSTALL_DATA) $(1) $(2)/lib/firmware/ath11k/$(3)/board-2.bin
endef

define ipq-wifi-install-one
  $(if $(filter $(suffix $(1)),.QCA4019 .qca4019),\
    $(call ipq-wifi-install-one-to,$(1),$(2),QCA4019/hw1.0),\
  $(if $(filter $(suffix $(1)),.QCA9888 .qca9888),\
    $(call ipq-wifi-install-one-to,$(1),$(2),QCA9888/hw2.0),\
  $(if $(filter $(suffix $(1)),.QCA9889 .qca9889),\
    $(call ipq-wifi-install-one-to,$(1),$(2),QCA9887/hw1.0),\
  $(if $(filter $(suffix $(1)),.QCA9984 .qca9984),\
    $(call ipq-wifi-install-one-to,$(1),$(2),QCA9984/hw1.0),\
  $(if $(filter $(suffix $(1)),.QCA99X0 .qca99x0),\
    $(call ipq-wifi-install-one-to,$(1),$(2),QCA99X0/hw2.0),\
  $(if $(filter $(suffix $(1)),.IPQ6018 .ipq6018),\
    $(call ipq-wifi-install-ath11-one-to,$(1),$(2),IPQ6018/hw1.0),\
  $(if $(filter $(suffix $(1)),.IPQ8074 .ipq8074),\
    $(call ipq-wifi-install-ath11-one-to,$(1),$(2),IPQ8074/hw2.0),\
  $(if $(filter $(suffix $(1)),.QCN9074 .qcn9074),\
    $(call ipq-wifi-install-ath11-one-to,$(1),$(2),QCN9074/hw1.0),\
    $(error Unrecognized board-file suffix '$(suffix $(1))' for '$(1)')\
  ))))))))

endef
# Blank line required at end of above define due to foreach context

define generate-ipq-wifi-package
  define Package/ipq-wifi-$(1)
    $(call Package/ipq-wifi-default)
    TITLE:=board-2.bin Overrides for $(2)
    CONFLICTS:=$(PREV_BOARD)
  endef

  define Package/ipq-wifi-$(1)/description
The $(2) requires board-specific, reference ("cal") data
that is not yet present in the upstream wireless firmware distribution.

This package supplies board-2.bin file(s) that, in the interim,
overwrite those supplied by the ath10k-firmware-* packages.

This is package is only necessary for the $(2).

Do not install it for any other device!
  endef

  define Package/ipq-wifi-$(1)/install-overlay
    $$$$(foreach IPQ_WIFI_BOARD_FILE,$$$$(wildcard $(PKG_BUILD_DIR)/board-$(1).*),\
      $$$$(call ipq-wifi-install-one,$$$$(IPQ_WIFI_BOARD_FILE),$$(1)))
  endef

  PREV_BOARD+=ipq-wifi-$(1)
endef

# To add local files for testing:
# 1. create a new directory in the ipq-wifi package directory called "files".
# 2. place there the needed board files.
#
# To send a board file upstream:
# 1. create a pull request in https://github.com/openwrt/firmware_qca-wireless or
#    send a patch to the mailing list adding the tag firmware/qca-wireless before
#     the PATCH tag.
# 2. create a pull request in the main openwrt repository bumping the ipq-wifi
#    package to the new version and adding the required lines to add the board file.
#
# Changes needed to the ipq-wifi Makefile:
# 1. Add board name to ALLWIFIBOARDS
# 2. Add $(eval $(call generate-ipq-wifi-package,<devicename>,<display name>))
#
# Board files should follow this name structure:
#   board-<devicename>.<qca4019|qca9888|qca9889|qca9984|qca99x0|ipq8074>

$(eval $(call generate-ipq-wifi-package,8devices_mango,8devices Mango))
$(eval $(call generate-ipq-wifi-package,arcadyan_aw1000,Arcadyan AW1000))
$(eval $(call generate-ipq-wifi-package,asus_rt-ax89x,Asus RT-AX89X))
$(eval $(call generate-ipq-wifi-package,buffalo_wxr-5950ax12,Buffalo WXR-5950AX12))
$(eval $(call generate-ipq-wifi-package,cambiumnetworks_xe34,Cambium Networks XE3-4))
$(eval $(call generate-ipq-wifi-package,cmcc_rm2-6,CMCC RM2-6))
$(eval $(call generate-ipq-wifi-package,compex_wpq873,Compex WPQ-873))
$(eval $(call generate-ipq-wifi-package,dynalink_dl-wrx36,Dynalink DL-WRX36))
$(eval $(call generate-ipq-wifi-package,edgecore_eap102,Edgecore EAP102))
$(eval $(call generate-ipq-wifi-package,edimax_cax1800,Edimax CAX1800))
$(eval $(call generate-ipq-wifi-package,linksys_mx4200,Linksys MX4200))
$(eval $(call generate-ipq-wifi-package,linksys_mx5300,Linksys MX5300))
$(eval $(call generate-ipq-wifi-package,linksys_mx8500,Linksys MX8500))
$(eval $(call generate-ipq-wifi-package,linksys_whw03,Linksys WHW03))
$(eval $(call generate-ipq-wifi-package,netgear_lbr20,Netgear LBR20))
$(eval $(call generate-ipq-wifi-package,netgear_rax120v2,Netgear RAX120v2))
$(eval $(call generate-ipq-wifi-package,netgear_sxk80,Netgear SXK80))
$(eval $(call generate-ipq-wifi-package,netgear_wax214,Netgear WAX214))
$(eval $(call generate-ipq-wifi-package,netgear_wax218,Netgear WAX218))
$(eval $(call generate-ipq-wifi-package,netgear_wax620,Netgear WAX620))
$(eval $(call generate-ipq-wifi-package,netgear_wax630,Netgear WAX630))
$(eval $(call generate-ipq-wifi-package,qnap_301w,QNAP 301w))
$(eval $(call generate-ipq-wifi-package,prpl_haze,prpl Haze))
$(eval $(call generate-ipq-wifi-package,redmi_ax6,Redmi AX6))
$(eval $(call generate-ipq-wifi-package,skspruce_wia3300-20,SKSpruce WIA3300-20))
$(eval $(call generate-ipq-wifi-package,spectrum_sax1v1k,Spectrum SAX1V1K))
$(eval $(call generate-ipq-wifi-package,tplink_eap660hd-v1,TP-Link EAP660 HD v1))
$(eval $(call generate-ipq-wifi-package,tplink_archer-c6-v2,TP-Link Archer C6 V2))
$(eval $(call generate-ipq-wifi-package,wallys_dr40x9,Wallys DR40X9))
$(eval $(call generate-ipq-wifi-package,xiaomi_ax3600,Xiaomi AX3600))
$(eval $(call generate-ipq-wifi-package,xiaomi_ax9000,Xiaomi AX9000))
$(eval $(call generate-ipq-wifi-package,yyets_le1,YYeTs LE1))
$(eval $(call generate-ipq-wifi-package,yuncore_ax880,Yuncore AX880))
$(eval $(call generate-ipq-wifi-package,yuncore_fap650,Yuncore FAP650))
$(eval $(call generate-ipq-wifi-package,zbtlink_zbt-z800ax,Zbtlink ZBT-Z800AX))
$(eval $(call generate-ipq-wifi-package,zte_mf269,ZTE MF269))
$(eval $(call generate-ipq-wifi-package,zte_mf287,ZTE MF287))
$(eval $(call generate-ipq-wifi-package,zte_mf287plus,ZTE MF287Plus))
$(eval $(call generate-ipq-wifi-package,zyxel_nbg7815,Zyxel NBG7815))

$(foreach PACKAGE,$(ALLWIFIPACKAGES),$(eval $(call BuildPackage,$(PACKAGE))))
