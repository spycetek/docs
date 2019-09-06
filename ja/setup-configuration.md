# App 設定

- [Websサーバ設定](#webserver-configuration)
    - [Apache設定](#apache-configuration)
    - [Nginx設定](#nginx-configuration)
    - [Lighttpd設定](#lighttpd-configuration)
    - [IIS設定](#iis-configuration)
- [アプリケーション設定](#app-configuration)
    - [デバッグモード](#debug-mode)
    - [セーフモード](#safe-mode)
    - [CSRF保護](#csrf-protection)
    - [最新版のアップデート](#edge-updates)
    - [パブリックフォルダの使用](#public-folder)
- [環境構築](#environment-config)
    - [基本環境の定義](#base-environment)
    - [ドメイン駆動環境](#domain-environment)
    - [DotEnv環境への変換](#dotenv-configuration)

Octoberの全てのコンフィグファイルは **config/** ディレクトリに保存されています。各オプションはドキュメント化されているので、ご自由に目を通して、利用可のなオプションに慣れてください。

<a name="webserver-configuration"></a>
## Webサーバ設定

OctorberにはWebサーバに適用されている基本設定があります。一般的なサーバとその設定は下記になります。

<a name="apache-configuration"></a>
### Apache設定

Apacheを使用している場合、下記の追加が必要です:

1. mod_rewriteをインストールしてください
1. AllowOverrideオプションをオンにしてください

`.htaccess`ファイルの下記コメントを外す必要がある場合があります:

    ##
    ## You may need to uncomment the following line for some hosting environments,
    ## if you have installed to a subdirectory, enter the name here also.
    ##
    # RewriteBase /

サブディレクトリをインストールしている場合は、サブディレクトリの名前の追加も必要です:

    RewriteBase /mysubdirectory/

<a name="nginx-configuration"></a>
### Nginx設定

Nginxで設定するには少しだけ設定が必要です。

`nano /etc/nginx/sites-available/default`

下記コードを **server** セクションで使用してください。Octorberをサブディレクトリにインストールしている場合は、ロケーションディレクティブの最初の `/` をOctorberがインストールされているディレクトリに置き換えてください:

    location / {
        # Let OctoberCMS handle everything by default.
        # The path not resolved by OctoberCMS router will return OctoberCMS's 404 page.
        # Everything that does not match with the whitelist below will fall into this.
        rewrite ^/.*$ /index.php last;
    }

    # Pass the PHP scripts to FastCGI server
    location ~ ^/index.php {
        # Write your FPM configuration here

    }

    # Whitelist
    ## Let October handle if static file not exists
    location ~ ^/favicon\.ico { try_files $uri /index.php; }
    location ~ ^/sitemap\.xml { try_files $uri /index.php; }
    location ~ ^/robots\.txt { try_files $uri /index.php; }
    location ~ ^/humans\.txt { try_files $uri /index.php; }

    ## Let nginx return 404 if static file not exists
    location ~ ^/storage/app/uploads/public { try_files $uri 404; }
    location ~ ^/storage/app/media { try_files $uri 404; }
    location ~ ^/storage/temp/public { try_files $uri 404; }

    location ~ ^/modules/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/behaviors/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/behaviors/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/widgets/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/widgets/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/formwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/formwidgets/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/reportwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/reportwidgets/.*/resources { try_files $uri 404; }

    location ~ ^/plugins/.*/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/behaviors/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/behaviors/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/reportwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/reportwidgets/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/formwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/formwidgets/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/widgets/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/widgets/.*/resources { try_files $uri 404; }

    location ~ ^/themes/.*/assets { try_files $uri 404; }
    location ~ ^/themes/.*/resources { try_files $uri 404; }

<a name="lighttpd-configuration"></a>
### Lighttpd設定

Lighttpdを使用している場合、下記設定を使用することができます。お好きなエディターでコンフィグファイルを開いてください。

`nano /etc/lighttpd/conf-enabled/sites.conf`

下記コードをエディターに貼り付け、 **host address** と **server.document-root** をプロジェクトに合うように変更してください。

    $HTTP["host"] =~ "domain.example.com" {
        server.document-root = "/var/www/example/"

        url.rewrite-once = (
            "^/(plugins|modules/(system|backend|cms))/(([\w-]+/)+|/|)assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
            "^/(system|themes/[\w-]+)/assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
            "^/storage/app/uploads/public/[\w-]+/.*$" => "$0",
            "^/storage/temp/public/[\w-]+/.*$" => "$0",
            "^/(favicon\.ico|robots\.txt|sitemap\.xml)$" => "$0",
            "(.*)" => "/index.php$1"
        )
    }

<a name="iis-configuration"></a>
### IIS設定

Internet Information Services (IIS)を使用している場合、 **web.config** コンフィグファイルで下記を使用することができます。

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <system.webServer>
            <rewrite>
                <rules>
                    <clear />
                    <rule name="OctoberCMS to handle all non-whitelisted URLs" stopProcessing="true">
                       <match url="^(.*)$" ignoreCase="false" />
                       <conditions logicalGrouping="MatchAll">
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/.well-known/*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/uploads/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/media/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/temp/public/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/themes/.*/(assets|resources)/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/plugins/.*/(assets|resources)/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/modules/.*/(assets|resources)/.*" negate="true" />
                       </conditions>
                       <action type="Rewrite" url="index.php" appendQueryString="true" />
                   </rule>
                </rules>
            </rewrite>
        </system.webServer>
    </configuration>

<a name="app-configuration"></a>
## アプリケーション設定

<a name="debug-mode"></a>
### デバッグモード

デバッグの設定は `config/app.php` コンフィグファイルの `debug` パラメータにあり、デフォルトで有効になっています。

この設定を有効にすると、他のデバッグ機能とともに発生した詳細なエラーメッセージが表示されます。 開発中は便利ですが、実際の運用サイトで使用する場合は、デバッグモードを常に無効にする必要があります。 これにより、潜在的な機密情報がエンドユーザーに表示されるのを防ぎます。

デバッグモード有効時は下記機能を使用します:

1. [詳細エラーページ](../cms/pages#error-page)が表示されます
1. ユーザ認証失敗には特定の理由を提示します
1. [結合されたアセット](../markup/filter-theme)はデフォルトでは縮小されません
1. [セーフモード](#safe-mode)はデフォルトで無効になっています

> **重要**: 本番環境では、常に `app.debug`設定を` false`に設定してください。

<a name="safe-mode"></a>
### セーフモード

セーフモードの設定は `config/cms.php` コンフィグファイル `enableSafeMode` パラメータにあり、デフォルト値は `null`になっています。

セーフモードが有効になっている場合、セキュリティ上の理由から、CMSテンプレートのPHPコードセクションは無効になっています。 `null`に設定すると、[デバッグモード](#debug-mode)が無効になっているときにセーフモードがオンになります。

<a name="csrf-protection"></a>
### CSRF保護

Octorberはクロスサイトリクエストフォージェリ（CSRF）からアプリケーションを保護する簡単な方法を提供しています。まず、ユーザーセッションにランダムトークンが配置されます。 次に[opening form tagが使用される](../services/html#form-tokens)と、トークンがページに追加され、各リクエストで送信されます。

CSRF保護はデフォルトで有効になっていますが、 `config/cms.php`コンフィグファイルの `enableCsrfProtection`パラメータで無効にできます。

<a name="edge-updates"></a>
### 最新版のアップデート

Octorberプラットフォームと一部のマーケットプレイスプラグインは、プラットフォームの全体的な安定性と整合性を確保するために2段階で変更を実装します。 つまりデフォルトの *stable build* に加えて *test build* があるということです。

`config / cms.php`設定ファイルの` edgeUpdates`パラメーターを変更することで、マーケットプレイスからのtest buildを優先するようにプラットフォームに指示できます。

    /*
    |--------------------------------------------------------------------------
    | Bleeding edge updates
    |--------------------------------------------------------------------------
    |
    | If you are developing with October, it is important to have the latest
    | code base, set this value to 'true' to tell the platform to download
    | and use the development copies of core files and plugins.
    |
    */

    'edgeUpdates' => false,

> **備考:** プラグインディベロッパーの場合、プラグイン設定ページから、マーケットプレイスにリストされているプラグインの **Test updates** を有効にすることをお勧めします。

> **備考:**  [Composer](../console/commands#console-install-composer) を使用してアップデート管理をする場合は、開発ブランチから直接アップデートをダウンロードするために、 `composer.json`ファイルのデフォルトのOctoberCMS要件を下記のように置き換えてください。

    "october/rain": "dev-develop as 1.0",
    "october/system": "dev-develop",
    "october/backend": "dev-develop",
    "october/cms": "dev-develop",
    "laravel/framework": "5.5.*@dev",

<a name="public-folder"></a>
### パブリックフォルダの使用

本番環境での最高のセキュリティのために、 **public/** フォルダーを使用してパブリックファイルのみにアクセスできるようにWebサーバーを設定できます。 最初に、 `october：mirror`コマンドを使用してパブリックフォルダーを生成する必要があります。

    php artisan october:mirror public/

これにより、プロジェクトの基本のディレクトリに **public/** という新しいディレクトリが作成されます。ここから、この新しいパスを *wwwroot* と呼ばれるホームディレクトリとして使用するようにWebサーバー設定を修正する必要があります。

> **備考**: 上記のコマンドは、システム管理者または *sudo* 権限で実行する必要がある場合があります。 また、各システムの更新後、または新しいプラグインがインストールされたときに実行する必要があります。

<a name="environment-config"></a>
## 環境構築

<a name="base-environment"></a>
### 基本環境の定義

多くの場合、アプリケーション実行中の環境に基づいて異なる設定値を使用すると便利です。そのためにはデフォルトで **production** に設定されている `APP_ENV`環境変数を設定します。 この値を変更するには、2つの一般的な方法があります:

1. Webサーバで直接 `APP_ENV` 値を設定します。

    例えばApacheでは、この行を `.htaccess` または `httpd.config` ファイルに追加することができます:

        SetEnv APP_ENV "dev"

2. 下記内容の **.env** ファイルをルートフォルダに作成する:

        APP_ENV=dev

上記の両例では、環境は新しい値 `dev`に設定されます。コンフィグファイルは **config/dev** で作成され、アプリケーションの基本設定はオーバーライドされます。

例えば `dev`環境でのみ異なるMySQLデータベースを使用するには、下記内容を使用した **config / dev / database.php** というファイルを作成します。:

    <?php

    return [
        'connections' => [
            'mysql' => [
                'host'     => 'localhost',
                'port'     => '',
                'database' => 'database',
                'username' => 'root',
                'password' => ''
            ]
        ]
    ];

<a name="domain-environment"></a>
### ドメイン駆動環境

Octoberは、特定のホスト名によって検出された環境の使用をサポートしています。 これらのホスト名は、環境設定ファイル、例えば **config / environment.php** に配置できます。

以下のこのファイルの内容を使用すると、　**global.website.tld**　を介してアプリケーションにアクセスした時、環境は `global`に設定され、他の環境も同様に設定されます。

    <?php

    return [
        'hosts' => [
            'global.website.tld' => 'global',
            'local.website.tld' => 'local',
        ]
    ];

<a name="dotenv-configuration"></a>
### DotEnv環境への変換

[基本環境設定](#base-environment) の代替として、コンフィグファイルを使用する代わりに、環境に共通の値を配置できます。 次に、 [DotEnv](https://github.com/vlucas/phpdotenv) 構文を使用してコンフィグにアクセスします。 `october：env`コマンドを実行して、一般的な設定値を環境に移動します:

    php artisan october:env

これにより、プロジェクトのルートディレクトリに **.env** ファイルが作成され、 `env`ヘルパー関数を使用するようにコンフィグファイルが変更されます。 最初の実引数には環境で見つかったキー名が含まれ、2番目の実引数にはオプションのデフォルト値が含まれます。

    'debug' => env('APP_DEBUG', true),

アプリケーションを使用するディベロッパーやサーバーごとに異なる環境設定が必要になる可能性があるため、 `.env`ファイルをアプリケーションのソース管理にコミットしないでください。

`.env`ファイルが本番環境で一般に非公開であることも重要です。 これを実現するには、[パブリックフォルダ](#public-folder)の使用を検討する必要があります
