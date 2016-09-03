# Shansible

ShangriLaが動作するサーバ群を構築するプロジェクトです。  
本プロジェクトは以下の2つで構成されています。

- Vagrant
- Ansible


# Vagrant

CentOS 7の仮想OS上に、ShangriLaプロジェクトのローカル開発環境を構築します。  
構築はVagrantとVirtualBoxを用いて行います。

ホストOSには40GB以上のディスク領域が必要です。
- https://atlas.hashicorp.com/centos/boxes/7


## 動作説明

Provisioningで以下のAnsible Playbookが実行されます。  
※ホストOSにAnsibleは不要です。  

    shansible/ansible/local.yml

実行後に作成されるShangriLaの各リポジトリは、以下のディレクトリに配置されます。

    shansible/repositories


## 実行要件

以下のソフトウェアとVagrantのプラグインをホストOSにインストールしてください。

### ソフトウェア

- Vagrant 1.8.4
- VirtualBox 5.0.X

### Vagrantプラグイン

- vagrant-vbguest
- vagrant-reload

Vagrantプラグインのインストール手順は以下になります。

```
$ vagrant plugin install <プラグイン名>
```

### for Windows Users

ホストOSにWindowsを利用する場合は、追加で以下の作業を行ってください。

#### RSyncのインストール

CygwinでRSyncをインストールしてください。  
Cygwinを利用しない場合は、Chocolateyでのインストールがお奨めです。  
Chocolateyをインストール後、以下のコマンドでインストールできます。

```
> choco install rsync
```

#### 環境変数の追加

環境変数に以下を追加してください。

- ```PATH```に```C:\ProgramData\chocolatey\lib```を追加（;区切り）
- ```CYGWIN```に```nodosfilewarning```を追加（スペース区切り）

#### Vagrantファイルの修正

Vagrant 1.8.1時点でのエラー回避のため、インストールしたVagrantのファイルを編集します。

##### 1. helper.rb

- ファイルの場所

```
HashiCorp\Vagrant\embedded\gems\gems\vagrant-1.8.1\plugins\synced_folders\rsync\helper.rb
```

48行目のhostpathの設定処理を以下のように変更する。
```rb
#hostpath = Vagrant::Util::Platform.cygwin_path(hostpath)
hostpath = "/cygdrive" + Vagrant::Util::Platform.cygwin_path(hostpath)
```

77-79行目をコメントアウト

```rb
rsh = [
  "ssh -p #{ssh_info[:port]} " +
  proxy_command +
#  "-o ControlMaster=auto " +
#  "-o ControlPath=#{controlpath} " +
#  "-o ControlPersist=10m " +
  "-o StrictHostKeyChecking=no " +
  "-o IdentitiesOnly=true " +
  "-o UserKnownHostsFile=/dev/null",
  ssh_info[:private_key_path].map { |p| "-i '#{p}'" },
].flatten.join(" ")
```

##### 2. guest.rb

- ファイルの場所

```
HashiCorp\Vagrant\embedded\gems\gems\vagrant-1.8.1\plugins\provisioners\ansible\config\guest.rb
```

41行目のremote_pathの設定処理を以下のようにを変更する。

```rb
#remote_path = Pathname.new(path).expand_path(@provisioning_path)
remote_path = File.expand_path(path, @provisioning_path)

# Remove drive letter if running on a Windows host
remote_path = remote_path.gsub(/^[a-zA-Z]:/, "")
```

※これは以下で修正済みのため、最新バージョンでは修正されている可能性があります。
- https://github.com/mitchellh/vagrant/commit/07f3d0b00dabc37281a01c6776eed22daeea7066


## 実行手順

ホストOSのコンソールで、以下のコマンドを実行してください。  

    $ vagrant up

rsyncの自動同期を行う場合は、ゲストOS起動後に以下のコマンドをホストOSで実行してください。

    $ vagrant rsync-auto


### 実行時間

すべてのタスクが完了するまで、約1時間ほどかかります。  


# ShangriLa Ansible Playbook

ShangriLaの実行環境を構築する、AnsibleのPlaybookです。

    shansible/ansible


## 実行要求

Vagrant以外から実行する場合は、実行環境にAnsible 2.0+ をインストールしてください。


### Ansible 2.1.0を使う場合の注意

Ansible 2.1.0ではunarchiveモジュールにバグが有るため、本playbookでunarchiveを利用している箇所がエラーになります。
回避するためには、以下のようにenvironmentを追加してください。

```yml
# TODO:environmentはAnsible2.1.0のバグ回避用
- name: unarchive the compressed Ant binaries
  unarchive: "copy=no src={{ src_dir }}/apache-ant-{{ ant_version }}-bin.tar.gz dest=/usr/local creates=/usr/local/apache-ant-{{ ant_version }}"
  environment:
    LANG: "C"
    LC_ALL: "C"
    LC_MESSAGES: "C"
  tags: ant
```


## 開発環境の構築手順

開発環境のサーバに```shansible/ansible```をコピーし、当該ディレクトリ上で以下のコマンドを実行してください。

    ansible-playbook -i local site.yml

※SSHのagent forwardingを使ってgit cloneしているので、GitHubの秘密鍵がssh-addで登録されている必要があります。
vagrant sshしてからansible-playbookコマンドを叩く場合も、ホストOS側でssh-addの実行をお願いします。


## 環境の説明

Playbookの実行により、以下がCentOS 7上に配置されます。

- Jenkins 2系最新
- MySQL 5.7
- OpenJDK 1.8
- sbt 最新
- ant 1.9.7
- git 2.9.3
- ruby 2.3.1
- rails 5
- ShangriLaプロジェクト関連のリポジトリ


### ShangriLaプロジェクトのリポジトリ

group_vars/all.ymlで定義されたapplication_dir配下（デフォルト：/home/vagrant/repositories）に、
ShangriLaプロジェクトの以下のリポジトリが作成されます。

- [sora-playframework-scala](https://github.com/Project-ShangriLa/sora-playframework-scala)
- [shangrila](https://github.com/Project-ShangriLa/shangrila)

shanagrilaのDDL/DMLを、MySQLのDB（anime_admin_development）に対して実行してください。


### MySQL

開発に利用するDBとして、anime_admin_developmentが作成されます。

また、以下のユーザがMySQLに作成されます。

| User   | Password  | Host      |
| :----- | :-------- | :-------- |
| root   | root      | localhost |
| admin  | admin     | localhost, 127.0.0.1, % |

初期設定されるパスワードは、group_vars/all.ymlの
mysql_root_password、mysql_admin_passwordで定義されています。


#### ログファイル

MySQLのログファイルは、以下のディレクトリに配置されます。（group_vars/all.ymlの変数で設定）

    mysql_log_dir: /var/log/mysql

