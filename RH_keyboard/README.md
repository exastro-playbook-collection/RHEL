# Ansible Role: RHEL/RH\_keyboard
=======================================================

## Description
このロールは、RHEL7.x のキーボードマッピングとX11レイアウトを変更するために使用されます。

## Supports
- 管理マシン(Ansibleサーバ)  
 * Linux系OS（RHEL）
 * Ansible バージョン 2.7 以上 (動作確認バージョン 2.7, 2.9)
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
  * VAR_RH_keyboard:        # キーボード設定項目  
      keytable: jp106           # キーボードマッピング変数、大小文字を区分  
      x11_layout: jp106         # X11レイアウト変数、大小文字を区分  
  ※すべて利用できるkeyboardは「localectl list-keymaps」の実行結果を参照する
~~~

### Optional variables  

~~~
  * VAR_RH_keyboard_reboot: false  # すぐに再起動する/しないことを指定する。true/false（デフォルト値）  
~~~

## Dependencies  

~~~
  * ロール：RH_reboot         # 再起動用ロール
~~~

## Usage  

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて指定します。
3. Playbookを実行します。

## Example Playbook

キーボードマッピングまたはx11レイアウトを変更する場合、 "roles"ディレクトリに次のRoleを入力してください。  
以下のようなPlaybookを作成してください。 

- フォルダ構成 
~~~
 - playbook
　   ├── hosts/
　   ├── roles/
　   │    └── RHEL/
　   │          ├── RH_reboot
　   │          └── RH_keyboard
　   │                ├── defaults
　   │                │       main.yml
　   │                ├── meta
　   │                │       main.yml
　   │                ├── tasks
　   │                │       main.yml
　   │                ├── vars
　   │                │       gathering_definition.yml
　   │                └─ README.md
　   └─ linux_keyboard.yml
~~~

- マスターPlaybook サンプル[linux\_keyboard.yml]  
~~~
#linux_keyboard.yml
　  - name: Linux Modify keyboard layout and x11 layout
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_keyboard
　        VAR_RH_keyboard:
　            keytable: jp106
　            x11_layout: jp106
　        VAR_RH_keyboard_reboot: false
　        tags:
　          - RH_keyboard
~~~

## Running Playbook

- キーボードレイアウトとx11レイアウトを変更するとき,以下のように実行します。  

~~~sh
> ansible-playbook linux_keyboard.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_keyboard\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_keyboard"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux\_keyboard.yml]
~~~
　---
　  - import_playbook: linux_keyboard_evidence.yml VAR_gathering_label=before
　  - name: Linux Modify keyboard layout and x11 layout
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_keyboard
　        VAR_RH_keyboard:
　           keytable: jp106
　           x11_layout: jp106
　        VAR_RH_keyboard_reboot: false
　        tags:
　          - RH_keyboard
　  - import_playbook: linux_keyboard_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧  
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    └─── command
　        ├── 0                          #　localectl status情報にVC Keymapの関連部分
　        ├── 1                          #　localectl status情報にX11 Layoutの関連部分
　        ├── 2                          #　localectl status情報（全部）
　        └── results.json               # 各コマンドの情報を格納するJSONファイル
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
