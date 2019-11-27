# Ansible Role: RHEL\NEC\_RH\_snmpd
=======================================================

## Description
本ロールは、 RHEL7.x サーバの snmpd の設定を行います。  

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

特にありません。

### Optional variables  

~~~
* VAR_NEC_RH_snmpd_info:                 # snmpdの関連情報を設定します。
      com2sec:                           # com2secの情報を指定します。子項目は繰り返して設定できます。
        - sec_name: 'testsec'            # セキュリティ名を指定します。
          source: '192.168.1.0/24'       # ネットワークアドレスを指定します。
          community: 'public'            # コミュニティ名を指定します。
      group:                             # groupの情報を指定します。子項目は繰り返して設定できます。
        - groupName: 'testgroup'         # グループ名を指定します。
          securityModel: 'v2c'           # モデル（v1, v2c, usm）を指定します。
          securityName: 'testsec'        # セキュリティ名を指定します。
      view:                              # viewの情報を指定します。子項目は繰り返して設定できます。
        - name: 'testview'               # viewの名前を指定します。
          incl_excl: 'included'          # included/excludedの設定を指定します。
          subtree: '1.3.6.1.2.1.1'       # ツリーを指定します。
          mask: 'fe'                     # マスク(指定しなくてもよい）を指定します。16進数になります。
      access:                            # accessの情報を指定します。子項目は繰り返して設定できます。
        - group: 'testgroup'             # グループ名を指定します。
          context: ''                    # contextの設定を指定します。
          sec_model: 'v2c'               # モデル（v1, v2c, usm, any）を指定します。
          sec_level: 'noauth'            # 認証レベルを指定します。
          prefix: 'exact'                # prefixの設定を指定します。
          read: 'testview'               # read権限をつけるviewを指定します。
          write: 'none'                  # write権限をつけるviewを指定します。
          notif: 'none'                  # notif権限をつけるviewを指定します。
      syslocation: 'honsha RHEL'         # 機器の場所などを書いておく場所（記述があれば何でもよい）を指定します。
      syscontact: 'root@localhost'       # 管理者のメールアドレスなどを書いておく場所（記述があれば何でもよい）を指定します。
      trap_agent:                        # trapエージェント側の設定情報を指定します。
          community:                     # コミュニティ情報を指定します。子項目は繰り返して設定できます。
            - trapcommunity: 'traptest'  # コミュニティ名を指定します。
              manager_ip: '192.168.1.1'  # マネージャのIPを指定します。
          user:                          # 内部監視用ユーザ情報を指定します。子項目は繰り返して設定できます。
            - username: 'internaluser'   # ユーザ名を指定します。
              hashmode: 'MD5'            # ハッシュ方式を指定します。
              password: 'internalpass'   # パスワードを指定します。
          notificationEvent:             # イベントが起きた際に通知する MIB の情報をオブジェクト名もしくは OID で指定します。子項目は繰り返して設定できます。
            - ENAME: 'linkUpTrap'        # イベント名を指定します。
              NOTIFICATION: 'linkUp  ifName  ifAdminStatus  ifOperStatus'
                                         # 監視対象OIDを指定します。
              OPTIONS: '-m'              # オプションを指定します。[-m] [-i OID | -o OID ]*などを設定できます。
          monitor:                       # 実際に監視するイベントとタイミングを定義します。子項目は繰り返して設定できます。
            - ENAME: 'linkUpTrap'        # イベント名を指定します。
              OPTIONS: '-r 20' -u internaluser  # オプションを指定します。
              NAME: 'Generate linkUp'    # NAMEはこの式の管理名で、mteTriggerTable（および関連テーブル）の索引付けに使用されます。
              EXPRESSION: 'ifOperStatus != 2'
                                         # '<監視対象OID> <条件記号> <値>などを指定します。
      trap_manager:                      # trapマネージャ側の設定を指定します。子項目は繰り返して設定できます。
        - behavior: 'log,execute,net'    # 挙動の設定を指定します。
          trapcommunity: 'traptest'      # 受け取るトラップのコミュニティ名を指定します。
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
　  │          └── NEC_RH_snmpd/
　  │                │── defaults/
　  │                │       main.yml
　  │                │── meta/
　  │                │       main.yml
　  │                │── tasks/
　  │                │       main.yml
　  │                │       modify.yml
　  │                │── vars/
　  │                │       main.yml
　  │                └─ README.md
　  └─ linux_snmpd.yml
~~~

- マスターPlaybook サンプル[linux\_snmpd.yml]

