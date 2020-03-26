# Ansible Role: RHEL\RH\_kdump
=======================================================

## Description

このロールはkdumpを設定するために使用されます。

## Supports
- 管理マシン(Ansibleサーバ)  
 * Linux系OS（RHEL7）
 * Ansible バージョン 2.4 以上 (動作確認バージョン 2.4, 2.9)
 * Python バージョン 2.x, 3.x  (動作確認バージョン 2.7, 3.6)
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
　* VAR_RH_kdump:			# 「/etc/kdump.conf」ファイルに設定項目
　     state: 'started'   		# kdumpサービスステータス ,started/stopped 
　     option:            		# kdumpの設定項目を指定する。「/etc/kdump.conf」ファイルに設定したい項目となる。
　     path: '/var/crash'		# 複数設定可能。指定できる項目についてはkdump.confのmanを参照してください。　
　     default: 'reboot' 
　     ....... 
　* VAR_RH_kdump_grub_crashkernel: '160M@64M'    # GRUB_CMDLINE_LINUXのcrashkernelの値を指定する。
　	                                                # 以下の四つのケースを対応できる。
　	                                                # 　・crashkernel=<size>
　	                                                # 　・crashkernel=<range1>:<size1>,<range2>:<size2>
　	                                                # 　・crashkernel=<size>@<starting physical address>
　	                                                # 　・crashkernel=auto
　  設定内容について、下記のURLを参考のこと：
　　　https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/kernel_crash_dump_guide/appe-supported-kdump-configurations-and-targets#sect-kdump-memory-requirements 
　　　https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/kernel_crash_dump_guide/sect-kdump-config-cli#sect-kdump-config-cli-target-type
　* VAR_RH_kdump_reboot: 'false'              	# すぐに再起動する/しないことを指定する。true/false（デフォルト値）
~~~

### Optional variables  

特にありません。

## Dependencies  

RHEL/RH_reboot

## Usage  

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて指定します。
3. Playbookを実行します。

## Example Playbook

kdumpを設定する場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、 
以下のようなPlaybookを作成してください。  

- フォルダ構成  
~~~
 - playbook
　   │── hosts/
　   │── roles/
　   │    └── RHEL/
　   │          └── RH_kdump
　   │                ├── defaults
　   │                │       main.yml
　   │                ├── meta
　   │                │       main.yml
　   │                ├── tasks
　   │                │       main.yml
　   │                │       pre_check.yml
　   │                │       modify_grub.yml
　   │                │       modify_kdump.yml
　   │                │       post_check.yml
　   │                └─ README.md
　   └─ linux_kdump.yml
~~~

- マスターPlaybook サンプル[linux_kdump.yml]  
~~~
 #linux_kdump.yml
　  - name: RH_kdump
　    hosts: rhel
　    gather_facts: false
　    roles:
　      - role: RHEL/RH_kdump
　        VAR_RH_kdump:
　          state: 'started' 
　          option:
　             path: '/var/crash'
　             default: 'reboot'
　        VAR_RH_kdump_grub_crashkernel: '160M@64M'
　        VAR_RH_kdump_reboot: false
　        tags: 
　          - RH_kdump
~~~

## Running Playbook

- kdumpを設定する時、以下のように実行します。
 
~~~sh
> ansible-playbook linux_kdump.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_kdump\_evidence.yml]
~~~
　---
　  - hosts: rhel 
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_kdump"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux\_kdump.yml]
~~~
　---
　  - import_playbook: linux_kdump_evidence.yml VAR_gathering_label=before
　  - name: RH_kdump
　    hosts: rhel
　    gather_facts: false
　    roles:
　      - role: RHEL/RH_kdump
　        VAR_RH_kdump:
　          state: 'started' 
　          option:
　             path: '/var/crash'
　             default: 'reboot'
　        VAR_RH_kdump_grub_crashkernel: '160M@64M'
　        VAR_RH_kdump_reboot: false
　        tags: 
　          - RH_kdump
　  - import_playbook: linux_kdump_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├── command
　    │   ├── 0                      # kexec-toolsパッケージのチェック
　    │   ├── 1                      # 管理対象マシンの起動時間
　    │   ├── 2                      # kdumpサービスの状態
　    │   └── results.json           # 各コマンドの情報を格納するJSONファイル
　    └── file
　        ├── boot
　        │   └── grub2
　        │       └── grub.cfg       # grub.cfg配置ファイル
　        └── etc
　            ├── default
　            │   └── grub           # grubファイル
　            └── kdump.conf         # kdump.conf配置ファイル
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
