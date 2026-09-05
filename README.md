# GA-X99-UD4P QFlash Above 4G Decoding, ReBar Cap Removed, MMIO High Size Raised to 1tb for the GA-X99-UD4P Rev 1.0 motherboard. 
## I spent like 40 straight hours on this so throw the repo a star if it helps you, it'll make me happy.

Backup filestore and guide: https://archive.org/details/x-99-ud-4-p.-23c
GA-X99-UD4P rev 1.0 BIOS F23c, patched so high-BAR GPUs (Teslas, big Quadros, etc.) can get 64-bit MMIO.

File: X99UD4P.23c Size: 16,777,216 bytes SHA256: c80eb675e55d84415094abc2445af1ab1787ea6f1269fb7dfa4469a57e1bb648

Official Gigabyte F23c. ME, MAC, and Intel descriptor are stock.

This is designed so it'll be incredibly hard to brick your system doing it. It's a modification on the original F23c QFlash image. I probably reflashed 16 times and it was fine.

Modified:

PciBus BAR cap removed (~36GB limit → unlimited)

Hidden Above 4G enabled (Setup 0x496=1)

Defaults locked: Windows 8/10, CSM Disabled, storage/display OpROM UEFI Only (survives Load Optimized Defaults)

IntelSetup MMIOHBase = 1T, MMIO High Size = 1024G

Q-Flash volume checksum off ($BDR) so this image is accepted

No ReBarDxe. No ME change. Not the GitHub GIGABYTE.BIN dump.

WARNING — address space: This image will assign huge 64-bit BARs into the Above-4G / MMIOH window. Firmware can map a card into that window; it cannot unmap. Same cards in the same slots can reboot fine — the old map is reused.

It breaks when the PCI tree changes: you move a high-BAR GPU to a different slot, pull one, or swap slots. Each slot is a different PCI path. The old path’s BAR range stays reserved, and the new path asks for another. The map then overlaps or walks past what the chipset can decode: hang, failed PCI assign, reboot loop.

Fix: pull the large-BAR cards, CMOS clear, boot with a small GOP display only (after resetting cmos i could boot with just an rx 470 fine). That rebuilds a clean map. Reflash this image if needed, then put the cards back. Do not keep power-cycling with the big cards still in the new layout — that does not free the old ranges.

Q-Flash:

Large-BAR GPUs out. Small GOP display in (RX 470 works). HD 7570 / GTS 230 / GTS 240 go black with CSM off.

FAT32 USB, root, filename exactly X99UD4P.23c

DEL → Q-Flash → Update BIOS From Drive → Main BIOS only

Do not use Q-Flash Plus (white rear BIOS USB + board button). That writes DualBIOS backup.

After flash:

Load Optimized Defaults once

Windows 8/10 Features = Windows 8/10, CSM Support = Disabled

Save

Use:

POST/Windows with the small GOP card only

Power off. Add high-BAR GPUs one at a time. Prefer a 40-lane CPU. Read the PCIE_n print; PCIE_4 shares lanes with PCIE_1

Pass: Device Manager → Resources → Large Memory (or nvidia-smi). If you later move or remove a high-BAR card and POST dies, use the CMOS fix above before trying the new layout.






##FULL GUIDE, EASY TO FOLLOW:
This text describes a custom-modified BIOS update for an older motherboard (the Gigabyte GA-X99-UD4P) designed to solve a very specific modern problem: getting heavy-duty workstation or AI graphics cards—like NVIDIA Teslas or large Quadros—to work properly.

Here is a plain-English translation of what this means, why it’s necessary, and a step-by-step tutorial on how to install and use it safely.

[[edit from author: one tip: PCIE_4 shares lanes with PCIE_1, so if you have something in both they'll go to x8 each. pcie2 has a full x16 lane that's not shared, the bottom pcie x16 might too but i cant remember if it's wired for x8 or x16, but you should be able to visibly see it)

What Does This Mod Do?
Older motherboards (like Intel X99) have a built-in memory limit (~36GB) for PCIe devices. Modern server-grade GPUs require massive amounts of memory space (64-bit MMIO) to function. Without modification, plugging in a card like a Tesla will result in a black screen, a motherboard crash, or the card simply not being recognized.

This modified BIOS:

Removes the memory limit, allowing unlimited memory allocation for your GPUs.

Permanently forces "Above 4G Decoding" and locks critical settings (like UEFI-only mode and disabled CSM) so they survive factory resets.

Allows you to flash it safely using the built-in Q-Flash tool without bricking your board.

⚠️ The Golden Rule / Big Trap (Read Before Doing Anything)
Because of how the X99 firmware handles memory mapping, it can map a big GPU into memory, but it cannot automatically "unmap" it if you move things around.

The Problem: If you install this BIOS, boot up, and later decide to move your Tesla to a different slot, add another card, or swap slots, your system will crash or get stuck in a reboot loop. The motherboard will still look for the old memory address while trying to create a new one, causing a conflict.

The Fix: If you ever change your hardware layout (move or swap slots), you must:

Pull out the large-BAR GPUs.

Clear your CMOS (reset BIOS settings).

Boot up using a basic, small display card (like an AMD RX 470) to let the system rebuild a clean memory map.

Put your big cards back in.

Step-by-Step Tutorial: How to Flash and Setup
Phase 1: Preparation
Remove your heavy GPUs: Take out your Tesla, large Quadro, or any high-BAR cards.

Install a basic "GOP" display card: You need a simple, modern-ish graphics card to see what you're doing (e.g., an AMD RX 470 or similar basic UEFI card). Note: Very old cards like an HD 7570 may show a black screen because CSM will be disabled. --- extra edit: you specifically need a GOP card (not VBIOS) with 8gb vram or less to configure this build properly. I used an rx470 but any card matching these specs will work and can be found very cheaply second hand.

Prepare a USB Drive: Take a standard USB flash drive and format it to FAT32.

Download/Rename the file: Place the modified BIOS file on the root of the USB drive and rename it to exactly: X99UD4P.23c.

Phase 2: Flashing the BIOS
Plug the USB drive into your computer and turn it on.

Tap DEL repeatedly to enter the BIOS.

Open the Q-Flash utility.

Select Update BIOS From Drive, choose your file, and make sure you update the Main BIOS only.

🛑 CRITICAL: Do not use Q-Flash Plus (the white rear USB port and board button), as that will write to the DualBIOS backup and mess up the mod.

Let the flash complete and reboot.

Phase 3: Post-Flash Configuration
Once it reboots, go back into the BIOS and Load Optimized Defaults once.

Ensure the following settings are locked in:

Windows 8/10 Features = Windows 8/10

CSM Support = Disabled

Save and Exit.

Phase 4: Bringing Your Big GPUs Online
Boot into Windows/POST using only your small temporary graphics card to ensure everything is stable.

Power down the system completely.

Insert your high-BAR GPU (like your Tesla) into a slot. (Tip: If you have a 40-lane CPU, prefer slot placement carefully; note that PCIE_4 shares lanes with PCIE_1 on this board).

Boot up. Check Device Manager ➔ Resources ➔ Large Memory or run nvidia-smi to verify the GPU is fully recognized and utilizing its memory space.

Summary Checklist
Flash with a small GPU installed? Yes.

Use Q-Flash (not Q-Flash Plus)? Yes.

Change hardware layout later without clearing CMOS? No (this will break things—follow the CMOS reset procedure if you move cards).
