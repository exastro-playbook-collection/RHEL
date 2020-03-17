# Ansible Role: RHEL\RH\_static-route
=======================================================

## Description
このロールは、RHEL7.xに静的ルートを設定するために使用されます。

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
注意事項：XXXXX というWindows/Linux共通の変数は廃止しました。RH_XXXXX、WIN_XXXXXというOS毎の変数設定が必要です。

### Mandatory variables
~~~
　  * VAR_RH_static_route:     # 静的ルート設定情報を設定する。子項目は繰り返して設定できる。
　      - interface: 'ens32'       # ネットワークインタフェース設定名を設定する。大小文字を区分する。※省略可能
　        dest: '192.168.1.1/24'   # ネットワーク宛先のIPアドレス（Prefixを付けることは可能）を指定する。
　                                 # destが設定されない及び0.0.0.0 、0.0.0.0/0の場合、固定ルートのゲートウェイとする。
　        gateway: '192.168.1.126' # ターゲットホストのゲートウェイを設定する。
　        metric: 1000             # metric を設定する。固定ルートのゲートウェイを設定する時、指定する必要なし。※省略可能
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

静的ルートを設定する場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、  
以下のようなPlaybookを作成してください。  

- フォルダ構成  
~~~
 - playbook
　  │── hosts/
　  │── roles/
　  │    └── RHEL/
　  │          └── RH_static-route
　  │                ├── defaults
　  │                │       main.yml
　  │                ├──　handlers
　  │                │       main.yml
　  │                ├── tasks
　  │                │       main.yml
　  │                │       pre_check.yml
　  │                │       modify.yml
　  │                │       post_check.yml
　  │                └─ README.md
　  └─ linux_static-route.yml
~~~

- マスターPlaybook サンプル[linux\_static-route.yml]  
~~~
#linux_static_routing.yml 
　  - name: linux_static_route setting  
　    hosts: rhel
　    gather_facts: false
　    roles:
　      - role: RHEL/RH_static-route
　        VAR_RH_static_route: 
　          - interface: 'ens32'
　            dest: '192.168.1.1/24' 
　            gateway: '192.168.1.126'
　            metric: 1000
　          - gateway: '192.168.10.254'
　        tags: 
　          - RH_static-route
~~~

## Running Playbook

- 静的ルートを設定する時、以下のように実行します。

~~~sh
> ansible-playbook linux_static-route.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_static-route_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_static-route"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux_static-route.yml]
~~~
　---
　  - import_playbook: linux_static-route_evidence.yml VAR_gathering_label=before
　  - name: linux_static_route setting
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_static-route
　        VAR_RH_static_route:
　          - interface: 'ens224'
　            dest: '172.28.145.0/25'
　            gateway: '172.28.145.254'
　            metric: 1005
　          - gateway: '172.28.159.254'
　        tags:
　          - RH_static-route
　  - import_playbook: linux_static-route_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├── command
　    │   ├── 0                           #　静的ルート設定情報
　    │   └── results.json                #　各コマンドの情報を格納するJSONファイル
　    └── file
　        └── etc
　            └── sysconfig
　                └── network-scripts     # 静的ルート設定情報関連フォルダ
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
