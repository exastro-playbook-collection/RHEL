# Ansible Role: RHEL\NEC\_RH\_grub2
=======================================================

## Description
このロールは、grub2設定を変更するために使用されます。

## Supports
- 管理マシン(Ansibleサーバ)
 * Linux系OS（RHEL7）
 * Ansible バージョン 2.4 以上
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
* VAR_NEC_RH_grub_options:          # grub2設定に関する設定情報を指定する。
                    		        # 設定したい項目のみ指定する。指定されないものは更新しないこと。
                    		        # 指定できる項目についてはgrubのmanを参照してください。
   GRUB_TIMEOUT: 5	                #
   GRUB_DEFAULT: 'saved'	        #
   ......    		                #
* VAR_NEC_RH_grub2_reboot: 'false'  # すぐに再起動する/しないことを指定する。
~~~

### Optional variables  

特にありません。

## Dependencies  

RHEL/NEC_RH_reboot

## Usage  

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて指定します。
3. Playbookを実行します。

## Example Playbook

grub2を設定する場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、  
以下のようなPlaybookを作成してください。  

- フォルダ構成  
~~~
 - playbook
　　　│── hosts/
　　　│── roles/
　　　│    └── RHEL/
　　　│          └── NEC_RH_grub2
　　　│                ├── defaults
　　　│                │       main.yml
　　　│                ├── meta
　　　│                │       main.yml
　　　│                ├── tasks
　　　│                │       main.yml
　　　│                │       modify.yml
　　　│                └─ README.md
　　　└─ linux_grub2.yml
~~~

- マスターPlaybook サンプル[linux_grub2.yml]  
~~~
　  - name: linux_grub2
　    hosts : rhel
　    gather_facts: false
　    roles:
　      - role: RHEL/NEC_RH_grub2
　        VAR_NEC_RH_grub2_options:
　            GRUB_TIMEOUT: 3
　            GRUB_DEFAULT: 'saved'
　            GRUB_DISABLE_RECOVERY: true
　        VAR_NEC_RH_grub2_reboot: false
　        tags:
　          - NEC_RH_grub2
~~~

## Running Playbook

- grub2を設定する時、以下のように実行します。

~~~sh
> ansible-playbook linux_grub2.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_grub2\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/NEC_RH_grub2"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux_grub2.yml]
~~~
　---
　  - import_playbook: linux_grub2_evidence.yml VAR_gathering_label=before
　  - name: linux_grub2
　    hosts : rhel
　    gather_facts: false
　    roles:
　      - role: RHEL/NEC_RH_grub2
　        VAR_NEC_RH_grub2_options:
　            GRUB_TIMEOUT: 3
　            GRUB_DEFAULT: 'saved'
　            GRUB_DISABLE_RECOVERY: true
　        VAR_NEC_RH_grub2_reboot: false 
　        tags:
　          - NEC_RH_grub2
　  - import_playbook: linux_grub2_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　　　├── command
　　　│   ├── 0                       # 管理対象マシンの起動時間情報
　　　│   └── results.json            # 各コマンドの情報を格納するJSONファイル
　　　└── file
　　　     ├── boot
　　　     │   └── grub2
　　　     │       └── grub.cfg        # grub.cfg配置ファイル
　　　     └── etc
　　　         └── default
　　　             └── grub            # grubファイル
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
