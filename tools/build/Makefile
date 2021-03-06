# ----------------------------------------------------------------------
# Create Image
# Maintainer: Buddy <buddy.zhang@aliyun.com>
#
# Copyright (C) 2017 BiscuitOS
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 2 as
# published by the Free Software Foundation.

# --------------------------
# ROOT_DEV
# --------------------------
ifeq ($(BOOTLOADER), Floppy)
  DEFAULT_MAJOR_ROOT := 4
  DEFAULT_MINOR_ROOT := 2
else
  DEFAULT_MAJOR_ROOT := 3
  DEFAULT_MINOR_ROOT := 1
endif

ifneq ($(DEBUG),)
  include $(srctree)/tools/debug/Makefile
endif

# ---------------------------
# Sector size
# ---------------------------
SECTOR_SIZE := 512

# ---------------------------
# Sectors per-track
# ---------------------------
SECTORS := 32

# ---------------------------
# Cylinder
# ---------------------------
CYLINDERS := 8

# ---------------------------
# Heads
# ---------------------------
HEADS := 8

# ---------------------------
# Bootsect sector
# ---------------------------
BOOTSECT_SECTORS := 1
BOOTSECT_START_SECTORS := 0

# ---------------------------
# Setup sector
# ---------------------------
SETUP_SECTORS := 4
SETUP_START_SECTORS := $(BOOTSECT_SECTORS)

# ---------------------------
# System sector
# ---------------------------
SYSTEM_SECTORS := 62
SYSTEM_START_SECTORS := $(shell expr $(SETUP_START_SECTORS) + $(SETUP_SECTORS))

# ---------------------------
# Bootload sector
# ---------------------------
BOOTLOADER_CUR      := $(shell expr $(SYSTEM_START_SECTORS) + $(SYSTEM_SECTORS))
RESERVED_BOOTLOADER := 10
BOOTLOADER_SECTORS  := $(shell expr $(BOOTLOADER_CUR) + $(RESERVED_BOOTLOADER))

# ---------------------------
# Image sector : CHS
# ---------------------------
#IMAGE_SECTORS := $(shell expr $(CYLINDERS) \* $(HEADS) \* $(SECTORS))
IMAGE_SECTORS := 20480   # 10M

all: Image
# ---------------------------
# Start debug system
# ---------------------------
ifneq ($(DEBUG),)

# ---------------------------
# Create floppy or disk image
# ---------------------------
ifeq ($(DEBUG_TOOLS), qemu)
Image: ECHO bootsect setup system
	$(Q)cp -f system system.tmp
	$(Q)$(STRIP) system.tmp
	$(OBJCOPY) -O binary -R .note -R .comment system.tmp kernel
	$(Q)dd if=bootsect bs=$(SECTOR_SIZE) count=1 of=Image \
				conv=notrunc > /dev/null 2>&1
	$(Q)dd if=setup bs=$(SECTOR_SIZE) seek=1 count=4 \
	                of=Image conv=notrunc > /dev/null 2>&1
	$(Q)dd if=kernel bs=$(SECTOR_SIZE) seek=5 count=2888 \
				of=Image conv=notrunc > /dev/null 2>&1
	$(Q)rm system.tmp
	$(Q)rm -rf kernel
	$(Q)sync

# -------------------------
# Runing system on Qemu
# More Qemu configure see tools/debug/qemu/Makefile.header
# -------------------------
start:
ifeq ($(BOOTLOADER), Floppy)
		$(Q)$(QEMU) $(QEMU_FLAGS)
else
		$(Q)$(QEMU) $(QEMU_FLAGS)
endif

# -------------------------
# Debug system on Qemu
# -------------------------
debug:
ifeq ($(BOOTLOADER), Floppy)
		$(Q)$(QEMU) $(QEMU_FLAGS) -s -S
else
		$(Q)$(QEMU) $(QEMU_FLAGS) -s -S
endif

else # DEBUG_TOOLS

# -------------------------
# Use other debug tools
# -------------------------
System: Image
	# Empty

start:
	# Empty

debug:
	# Empty


endif # End of EBUG_TOOLS
# --------------------------
# Don't debug system
# --------------------------
else # DEBUG
System: Image
	# Empty

start:
	# Empty

debug:
	# Empty

endif # End DEBUG

# -------------------------
# Link all objects
# -------------------------

bootsect: ECHO
	$(Q)$(LD) $(LDFLAGS) -o $@ $(srctree)/arch/$(ARCH)/boot/bootsect.o
ifneq ($(DEBUG),)
		$(Q)cp $@ .debug_bootsect
endif
	$(Q)$(OBJCOPY) -R .pdr -R .comment -R.note -S -O binary $@

setup: ECHO
	$(Q)$(LD) $(LDFLAGS) -o $@ $(srctree)/arch/$(ARCH)/boot/setup.o
ifneq ($(DEBUG),)
		$(Q)cp $@ .debug_setup
endif
	$(Q)$(OBJCOPY) -R .pdr -R .comment -R.note -S -O binary $@


system: ECHO
	$(LD) $(LDFLAGS) -e startup_32 -o $@ $(srctree)/arch/$(ARCH)/boot/head.o \
	                           $(srctree)/init/main.o \
							   $(IMAGE_PACKAGE)
	$(Q)$(NM) $@ | grep -v '\(compiled\)\|\(\.o$$\)\|\( [aU] \)\|\(\.\.ng$$\)\|\(LASH[RL]DI\)'| sort > System.map

ECHO:

.PHONY: clean
clean:
	$(Q)rm -rf bootsect setup system System.map Image
ifneq ($(DEBUG),)
		$(Q)rm -rf .debug_bootsect .debug_setup
endif
