CC = gcc
AS = as
LD = ld
GRUB_FILE = grub-file
GRUB_MKRESCUE = grub-mkrescue

BUILD = build
ISO_DIR = iso
KERNEL = $(BUILD)/kernel.elf
ISO = $(BUILD)/pyos.iso
DISK = $(BUILD)/pyos-disk.img
WRITE_TEST_DISK = $(BUILD)/pyos-disk-write-test.img
DISK_SIZE_MB = 32

.PHONY: all iso run clean disk run-write-test bootstrap install-plan demo bootcfg login

all: $(KERNEL)

bootstrap:
	python3 userspace/bootstrap.py --apply

install-plan:
	python3 userspace/installer.py --dry-run --desktop kde-plasma --write-manifest build/install-plan.json

bootcfg:
	python3 userspace/bootloader.py --install

login:
	python3 userspace/login_manager.py --status

demo:
	python3 userspace/display_manager.py --preview --width 1920 --height 1080 --monitor builtin

$(BUILD):
	mkdir -p $(BUILD)

$(BUILD)/boot.o: kernel/boot.S | $(BUILD)
	$(AS) --32 $< -o $@

$(BUILD)/kernel.o: kernel/kernel.c | $(BUILD)
	$(CC) -m32 -ffreestanding -fno-pie -fno-stack-protector \
		-fno-builtin -Wall -Wextra -c $< -o $@

$(KERNEL): $(BUILD)/boot.o $(BUILD)/kernel.o kernel/linker.ld
	$(LD) -m elf_i386 -T kernel/linker.ld -o $@ \
		$(BUILD)/boot.o $(BUILD)/kernel.o

iso: $(ISO)

$(ISO): $(KERNEL) iso/boot/grub/grub.cfg
	mkdir -p $(ISO_DIR)/boot
	cp $(KERNEL) $(ISO_DIR)/boot/kernel.elf
	$(GRUB_MKRESCUE) -o $(ISO) $(ISO_DIR)

# Create a new main disk only when one doesn't already exist.
# Existing virtual disk data is never overwritten automatically.
disk: | $(BUILD)
	@if [ ! -f "$(DISK)" ]; then \
		dd if=/dev/zero of="$(DISK)" bs=1M count=$(DISK_SIZE_MB) status=progress; \
		mkfs.fat -F 16 "$(DISK)"; \
		echo "Created FAT16 disk: $(DISK)"; \
	else \
		echo "Keeping existing disk: $(DISK)"; \
	fi

run: iso disk
	qemu-system-i386 \
		-display none \
		-serial stdio \
		-monitor none \
		-cdrom $(ISO) \
		-drive file=$(DISK),format=raw,if=ide,index=0,media=disk \
		-boot d

run-graphics: iso disk
	qemu-system-i386 \
		-cdrom $(ISO) \
		-drive file=$(DISK),format=raw,if=ide,index=0,media=disk \
		-boot d

# This target never creates or formats the test disk. It refuses to run
# if the separate test image is missing, protecting the main disk by design.
run-write-test: iso
	@if [ ! -f "$(WRITE_TEST_DISK)" ]; then \
		echo "Missing $(WRITE_TEST_DISK). Create/restore a test image first."; \
		exit 1; \
	fi
	qemu-system-i386 \
		-cdrom $(ISO) \
		-drive file=$(WRITE_TEST_DISK),format=raw,if=ide,index=0,media=disk \
		-boot d

clean:
	rm -f $(BUILD)/boot.o $(BUILD)/kernel.o \
		$(BUILD)/kernel.elf $(BUILD)/pyos.iso
	rm -f $(ISO_DIR)/boot/kernel.elf
