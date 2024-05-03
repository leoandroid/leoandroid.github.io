---
show_subscribe: false	

title: Mac 常用的命令
tags: 
    - Mac
---


## 1. Mac系统设置任何来源安装第三方软件
    ```shell
    sudo spctl --master-disable
    ```

## 2. 查询 IP
```shell
# wifi ip
ipconfig getifaddr en0
# ethernet ip
ipconfig getifaddr en1
```