# lvm-xfs-ext4-lab
📦 Linux 存储管理实战
— LVM / XFS / EXT4 扩容与缩容的完整实验记录

作为一名正在成长的运维实习生，我把最近练习磁盘管理、LVM、XFS/EXT4 文件系统扩容与缩容的全过程整理成这个仓库，用来巩固基础，也方便未来复习。

🧱 1. 新增磁盘与挂载
使用 fdisk 创建分区
用 mkfs.xfs / mkfs.ext4 格式化
挂载到 /data、/xdata
在 /etc/fstab 完成持久化挂载

🧩 2. 创建 LVM 逻辑卷环境
包含完整的 PV / VG / LV 工作流程：
pvcreate 初始化物理卷
vgcreate 创建卷组
lvcreate 创建逻辑卷
配置 XFS/EXT4 文件系统

🚀 3. XFS 文件系统扩容（在线扩容）
XFS 支持在线扩容：
lvextend
xfs_growfs

实战扩容案例：
10G → 20G
整个过程无需卸载，扩容速度极快。

🔧 4. XFS 文件系统缩容（迁移法）
由于 XFS 不支持 shrink，需要通过迁移法完成缩容：
迁移法步骤：
创建更小的 LV（如 20G → 9G）
格式化为 XFS
rsync -avx 迁移数据
使用 diff -r 校验目录一致性
卸载所有挂载点
lvremove 删除旧 LV
lvrename 切换到原 LV 名称
再挂载回原路径
这是企业生产环境中 唯一安全可靠 的 XFS 缩容方式。

📐 5. EXT4 文件系统扩缩容（原地 shrink）
EXT4 支持在线扩容，也支持离线 shrink。
扩容：lvextend + resize2fs
缩容：umount → e2fsck → resize2fs
同时对比了 EXT4 与 XFS 在结构与限制上的区别。

🛡 6. Swap、inode、磁盘分析
内容包括：
创建 swapfile
开机自动启用 swap（fstab）
df -i 检查 inode 占用
du -sh / df -Th 做空间分析

🔍 7. 数据完整性校验（MD5 哈希）
为了确保缩容场景下数据无损，我使用：
md5sum file
diff -r old/ new/

验证迁移前后文件的哈希值完全一致。
这是我收获最大的部分，也是最贴近企业实际的数据迁移操作。

🎯 我的目标
虽然我还是一名运维实习生，但我希望把 Linux 存储管理的基础打牢：
能独立处理磁盘扩容/缩容
能正确使用 LVM 管理存储
能在迁移文件系统时保证数据 100% 安全

为未来的云原生和 K8s 存储体系打基础

这个仓库是我学习道路上的一个阶段成果，也记录了我不断“动手实战”的态度。
