---
title: Ansible 介紹
date: 2018-05-04 20:20
tags:
---
## 架構


![ansible-structure](https://i.imgur.com/Pn9uTLe.png)


## 角色

Control machine：管理主機；主控台；中控台。

Managed node/host：被管理的主機。

Ansible 基本運作模式：push

<!-- more -->

### Control machine 安裝方法: 
* [Control machine 安裝方法](#Ansible-Control環境建置)

### Managed node 基本需求
Linux: Python2.6或2.7、SSH
Windows: Framework 4.0、PowerShell 3.0，[windows ansible 環境設定](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1)

* [參考文件](http://docs.ansible.com/ansible/intro_windows.html#windows-system-prep)

### 元素
Inventory：主機列表；主機清冊。
Playbook：劇本。
Task：個別組態／步驟。
Module：Ansible 內建指令。

## Ansible Control環境建置
以下是以Ubuntu系統為說明
Ubuntu版本:16.04

### 安裝add-apt-repository
安裝ansible前，先安裝add-apt-repository 必要套件

```
apt-get install python-software-properties software-properties-common -y
```

### 安裝Ansible 官方的 PPA 
安裝Ansible 官方的 PPA 套件來源

```
apt-add-repository ppa:ansible/ansible -y && apt-get update
```

### 安裝 Ansible
安裝 Ansible

```
apt-get install ansible -y
```

### 檢查ansible版本
檢查ansible版本

```
ansible --version
```

### [其他系統的安裝方式可參考](http://ansible.tw/#docs/installation.md)

### 設定Control 初始環境
ansible安裝完畢後會在/etc/ansible，個人習慣會將/etc/ansible資料夾放置在個人user目錄底下

```
cp -R /etc/ansible/ .
```

修改ansible.cfg，指定個人目錄底下的hosts

```
vi ansible.cfg
```
設定hosts檔案清單，hosts檔案清單可以用群組分類，當執行playbook時可以使用此群組

```
[Windows]
10.12.12.2
10.12.12.3
10.12.12.4

```

[windows hosts 環境建置](#Ansible-windows-host環境建置)

[linux hosts 環境建置](#Ansible-linux-host環境建置)


## Ansible windows host環境建置

### Ansible Windows連線Port
Ansible windows 連線Port為5986(https)、5985(http)

### 建立Windows執行帳號集合
在個人ansible目錄底下建立群組共用參數集合
```
mkdir group_vars
```

在group_vars建立windows.yml檔案
```
 vi group_vars/windows.yml
```

```
ansible_user: 遠端主機登入帳號
ansible_password: 遠端主機登入密碼
ansible_port: 5986
ansible_connection: winrm
ansible_winrm_server_cert_validation: ignore
```

### 安裝遠端執行windows命令模組

在control 環境上安裝winrn模組，已供遠程執行windows命令操作
```
apt-get install python-pip -y
apt-get update
pip install "pywinrm>=0.1.1"
```

### Windows命令測試
執行windows 主機命令測試
```
ansible windows -m win_ping
```

## Ansible linux host環境建置

以下是以Ubuntu系統為說明

Ubuntu版本:16.04

### 建立Linux上執行anbile的帳號
在Ansible Control建立一個使用者供ansible執行服務用。
```
useradd ansible_user -s /bin/bash -m
```

### 建立放置帳號ssh key 資料夾
切換到ansible_user，並在home底下並建立.ssh 資料夾
```
sudo su ansible_user
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

### 產生帳號ssh key
輸入指令產生ssh rsa key，一直Eenter即可
```
ssh-keygen -t rsa
```


### 複製ssh 金鑰至google上
開啟home底下的.ssh/id_rsa.pub，複製內容到google 中繼資料內的SSH 金鑰
```
vi ~/.ssh/id_rsa.pub
```
![step4](https://i.imgur.com/ZwoBTls.png)

#### 建立linux執行帳號集合
在個人ansible目錄底下建立群組共用參數集合
```
mkdir group_vars
```

在group_vars建立linux.yml檔案

```
vi group_vars/linux.yml
```

```
ansible_ssh_user: ansible_user
ansible_ssh_private_key_file: /home/ansible_user/.ssh/id_rsa
```

ansible_ssh_user: 遠端主機登入帳號

ansible_ssh_private_key_file: /home/遠端主機登入帳號/.ssh/id_rsa

### linux命令測試
執行linux 主機命令測試，會出現允許遠端連線的提示訊息，需輸入yes
```
ansible linux -m ping
```

## Ansible語法

ansible 指令有分兩種，一種是ansible，另一種為absible-playbook。

### ansible 指令說明:
ansible 主機 -m command -a 參數 -f 同時執行數量
```
ansible windows -m win_shell -a "ipconfig" -f 5
```

### ansible-playbook指令說明:

ansible-playbook .yaml檔案 -vvv(詳細列出執行訊息) -f 同時執行數量

test.yaml 內容如下

```
- name: host ip infomation
  hosts: linux
  remote_user: root
  tasks:
  - name: ifconfig
    shell: ifconfig
```

```
ansible-playbook test.yaml -vvv -f 5
```

## Ansible實例

[維護腳本](https://github.com/lj010205/devops/tree/master/MaintenanceTools)