~~~
# playbook-snmpd.yml
---
- name: snmpd setting 
  hosts: rhel
  gather_facts: true
  roles:
    - role: RHEL/NEC_RH_snmpd
      VAR_NEC_RH_snmpd_info: 
          com2sec: 
            - sec_name: "testsec"
              source: "192.168.1.0/24"
              community: "public"
            - sec_name: "local"
              source: "localhost"
              community: "private"
          group: 
            - groupName: "testgroup"
              securityModel: "v1"
              securityName: "testsec"
            - groupName: "testgroup"
              securityModel: "v2c"
              securityName: "testsec"
            - groupName: "testgroup"
              securityModel: "v1"
              securityName: "local"
          view: 
            - name: "testview"
              incl_excl: "included"
              subtree: ".1.3.6.1.2.1.1"
              mask: "fe"
            - name: "testview"
              incl_excl: "included"
              subtree: ".1.3.6.1.2.1.25.1.1"
          access: 
            - group: "testgroup"
              context: ""
              sec_model: "v1"
              sec_level: "noauth"
              prefix: "exact"
              read: "testview"
              write: "none"
              notif: "none"
          syslocation: "honsha CentOS"
          syscontact: "root@localhost"
          trap_agent: 
              community:
                - trapcommunity: "traptest"
                  manager_ip: "192.168.1.1"
              user: 
                - username: "internaluser"
                  hashmode: "MD5"
                  password: "internalpass"
              notificationEvent: 
                - ENAME: "linkUpTrap"
                  NOTIFICATION: "linkUp  ifName  ifAdminStatus  ifOperStatus"
                  OPTIONS: "-m"
                - ENAME: "memfree"
                  NOTIFICATION: "memTotalFree"
              monitor: 
                - ENAME: "linkUpTrap"
                  OPTIONS: "-r 20  -u internaluser"
                  NAME: "Generate linkUp"
                  EXPRESSION: "ifOperStatus != 2"
                - ENAME: "memfree"
                  NAME: "memory non"
                  EXPRESSION: "memTotalFree < 1111000"
          trap_manager: 
            - behavior: "log,execute,net"
              trapcommunity: "traptest"
      tags:
        - NEC_RH_snmpd
~~~

## Running Playbook

~~~sh
>ansible-playbook linux_snmpd.yml
~~~

### ■エビデンスを取得する場合の呼び出す方法

エビデンスを収集する場合、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_snmpd\_evidence.yml]

~~~
---
- hosts: rhel
  roles:
    - role: gathering
      VAR_gathering_definition_role_path: "roles/RHEL/NEC_RH_snmpd"
~~~

- マスターPlaybookにエビデンスコマンドを追加します。 サンプル[linux\_snmpd.yml]

~~~
---
- import_playbook: linux_snmpd_evidence.yml VAR_gathering_label=before

  hosts: rhel
  gather_facts: true
  roles:
    - role: RHEL/NEC_RH_snmpd
      VAR_NEC_RH_snmpd_info: 
          com2sec: 
            - sec_name: "testsec"
              source: "192.168.1.0/24"
              community: "public"
            - sec_name: "local"
              source: "localhost"
              community: "private"
          group: 
            - groupName: "testgroup"
              securityModel: "v1"
              securityName: "testsec"
            - groupName: "testgroup"
              securityModel: "v2c"
              securityName: "testsec"
            - groupName: "testgroup"
              securityModel: "v1"
              securityName: "local"
          view: 
            - name: "testview"
              incl_excl: "included"
              subtree: ".1.3.6.1.2.1.1"
              mask: "fe"
            - name: "testview"
              incl_excl: "included"
              subtree: ".1.3.6.1.2.1.25.1.1"
          access: 
            - group: "testgroup"
              context: ""
              sec_model: "v1"
              sec_level: "noauth"
              prefix: "exact"
              read: "testview"
              write: "none"
              notif: "none"
          syslocation: "honsha CentOS"
          syscontact: "root@localhost"
          trap_agent: 
              community:
                - trapcommunity: "traptest"
                  manager_ip: "192.168.1.1"
              user: 
                - username: "internaluser"
                  hashmode: "MD5"
                  password: "internalpass"
              notificationEvent: 
                - ENAME: "linkUpTrap"
                  NOTIFICATION: "linkUp  ifName  ifAdminStatus  ifOperStatus"
                  OPTIONS: "-m"
                - ENAME: "memfree"
                  NOTIFICATION: "memTotalFree"
              monitor: 
                - ENAME: "linkUpTrap"
                  OPTIONS: "-r 20  -u internaluser"
                  NAME: "Generate linkUp"
                  EXPRESSION: "ifOperStatus != 2"
                - ENAME: "memfree"
                  NAME: "memory non"
                  EXPRESSION: "memTotalFree < 1111000"
          trap_manager: 
            - behavior: "log,execute,net"
              trapcommunity: "traptest"
      tags:
        - NEC_RH_snmpd

- import_playbook: linux_snmpd_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧

~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├── command
　    │   ├── 0                      #　net-snmpパッケージインストール情報
　    │   ├── 1                      #　snmpdサービス起動情報
　    │   ├── 2                      #　snmptrapdサービス起動情報
　    │   └── results.json           #　各コマンドの情報を格納するJSONファイル
　    └── file
　        └── etc
　            └── snmp
　                ├── snmpd.conf     # snmpd、trap agent情報関連配置ファイル
　                └── snmptrapd.conf # trap マネージャ側情報関連配置ファイル
~~~

## License

## Copyright

Copyright (c) 2017-2019 NEC Corporation

## Author Information

NEC Corporation
