# ansible-wordpress

ansible を 使って ブログ作成ツール WORDPRESS を構築するサンプルスクリプト。

ansible での WordPress 構築部分の元ネタは[ansible-example](https://github.com/ansible/ansible-examples/tree/master/wordpress-nginx) より取得しています。


本サンプルでは、Dockerサーバ上に sshdが起動したCentOS6 の Dockerコンテナを起動し、このコンテナの中に ansible-playbook で WordPress の環境を構築する手順となります。

Docker関連パッケージがインストールされていない場合、Docker関連パッケージインストール用のPlaybook **docker_install.yml** を実行すれば環境が構築できます。

### 本手順で利用するサーバ構成

種類 | IP ADDR | SSH PORT | 備考
--- | --- | --- | ---
ansible-tower | 192.168.99.89 | - | keypair作成・設定対象への配布が必要
docker server | 192.168.99.212 | 22 | group_vars/docker で起動コンテナを指定
container | 192.168.99.212 | 10022 | centos601
container | 192.168.99.212 | 20022 | centos602



### ファイル構成

```
.
└── <ansible-wordpress>
    └── <DockerScripts>
    │  └── <centos6-ssh>
    │      ├── Dockerfile         sshdを起動する CentOS6 の Dockerfile
    │      ├── authorized_keys    Dockerコンテナへ設定する SSH Public Key
    │      └── init.sh            Dockerコンテナ初期起動スクリプト
    ├── <group_vars>
    │   ├── all                   WordPress構築用変数
    │   └── docker                Docker構築用変数
    ├── hosts                     Inventryファイル
    ├── <roles>                   WordPressインストール Playbookの実態
    ├── site.yml                  WordPressインストール 時の実行 Playbook
    ├── docker_images.yml         CentOS6 DockerImage作成用 Playbook
    ├── docker_install.yml        Dockerパッケージインストール用 Playbook
    ├── docker_run.yml            Dockerコンテナ起動 Playbook
    └── docker_rm.yml             Dockerコンテナ停止 Playbook
```

dockerサーバに起動するコンテナ数とポートフォワード用のポート番号は、**group_vars/docker** で指定。起動するコンテナにWordPressを導入するAnsible-Playbookを実行するので、 **hosts** の **[wordpress-server]** にDockerコンテナの設定が必要となります。

### 事前準備

1. スクリプトの取得 (git clone を Ansibleサーバで実施)

		# git clone https://github.com/riry111/ansible-wordpress.git

2. Ansibleサーバにて、SSHキーペアを作成し、配布する
	
	1) キーペアの作成

		# ssh-kyegen -t rsa   <-パスフレーズなしで作成する 
	
	2) 公開鍵 を Dockerサーバ [例:192.168.99.212]に配布
	
		# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.99.212 
	
	3) Docker Image作成元ファイルに 公開鍵 を設定

		# cat ~/.ssh/id_rsa.pub > ansible-wordpress/DockerScripts/centos6-ssh/authorized_keys
		

### Docker環境構築
		
1. Ansibleサーバにて、Dockerインストール用 Playbook実行

	1) dockerサーバのアドレスを hosts に設定する (以下の3か所)
	
		[docker]
		192.168.99.212
		[wordpress-server]
		wp01 ansible_ssh_port=10022 ansible_ssh_host=192.168.99.212
		wp02 ansible_ssh_port=20022 ansible_ssh_host=192.168.99.212
	
	2) Dockerインストール用 Playbookを実行
	
		# ansible-playbook -i hosts ./docker_install.yml

2. CentOS6 Docker Image作成

	1) Ansibleサーバで Docker Image作成用 Playbookを実行
	
		# ansible-playbook -i hosts ./docker_images.yml


### WordPress環境構築
1. Dockerコンテナの起動 <br>

		# ansible-playbook -i hosts ./docker_run.yml

   **[補足]** group_vars/docker ファイルの変数 DOCKER_ENVS で起動するコンテナ数、コンテナ名、ポートフォワード用のポート番号を変更できます。この値を変更した場合は、WordPress導入先が記載されている **hosts** も変更してください。

2. WordPressインストール

		# ansible-playbook -i hosts ./site.yml 

3. ブラウザからWordPressにアクセスします (ポート番号は group_vars/docker で設定)

		http://<Dockerサーバのアドレス>:<ポート番号>

### WordPress環境削除
1. Dockerコンテナ削除

		# ansible-playbook -i hosts ./docker_rm.yml


以　上
