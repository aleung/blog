---
title: NAS and file system
date: 2022-12-29 22:44:15
tags:
- SysAdmin
- Linux
---

折腾了Linux台式机和NAS，记录一下。(原文写于2022年底，但放下了就没有全部写完)

其实我没有很高的NAS的服务需求，主要是数据得有备份，因此也没有专门买NAS硬件也没有用专门的NAS软件，只是用了闲置下来一台台式机，安装Ubuntu。故此，我就有两台Ubuntu电脑，一台是日常用机，称为desktop，另一台是NAS。

## NAS

### Btrfs RAID 1

NAS需要保障数据安全，组个RAID会安心一点。

这台电脑用了一个128G的SSD作为系统盘，挂两个HDD组成RAID 1做数据盘，以后有需求可以再增加一个。

数据盘的文件系统采用BTRFS，考虑BTRFS支持RAID，并且可以用多个容量不一的磁盘来组RAID 1，正好用上现有的硬盘，以后添加也可以根据到时最经济的大小来买。添加（`btrfs device add`）或者替换（`btrfs replace`）磁盘后，执行 `btrfs balance` 就会将数据在RAID中的磁盘中重新分配，它会根据数据副本数量自动考虑各个磁盘的空闲空间大小来分配。

BTRFS 的 scrub 命令可以检查和修复一部分磁盘错误。它对每个数据块读取并检查 checksum，如果发现读取失败或者数据校验错误，就会尝试用 RAID 1 中的其他副本来替换。建议定期（例如每个月）执行。

## Desktop

工作电脑要解决的问题是磁盘容量和性能的平衡，还有文件的历史版本保存，方便出问题时回退。

首先，系统根文件系统我不想搞复杂，怕出问题，还是简单在SSD上分配，用 ext4 文件系统。然后，home 和一些数据目录再考虑其他方案。

### Cached LVM

照片、视频、数据等等各种东西加起来，需要的磁盘容量不小。考虑到使用方便和存储速度，倾向于保存在desktop上，而不是从NAS远程访问。

如果都用SSD，大容量SSD也太贵了，HDD速度又太慢。以前用黑苹果的时候，是用 SSD + HDD 组成了 fusion drive，体验相当好，故此在 Linux 下也寻找类似的方案。最终选择了使用 LVM cache。

LVM 的基本概念是三层结构：

- LV - Logical Volume，在VG上分配LV，LV上创建文件系统
- VG - Volume Group，多个PV组合成一个VG
- PV - Physical Volume，对应着物理存储设备（磁盘或者分区）

一个 VG 中的 PV 可以包括 SSD（高速）和 HDD（低速），cached LV 就是将一个 cache LV 和一个 main LV 绑定起来，其中 cache LV 的空间分配自高速的 PV，而 main LV 的空间分配自低速的 PV，利用 cache 对读写操作进行缓冲，提高访问速度。

Cached LV 有两种类型：cache both read and write (type  `cache`) 和 cache only write operations (type `writecache` )，在我的应用场景中当然是希望要能够对读操作加速。有两种 cache 模式：writeback 和 writethrough，writethrough 安全但似乎就起不到加速写操作的作用，但如果指定 writeback 模式，创建的时候会有警告，不推荐使用。

很多教程、文档上写的方法都是一步步按部就班进行：

1. 创建 main LV
2. 创建 cache data LV
3. 创建 cache meta LV
4. 将 cache data LV 和 cache meta LV 合并为一个 cache pool
5. 将 cache pool 绑定到 main LV，转换为 cached LV

其实用 cachevol 比用 cache pool 要简单， 就是直接在高速 PV 上创建一个 LV，将这个 LV 作为 cache。它与 cache pool 不同在于 cachevol 无法将 data 与 meta 分开到不同磁盘上，但通常家用场景也没有这样的需求

然而，还有一个终极大招：一个命令完成创建 main LV （例子中为`/dev/sdd2`）、创建 cachevol (例子中为`/dev/sda5`)、转换为 cached LV 的全过程：

```
lvcreate --type cache -l 100%FREE --cachedevice /dev/sda5 vg2 -n lv_name /dev/sdd2
```

如果在创建的过程中出现关于 chunksize 的错误信息，需要增加参数 `--chunksize` 来调整。超过100GB的cache需要256k chunk size。

