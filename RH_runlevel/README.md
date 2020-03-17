# Ansible Role: RHEL/RH\_runlevel
=======================================================

## Description
このロールは、RHEL7.x のシステムランレベルを変更するために使用されます。

## Supports
- 管理マシン(Ansibleサーバ)  
 * Linux系OS（RHEL）
 * Ansible バージョン 2.7 以上
 * Python バージョン 2.7  
- 管理対象マシン  
 * RHEL7.x

## Requirements
- 管理マシン(Ansibleサーバ)  
 * Ansibleホストは管理対象マシンへSSH接続できる必要がある。
- 管理対象マシン  
 * RHEL7.x

## Role Variables
Role の変数値について説明します。

### Mandatory variables
~~~
    * VAR_RH_runlevel: multi-user.target     # 設定のシステムランレベルを指定する
                                                 # 設定可能な値は次のとおり、大小文字区別:
                                                 #   poweroff.target    →runlevel0.targetと対応する
                                                 #   rescue.target      →runlevel1.targetと対応する
                                                 #   multi-user.target  →runlevel3.targetと対応する
                                                 #   graphical.target   →runlevel5.targetと対応する
                                                 #   reboot.target      →runlevel6.targetと対応する
~~~

### Optional variables  

特にありません。

## Dependencies  

特にありません。

## Usage  

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて指定します。
3. Playbookを実行します。

## Example Playbook

システムのランレベルを変更したい場合は、 "roles"ディレクトリに次のRoleを入力してください。  
以下のようなPlaybookを作成してください。 

- フォルダ構成 
~~~
 - playbook
　  ├── hosts/
　  ├── roles/
　  │    └── RHEL/
　  │          └── RH_runlevel
　  │                ├── defaults
　  │                │       main.yml
　  │                ├── meta
　  │                │       main.yml
　  │                ├── tasks
　  │                │       main.yml
　  │                ├── vars
　  │                │       gathering_definition.yml
　  │                └─ README.md
　  └─ linux_runlevel.yml
~~~

- マスターPlaybook サンプル[linux\_runlevel.yml]  
~~~
#linux\_runlevel.yml
　  - name: Linux Modify the system run level
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_runlevel
　        VAR_RH_runlevel: 'multi-user.target'
　        tags:
　          - RH_runlevel
~~~

## Running Playbook

- システムのデフォルトランレベルを変更し、以下の操作を実行します。

~~~
> ansible-playbook linux_runlevel.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_runlevel\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_runlevel"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux\_runlevel.yml]
~~~
　---
　  - import_playbook: linux_runlevel_evidence.yml VAR_gathering_label=before
　  - name: Linux Modify the system run level
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_runlevel
　        VAR_RH_runlevel: 'multi-user.target'
　        tags:
　          - RH_runlevel
　  - import_playbook: linux_runlevel_evidence.yml  VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    └─── command
　        ├── 0                          #　systemctl get-default情報
　        └── results.json               # 各コマンドの情報を格納するJSONファイル
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
