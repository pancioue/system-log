## 系統log
> 在 systemd 系統中（如 Ubuntu 16.04+、CentOS 7+、Debian 8+），journalctl 幾乎可以查到全部。

但要注意，php、Mysql的錯誤一般不會寫到系統Log

## journalctl 一些常見查詢語法
* 從某時之後
  ```
  journalctl --since "2025-04-26 23:00"
  ```
* 查詢區間
  ```
  journalctl --since "2025-04-26 23:00" --until "2025-04-27 01:00"
  ```
* 查某天晚上 11 點到今天凌晨 1 點之間，且只看「error」的 log, -i 忽略大小寫
  ```
  journalctl --since "2025-04-26 23:00" --until "2025-04-27 01:00" | grep -i error
  ```
  或搜尋多個字
  ```
  journalctl --since "2025-04-26 23:00" --until "2025-04-27 01:00" | grep -Ei "error|fail"
  ```
* 只想看 nginx.service 相關錯誤
  ```
  journalctl -u nginx.service --since "2025-04-26 23:00" --until "2025-04-27 01:00" | grep -Ei "error|fail"
  ```
* 若要查看常見的 ext4 錯誤訊息關鍵字
  ```
  journalctl -k | grep -i ext4
  ```
  journalctl -k（或 --dmesg）：表示只顯示來自核心（kernel）來源的訊息，相當於 dmesg 輸出。
  等同於 dmesg
  
## dmesg
* dmesg 最前面的[875743.640804] 是自從開機後經過的秒數
* 若要換成人類時間
  `dmesg --ctime`

* 硬碟壞掉可能的相關錯誤
  - I/O error
  - ata error
  - read error
  - write error
  - reset
  - disk failure

ex. 
```
dmesg --ctime | grep -iE 'error|fail|reset'
```

## syslog、kern.log
`kern.log` 基本上可以看做是 dmesg，
> 大多數新發行版（如 Ubuntu 20.04+、Fedora、Arch Linux）預設不再依賴 /var/log/kern.log，而是統一用
> ```
> journalctl -k                  # 核心訊息
> journalctl -b                  # 本次開機所有 log
> journalctl -p err..alert       # 嚴重錯誤
> ```

而 syslog 基本上就是 journalctl
> Ubuntu 預設使用 systemd-journald 收集 log，再轉送一份給 rsyslog 產生 /var/log/syslog 等檔案。

## logrotate 切系統檔案
```
/var/log/syslog
{
    rotate 7
    daily
    missingok
    notifempty
    compress  // 要壓縮
    delaycompress // 延遲壓縮 .1 是未壓縮的，.2 才開始壓

    # 是為了確保新寫入的檔案是 syslog，而不是變成 syslog.1
    postrotate 
        /usr/lib/rsyslog/rsyslog-rotate
    endscript
}
```
延遲壓縮的效果
```
syslog.3.gz ← syslog.2.gz
syslog.2.gz ← syslog.1
syslog.1    ← syslog
syslog      ← 新建空檔，開始寫入新的日誌
```
