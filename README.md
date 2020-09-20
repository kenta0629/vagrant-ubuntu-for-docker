# vagrant-ubuntu-for-docker解説

## 事前準備
下記をインストールしておくこと

- VirtualBox
- Vagrant
- Git

※プロキシを介してVagrantを使う場合はシステム環境変数に下記を追加すること
- HTTP_PROXY : http://<プロキシホスト名>:<プロキシポート>
- HTTPS_PROXY : http://<プロキシホスト名>:<プロキシポート>


## Vagrantのプラグインをインストール(※後からでも可)

### vagrant-vbguestをインストールする
`vagrant plugin install vagrant-vbguest` を実行する。
### プロキシを介している場合はvagrant-proxyconfをインストールする
`vagrant plugin uninstall vagrant-proxyconf` を実行する。

## リポジトリクローン
```
  git clone git@github.com:kenta0629/vagrant-ubuntu-for-docker.git
```

## 共有フォルダの作成
```
mkdir src
```

## Vagrantfileの修正

### プロキシ設定
- プロキシを介している場合、下記のようにプロキシ情報を設定してください。
- プロキシを介していない場合、プロキシのプラグインをインストールしていなければ修正の必要ないです。プラグインをインストールしている場合は「config.proxy.enabled  = false」にしてください。
```
  ...
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.enabled  = true  # => true; all applications enabled, false; all applications disabled
    config.proxy.http     = "http://<プロキシホスト名>:<プロキシポート>"
    config.proxy.https    = "http://<プロキシホスト名>:<プロキシポート>"
    config.proxy.no_proxy = "localhost,127.0.0.1"
  end
  ...
```

### その他設定
特に設定する必要はありませんが、
ポートフォワードを設定したい、IPアドレスを変更したい場合は想定する環境に合わせて設定してください。
また、Windowsで起動することを想定しているため`Encoding.default_external = 'SJIS'`と設定しています。エンコードでエラーが発生する場合はコメントアウトなり対応するエンコードに設定してください。

## 起動
Vagrantfileの配置されているディレクトリで`vagrant up`で起動します。

※初回起動時はprovisionが起動されdocker,docker-compose等がインストールされます。

### dockerのコンテナ起動

vagrant起動時にコンテナも起動させるように設定しています。
以下のログが流れたら完了
```
==> default: Creating docker-for-laravel_db_1 ...
Creating docker-for-laravel_db_1 ... done
==> default: Creating docker-for-laravel_app_1 ...
Creating docker-for-laravel_app_1 ... done
==> default: Creating docker-for-laravel_web_1 ...
Creating docker-for-laravel_web_1 ... done
```

### 起動で失敗した場合

上記までの処理で起動が失敗した場合は修正後に以下コマンド入力
```
vagrant provision
```

## ゲストOSへアクセス

```
vagrant ssh
```

Vagrantfileで設定したIP（デフォルト: 192.168.33.10）へSSHでアクセスできます。
(ID / PW : vagrant / vagrant)


## コンテナ確認

```
docker ps
```
docker-for-laravel_db_1, docker-for-laravel_app_1, docker-for-laravel_web_1が表示されれば正常です。

## コンテナログイン

```
docker exec -it {appコンテナID} bash
```

Laravelのインストールをするため、コンテナにログインします。

## Laravelインストール

```
composer global require laravel/installer
composer create-project --prefer-dist laravel/laravel src
```

※それぞれ1行ずつ流してください

## 階層合わせ

```
cd src
mv .[^\.]* ../
rmdir src
```

現在のままだと/src/srcの状態になっていて共有できないので、/
src配下にインストールしたファイル、ディレクトリを移動します。
※要修正箇所
※それぞれ1行ずつ流してください

## ブラウザアクセス確認

URLに192.168.33.10を入力してLaravelのWelcomページが表示されたら完了です。

## 停止

```
vagrant halt
```

Vagrantfileの配置されているディレクトリで停止します。

