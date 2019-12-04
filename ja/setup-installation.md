# インストール

- [システムの最低条件](#system-requirements)
- [インストーラを使ったインストール](#wizard-installation)
    - [トラブルシューティング](#troubleshoot-installation)
- [コマンドラインを使ったインストール](#command-line-installation)
- [インストール後の手順](#post-install-steps)
    - [インストールファイルの削除](#delete-install-files)
    - [コンフィグレーションのレビュー](#config-review)
    - [スケジューラーのセットアップ](#crontab-setup)
    - [キューワーカーのセットアップ](#queue-setup)

Octoberをインストールするには2つの方法があります。 [インストーラー](#wizard-installation) を使う方法か、 [コマンドラインでのインストール](../console/commands#console-install) の説明に沿ってインストールする方法があります。 インストールする前に、サーバが条件と合っているか確認してください。

<a name="system-requirements"></a>
## システムの最低条件

October CMSには下記のウェブホスティングサーバが必要です。

1. PHP version 7.0 or higher
1. PDO PHP Extension
1. cURL PHP Extension
1. OpenSSL PHP Extension
1. Mbstring PHP Library
1. ZipArchive PHP Library
1. GD PHP Library

PHP JSON と XML extensionsのインストールが必要なディストリビューションもあります。例えばUbuntuの場合、それぞれ `apt-get install php7.0-json` と `apt-get install php7.0-xml` でインストールできます。

SQLサーバデータベースエンジンを使う場合は、[group concatenation](https://groupconcat.codeplex.com/)user-defined aggregateのインストールが必要です。

<a name="wizard-installation"></a>
## インストーラを使ったインストール

Octorberをインストールするにはインストーラを使用するのが推奨されています。 コマンドラインを使ってインストールするよりもシンプルで、特別なスキルも必要ありません。

1. 空のサーバー上のディレクトリを準備してください。これがサブディレクトリ、ドメインルート、サブドメインになります。
1. インストーラの圧縮ファイルを[ダウンロード](http://octobercms.com/download)します。
1. 準備したディレクトリでインストーラの圧縮ファイルを解凍します。
1. インストールディレクトリとそのすべてのサブディレクトリとファイルに対する書き込み権限を付与します。
1. Webブラウザーでinstall.phpスクリプトに移動します。
1. インストール手順に従って進めます。

![image](https://github.com/octobercms/docs/blob/master/images/wizard-installer.png?raw=true) {.img-responsive .frame}

<a name="troubleshoot-installation"></a>
### トラブルシューティング

1. **アプリケーションファイルのダウンロード時にError 500が表示される**: Webサーバのタイムアウト時間を延長するか設定を無効にする必要があります。例えばApache's FastCGIでは `-idle-timeout` オプションが30秒に設定されていることがあります。

1. **アプリケーションを開いた時にブランクの画面が表示される**: `/storage`ファイルとフォルダはWebサーバーに対して書き込み可能である必要があるため、アクセス許可が正しく設定されているかを確認してください。

1. **エラーコード"liveConnection"が表示される**: インストーラはポート80を使用してインストールサーバへの接続テストを行います。WebサーバーがPHP経由でポート80に接続できることを確認してください。サーバファイアウォールの設定でよく見られるため、 ホスティングプロバイダにお問い合わせください。

1. **バックエンドエリアに"Page not found" (404)と表示される**: アプリケーションがデータベースを見つけることができない場合に、バックエンドの404ページが表示されます。[デバックモード](../setup/configuration#debug-mode)を有効にして、もととなっているエラーメッセージを確認してください。

> **備考:** インストールログの詳細は `install_files/install.log` ファイルにあります。

<a name="command-line-installation"></a>
## コマンドラインを使ったインストール

コマンドラインを使い慣れている、または、コンポーザを使いたい場合は、 [コンソールインターフェースページ](../console/commands#console-install)にCLIのインストール手順があります。

<a name="post-install-steps"></a>
## インストール後の手順

インストール完了後に、いくつかの設定が必要な場合があります。

<a name="delete-install-files"></a>
### インストールファイルの削除

[インストーラ](#wizard-installation)を使用した場合、セキュリティ上の理由からインストールファイルを削除してください。Octorberは自動的にファイルを削除しないため、下記のファイルとディレクトリを手動で削除してくださ。

    install_files/      <== Installation directory
    install.php         <== Installation script

<a name="config-review"></a>
### コンフィグレーションのレビュー

コンフィグレーションファイルは、アプリケーションの **config** ディレクトリに保存されます。各ファイルには各設定についての説明が含まれていますが、状況に応じて利用可能な[一般的な設定オプション](../setup/configuration)を確認することが重要です。

例えば、本番環境では [CSRF 保護](../setup/configuration#csrf-protection)を有効にすることができ、開発環境では [最新版への更新](../setup/configuration#edge-updates)を有効にすることができます。

ほとんどの設定はオプションですが、本番環境では[デバックモード](../setup/configuration#debug-mode) を無効にすることを強くお勧めします。 セキュリティ強化のために[パブリックフォルダ](../setup/configuration#public-folder) の使用も可能です。

<a name="crontab-setup"></a>
### スケジューラーのセットアップ

*スケジジュールされたタスク* が正しく動作するには、次のCronエントリをサーバーに追加する必要があります。 Crontabの編集は、一般的にコマンド `crontab -e`で実行されます。

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

必ず **/path/to/artisan** を、Octorberのルートディレクトリにある *artisan* ファイルへの絶対パスに置き換えてください。このCronは毎分コマンドスケジューラーをコールします。そしてスケジュールされたタスクを算定して、期限のあるタスクを実行します。

> **備考**: これを `/etc/cron.d` に追加する場合、`* * * * *`のすぐ後にユーザを指定する必要があります。

<a name="queue-setup"></a>
### キューワーカーのセットアップ

*queued jobs* を処理するための外部キューをオプションで設定できます。デフォルトでは、これらはプラットフォームによって非同期的に処理されます。 この動作は`config/queue.php`の`default`のパラメーターを設定することで変更できます。

`database`キュードライバーを使用する場合、`php artisan queue:work --once`コマンドをCrontabに追加して、キュー内のジョブを１つずつ処理することをお勧めします。
