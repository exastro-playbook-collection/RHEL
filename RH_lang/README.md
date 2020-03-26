# Ansible Role: RHEL/RH\_lang
=======================================================

## Description
このロールは、RHEL7.x のシステムデフォルト言語を変更するために使用されます。

* 注意事項
  * OSインストール直後に収集・パラメータ生成コードを用いて生成したパラメータファイルを本ロールで使用すると、以下エラーが発生する場合があります。
~~~
fatal: ～ "Parameter VAR_RH_lang is not defined."}
~~~
パラメータファイルの内容を確認して以下のようになっている場合
~~~
VAR_RH_lang: '"ja_JP.UTF-8"'
~~~
以下のように引用符を削除してロールを再実行してください。
~~~
VAR_RH_lang: ja_JP.UTF-8
~~~

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
    * VAR_RH_lang: C     # システムのデフォルト言語を指定する
                             # 設定可能な値：C及びlocalectl list-localesコマンドで取得したシステムロケール言語
                             # localectl list-localesの結果に“UTF-8”がないが、運用習慣により、“utf8”と“UTF-8”両方ともサポートする
                             # 大小文字区別
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

システムのデフォルト言語を変更する場合は、 "roles"ディレクトリに次のRoleを入力してください。  
以下のようなPlaybookを作成してください。 

- フォルダ構成 
~~~
 - playbook
　  ├── hosts/
　  ├── roles/
　  │    └── RHEL/
　  │          └── RH_lang
　  │                ├── defaults
　  │                │       main.yml
　  │                ├── meta
　  │                │       main.yml
　  │                ├── tasks
　  │                │       main.yml
　  │                ├── vars
　  │                │       gathering_definition.yml
　  │                └─ README.md
　  └─ linux_lang.yml
~~~

- マスターPlaybook サンプル[linux\_lang.yml]  
~~~
#linux_lang.yml
　  - name: Linux Set the system default language
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_lang
　        VAR_RH_lang: 'ja_JP.utf8'
　        tags:
　          - RH_lang
~~~

## Running Playbook

- システムのデフォルト言語を変更する、以下の操作を実行します。

~~~
ansible-playbook linux_lang.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_lang\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_lang"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux\_lang.yml]
~~~
　---
　  - import_playbook: linux_lang_evidence.yml VAR_gathering_label=before
　  - name: Linux Set the system default language
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_lang
　        VAR_RH_lang: 'ja_JP.utf8'
　        tags:
　          - RH_lang
　  - import_playbook: linux_lang_evidence.yml  VAR_gathering_label=after
~~~

- エビデンス収集結果一覧
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    └─── file
　         └── etc
　             └── locale.conf       #言語関連の設定ファイル
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
