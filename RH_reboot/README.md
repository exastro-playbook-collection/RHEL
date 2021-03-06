# Ansible Role: RHEL\RH\_reboot
=======================================================

## Description
本ロールは、RHEL7.xのサーバ再起動を行います。  
以下の用途で使用することができます。
* 単独で再起動を行いたい場合
* 複数のロールを呼び出し、それぞれで再起動が必要になる可能性があるが、再起動は最後に一度だけ行えば良い場合

なお、Ansible 2.7からrebootモジュールが提供されたため、単独再起動、かつ、Ansible 2.7以上をご利用の場合は、本ロールは使用せず[reboot モジュール](https://docs.ansible.com/ansible/latest/modules/reboot_module.html#reboot-module)をご利用ください。

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

単独再起動の場合、特に変数を設定する必要はありません。  
複数ロール間で再起動を制御する場合、以下の2つの変数を利用します。  

|変数名|意味|
|---|---|
|`VAR_<設定ロール名>_reboot`|各設定ロール内で処理後すぐ再起動させるかを設定する変数(Playbookで設定)|
|`RH_reboot_required`|後続のタスクで再起動が必要かを設定する変数(各設定ロールで設定)|

　**※具体的な使い方は「複数ロール間で再起動を制御する方法」を参照してください。**  

## Dependencies

特にありません。

## Usage

1. 本Roleを用いたPlaybookを作成します。
2. 変数を必要に応じて設定します。
3. Playbookを実行します。

## Example Playbook

### ■単独で再起動を行う方法

本ロールを"roles"ディレクトリに配置して、以下のようなPlaybookを作成してください。

- フォルダ構成
~~~
 - playbook
　  │── hosts
　  │── roles/
　  │    └── RHEL/
　  │          └── RH_reboot
　  │                │── handlers
　  │                │       main.yml
　  │                │── tasks
　  │                │       main.yml
　  │                └─ README.md
　  └─ linux_reboot.yml
~~~

- マスターPlaybook サンプル[linux\_reboot.yml]
    ~~~
    #linux_reboot.yml
      - name: Linux reboot
        hosts: rhel
        gather_facts: false
        roles:
          - role: RHEL/RH_reboot
    ~~~

### Running Playbook
~~~sh
> ansible-playbook linux_reboot.yml
~~~

### ■複数ロール間で再起動を制御する方法

本ロールを"roles"ディレクトリに配置して、以下のようなPlaybookを作成してください。

- フォルダ構成（2つの設定ロール`RH_test1`/`RH_test2`とともに使用する例）
~~~
 - playbook
　  │── hosts
　  │── roles/
　  │    └── RHEL/
　  │          │── RH_test1
　  │          │     │── meta
　  │          │     │       main.yml
　  │          │     │── tasks
　  │          │     │       main.yml
　  │          │     └─ README.md
　  │          │── RH_test2
　  │          │     │── meta
　  │          │     │       main.yml
　  │          │     │── tasks
　  │          │     │       main.yml
　  │          │     └─ README.md
　  │          └── RH_reboot
　  │                │── handlers
　  │                │       main.yml
　  │                │── tasks
　  │                │       main.yml
　  │                └─ README.md
　  └─ linux_test.yml
~~~

- `RH_test1/meta/main.yml` と `RH_test2/meta/main.yml`
    ~~~
    ---
    #meta
        dependencies:
          - {role: RH_reboot, RH_reboot_nop: yes }
    ~~~

- `RH_test1/task/main.yml` と `RH_test2/task/main.yml`
    ~~~
    ---
    #linux test
      # rebootロールの内部用変数を初期化
      - set_fact:
            RH_reboot_required: "{{ RH_reboot_required | d(false) }}"

      # ロールRH_test1/RH_test2の処理
      - name: processing of test role
        #
        # 実際の処理を記載
        #
        register: notify_ret

      # 本ロールの処理により再起動を行うかどうかを判断
      - name: notify handlers
        raw: echo "Reboot ..."
        notify:
          - Run linux reboot command
          - Wait for ssh connection down
          - Wait for ssh connection up
        when:
          - notify_ret is changed
          - VAR_RH_test1_reboot == true         # RH_test1の場合の例

      # 上記のnotify handlers処理が実施された場合はここでhandlerを実行する
      - meta: flush_handlers

      # 本ロールの処理で再起動の必要あり、かつ、再起動を実施しない場合は、
      # 変数RH_reboot_requiredをtrueに設定し、以後のrole/playbookに任せる
      - name: changed required value
        set_fact:                                   # RH_test1の場合の例
            RH_reboot_required: "{{ not VAR_RH_test1_reboot }}"
        when: notify_ret is changed
    ~~~

- linux_test.yml
    ~~~
    ---
      - name: Linux test
        hosts: rhel
        gather_facts: false
        roles:                                # 必要なロールを順次呼び出し、最後にpost_tasksで再起動を実施する例
          - role: RHEL/RH_test1
            VAR_RH_test1_reboot: false    # ロールRH_test1の処理後、再起動をしない例のためfalseに設定
          - role: RHEL/RH_test2
            VAR_RH_test2_reboot: false    # ロールRH_test2の処理後、再起動をしない例のためfalseに設定
        # reboot
        post_tasks:
          - include_role:
                name: RHEL/RH_reboot
            when: RH_reboot_required == true
    ~~~

### Running Playbook
~~~sh
> ansible-playbook linux_test.yml
~~~

## License

## Copyright

Copyright (c) 2017-2019 NEC Corporation

## Author Information

NEC Corporation
