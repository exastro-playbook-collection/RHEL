# Ansible Role: RHEL/RH\_password-rules
=======================================================

## Description
このロールは、RHEL7.x のパスワードポリシーを変更するために使用されます。

## Supports
- 管理マシン(Ansibleサーバ)  
 * Linux系OS（RHEL）
 * Ansible バージョン 2.7 以上
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

### Mandatory variables

特にありません。

### Optional variables  

~~~
  * VAR_RH_pw_validity_period:         # パスワードの有効期限を設定する項目
        pass_max_days: 90                  # パスワードが期限切れになる最大日数を設定する
        pass_min_days: 0                   # パスワードの変更間隔の最小値を設定する
        pass_warn_age: 3                   # パスワード有効期限の警告日を設定する

  * VAR_RH_pw_quality_flag: requisite  # パスワード強度のフラグを設定する（「pam_pwquality.so」を利用）
                                           # 設定可能な項目：required、requisite、sufficient、optional
  * VAR_RH_pw_quality_arg1:            # パスワード強度の引数を設定する（「key=value」ではない項目）
      - xxx                                # 設定可能な項目はコマンドラインで「man pam_pwquality」の結果を参照のこと
      - yyy                                # 複数値を設定できる
      ・・・・                              # 
  * VAR_RH_pw_quality_arg2:            # パスワード強度の引数を設定する（「key=value」の項目）
        aaa: mm                            # 設定可能な項目はコマンドラインで「man pam_pwquality」の結果を参照のこと
        bbb: nn                            # 複数値を設定できる
      ・・・・                              # 

  * VAR_RH_pw_authentication_flag: requisite  # パスワード認証のフラグを設定する（「pam_unix.so」を利用）
                                           # 設定可能な項目：required、requisite、sufficient、optional
  * VAR_RH_pw_authentication_arg1:     # パスワード認証の引数を設定する（「key=value」ではない項目）
      - xxx                                # 設定可能な項目はコマンドラインで「man pam_unix」の結果を参照のこと
      - yyy                                # 複数値を設定できる
      ・・・・                              # 
  * VAR_RH_pw_authentication_arg2:     # パスワード認証の引数を設定する（「key=value」の項目）
        aaa: mm                            # 設定可能な項目はコマンドラインで「man pam_unix」の結果を参照のこと
        bbb: nn                            # 複数値を設定できる
      ・・・・                              # 
~~~

## Dependencies  

特にありません。

## Usage  

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて指定します。
3. Playbookを実行します。

## Example Playbook

パスワードポリシーを変更する場合、 "roles"ディレクトリに次のRoleを入力してください。  
以下のようなPlaybookを作成してください。 

- フォルダ構成 
~~~
 - playbook
　   ├── hosts/
　   ├── roles/
　   │    └── RHEL/
　   │          └── RH_password-rules
　   │                ├── defaults
　   │                │       main.yml
　   │                ├── meta
　   │                │       main.yml
　   │                ├── tasks
　   │                │       main.yml
　   │                ├── vars
　   │                │       gathering_definition.yml
　   │                └─ README.md
　   └─ linux_password_rules.yml
~~~

- マスターPlaybook サンプル[linux\_password\_rules.yml]  
~~~
#linux_password_rules.yml
　  - name: Linux Modify password policy
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_password-rules
　        VAR_RH_pw_validity_period:
　            pass_max_days： 10
　            pass_min_days： 3
　            pass_warn_age： 1
　        VAR_RH_pw_quality_flag: requisite
　        VAR_RH_pw_quality_arg1:
　          - enforce_for_root
　          - debug
　        VAR_RH_pw_quality_arg2:
　            retry： 3
　            minlen： 10
　        VAR_RH_pw_authentication_flag: requisite
　        VAR_RH_pw_authentication_arg1:
　          - sha512
　        VAR_RH_pw_authentication_arg2:
　            remember: 5
　        tags:
　          - RH_password_rules
~~~

## Running Playbook

- パスワードポリシーを変更するとき,以下のように実行します。  

~~~sh
> ansible-playbook linux_password_rules.yml
~~~

## Evidence Description

エビデンス収集場合は、以下のようなEvidence収集用のPlaybookを作成してください。  

- エビデンスplaybook サンプル[linux\_password\_rules\_evidence.yml]
~~~
　---
　  - hosts: rhel
　    roles:
　      - role: gathering
　        VAR_gathering_definition_role_path: "roles/RHEL/RH_password-rules"
~~~

- マスターPlaybookにエビデンスコマンド追加 サンプル[linux\_password\_rules.yml]
~~~
　---
　  - import_playbook: linux_password_rules_evidence.yml VAR_gathering_label=before
　  - name: Linux Modify keyboard layout and x11 layout
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_password-rules
　        VAR_RH_pw_validity_period:
　            pass_max_days： 10
　            pass_min_days： 3
　            pass_warn_age： 1
　        VAR_RH_pw_quality_flag: requisite
　        VAR_RH_pw_quality_arg1:
　          - enforce_for_root
　          - debug
　        VAR_RH_pw_quality_arg2:
　            retry： 3
　            minlen： 10
　        VAR_RH_pw_authentication_flag: requisite
　        VAR_RH_pw_authentication_arg1:
　          - sha512
　        VAR_RH_pw_authentication_arg2:
　            remember: 5
　        tags:
　          - RH_password_rules
　  - import_playbook: linux_password_rules_evidence.yml VAR_gathering_label=after
~~~

- エビデンス収集結果一覧  
~~~
#エビデンス構成
.
└── 管理対象マシンIPアドレス
　    ├─── file
　    │    └── etc
　    │        ├── login.defs       # パスワードの有効期間に関連する項目を格納する
　    │        └── pam.d
　    │            └── system-auth  # パスワード強度と認証の情報を格納する
　    └─── command
　        ├── 0                     #　cat /etc/pam.d/system-auth からpassword及びpam_pwquality.soの行を抽出
　        ├── 1                     #　cat /etc/pam.d/system-auth からpassword及びpam_unix.soの行を抽出
　        └── results.json          # 各コマンドの情報を格納するJSONファイル
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
