#
# Copyright (C) 2012-2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

UBIFS_OPTS = -F -m 2048 -e 124KiB -c 4096 -U
UBI_OPTS = -m 2048 -p 128KiB -s 512 -O 2048

FAT32_BLOCK_SIZE=1024
FAT32_BLOCKS=$(shell echo $$(($(CONFIG_BBB_BOOT_PARTSIZE)*1024*1024/$(FAT32_BLOCK_SIZE))))

define Image/BuildKernel
	$(CP) $(KDIR)/zImage $(BIN_DIR)/openwrt-$(BOARD)-zImage
 ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
	$(CP) $(KDIR)/zImage-initramfs $(BIN_DIR)/openwrt-$(BOARD)-zImage-initramfs
 endif

 ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_KERNEL),)
	$(INSTALL_DIR) $(TARGET_DIR)/boot
	$(CP) $(BIN_DIR)/openwrt-$(BOARD)-zImage $(TARGET_DIR)/boot/zImage
 endif
 ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_DTB),)
	$(INSTALL_DIR) $(TARGET_DIR)/boot
	$(CP) $(DTS_DIR)/am335x*.dtb $(TARGET_DIR)/boot/
	$(CP) $(DTS_DIR)/omap3*.dtb $(TARGET_DIR)/boot/
	$(CP) $(DTS_DIR)/omap4*.dtb $(TARGET_DIR)/boot/
 endif
	-mkdir $(BIN_DIR)/dtbs
	-$(CP) $(DTS_DIR)/am335x*.dtb $(BIN_DIR)/dtbs/
	-$(CP) $(DTS_DIR)/omap3*.dtb $(BIN_DIR)/dtbs/
	-$(CP) $(DTS_DIR)/omap4*.dtb $(BIN_DIR)/dtbs/
endef

define Image/Build/BBB
	rm -f $(KDIR)/boot.img

	mkdosfs $(KDIR)/boot.img -C $(FAT32_BLOCKS)
	mcopy -i $(KDIR)/boot.img $(BIN_DIR)/uboot-$(BOARD)-am335x_evm/MLO ::
	mcopy -i $(KDIR)/boot.img $(BIN_DIR)/uboot-$(BOARD)-am335x_evm/u-boot.img ::
	../../brcm2708/image/gen_rpi_sdcard_img.sh $(BIN_DIR)/$(IMG_PREFIX)-combined-vfat-$(1).img $(KDIR)/boot.img $(KDIR)/root.$(1) \
		$(CONFIG_BBB_BOOT_PARTSIZE) $(CONFIG_TARGET_ROOTFS_PARTSIZE)
endef

define Image/Build
	$(call Image/Build/$(1),$(1))

 ifneq ($(CONFIG_BBB_IMAGES),)
	$(call Image/Build/BBB,$(1))
 endif
endef

define Image/Build/jffs2-64k
	dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/openwrt-$(BOARD)-$(1).img bs=65536 conv=sync
endef

define Image/Build/jffs2-128k
	dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/openwrt-$(BOARD)-$(1).img bs=131072 conv=sync
endef

define Image/Build/squashfs
	$(call prepare_generic_squashfs,$(KDIR)/root.squashfs)
	dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/openwrt-$(BOARD)-$(1).img bs=131072 conv=sync
endef

$(eval $(call BuildImage))
