# Ansible Role:RHEL\RH\_ntp
=======================================================

## Description
本ロールは、 RHEL7.x サーバのntpの設定を行います。  

## Supports
- 管理マシン(Ansibleサーバ)
  * Linux系OS（RHEL7）
  * Ansible バージョン 2.4 以上 (動作確認バージョン 2.4, 2.9)
  * Python バージョン 2.x, 3.x  (動作確認バージョン 2.7, 3.6)
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
* VAR_RH_ntp_servers:         # NTPサーバの情報を設定します。
                                  # ※子項目は繰り返して設定できます。
    - server: "192.168.1.1"       # NTPサーバのIPアドレス(IPv6非対応)あるいはホスト名を指定します。親項目が設定されたら、設定必須となります。
      prefer: true                # 優先使用のサーバを指定します。（True/False）opがadd/updの場合、設定必須となります。
      op: "add"                   # 操作タイプ（add/upd/del）を指定します。大小文字を区別しません。設定必須となります。
    - server: "192.168.10.1"
      prefer: false
      op: "upd"
    - server: "192.168.20.1"
      op: "del"
* VAR_RH_ntp_clearflag: True  # 既存servers設定を全部クリアするかどうかを設定します。True/False（デフォルト値）
* VAR_RH_ntp_options:         # NTPオプション情報を設定します。
      slew: True                  # slewモード（True/False）を指定します。
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
　  │          └── RH_ntp/
　  │                │── defaults/
　  │                │       main.yml
　  │                │── meta/
　  │                │       main.yml
　  │                │── tasks/
　  │                │       main.yml
　  │                │       modify.yml
　  │                │── vars/
　  │                │       gathering_definition.yml
　  │                └─ README.md
　  └─ linux_ntp.yml
~~~

- マスターPlaybook サンプル[linux\_ntp.yml]

~~~
# linux_ntp.yml
---
- name: ntp setting
  gather_facts: true
  hosts: rhel
  roles:  
    - role: RHEL/RH_ntp
      VAR_RH_ntp_servers:
        - server: '192.168.1.1'
          prefer: True
          op: 'add'
        - server: '192.168.10.1'
          prefer: False
          op: 'upd'
        - server: '192.168.21.1'
          op: 'del'
      VAR_RH_ntp_clearflag: True
      VAR_RH_ntp_options:
          slew: True
      tags:
        - RH_ntp
~~~

## Running Playbook

~~~sh
>ansible-playbook linux_ntp.yml
~~~

### ■エビデンスを取得する場合の呼び出す方法

エビデンスを収集する場合、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_ntp\_evidence.yml]

~~~
---
- hosts: rhel
  roles:
    - role: gathering
      VAR_gathering_definition_role_path: "roles/RHEL/RH_ntp"
~~~

- マスターPlaybookにエビデンスコマンドを追加します。 サンプル[linux_ntp.yml]

~~~
---
- import_playbook: linux_ntp_evidence.yml VAR_gathering_label=before

- name: ntp setting
  gather_facts: true
  hosts: rhel
  roles:  
    - role: RHEL/RH_ntp
      VAR_RH_ntp_servers:
        - server: '192.168.1.1'
          prefer: True
          op: 'add'
        - server: '192.168.10.1'
          prefer: False
          op: 'upd'
        - server: '192.168.21.1'
          op: 'del'
      VAR_RH_ntp_clearflag: True
      VAR_RH_ntp_options:
          slew: True

- import_playbook: linux_ntp_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧

~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├── command
　    │   ├── 0              # ntpパッケージのチェック
　    │   ├── 1              # ntpサービスの状態
　    │   └── results.json   # 各コマンドの情報を格納するJSONファイル
　    └── file
　        └── etc
　            ├── ntp.conf   # ntp.confファイル
　            └── sysconfig
　                └── ntpd   # ntpdファイル
~~~

## License

## Copyright

Copyright (c) 2017-2019 NEC Corporation

## Author Information

NEC Corporation
