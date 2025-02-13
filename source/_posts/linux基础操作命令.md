---
layout: linux
title: linux基础操作命令
date: 2025-02-13 19:54:44
tags: linux

---

```bash
# 查看文件、文件夹
ls

# 查看文件详情
ls -l

# 显示所有文件（包括隐藏文件）
ls -a

# 以人类可读的格式显示文件和目录的大小
ls -h

# 通常与l配合使用
ls -lh

# 组合使用
ls -lha

# 查看包含隐藏文件、系统文件
ls ./*

# 删除所有 
rm -rf ./*

# 查看当前所在的绝对路径
pwd

# 创建文件 若文件存在 则访问时间和修改时间更新为当前时间
touch example.txt

# 添加文件权限 
#-R：递归地更改目录及其所有子目录和文件的权限。
# 775：设置权限为 775，其中：
# 7（所有者）：读、写、执行权限（rwx）。
# 7（所属组）：读、写、执行权限（rwx）。
# 5（其他用户）：读和执行权限（r-x）。
sudo chmod -R 775 /home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/

# 更改所有者
sudo chown -R lighthouse:lighthouse /home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/


```