这样组成的 cached LV 跟 MacOS 的 APFS Fusoin Drive 还是有区别：cached LV 的有效大小是 main LV 的大小，cache volume 完全只是作为 cache，甚至可以把它摘掉也不会影响存储的数据；而 Fusion Drive 的容量是组成的多个分区的总大小，它内部自动将文件在不同设备之间迁移，尽可能将热数据保存在快速设备上。

### Btrfs, subvolumes and snapshots

上面步骤创建好的 LV 等价于磁盘分区，还需要在上面创建文件系统。考虑到 BTRFS 支持 snapshot，可以实现文件的历史版本记录，选择了使用BTRFS。

#### subvolume

(没写完)

#### snapper

snapper是一个管理 btrfs snapshot 的工具，对每个 subvolume 配置 snapshot 的保留策略（每小时/每天/每周/每月/每年的分别保存多少份），定时创建/清理 btrfs snapshot。

snapper是命令行工具，有一个snapper-gui项目，但在Ubuntu上工作不是很正常。用[btrfs-assistant](https://gitlab.com/btrfs-assistant/btrfs-assistant) GUI tool, 代替了 snapper-gui

实际使用中发现，如果文件系统剩余空间不多（例如不足10%），磁盘就会非常繁忙，系统卡顿，应该是数据整理非常耗费资源。要保证文件系统剩余足够空闲空间，这限制的snapshot的数量。另外一个问题是，删除snapshot的时候，磁盘会非常繁忙，因此也要减少 snapshot 的数量。期望单靠 snapshot 来找回文件历史版本并不是合适的方案，应该还是使用 backup 工具创建定期的备份（见下面的章节），而对于需要**真正**的文件版本的场合，使用专门的版本管理系统。

如果重新再装系统，我就会考虑snapshot这个功能需求是否有那么重要了，也许并不需要使用btrfs。

#### 其他

启用 quota 功能可以统计各个 subvolume 使用的空间大小：`btrfs enable quota /path`，但是 quota 会造成很严重的性能问题！所以没必要还是不要启用。

[btrfs-list](https://github.com/speed47/btrfs-list) 方便查看所有的 subvolume, snapshot 以及它们所占的空间大小

## Backup

从desktop备份数据到NAS，用的是 [Kopia](https://kopia.io)。类似的工具还有 [Borg](https://www.borgbackup.org)，都是deduplication的备份工具，对于多个备份中的相同的数据（基于hash）不会重复传输和存储，可以高效的进行增量备份，节省存储空间。

Kopia既有命令行也有GUI，我在desktop上安装了KopiaUI，使用相当方便。存储仓库用SFTP指向NAS，在NAS上不需要安装软件，配置好ssh就可以。

在desktop上，为每个要备份的目录（我这里是对应着BTRFS的subvolume）创建一个policy，policy中可以配置备份保持的周期（每小时/每天/每周/每月/每年的分别保存多少份），执行间隔时间，设定是否压缩，等等。

如果有些文件/目录不想备份，可以写到`.kopiaignore`文件中，类似于`.gitignore`的格式，这也很方便，比起在软件中配置要简单。

Desktop 上使用 BTRFS，并且已经使用 snapper 定时（每小时）创建 Btrfs snapshot，那最好是能够让备份工具备份最近一个文件系统snapshot，而不是直接备份当前文件，这样的好处是能够保证一个备份中的文件都是处于同一个时刻的，而不是有时间先后。

Kopia 的policy支持配置 before-snapshot action，正好可以做这个事情。这个 action 的脚本做的事情就是找过要备份的 btrfs subvolume 中最后一个 snapshot 的路径：

```sh
snapshots=$KOPIA_SOURCE_PATH/.snapshots
latest=`ls -Art $snapshots | tail -n 1`
echo KOPIA_SNAPSHOT_PATH=$snapshots/$latest/snapshot
```

Kopia 会备份这个 action 返回的 KOPIA_SNAPSHOT_PATH，而不是原始的 KOPIA_SOURCE_PATH。

以前用MacOS，它的 TimeMachine 很好用的一点是可以在 timeline 中很方便的浏览备份的文件，快速的定位到想要恢复的文件在那个日期的备份中存在。看了一轮 Linux 上的备份工具的介绍，在数据恢复方面都没有能够像 TimeMachine 那么方便的，需要一个个备份去查看。有些工具能够提供文件在不同备份之间 diff 的功能，可惜 Kopia 也没有。

<!-- more -->
