# Ansible Role: IIS\_Install
===============================

## Description

本Ansible RoleはWebサーバソフトウェアである"IIS"のインストールを行います。
対象バージョンは以下のバージョンです。

- Windows Server 2016 Internet Information Server10

IISインストール時に、".NET Framework 3.5 Features"や".NET Framework 4.6 Features"の
インストールを行うこともできます。

## Supports

- 管理マシン(Ansibleサーバ)
  - Linux系OS（RHEL/CentOS）
  - Ansible バージョン2.6, 2.9
  - Python バージョン2.6, 2.7, 3.6

- 管理対象マシン(インストール対象マシン)
  - Windows Server 2016
  - PowerShell 4.0

## Requirements
- 管理マシン(Ansibleサーバ)
  - 管理対象マシンへWinRM接続できること
  - WinRM接続設定は次のサイトを参考にすること  
    ・https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html#winrm-setup
- 管理対象マシン(インストール対象マシン)
  - PowerShell 4.0を利用できること
  - ファイアフォールが適切に設定されていること

## Role Variables
### Mandatory Variables

以下の変数は必ず指定します。

- WinRM設定（group\_varsもしくはhost\_varsに設定）
  * ''ansible\_user'': ユーザ名(string)
  * ''ansible\_password'': パスワード(string)
  * ''ansible\_port'': ポート(number)
  * ''ansible\_connection'': 接続方式(string)
  * ''ansible\_winrm\_server\_cert\_validation'': 暗号化有無(string)

- 共通変数
  * ''VAR\_IIS\_OS\_Version'': 管理対象マシンのOSバージョンを設定します (string, 例："Windows Server 2016")

- IISインストールの関連変数
  * ''VAR\_IIS\_INSTALLTYPE'': インストール方法を設定します (string, 例："CUSTOM" あるいは "DEFAULT" あるいは "NONE")

### Optional variables

以下の変数は任意で指定します。

- IISインストールの関連変数
  * ''VAR\_IIS\_FEATURE\_NAME'': IISの必須機能以外、インストールしたい機能を設定します  
    (string, "Web-Mgmt-Console,Web-Scripting-Tools,Web-Mgmt-Service")
  * ''VAR\_IIS\_SOURCEPATH'': 管理対象マシンにIIS ソースファイルのパスを設定します（「.NET Framework 3.5 Features」がインストールする必要の場合指定必須）  
    (string, 例："C:\\\\Temp\\\\")
  * ''VAR\_Installer\_Name'': 「.NET Framework 3.5 Features」のインストーラ物件のファイル名を設定します  
    (string, 例："microsoft-windows-netfx3-ondemand-package.cab")  
  * ''VAR\_Installer\_URL'': インストールパッケージのファイルのWeb格納URLを設定します（この値が設定されない場合、win\_copyでファイルを転送します）  
    (string, 例："http://192.168.1.18:8080")  
  * ''VAR\_IIS\_State'': VAR_IIS_INSTALLTYPEを"NONE"に設定すると、IISサービス起動/停止のみを行えます（START：起動（デフォルト）；STOP：停止）、  
    (string, 例："START")  

## Dependencies

特にありません。

## Usage

1. 本Roleを用いたPlaybookを作成します。  
2. 「.NET Framework 3.5 Features」のインストーラ物件を取得します。  
  - Winodws Server 2016のインストールCDに「sources\sxs」フォルダから「microsoft-windows-netfx3-ondemand-package.cab」ファイルを取得します。  
  - ファイル名は変数「VAR\_Installer\_Name」で指定します。  
3. 「microsoft-windows-netfx3-ondemand-package.cab」ファイルを、管理マシン上に配置します。  
  - コピー先は{{本ソースのフォルダ}}/playbook/roles/IIS\_Install/files です。  
  - win\_get\_urlでファイル転送の場合、以下の方法でインストールパッケージを共有します：  
    管理マシンにコマンドラインで以下のように実行して、 「http://管理マシン:8080/」 でアクセスして、ファイルをダウンロードできます。  

    ~~~sh  
    cd {{本ソースのフォルダ}}/playbook/roles/IIS_Install/files  
    nohup python -m SimpleHTTPServer 8080 &  
    ~~~  

4. 必須変数を指定します。  
5. 任意変数を必要に応じて指定します。  
6. Playbookを実行します。  


## Example Playbook

インストールとすべての設定をする場合は、提供した以下のRoleを"roles"ディレクトリ配置したうえで、
以下のようなPlaybookを作成してください。

- フォルダ構成
~~~
  - group_vars/
    ・ server1
    ・ server2
  - host_vars/
    ・ host1
    ・ host2
  - roles/
    ・ IIS_Install/
    ・ IIS_Setup/
  - IISinstall.yml
  - IISsetup.yml
  - conf.yml
  - hosts
~~~

- マスターPlaybook サンプル「IISinstall.yml」
~~~
# IISinstall.yml
 - name: install IIS10
   hosts: all
   gather_facts: true
   roles:
     - role: IIS_Install
       VAR_IIS_INSTALLTYPE: DEFAULT
       tags:
         - IIS_Install
~~~


## Running Playbook
- extra-vars を利用する場合の実行例
~~~sh
ansible-playbook IISinstall.yml  -i hosts --extra-vars="@conf.yml"
~~~

- group_vars を利用する場合の実行例  
 group_vars で指定したグループ名が webserver1 の場合
~~~sh
ansible-playbook IISinstall.yml  -i hosts -l webserver1
~~~

- host_vars を利用する場合の実行例  
 host_vars で指定したグループ名が server1 の場合
~~~sh
ansible-playbook IISinstall.yml  -i hosts -l server1
~~~

- 本Roleのみを実行する場合は、 --tags "IIS_Install" を付け加える
~~~sh
ansible-playbook IISinstall.yml  -i hosts --extra-vars="@conf.yml" --tags "IIS_Install"
~~~


# Copyright
---------
Copyright (c) 2019 NEC Corporation

# Author Information
------------------
NEC Corporation
