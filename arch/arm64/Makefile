#
# arch/arm64/Makefile
#
# This file is included by the global makefile so that you can add your own
# architecture-specific flags and dependencies.
#
# This file is subject to the terms and conditions of the GNU General Public
# License.  See the file "COPYING" in the main directory of this archive
# for more details.
#
# Copyright (C) 1995-2001 by Russell King

ifeq ($(CONFIG_RELOCATABLE_KERNEL), y) 
LDFLAGS_vmlinux	:=-p --no-undefined -X -pie -Bsymbolic
else 
LDFLAGS_vmlinux	:=-p --no-undefined -X
endif 

CPPFLAGS_vmlinux.lds = -DTEXT_OFFSET=$(TEXT_OFFSET)
OBJCOPYFLAGS	:=-O binary -R .note -R .note.gnu.build-id -R .comment -S
GZFLAGS		:=-9

LIBGCC 		:= $(shell $(CC) $(KBUILD_CFLAGS) -print-libgcc-file-name)

KBUILD_DEFCONFIG := defconfig

KBUILD_CFLAGS	+= -mgeneral-regs-only
KBUILD_CFLAGS	+= -fno-pic
KBUILD_CFLAGS	+= -fno-asynchronous-unwind-tables
KBUILD_CFLAGS	+= $(call cc-option, -mpc-relative-literal-loads)

KBUILD_CFLAGS	+= $(call cc-ifversion, -ge, 0500, -DCONFIG_ARCH_SUPPORTS_INT128)

ifeq ($(CONFIG_CPU_BIG_ENDIAN), y)
KBUILD_CPPFLAGS	+= -mbig-endian
AS		+= -EB
LD		+= -EB
else
KBUILD_CPPFLAGS	+= -mlittle-endian
AS		+= -EL
LD		+= -EL
endif

CHECKFLAGS	+= -D__aarch64__

ifeq ($(CONFIG_ARM64_ERRATUM_843419), y)
KBUILD_CFLAGS_MODULE	+= -mcmodel=large
endif

# Default value
head-y		:= arch/arm64/kernel/head.o

# The byte offset of the kernel image in RAM from the start of RAM.
ifeq ($(CONFIG_ARM64_RANDOMIZE_TEXT_OFFSET), y)
TEXT_OFFSET := $(shell awk 'BEGIN {srand(); printf "0x%03x000\n", int(512 * rand())}')
else
TEXT_OFFSET := 0x00080000
endif

export	TEXT_OFFSET GZFLAGS

core-y		+= arch/arm64/kernel/ arch/arm64/mm/
core-$(CONFIG_NET) += arch/arm64/net/
core-$(CONFIG_KVM) += arch/arm64/kvm/
core-$(CONFIG_XEN) += arch/arm64/xen/
core-$(CONFIG_CRYPTO) += arch/arm64/crypto/
libs-y		:= arch/arm64/lib/ $(libs-y)
libs-y		+= $(LIBGCC)
libs-$(CONFIG_EFI_STUB) += drivers/firmware/efi/libstub/

# HotPlugs #
obj-y		:= intelli_hotplug.o autosmp.o cluster_plug.o lazyplug.o aio_hotplug.o msm_zen_decision.o msm_sleeper.o 
core-$(CONFIG_INTELLI_HOTPLUG)	+= arch/arm64/hotplug/INTELLIPLUG/
core-$(CONFIG_ALUCARD_HOTPLUG)	+= arch/arm64/hotplug/ALUCARD/
core-$(CONFIG_BLU_HOTPLUG)	+= arch/arm64/hotplug/BLUPLUG/
core-$(CONFIG_BRICKED_HOTPLUG)	+= arch/arm64/hotplug/BRICKED/
core-$(CONFIG_CLUSTER_HOTPLUG)	+= arch/arm64/hotplug/CLUSTER/
core-$(CONFIG_FAST_HOTPLUG)	+= arch/arm64/hotplug/FASTPLUG/
core-$(CONFIG_MSM_HOTPLUG)	+= arch/arm64/hotplug/MSMHOTPLUG/
core-$(CONFIG_MSM_SLEEPER)	+= arch/arm64/hotplug/MSMSLEEPER/
core-$(CONFIG_THUNDERPLUG_CONTROL)	+= arch/arm64/hotplug/THUNDERPLUG/
core-$(CONFIG_MAKO_HOTPLUG)	+= arch/arm64/hotplug/MAKO/
core-$(CONFIG_ZEN_DECISION)	+= arch/arm64/hotplug/ZENDECISION/
core-$(CONFIG_ASMP)	+= arch/arm64/hotplug/AUTOSMP/
core-$(CONFIG_ASMPTS)	+= arch/arm64/hotplug/AUTOSMPTS/
core-$(CONFIG_AIO_HOTPLUG)	+= arch/arm64/hotplug/AIO_PLUG/
core-$(CONFIG_HIMA_HOTPLUG)	+= arch/arm64/hotplug/HIMA_PLUG/
core-$(CONFIG_LAZYPLUG)		+= arch/arm64/hotplug/LAZYPLUG/
core-$(CONFIG_ARM_AUTO_HOTPLUG)		+= arch/arm64/hotplug/KTAUTO/

# Default target when executing plain make
ifeq ($(CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE),y)
KBUILD_IMAGE	:= Image.gz-dtb
else
KBUILD_IMAGE	:= Image.gz
endif

KBUILD_DTBS	:= dtbs

all:	$(KBUILD_IMAGE) $(KBUILD_DTBS)

boot := arch/arm64/boot

Image Image.gz: vmlinux
	$(Q)$(MAKE) $(build)=$(boot) $(boot)/$@

zinstall install: vmlinux
	$(Q)$(MAKE) $(build)=$(boot) $@

%.dtb: scripts
	$(Q)$(MAKE) $(build)=$(boot)/dts $(boot)/dts/$@

dtbs: scripts
	$(Q)$(MAKE) $(build)=$(boot)/dts dtbs

Image-dtb Image.gz-dtb: vmlinux scripts dtbs
	$(Q)$(MAKE) $(build)=$(boot) $(boot)/$@

PHONY += vdso_install
vdso_install:
	$(Q)$(MAKE) $(build)=arch/arm64/kernel/vdso $@

# We use MRPROPER_FILES and CLEAN_FILES now
archclean:
	$(Q)$(MAKE) $(clean)=$(boot)

# We need to generate vdso-offsets.h before compiling certain files in kernel/.
# In order to do that, we should use the archprepare target, but we can't since
# asm-offsets.h is included in some files used to generate vdso-offsets.h, and
# asm-offsets.h is built in prepare0, for which archprepare is a dependency.
# Therefore we need to generate the header after prepare0 has been made, hence
# this hack.
prepare: vdso_prepare
vdso_prepare: prepare0
	$(Q)$(MAKE) $(build)=arch/arm64/kernel/vdso include/generated/vdso-offsets.h

define archhelp
  echo  '* Image.gz      - Compressed kernel image (arch/$(ARCH)/boot/Image.gz)'
  echo  '  Image         - Uncompressed kernel image (arch/$(ARCH)/boot/Image)'
  echo  '* dtbs          - Build device tree blobs for enabled boards'
  echo  '  install       - Install uncompressed kernel'
  echo  '  zinstall      - Install compressed kernel'
  echo  '                  Install using (your) ~/bin/installkernel or'
  echo  '                  (distribution) /sbin/installkernel or'
  echo  '                  install to $$(INSTALL_PATH) and run lilo'
endef
