# **📌 Linux Server Backup Options**  

---

## 🧭 1️⃣ Local Backup (On the Same Server)

Local backups are stored on the same machine — fast to create and restore, perfect for system admins managing standalone or test environments.

---

### ✅ **Using `tar` (Archive & Compress)**

**Best for:** Small servers, quick directory-level backups

```bash
sudo tar -czvf /backup/mybackup.tar.gz /home /etc /var/www
```

📌 **Explanation:**
Creates a compressed archive (`.tar.gz`) containing `/home`, `/etc`, and `/var/www`.
Easy to move or store, but restoring individual files can be slower.

🧠 *Memorization Trick:*

> “**tar czvf** → Compress Ze Very Files!”
> `c=create`, `z=gzip`, `v=verbose`, `f=file`.

---

### ✅ **Using `rsync` (Efficient Incremental Backup)**

**Best for:** Regular local backups or syncing directories efficiently

```bash
sudo rsync -av --delete /home /backup/
```

📌 **Explanation:**
Synchronizes `/home` to `/backup/`:

* Copies only **changed files** (incremental)
* `--delete` removes files in destination not in source
  Perfect for cron-based automated backups.

🧠 *Memorization Trick:*

> “**rsync** keeps things in **sync**.”

---

### ✅ **Using `dd` (Full Disk Backup)**

**Best for:** Disk cloning, low-level recovery

```bash
sudo dd if=/dev/sda of=/backup/disk.img bs=4M status=progress
```

📌 **Explanation:**
Copies every bit from `/dev/sda` into `disk.img`.
Useful for cloning drives or creating bootable recovery images.
⚠️ **Caution:** Large and slow; restores overwrite disks entirely.

🧠 *Memorization Trick:*

> “**dd** = disk duplicate.”

---

### ✅ **Using `Timeshift` (System Snapshot Tool)**

**Best for:** System restore points, rollback before major updates or driver installs

```bash
sudo timeshift --create --comments "Before major update" --tags D --snapshot-device /dev/sdb2
```

📌 **Explanation:**
Creates a snapshot of your entire system (using **rsync** or **Btrfs**) that can be restored later if something breaks.
Unlike `tar` or `rsync`, Timeshift focuses on **system files**, not personal data.

🧰 **Common Commands:**

```bash
sudo timeshift --list                       # List snapshots
sudo timeshift --restore                    # Restore snapshot interactively
sudo timeshift --delete --tags D            # Delete daily snapshots
```

🧠 *Memorization Trick:*

> “When time drifts, use Timeshift.”
> Think of it as your **system time machine**.

---

### ⚙️ **Quick Comparison**

| Tool        | Backup Type     | Speed    | Incremental | Suitable For           | Restore Ease |
| ----------- | --------------- | -------- | ----------- | ---------------------- | ------------ |
| `tar`       | File-level      | ⚡ Medium | ❌ No        | Small, manual archives | 🟡 Moderate  |
| `rsync`     | File-level      | ⚡ Fast   | ✅ Yes       | Regular backups        | 🟢 Easy      |
| `dd`        | Disk-level      | 🐢 Slow  | ❌ No        | Full disk clone        | 🔴 Risky     |
| `timeshift` | System snapshot | ⚡ Fast   | ✅ Yes       | OS restore points      | 🟢 Very Easy |

---

### 🧠 *Memory Hook Summary:*

> * **tar** → “Pack it all up.”
> * **rsync** → “Keep it synced.”
> * **dd** → “Duplicate disk.”
> * **timeshift** → “Turn back time.”


---

## **2️⃣ Remote Backup (To Another Server or Cloud)**  
### ✅ **Using `rsync` Over SSH**
**Best for:** Secure remote backups  
```sh
rsync -avz -e "ssh -p 22" /home user@remote-server:/backup/
```
📌 *Syncs `/home` to `/backup/` on a remote server using SSH (`port 22`).*  

### ✅ **Using `scp` (Secure Copy)**
**Best for:** Simple file transfer  
```sh
scp -r /backup/ user@remote-server:/backup/
```
📌 *Copies `/backup/` directory to a remote server via SSH.*  

### ✅ **Using `Bacula` (Enterprise-Level Backup)**
**Best for:** Large-scale, multi-server backup management  
```sh
apt install bacula
```
📌 *Bacula automates **backup scheduling, storage management, and recovery**.*  

---

## **3️⃣ Cloud Backup Solutions**  
### ✅ **AWS S3 (Using `aws s3 cp` or `s3 sync`)**
**Best for:** Scalable cloud backups  
```sh
aws s3 cp /backup/mybackup.tar.gz s3://mybucket/
```
📌 *Uploads the backup to AWS S3.*  
```sh
aws s3 sync /backup/ s3://mybucket/
```
📌 *Syncs an entire directory to S3.*  

### ✅ **Google Drive (`rclone`)**
**Best for:** Free cloud storage  
```sh
rclone copy /backup remote:backup-folder
```
📌 *Copies `/backup/` to Google Drive or another configured cloud storage.*  

### ✅ **Dropbox CLI**
**Best for:** Personal backups  
```sh
dropbox_uploader.sh upload /backup/mybackup.tar.gz /
```
📌 *Uploads a backup to Dropbox using Dropbox Uploader.*  

---

## **4️⃣ Backup Automation & Scheduling**  
### ✅ **Using `cron` (Schedule Backups)**
Add a backup job to **crontab**:  
```sh
crontab -e
```
Example: Backup `/home` every night at 2 AM  
```sh
0 2 * * * tar -czvf /backup/home-$(date +\%F).tar.gz /home
```
📌 *Creates a timestamped backup every day at `2 AM`.*  

---

## **5️⃣ Database Backup (MySQL / PostgreSQL)**  
### ✅ **MySQL Backup (`mysqldump`)**
```sh
mysqldump -u root -p database_name > /backup/db_backup.sql
```
📌 *Exports `database_name` to `/backup/db_backup.sql`.*  

### ✅ **PostgreSQL Backup (`pg_dump`)**
```sh
pg_dump -U postgres -F c database_name > /backup/db_backup.pgdump
```
📌 *Creates a PostgreSQL database dump (`.pgdump`).*  

---

## **6️⃣ Backup & Recovery Solutions**  
| **Backup Type** | **Tool** | **Use Case** |  
|-----------------|---------|-------------|  
| **File Backup** | `tar`, `rsync` | Local file and directory backups |  
| **Full System Image** | `dd`, `Clonezilla` | Full disk cloning |  
| **Automated Backup** | `Bacula`, `Duplicity` | Enterprise backup solutions |  
| **Remote Backup** | `rsync`, `scp`, `S3` | Backup to another server or cloud |  
| **Database Backup** | `mysqldump`, `pg_dump` | Backup MySQL/PostgreSQL databases |  
| **Incremental Backup** | `rsync`, `BorgBackup` | Backup only changed files |  

---

## **📌 Final Thoughts**
✅ **Choose a backup strategy** based on your needs:  
- **Small servers?** → `tar` / `rsync`  
- **Remote backup?** → `rsync` / `AWS S3`  
- **Enterprise backup?** → `Bacula` / `Duplicity`  
- **Database backup?** → `mysqldump` / `pg_dump`  

[Home](../../README.md) · [All topics](../../docs/topics.md) · [Reading queue](../../docs/LATEST.md)

## New notes

<!-- new-notes -->
