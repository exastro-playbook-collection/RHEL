# Ansible Role: RHEL\RH\_logrotate
=======================================================

## Description
このロールは、logrotateサービスを設定するために使用されます。

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
　* VAR_RH_logrotate_options:    # logrotateの設定情報を指定する。
　      cycle: 'weekly'              # 回転周期 yearly\mothly\weekly\daily
　      rotate_num: '10'             # サイクル数 1\2\3...
　      create: True                 # 古いものを回転させた後に新しい（空の）ログファイルを作成する
　      dateext: True                # 回転されたファイルの接尾辞として日付を使用する
　      compress: True               # ログファイルを圧縮する場合は、このコメントを外してください
　* VAR_RH_logrotate_d_files:    # ユーザで作成したログローテーション設定ファイルを「/etc/logrotate.d/」に追加/置換したい場合、
　                                   # 対応するログローテーション設定ファイル名を指定する。
　    - 'test.conf'                  # 複数指定可能
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

logrotate情報を設定する場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、  
以下のようなPlaybookを作成してください。  

- フォルダ構成  
~~~
 - playbook
　   │── hosts/
　   │── roles/
　   │    └── RHEL/
　   │          └── RH_logrotate
　   │                ├── defaults
　   │                │       main.yml
　   │                ├── files  
　   │                │       userconfig01（仮）
　   │                │       userconfig02（仮）
　   │                ├── tasks
　   │                │       main.yml
　   │                │       modify_logrotate.yml
　   │                │       modify_logrotate_d_files.yml
　   │                └── README.md
　   └─ linux_logrotate.yml
~~~

- マスターPlaybook サンプル[linux_logrotate.yml]  
~~~
#linux_logrotate.yml
　  - name: Linux logrotate
　    hosts: rhel
　    gather_facts: false
　    roles:
　      - role: RHEL/RH_logrotate
　          VAR_RH_logrotate_options:
　            cycle: 'weekly'
　            rotate_num: 6
　            create: True
　            dateext: True
　            compress: False
　          VAR_RH_logrotate_d_files:
　            - 'userconfig01'
　            - 'userconfig02'
　          tags:
　            - RH_logrotate
~~~

## Running Playbook

- logrotate情報を設定する時、以下のように実行します。

~~~sh
> ansible-playbook linux_logrotate.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_logrotate\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_logrotate"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux\_logrotate.yml]
~~~
　---
　  - import_playbook: linux_logrotate_evidence.yml VAR_gathering_label=before
　  - name:  linux logrotate
　    hosts : rhel
　    gather_facts:  true
　    roles:
　      - role:  RHEL/RH_logrotate
　        VAR_RH_logrotate_options:
　            cycle: 'weekly'
　            rotate_num: 6
　            create: True
　            dateext: True
　            compress: False
　        VAR_RH_logrotate_d_files:
　          - 'userconfig01'
　          - 'userconfig02'
　        tags:
　          - RH_logrotate
　  - import_playbook: linux_logrotate_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    └── file
　        └── etc
　            ├── logrotate.conf # 「logrotate.conf」関連配置ファイル
　            └── logrotate.d    # 「logrotate.d」関連カスタマイズ設定フォルダ
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
