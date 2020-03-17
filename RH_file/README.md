# Ansible Role: RHEL\RH\_file
=======================================================

## Description
このロールは、RHEL7.xでファイルを作成するために使用されます。

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

### Mandatory variables
~~~
　 * VAR_RH_file_path: '/home/oracle/test.txt'    # 作成したいファイルのフルパスを指定する。
　 * VAR_RH_file_permission:                      # 作成したいファイルのACL情報を指定する。
　       user: 'root'                                 # 作成したいファイルの所属ユーザを指定する。
　       group: 'root'                                # 作成したいファイルの所属グループを指定する。
　       mode: '0644'                                 # 作成したいファイルの権限を指定する。
　 * VAR_RH_file_state:                           # 指定したファイルに対して、作成操作（present）/削除操作（absent）を指定する。
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

指定のファイルを作成/削除する場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、  
以下のようなPlaybookを作成してください。  

- フォルダ構成  
~~~
 - playbook
　  │── hosts/
　  │── roles/
　  │    └── RHEL/
　  │          └── RH_file
　  │                │── defaults
　  │                │       main.yml
　  │                │── tasks
　  │                │       main.yml
　  │                │       file.yml
　  │                └─ README.md
　  └─ linux_file.yml
~~~

- マスターPlaybook サンプル[linux\_file.yml]  
~~~
１）呼出す例：(ファイル作成)
　  - name: Linux File Create
　    hosts: rhel
　    gather_facts: true
　    roles:
　      - role: RHEL/RH_file
　        VAR_RH_file_path: '/root/test/my.txt'
　        VAR_RH_file_permission:
　            user: 'root'
　            group: 'root'
　            mode: '0644'
　        VAR_RH_file_state: 'present'
　        tags:
　          - RH_file
　 
２）呼出す例：(ファイル削除)
　  - name: Linux File Delete
　    hosts:  rhel
　    gather_facts:  true
　    roles:
　      - role: RHEL/RH_file
　        VAR_RH_file_path: '/home/oracle/test.txt'
　        VAR_RH_file_state: 'absent'
　        tags:
　          - RH_file
~~~

## Running Playbook

- 指定のファイルを作成/削除する時、以下のように実行します。

~~~sh
> ansible-playbook linux_file.yml
~~~

## License

## Copyright

Copyright (c) 2017-2018 NEC Corporation

## Author Information

NEC Corporation
