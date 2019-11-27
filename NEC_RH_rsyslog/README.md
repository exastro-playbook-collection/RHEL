# Ansible Role:RHEL\NEC\_RH\_rsyslog
=======================================================

## Description
このロールは、RHEL7.xでSyslogを設定するために使用されます。

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
- 管理対象マシン(インストール対象マシン)
 * RHEL7.x

## Role Variables
Role の変数値について説明します。

### Mandatory variables
~~~
　  * VAR_NEC_RH_rsyslog_rules:          # syslog配置のルールを設定する。子項目は複数個設定可能
　      - selector: 'local7.*'           # ファシリティー/優先度ベースのフィルター（<ファシリティ>.<プライオリティ>）、
　                                       # 複数のセレクターを 1 行内で定義するには、セミコロン (;) でそれらを区切る。
　        action: '/var/log/boot.log'    # ログ出力先、デフォルトでは、syslog メッセージの生成時に毎回ログファイルは同期される。
　                                       # 同期を省略する場合は、ダッシュ記号 (-) を該当するファイルパスの接頭辞として使う。 
　  * VAR_NEC_RH_rsyslog_d_files:        # ユーザで作成したシステムログ設定ファイルを「/etc/rsyslog.d/」に追加/置換したい場合、
　      - 'userconfig01'                 # 対応するシステムログ設定ファイル名を指定する。（複数指定可能）
　      - 'userconfig02'                 # 
　      - ......
~~~

### Optional variables  

特にありません。

### Dependencies  

特にありません。

## Usage  

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて指定します。
3. Playbookを実行します。

## Example Playbook

Syslogを設定する場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、  
以下のようなPlaybookを作成してください。 

- フォルダ構成  
~~~
 - playbook
　  │── hosts
　  │── roles/
　  │    └── RHEL/
　  │          └── NEC_RH_rsyslog
　  │                ├── defaults
　  │                │       main.yml
　  │                ├── files
　  │                │       userconfig01（仮）
　  │                │       userconfig02（仮）
　  │                ├── tasks
　  │                │       main.yml
　  │                │       modify.yml
　  │                │       modify_rsyslog_d_files.yml
　  │                └─ README.md
　  └─ playbook-rsyslog.yml
~~~

- マスターPlaybook サンプル[playbook-rsyslog.yml]
~~~
# playbook-rsyslog.yml
　  - name: rsyslog setting
　    gather_facts: true
　    hosts: rhel
　    roles:
　      - role: RHEL/NEC_RH_rsyslog
　        VAR_NEC_RH_rsyslog_rules:
　          - selector: "*.info;mail.none;authpriv.none;cron.none"
　            action: "/var/log/messages"
　          - selector: "lpr.info"
　            action: "/var/log/lpr.log"
　          - selector: "cron.*"
　            action: "/var/log/cron.log"
　          - selector: "mail .=err"
　            action: "-/var/log/maillog"
　          - selector: "user.!debug"
　            action: "/var/log/user.log"
　          - selector: "local7.*"
　            action: "/var/log/boot.log"
　        VAR_NEC_RH_rsyslog_d_files:
　          - 'userconfig01'
　          - 'userconfig02'
　        become_user: yes
　        tags:
　          - NEC_RH_rsyslog
~~~

## Running Playbook

Syslogの設定時、以下のように実行します。

~~~sh
>ansible-playbook playbook-rsyslog.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_rsyslog\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/NEC_RH_rsyslog"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[playbook-rsyslog.yml]
~~~
　---
　  - import_playbook: linux_rsyslog_evidence.yml VAR_gathering_label=before
　  - name: rsyslog setting
　    hosts: rhel
　    gather_facts:  true
　    roles:
　      - role: RHEL/NEC_RH_rsyslog
　        VAR_NEC_RH_rsyslog_rules:
　          - selector: "*.info;mail.none;authpriv.none;cron.none"
　            action: "/var/log/messages"
　          - selector: "lpr.info"
　            action: "/var/log/lpr.log"
　          - selector: "cron.*"
　            action: "/var/log/cron.log"
　          - selector: "mail .=err"
　            action: "-/var/log/maillog"
　          - selector: "user.!debug"
　            action: "/var/log/user.log"
　          - selector: "local7.*"
　            action: "/var/log/boot.log"
　        VAR_NEC_RH_rsyslog_d_files:
　          - 'userconfig01'
　        become_user: yes
　        tags:
　          - NEC_RH_rsyslog
　  - import_playbook: linux_rsyslog_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├── command
　    │   ├── 0                   #　サービス再起動情報
　    │   └── results.json        #　各コマンドの情報を格納するJSONファイル
　    └── file
　        └── etc
　            ├── rsyslog.conf    # rsyslog関連配置ファイル
　            └── rsyslog.d       # rsyslog関連書き込みフォルダ
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
