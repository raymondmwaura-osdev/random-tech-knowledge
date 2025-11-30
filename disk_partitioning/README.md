# Disk Parititioning

## What is Disk Partitioning

* **Definition**: Disk partitioning (or “disk slicing”) is the process of dividing a single physical storage device (hard disk drive, SSD, etc.) into multiple independent regions, called *partitions*.
* Each partition behaves (to the operating system) like a separate logical disk. Before data can be stored in a usable filesystem, the disk must first be partitioned, then each partition formatted with a filesystem.
* The layout and metadata about these partitions is stored in a *partition table*, which the OS reads during boot or when the disk is mounted.

**Why partition a disk?**

* Logical separation: you might want different partitions for system files, user data, swap space, backups, etc., making maintenance, backup, and OS reinstallation easier.
* Multiple operating systems: partitions let you install and boot different OSes on the same physical disk.
* Filesystem flexibility: different partitions can use different filesystems or mount options.
* Performance and reliability: isolating data can reduce risk of corruption; partitioning may also improve disk organization and management.

---

## Partitioning Schemes: MBR vs GPT

When partitioning a disk, you typically choose a *partition table scheme*. On PC-compatible systems two dominate: Master Boot Record (MBR) and GUID Partition Table (GPT).

### MBR (Master Boot Record)

* MBR is the older standard, invented in the early PC-era (since around 1983).
* On an MBR disk you can have up to **four “primary” partitions**.
* To exceed that limit, one of the primary partitions may be defined as an **“extended partition”**; within the extended partition you can create multiple **logical partitions**. This “extended + logical” scheme enables more partitions but is more complex.
* Disadvantages / limitations of MBR:

  * Maximum disk size ~ 2 TiB. Usage beyond that may be impossible with MBR.
  * Boot information and partition metadata are stored in a single region (the “boot sector” / MBR), which presents a single point of failure.

### GPT (GUID Partition Table)

* GPT is the modern standard, part of the firmware specification for modern PCs (UEFI-based systems).
* GPT supports **many more partitions**; e.g. typical OS implementations (like Windows) allow up to 128 partitions by default.
* GPT uses 64-bit block addressing (versus 32-bit in MBR), so disks much larger than 2 TiB are possible; making GPT suitable for modern large-capacity drives.
* GPT also stores its metadata redundantly: both a primary and a backup partition table/headers (at the start and end of the disk), improving reliability and recovery in case of corruption.
* On modern platforms (OS + firmware), GPT is increasingly the default and preferred partition scheme.

*(TI-prefixes = Tebibyte; ZB = Zettabyte)*

---

## Partition Types and Their Roles (on MBR and GPT)

### On MBR-based Disks

* **Primary partition**: A standalone partition that can contain a filesystem or operating system. Up to four primary partitions allowed.
* **Extended partition**: A special kind of partition that doesn’t hold a filesystem directly, but acts as a container for **logical partitions**. Only one extended partition allowed per disk.
* **Logical partitions**: Subdivisions within the extended partition. They behave as independent partitions (own filesystems), giving flexibility for more than four partitions on one physical disk.
* **Boot / System partition**: Among primary partitions one can be flagged as “active/bootable,” telling firmware to load bootloader/OS from it. Legacy BIOS-based systems normally boot from a primary partition.

### On GPT-based Disks

* There is no concept of “extended” vs “logical” partitions: GPT uses only primary-type partitions. Each partition is described in the GPT’s partition entries.
* Partitions are identified by globally unique identifiers (GUIDs); both for the partition itself and for the type of partition (e.g. “EFI System”, “Linux filesystem”, etc.).
* For bootable GPT disks on modern systems, a special partition (e.g. “EFI System Partition”) is typically used to store bootloader and firmware-related data.

---

## Basic Workflow for Partitioning a Disk

Here’s a simplified, conceptual workflow to partition a disk (before formatting or installing an OS):

1. **Choose a partitioning scheme**; MBR (legacy / compatibility) or GPT (modern / large-disk ready), based on disk capacity, intended OS, and boot firmware (BIOS vs UEFI).
2. **Initialize the disk**; write a partition table header (MBR or GPT) to the disk; this tells the OS how partitions will be described.
3. **Define partitions**; specify for each partition start and end (or size), and mark partition type (filesystem type or purpose: OS, data, swap, boot, etc.).
4. **(Optional) For MBR: create extended partition → logical partitions** if you need more than 4 partitions.
5. **Write the partition table**; this writes metadata to disk so OS and firmware recognize the partitions.
6. **Format partitions**; once partitions exist, each must be formatted with a filesystem (e.g. ext4, NTFS, FAT32, etc.), so data can be stored.
7. **Mount or assign drive letters / mount points**; so OS can access the partitions.

In many real-world scenarios, partitioning is done via partition-management utilities (graphical or command-line), which abstract low-level details. For example, in Linux distributions or system rescue CDs, such tools allow safe creation, resizing, deletion of partitions.

---

## Choosing the Right Partitioning Strategy; When MBR vs GPT, and Partition Layout Planning

When designing a partitioning layout for a disk, consider factors such as:

* **Disk size**: If disk is larger than ~2 TiB, GPT is strongly recommended (MBR can’t address beyond ~2 TiB).
* **System firmware / boot mode**: If your system supports and uses UEFI (modern PCs), GPT is usually preferred; for older BIOS-based hardware or legacy OSes, MBR might be necessary.
* **Number of partitions needed**: If you need many partitions (OS + data + swap + backups + recovery + more), GPT’s high partition limit is convenient.
* **Reliability and recovery**: GPT’s redundant metadata (primary + backup tables) reduces risk of partition-table corruption compared to MBR’s single table.
* **Operating system / cross-platform requirements**: Some older OSes or tools might expect MBR; some OSes (or dual-boot configurations) may have specific requirements for system vs data partitions.
* **Future expandability**: Select a layout that allows future resizing or addition of partitions without redoing the entire disk.

---

## Common Pitfalls and Best Practices

* **Backing up data before partitioning**; partitioning operations (especially table initialization, resizing, converting between schemes) can destroy existing data. Always backup important files first.
* **Care when converting schemes**; converting a disk from MBR to GPT (or vice versa) often requires erasing partitions. Some tools allow non-destructive conversion, but reliability and OS support must be verified.
* **Proper boot / system partition placement**; ensure bootable partition is configured correctly (active flag on MBR, correct EFI partition on GPT) otherwise the system may not boot.
* **Avoid over-complicating layout**; there’s no need for dozens of partitions unless justified; keep partitioning rational and meaningful (e.g. OS, data, swap, recovery).
* **Account for filesystem requirements and alignment**; ensure partitions start at suitable boundaries/alignment (especially with SSDs or modern storage) and choose appropriate filesystem types.
* **Document your partition layout**; especially on multi-OS or server systems; knowing what partition holds what is essential for maintenance and backups.

---
