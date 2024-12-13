# How to Delete the Recovery Partition and Extend Partition C

## Prerequisites
1. **Backup Your Data**: Ensure you have a recent backup of your files. Deleting the recovery partition is permanent and carries a small risk.

---

## Step 1: Open Diskpart
1. Press `Windows + R`, type `cmd`, and press `Ctrl + Shift + Enter` to open the Command Prompt as Administrator.
2. In the Command Prompt, type:
   ```
   diskpart
   ```
   and press `Enter`.

---

## Step 2: List Drives
To display all drives, enter:
```bash
list disk
```
Identify the disk that contains Partition C and the recovery partition.

---

## Step 3: Select the Drive
Select the target disk (e.g., Disk 0):
```bash
select disk 0
```

---

## Step 4: List Partitions
Display all partitions on the selected disk:
```bash
list partition
```
Locate the recovery partition (e.g., Partition 4).

---

## Step 5: Delete the Recovery Partition
1. Select the recovery partition by entering:
   ```bash
   select partition <Partition_Number>
   ```
   Replace `<Partition_Number>` with the actual number (e.g., `4`).

2. Delete the partition:
   ```bash
   delete partition override
   ```
   This will free up the space as "unallocated."

---

## Step 6: Extend Partition C
1. Open **Disk Management**:
   - Right-click on "This PC" > Manage > Disk Management.

2. Locate Partition C.
3. Right-click on Partition C and select **Extend Volume**.
4. Follow the wizard to add the unallocated space to Partition C.

---

## Notes
- Deleting the recovery partition disables the Windows Recovery Environment (WinRE). To recreate it, refer to the [Restore WinRE Instructions](#).
- Always ensure critical data is backed up before modifying partitions.

---

By following these steps, you can safely delete the recovery partition and extend your C drive.
