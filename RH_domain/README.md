# Ansible Role: RHEL\RH\_domain
=======================================================

## Description

このロールは、RHEL7.xでドメインを変更するために使用されます。

## Supports

- 管理マシン(Ansibleサーバ)  
  * Linux系OS（RHEL）
  * Ansible バージョン 2.4 以上 (動作確認バージョン 2.4, 2.9)
  * Python バージョン 2.x, 3.x  (動作確認バージョン 2.7, 3.6)
- 管理対象マシン(構築対象マシン)  
  * RHEL7.x

## Requirements

- 管理マシン(Ansibleサーバ)  
  * Ansibleホストは管理対象マシンへSSH接続できる必要がある。
- 管理対象マシン(インストール対象マシン)  
  * RHEL7.x
  * 「/etc/sysconfig/network-scripts/ifcfg-インタフェース名」というファイルに“PEERDNS=no”を設定する必要。  
   “PEERDNS=no”になっていない場合、networkサービスが再起動すると、「/etc/resolv.conf」の“nameserver”が  
   自動的に上記の「ifcfg-インタフェース名」のDNS情報により更新される。

## Role Variables
Role の変数値について説明します。

### Mandatory variables
~~~
　 * VAR_RH_domain:
　       hostname: "AD2.test.com"    # ホスト名
　       hostip: "192.168.1.1"       # ホストIP
　       domainname: "test.com"      # ドメイン名
　       user: "administrator"       # 登録ユーザ名
　       password: "abc123$%"        # 登録ユーザのパスワード
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

ドメインを変更する場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、  
以下のようなPlaybookを作成してください。  

- フォルダ構成  
~~~
 - playbook
　  │── hosts
　  │── roles/
　  │    └── RHEL/
　  │          └── RH_domain
　  │                │── defaults
　  │                │       main.yml
　  │                │── tasks
　  │                │       main.yml
　  │                │       pre_check.yml
　  │                │       modify.yml
　  │                │       post_check.yml
　  │                └─ README.md
　  └─ playbook-domain.yml
~~~

- マスターPlaybook サンプル[playbook-domain.yml]
~~~
#playbook-domain.yml
　 - name: domain setting 
　   gather_facts: true
　   hosts: rhel
　   roles:  
　     - role: RHEL/RH_domain
　       become_user: yes
　       VAR_RH_domain:
　           hostname: "AD2.test.com"
　           hostip: "192.168.1.1"
　           domainname: "test.com"
　           user: "administrator"
　           password: "abc123$%"
　       tags:
　         - RH_domain
~~~

## Running Playbook

ドメインを変更する時、以下のように実行します。

~~~sh
>ansible-playbook playbook-domain.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_domain\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_domain"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[playbook-domain.yml]
~~~
　---
　  - import_playbook: linux_domain_evidence.yml VAR_gathering_label=before
　  - name: domain setting
　    gather_facts: true
　    hosts: rhel
　    roles:
　      - role: RHEL/RH_domain
　        become_user: yes
　        VAR_RH_domain:
　            hostname: "AD2.test.com"
　            hostip: "192.168.1.1"
　            domainname: "test.com"
　            user: "administrator"
　            password: "abc123$%"
　        tags:
　          - RH_domain
　  - import_playbook: linux_domain_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　   ├── command
　   │   ├── 0                                  # ドメインの情報確認
　   │   ├── 1                                  # ntpサービスの状態
　   │   ├── 2                                  # krb5関連パッケージの確認
　   │   ├── 3                                  # samba関連パッケージの確認
　   │   ├── 4                                  # ネットワークの状態表示
　   │   └── results.json                       # 各コマンドの情報を格納するJSONファイル
　   └── file
　       └── etc
　           ├── hosts                          # hostsファイル
　           └── sysconfig
　                └── network-scripts           # ネットワーク配置ファイルのフォルダ
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
