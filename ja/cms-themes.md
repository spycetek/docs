# CMS テーマ

- [導入](#introduction)
- [ディレクトリ構造](#directory-structure)
    - [サブディレクトリ](#subdirectories)
- [テンプレート構造](#template-structure)
    - [設定セクション](#configuration-section)
    - [PHPコードセクション](#php-section)
    - [Twigマークアップセクション](#twig-section)
- [テーマ ロギング](#theme-logging)

<a name="introduction"></a>
## 導入

テーマは、Octoberで構築されたWebサイトまたはWebアプリケーションの見た目を定義します。Octoberのテーマは完全にファイルベースで、例えばGitのどのバージョンでも管理されています。このページはOctoberテーマについてのハイレベル説明書です。 [ページ](pages)、[パーシャル](partials)、[レイアウト](layouts)、[コンテンツファイル](content)の詳細は該当記事をご覧ください。

テーマは初期設定で **/themes** に属するディレクトリです。テーマは下記のオブジェクトを含みます：

オブジェクト | 説明
------------- | -------------
[ページ](pages) | webサイトのページを意味する
[パーシャル](partials) | 再利用可能なHTMLマークアップのチャンクを含む
[レイアウト](layouts) | スキャホールドページを定義する.
[コンテンツファイル](content) | ページやレイアウトとは別に編集できるtext、HTML、[マークダウン](http://daringfireball.net/projects/markdown/syntax)ブロック
**アセットファイル** | images、CSS、JavaScriptの様なリソースファイル

<a name="directory-structure"></a>
## ディレクトリ構造

下記がテーマディレクトリ構造の例です。Octoberの各テーマは個別のディレクトリで表され、通常1つのアクティブなテーマがWebサイトの表示に使用されます。この例は"website"テーマディレクトリを表示しています。.

    themes/
      website/           <=== Theme starts here
        pages/           <=== Pages directory
          home.htm
        layouts/         <=== Layouts directory
          default.htm
        partials/        <=== Partials directory
          sidebar.htm
        content/         <=== Content directory
          intro.htm
        assets/          <=== Assets directory
          css/
            my-styles.css
          js/
          images/

> アクティブなテーマは `config/cms.php` ファイルにある `activeTheme` パラメータで設定される。もしくは、バックエンドページにある設定＞CMS＞フロントエンドのテーマで設定される。

<a name="subdirectories"></a>
### サブディレクトリ

Octoberでは **ページ**、**パーシャル**、**レイアウト**、**コンテンツ** ファイルの一階層のサブディレクトリをサポートしていますが、**assets** ディレクトリは任意の構造を持っています。 これは広大なWebサイトの構成を簡易化しています。下記のディレクトリ構造の例では、**pages** と **partials** ディレクトリには **blog** サブディレクトリが含まれていて、**content** ディレクトリには **home** サブディレクトリが含まれています。

    themes/
      website/
        pages/
          home.htm
          blog/                  <=== Subdirectory
            archive.htm
            category.htm
        partials/
          sidebar.htm
          blog/                  <=== Subdirectory
            category-list.htm
        content/
          footer-contacts.txt
          home/                  <=== Subdirectory
            intro.htm
        ...

サブディレクトリからパーシャルファイルまたはコンテンツファイルを参照するには、テンプレートの名前の前にサブディレクトリの名前を指定します。 サブディレクトリからパーシャルをレンダリングする例：

    {% partial "blog/category-list" %}

> **備考：** テンプレートのパスは常に絶対です。 パーシャルで同じサブディレクトリから別のパーシャルをレンダリングする場合、サブディレクトリの名前を指定する必要があります。

<a name="template-structure"></a>
## テンプレート構造

ページ、パーシャル、レイアウトテンプレートには最大3つのセクションを含めることができます： **設定**、**PHPコード**、**Twigマークアップ**

セクションは `==`シーケンスで区切られます。
例えば:

    url = "/blog"
    layout = "default"
    ==
    function onStart()
    {
        $this['posts'] = ...;
    }
    ==
    <h3>Blog archive</h3>
    {% for post in posts %}
        <h4>{{ post.title }}</h4>
        {{ post.content }}
    {% endfor %}

<a name="configuration-section"></a>
### 設定セクション

設定セクションはパラメータのテンプレートを設定しています。サポートされている設定パラメーターは、さまざまなCMSテンプレート固有のものであり、対応するドキュメント記事で説明されています。設定セクションでは、文字列パラメーター値が引用符で囲まれているシンプルな[INIフォーマット](http://en.wikipedia.org/wiki/INI_file)を使っています。ページテンプレートの設定セクションの例：

    url = "/blog"
    layout = "default"

    [component]
    parameter = "value"

<a name="php-section"></a>
### PHPコードセクション

PHPセクションのコードは、テンプレートがレンダリングされる前に毎回実行されます。PHPセクションはすべてのCMSテンプレートのためのオプションであり、その内容は定義されているテンプレートタイプによって異なります。PHPコードセクションは、テキストエディターで構文のハイライトを有効にするための開始および終了PHPタグをオプションで含んでいます。開始タグと終了タグは、常にセクションを区切る記号 `==`とは異なる行で指定する必要があります。

    url = "/blog"
    layout = "default"
    ==
    <?
    function onStart()
    {
        $this['posts'] = ...;
    }
    ?>
    ==
    <h3>Blog archive</h3>
    {% for post in posts %}
        <h4>{{ post.title }}</h4>
        {{ post.content }}
    {% endfor %}

PHPセクションでは、関数を定義し、PHPの `use`キーワードで名前空間を参照することしかできません。他のPHPコードは許可されていません。これはページの解析時にPHPクラスに変換されるためです。 名前空間参照の使用例：

    url = "/blog"
    layout = "default"
    ==
    <?
    use Acme\Blog\Classes\Post;

    function onStart()
    {
        $this['posts'] = Post::get();
    }
    ?>
    ==

一般的な変数の設定方法として、 `$this`で配列アクセスメソッドを使用する必要がありますが、簡略化するために読み込み専用オブジェクトのようにも使用できます。例として：

    // Write via array
    $this['foo'] = 'bar';

    // Read via array
    echo $this['foo'];

    // Read-only via object
    echo $this->foo;

<a name="twig-section"></a>
### Twigマークアップセクション

Twigセクションは、テンプレートによってレンダリングされるマークアップを定義しています。Twigセクションでは、[Octoberが提供する関数、タグ、フィルター](../markup)、すべての[Twig本来の機能](http://twig.sensiolabs.org/documentation)、[プラグインが提供する機能](../plugin/registration#extending-twig)を使用できます。Twigセクションのコンテンツは、テンプレートのタイプ（ページ、レイアウト、パーシャル）によって異なります。 特定のTwigオブジェクトの詳細については、ドキュメントを参照してください。

詳細は[マークアップガイド](../markup)をご覧ください。

<a name="theme-logging"></a>
## テーマロギング

OctoberCMSにはテーマロギングという非常に便利な機能が備わっていますが、 デフォルトで無効になっています。

レイアウトとページではほとんどのデータがフラットファイルに保存されるため、誤ってコンテンツを失う可能性があります。例えばページのレイアウトを切り替えると、ページのスキャホールドが変更されるため、データが失われます。

テーマロギングを有効にするには、 **設定 -> Log settings** に移動して **Log theme changes** を有効にします。 これですべての変更が記録されます。

テーマ変更ログは **設定 -> Theme log** から見ることができます。各変更には、追加/削除された内容の概要と、その前後に変更されたファイルのコピーがあります。必要に応じて、この情報を適切なアクションの決定や変更のリバージョンのために使用できます。
