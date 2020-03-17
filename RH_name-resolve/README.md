# Ansible Role: RHEL\RH\_name-resolve
=======================================================

## Description
本ロールは、 RHEL7.x サーバの名前解決設定を行います。  

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
  * 「/etc/sysconfig/network-scripts/ifcfg-インタフェース名」というファイルに`PEERDNS=no`を設定する必要です。  
    `PEERDNS=no`となっていない場合、networkサービスを再起動すると、「/etc/resolv.conf」の`nameserver`が自動的に上記の「ifcfg-インタフェース名」のDNS情報により更新されます。

## Role Variables
ロールの変数について説明します。
注意事項：XXXXX というWindows/Linux共通の変数は廃止しました。RH_XXXXX、WIN_XXXXXというOS毎の変数設定が必要です。

### Mandatory variables

特にありません。

### Optional variables  

~~~
* VAR_RH_name_resolve_hosts:          # hostsファイルに設定する情報を設定します。
                                          # ※子項目は繰り返して設定できます。
    - ip: 192.168.1.1                     # IPを指定します。
      hostname: test.domain.com           # ホストネームを指定します。
* VAR_RH_name_resolve_ipv6_disabled:  # hostsファイルにIPV6に関する情報を無効にするかどうか指定します。true\false（デフォルト値）
* VAR_RH_name_resolve_dns:            # DNSサーバ配置に関する変数を設定します。
      servers:                            # DNSサーバのIPを指定します。複数のDNSサーバを設定できます。親項目が定義されたら、設定必須となります。
        - 192.168.0.1
        - 192.168.10.1
      suffix:                             # DNSサーバのドメイン名を指定します。複数のDNSサーバを設定できます。
        - test.com
        - localdomain
* VAR_RH_name_resolve_nsswitch:       # 名前解決の参照先の順序を設定します。
                                          # ※子項目は繰り返して設定できます。
    - method: dns                         # メソッドを指定します。親項目が定義されたら、設定必須となります。設定可能な値は下記の表（★1）を参照してください。
      actions: '[NOTFOUND=return]'        # アクションを指定します。設定可能な値は下記の表（★2）を参照してください。
~~~

**★1　設定可能なメソッド一覧**

| メソッド | 意味 |
|:---|:---|
|`nisplus`| Use NIS+ (NIS version 3) |
|`nis`| Use NIS (NIS version 2), also called YP |
|`dns`| Use DNS (Domain Name Service) |
|`files`| Use the local files |
|`db`| Use the local database (.db) files |
|`compat`| Use NIS on compat mode |
|`hesiod`| Use Hesiod for user lookups |

**★2　アクションの設定仕様**

| 説明内容 | 詳細 |
|:---|:---|
| フォーマット１ | `[STATUS=ACTION]` |
| フォーマット２ | `[!STATUS=ACTION]` |
| STATUS値取 | `success`、`notfound`、`unavail`、`tryagain` |
| ACTION値取 | `return`、`continue` |

## Dependencies  

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
　  │          └── RH_name-resolve/
　  │                ├── defaults/
　  │                │       main.yml
　  │                ├── handlers/
　  │                │       main.yml
　  │                ├── tasks/
　  │                │       main.yml
　  │                │       pre_check_nm.yml
　  │                │       modify_hosts.yml
　  │                │       modify_resolve.yml
　  │                │       modify_nsswitch.yml
　  │                └─ README.md
　  └─ linux_name_resolve.yml
~~~

- マスターPlaybook サンプル[linux\_name\_resolve.yml]  

~~~
#linux_name_resolve.yml
---
- name: Linux NameResolve
  hosts: rhel
  gather_facts: true
  roles:
    - role: RHEL/RH_name-resolve
      VAR_RH_name_resolve_hosts:
        - ip: '192.168.1.1'
          hostname: 'test.domain.com'
        - ip: '192.168.1.2'
          hostname: 'AD1.domain.com'
      VAR_RH_name_resolve_ipv6_disabled: false
      VAR_RH_name_resolve_dns:
          servers:
            - '192.168.10.1'
            - '192.168.20.1'
          suffix:
            - 'localdomain'
            - 'test.com'
      VAR_RH_name_resolve_nsswitch:
        - method: 'dns'
          actions: '[SUCCESS=continue]'
        - method: 'files'
          actions: '[NOTFOUND=return]'
      tags:
        - RH_name-resolve
~~~

## Running Playbook

~~~sh
> ansible-playbook linux_name_resolve.yml
~~~

### ■エビデンスを取得する場合の呼び出す方法

エビデンスを収集する場合、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_name\_resolve\_evidence.yml]

~~~
---
- hosts: rhel
  roles:
    - role: gathering
      VAR_gathering_definition_role_path: "roles/RHEL/RH_name-resolve"
~~~

- マスターPlaybookにエビデンスコマンドを追加します。 サンプル[linux\_name\_resolve.yml]

~~~
---
- import_playbook: linux_name_resolve_evidence.yml VAR_gathering_label=before

- name: Linux NameResolve
  hosts: rhel
  gather_facts: true
  roles:
    - role: RHEL/RH_name-resolve
      VAR_RH_name_resolve_hosts:
        - ip: '192.168.1.1'
          hostname: 'test.domain.com'
        - ip: '192.168.1.2'
          hostname: 'AD1.domain.com'
      VAR_RH_name_resolve_ipv6_disabled: false
      VAR_RH_name_resolve_dns:
          servers:
            - '192.168.10.1'
            - '192.168.20.1'
          suffix:
            - 'localdomain'
            - 'test.com'
      VAR_RH_name_resolve_nsswitch:
        - method: 'dns'
          actions: '[SUCCESS=continue]'
        - method: 'files'
          actions: '[NOTFOUND=return]'

- import_playbook: linux_name_resolve_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧

~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    └── file
　        └── etc
　            ├── hosts         # hosts関連配置ファイル
　            ├── nsswitch.conf # nsswitch関連配置ファイル
　            └── resolv.conf   # resolv関連配置ファイル
~~~

## License

## Copyright

Copyright (c) 2017-2019 NEC Corporation

## Author Information

NEC Corporation
