# Ansible研修用リポジトリ
本リポジトリでは下記のAnsibleタスクを用意している。

* httpdインストール
* httpdコンフィグ設定
  * /etc/httpd/conf/httpd.conf,/var/www/html/index.html の編集テンプレートあり。
* httpdリロード
* httpd再起動

## 動作確認環境
* OS  
  * Amazon Linux release 2023.8.20250818 (Amazon Linux)

* カーネル
  * Linux version 6.1.147-172.266.amzn2023.x86_64 (mockbuild@ip-10-0-55-115) (gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-5), GNU ld version 2.41-50.amzn2023.0.3) #1 SMP PREEMPT_DYNAMIC Thu Aug  7 19:30:40 UTC 2025

※[Terraform統合構成：ALB + EC2 + Bastion + Ansible実行サーバ（config-manager）](https://github.com/watanabe-toshi/test-terraform/tree/test-20250730) の
Ansible実行用EC2で動作を想定

## 実行環境の準備
Ansibleを実行するEC2内で下記の手順を実施します。

### (1) Ansibleのインストール状態確認

Ansible のバージョンを確認
```
ansible --version
```

### (2) Git インストール

git をインストール
```
sudo yum install -y git
```

git のバージョンを確認
```
git -v
```

### (3) test-ansible リポジトリのクローン

ec2-userの状態でホームディレクトリに移動して、git cloneを行う。
```
id
cd
git clone https://github.com/stsuda218/test-ansible.git
```

test-ansibleリポジトリの中身を確認
```
cd ~/test-ansible/
ls -l
```

### (4) Amazon EC2 インベントリプラグイン有効化
EC2タグでAnsible実行対象のクライアントサーバを判別させる為、
Ansible Galaxyから ```amazon.aws``` コレクションをインストールする。

```
ansible-galaxy collection install amazon.aws
ansible-galaxy collection list | grep amazon.aws
```

Amazon EC2 インベントリプラグインに必要な boto3 と botocoreをインストールする。
```
sudo yum install -y python3-pip
pip3 install boto3 botocore --user
```

AnsibleインベントリにEC2タグで判別されたサーバがそれぞれ表示されることを確認する。  
※EC2 IAMロールに AmazonEC2ReadOnlyAccess ポリシーが必要
```
ansible-inventory -i aws_ec2.yml --graph
```

### (5) Ansible Playbook 実行
Ansible Playbook を実行する。
```
ansible-playbook <Playbookファイル> --tags=<Playbookタスクタグ> -CD
```
* <Playbookファイル> は playbookとして記述された.ymlファイルを指定する。
* --tags=<Playbookタスクタグ> は Playbook内に定義されたタグを指定する。
* -Cオプション は Playbook実行のテストが行えるオプション。実際に変更は及ばさずに実行結果の検査が行える。
* -Dオプション は Playbook実行時に詳細な差分が表示されるオプション

コマンド例①： Webサーバに対してhttpdをインストールするチェックを行う
```
ansible-playbook Group_Web_Server.yml --tags=httpd_install -CD
```

コマンド例②： Webサーバに対してhttpdのリロードを行う
```
ansible-playbook Group_Web_Server.yml --tags=httpd_reload -D
```

下記のコマンドでPlaybookファイルがどのタスクを実行するよう定義されているかが確認できる。
```
ansible-playbook --list-tasks Group_Web_Server.yml
```


## 参考

https://porain.hatenablog.com/entry/2021/10/06/203055  
https://zenn.dev/ohsawa0515/articles/enable-ec2-dynamic-inventory-by-ansible
https://blog.denet.co.jp/ansiblejinja2/
