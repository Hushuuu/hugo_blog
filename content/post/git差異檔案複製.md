---
title: "Git差異檔案複製"
description: "比對提交差異檔案並複製到另外一個資料夾"
date: 2025-05-06T11:51:53+08:00
image:
draft: false
license:
hidden: false
comments: true
categories:
  - Git
tags: ["Git"]
---

## 前言

要提交給客戶原始碼，有時候只需異動之檔案  
一個一個提交作比對太花時間了，可以用`git command` 來列出差異並下指令複製

## 主要內容

到專案資料夾底下開啟`cmd`

```bash
$ git log
```

```bash
#   commit 內容
#下行就是 commit的ID
#commit ec20127e15c1b1055260a695e632083e06a74e90
#Author:
#Date:   Mon Aug 7 15:42:45 2023 +0800
```

將下方這段`shell script`存至檔案`copy_changed_files.sh`

```bash
#!/bin/bash

# 使用方式: ./copy_changed_files.sh <old_commit> <new_commit>

if [ "$#" -ne 2 ]; then
    echo "用法: $0 <舊的 commit> <新的 commit>"
    exit 1
fi

OLD_COMMIT=$1
NEW_COMMIT=$2
OUTPUT_DIR="diff_output"

# 建立輸出資料夾
mkdir -p "$OUTPUT_DIR"

# 取得異動檔案清單
git diff --name-only "$OLD_COMMIT" "$NEW_COMMIT" > changed_files.txt

# 複製檔案，保留資料夾結構
cat changed_files.txt | while read file; do
    # 檢查檔案是否存在於當前工作目錄中（可能已刪除）
    if [ -f "$file" ]; then
        mkdir -p "$OUTPUT_DIR/$(dirname "$file")"
        cp "$file" "$OUTPUT_DIR/$file"
    else
        echo "⚠️ 檔案不存在（可能已刪除）: $file"
    fi
done

echo "✅ 完成，檔案已複製到 $OUTPUT_DIR/"
```

`windows`環境下可藉由`Git Bash`來執行`shell script`  
在資料夾右鍵使用`Git`右鍵菜單`Open Git Bash Here`

```bash
chmod +x copy_changed_files.sh #權限
./copy_changed_files.sh 8fa67f3 2c9ff39 #(兩個commit ID)
```

## 小結
