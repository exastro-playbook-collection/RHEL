# Ansible Role: RHEL\NEC\_RH\_sshd
=======================================================

## Description
本ロールは、 RHEL7.x サーバの sshd の設定を行います。  

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

~~~
* VAR_NEC_RH_sshd_config:           # 「/etc/ssh/sshd_config」に設定したい項目を設定します。必須項目となります。
                                    # 設定したい項目のみ指定します。設定できる項目についてはsshd_configの man を参照してください。
    - key: PermitRootLogin          # （例）rootユーザのログインを許可
      value: yes                    # 
    - key: PasswordAuthentication   # （例）パスワード認証を許可
      value: yes                    # 
      ......                        # 
※`PermitRootLogin: yes`　と　`PasswordAuthentication: yes`になることが必要です。
※そうしないと、Ansibleホストから管理対象マシンに　SSH　でアクセスする時、失敗になる可能性があります。
~~~

### Optional variables  

特にありません。

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
　  │          └── NEC_RH_sshd/
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
　  └─ linux_sshd.yml
~~~

- マスターPlaybook サンプル[linux\_sshd.yml]

~~~
# linu_sshd.yml
---
- name: sshd setting 
  gather_facts: true
  hosts: rhel
  roles:
    - role: RHEL/NEC_RH_sshd
      VAR_NEC_RH_sshd_config:
        - key: Compression
          value: yes
      become_user: yes
      tags:
        - NEC_RH_sshd
~~~

## Running Playbook

~~~sh
>ansible-playbook linux_sshd.yml
~~~

### ■エビデンスを取得する場合の呼び出す方法

エビデンスを収集する場合、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_sshd\_evidence.yml]

~~~
---
- hosts: rhel
  roles:
    - role: gathering
      VAR_gathering_definition_role_path: "roles/RHEL/NEC_RH_sshd"
~~~

- マスターPlaybookにエビデンスコマンドを追加します。 サンプル[linux\_sshd.yml]

~~~
---
- import_playbook: linux_sshd_evidence.yml VAR_gathering_label=before

- name: sshd setting 
  gather_facts: true
  hosts: rhel
  roles:
    - role: RHEL/NEC_RH_sshd
      VAR_NEC_RH_sshd_config:
        - key: Compression
          value: yes
      become_user: yes

- import_playbook: linux_sshd_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧

~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├── command
　    │   ├── 0                       #　sshdインストール情報
　    │   ├── 1                       #　sshd起動情報
　    │   └── results.json            #　各コマンドの情報を格納するJSONファイル
　    └── file
　        └── etc
　            └── ssh
　                └── sshd_config     # sshd_config関連配置ファイル
~~~

## License

## Copyright

Copyright (c) 2017-2019 NEC Corporation

## Author Information

NEC Corporation
