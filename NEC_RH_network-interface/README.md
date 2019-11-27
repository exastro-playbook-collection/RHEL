# Ansible Role: RHEL\NEC\_RH\_network-interface
=======================================================

## Description
本ロールは、 RHEL7.x サーバのネットワークインタフェースの設定を行います。  

## Supports
- 管理マシン(Ansibleサーバ)
  * Linux系OS（RHEL7）
  * Ansible バージョン 2.4 以上
  * Python バージョン 2.7
- 管理対象マシン
  * RHEL7.x

## Requirements
- 管理マシン(Ansibleサーバ)
  * Ansibleサーバは管理対象マシンへSSH接続できる必要があります。
- 管理対象マシン
  * RHEL7.x

## Role Variables
ロールの変数について説明します。

### Mandatory variables

特にありません。

### Optional variables  

~~~
* VAR_NEC_RH_network_interface:      # ネットワークインタフェース設定情報を設定します。複数のNICに対して、設定可能となります。
    - name: "end32"                  # ネットワークインターフェース名を設定します。
      ipaddresses:                   # IPアドレス情報を設定します。
        - ip: "192.168.1.1"          # ip address(IPv6非対応)
          prefix: 24                 # prefixを指定します。
          gateway: "192.168.1.126"   # ゲートウェイを指定します。
      dhcp: false                    # DHCPが有効：true/無効：false を設定します。
      state: yes                     # イーサネットの有効：yes/無効フラグ：no を設定します。
~~~

### Dependencies  

特にありません。

## Usage  

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて設定します。
3. Playbookを実行します。

## Example Playbook

### ■エビデンスを取得しない場合の呼び出す方法

本ロールを"roles"ディレクトリに配置して、以下のようなPlaybookを作成してください。

- フォルダ構成  

~~~
 - playbook/
　  │── hosts
　  │── roles/
　  │    └── RHEL/
　  │          └── NEC_RH_interface/
　  │                │── defaults/
　  │                │       main.yml
　  │                │── meta/
　  │                │       main.yml
　  │                │── tasks/
　  │                │       main.yml
　  │                │       pre_check.yml
　  │                │       modify.yml
　  │                │       post_check.yml
　  │                │── vars/
　  │                │       gathering_definition.yml
　  │                └─ README.md
　  └─ linux_interface.yml
~~~

- マスターPlaybook サンプル[linux\_interface.yml]

~~~
# linux_interface.yml
---
- name: interface setting 
  gather_facts: true
  hosts: rhel
  roles:  
    - role: RHEL/NEC_RH_network-interface
      become_user: yes
      VAR_NEC_RH_network_interface:
        - name: "ens32"
          ipaddresses: 
            - ip: "192.168.1.1"
              prefix: 24
              gateway: "192.168.1.126"
          dhcp: false 
          state: yes
        - name: "ens33"
          ipaddresses: 
            - ip: "192.168.10.1"
              prefix: 24
              gateway: "192.168.10.126"
          dhcp: false 
          state: yes
      tags:
        - NEC_RH_network-interface
~~~

## Running Playbook

~~~sh
>ansible-playbook linux_interface.yml
~~~

### ■エビデンスを取得する場合の呼び出す方法

エビデンスを収集する場合、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_interface\_evidence.yml]

~~~
---
- hosts: rhel
  roles:
    - role: gathering
      VAR_gathering_definition_role_path: "roles/RHEL/NEC_RH_network-interface"
~~~

- マスターPlaybookにエビデンスコマンドを追加します。 サンプル[linux_interface.yml]

~~~
---
- import_playbook: linux_interface_evidence.yml VAR_gathering_label=before

- name: interface setting 
  gather_facts: true
  hosts: rhel
  roles:  
    - role: RHEL/NEC_RH_network-interface
      become_user: yes
      VAR_NEC_RH_network_interface:
        - name: "ens32"
          ipaddresses: 
            - ip: "192.168.1.1"
              prefix: 24
              gateway: "192.168.1.126"
          dhcp: false 
          state: yes
        - name: "ens33"
          ipaddresses: 
            - ip: "192.168.10.1"
              prefix: 24
              gateway: "192.168.10.126"
          dhcp: false 
          state: yes

- import_playbook: linux_interface_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧

~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├── command
　    │   ├── 0                          #　ifconfig情報
　    │   ├── 1                          #　networkサービス起動情報
　    │   └── results.json               # 各コマンドの情報を格納するJSONファイル
　    └── file
　        └── etc
　            └── sysconfig
　                └── network-scripts    # networkサービス関連配置フォルダ
~~~

## License

## Copyright

Copyright (c) 2017-2019 NEC Corporation

## Author Information

NEC Corporation
