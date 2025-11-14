# 📦 Linux 存储管理实战 — 新磁盘挂载 / 扩容 / inode 排查 / Swap 配置

（基于 CentOS7）

本文记录了我在实战中完成的「第一周任务 1」全过程，包括新增磁盘、分区、格式化、挂载、扩容、inode 排查、swap 设置等完整实验流程。
内容来自我本地的实际操作，并整理成结构化文档，方便复现与学习。

# 🟦 一、挂载一块新磁盘

## 1.添加一款磁盘

检查当前磁盘状态

[root@localhost ~]# lsblk

<img width="430" height="126" alt="image" src="https://github.com/user-attachments/assets/73f5585e-f533-47b2-ba4b-538bef72d55c" />

在虚拟机关机状态添加一块硬盘

<img width="504" height="364" alt="image" src="https://github.com/user-attachments/assets/113f01de-1f15-4ce7-8263-b2ced8451853" />


（配置保持默认，添加后再开机）

<img width="519" height="422" alt="image" src="https://github.com/user-attachments/assets/ae6e2ba1-94ee-4f97-8740-369ecf3dc617" />

<img width="362" height="211" alt="image" src="https://github.com/user-attachments/assets/d606c2a2-e589-4d1b-8104-827c279babc0" />

开机后再次执行 lsblk，可看到新磁盘 sdb 已识别。

[root@localhost ~]# lsblk

<img width="501" height="157" alt="image" src="https://github.com/user-attachments/assets/2a637e15-0035-4073-983d-0f4431d3e223" />

## 2. 手动分区

[root@localhost ~]# fdisk /dev/sdb

n → p → 1 → Enter → Enter → w

<img width="554" height="316" alt="image" src="https://github.com/user-attachments/assets/23794b07-2581-434e-9ade-aa9c1c040eab" />


重新查看：

[root@localhost ~]# lsblk

<img width="554" height="165" alt="image" src="https://github.com/user-attachments/assets/d7573107-8be6-4d6e-b1c1-644f943192e7" />

## 3. 格式化为文件系统

你可以选 XFS 或 EXT4：

XFS：

mkfs.xfs /dev/sdb1

EXT4：

mkfs.ext4 /dev/sdb1

XFS	    稳定通用、支持日志恢复	      小文件系统、兼容性好

ext4	   性能强大、支持超大文件、CentOS 默认	    大容量数据、数据库、/data

我这里选得是XFS

[root@localhost ~]# mkfs.xfs /dev/sdb1

<img width="554" height="135" alt="image" src="https://github.com/user-attachments/assets/a4070909-5ba6-4ed6-8a33-b5e1fa334d56" />

## 4. 创建挂载点并挂载

创建、挂载、检查

[root@localhost ~]# mkdir  /data

[root@localhost ~]# mount /dev/sdb1 /data

[root@localhost ~]# df -h | grep data

<img width="448" height="72" alt="image" src="https://github.com/user-attachments/assets/5ba9d8ba-f145-4e51-8cee-2024c00e0c1d" />

## 5. 配置永久挂载（fstab）

查看 UUID：

[root@localhost ~]# blkid /dev/sdb1

/dev/sdb1: UUID="650f9d23-f939-4566-9faa-eb6eaec79306" TYPE="xfs" 

写入 fstab：

[root@localhost ~]# vi /etc/fstab 

 将获得的UUID填写到文件最下面

UUID=650f9d23-f939-4566-9faa-eb6eaec79306 /data xfs defaults 0 0


验证：

[root@localhost ~]# umount /data

[root@localhost ~]# mount -a

[root@localhost ~]# df -h | grep data

<img width="554" height="87" alt="image" src="https://github.com/user-attachments/assets/ad5fe4fc-8733-4a22-8aa8-093d97818fc1" />

# 🟧 二、扩容磁盘（20G → 30G）

目标：在原磁盘基础上扩大磁盘容量并扩容文件系统   30G

## 1. 虚拟机扩容硬盘

关机 → 设置 → 磁盘 → 修改容量为 30G

（注意不能有快照）

<img width="554" height="363" alt="image" src="https://github.com/user-attachments/assets/2b72a483-5b06-4384-a0f2-ef966c82be77" />

开机后查看大小已变化，但分区未扩展：

<img width="554" height="289" alt="image" src="https://github.com/user-attachments/assets/04d61fe4-35e4-48eb-950f-e3fbbed9ed00" />

## 2. 扩展分区（保持起始扇区不变）

记录原起始扇区：

比如：Start = 2048

[root@localhost ~]# fdisk -l /dev/sdb

重新分区：

[root@localhost ~]# fdisk /dev/sdb

<img width="577" height="165" alt="image" src="https://github.com/user-attachments/assets/b85c72bc-62f6-4bbd-a142-765135dde84e" />

<img width="554" height="324" alt="image" src="https://github.com/user-attachments/assets/f6b0a028-f7fa-46f3-a547-5c4023c24d44" />

刷新分区表：

[root@localhost ~]# partprobe /dev/sdb

<img width="554" height="159" alt="image" src="https://github.com/user-attachments/assets/0cf58f48-a73f-4011-9ae7-64654bcd61e8" />



## 3. 扩容 XFS 文件系统
   
[root@localhost ~]# xfs_growfs /data

查看扩容结果：

[root@localhost ~]# df -h

# 🟩 三、inode 排查

1. 查看 inode 使用情况

[root@localhost ~]# df -i

<img width="554" height="137" alt="image" src="https://github.com/user-attachments/assets/7b5c9af2-75be-4bee-9182-a8fbd7febdec" />

2. 找出产生大量小文件的目录

[root@localhost ~]# du --inodes -d 2 /data | sort -nr | head

3. 删除大量旧文件（例如 7 天前）以及空目录

find /data/tmp -type f -mtime +7 -delete
find /data/tmp -type d -empty -delete

4. 验证 inode 恢复情况

df -i

# 🟨 四、配置 Swap（扩展为 4G）

## 1. 创建 swap 文件

[root@localhost ~]# dd if=/dev/zero of=/swapfile bs=1M count=4096

[root@localhost ~]# chmod 600 /swapfile

## 2. 格式化为 swap

[root@localhost ~]# mkswap /swapfile

## 3. 启用 swap

swapon /swapfile

free -h

## 4. 持久化（写入 fstab）

[root@localhost ~]# echo "/swapfile swap swap defaults 0 0" >> /etc/fstab

验证生效：

mount -a

free -h

<img width="554" height="367" alt="image" src="https://github.com/user-attachments/assets/3200f547-f864-4136-b773-ff19fe81d333" />

