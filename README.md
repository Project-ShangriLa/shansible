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

- Vagrant 1.8+
- VirtualBox 5+

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

ゲストOS起動後に、rsyncの自動同期を行う場合は、以下のコマンドをホストOSで実行してください。

    $ vagrant rsync-auto

# ShangriLa Ansible Playbook

ShangriLaの実行環境を構築する、AnsibleのPlaybookです。

    shansible/ansible

## 実行要求

Vagrant以外から実行する場合は、実行環境にAnsible 1.9+ をインストールしてください。

また、```/etc/ansible/ansible.cfg```のhost_key_checkingをアンコメントし、
host_key_checkingをFalseに変更して、SSH時のknown_hostsチェックを無効にしてください。

```
host_key_checking = False
```


## ローカル環境の構築手順

開発環境のサーバに```shansible/ansible```をコピーし、当該ディレクトリ上で以下のコマンドを実行してください。

    ansible-playbook -i local local.yml

## 構築されるリポジトリ

- [sora-playframework-scala](https://github.com/Project-ShangriLa/sora-playframework-scala)

## 実行するDDL

- [shangrila](https://github.com/Project-ShangriLa/shangrila)
