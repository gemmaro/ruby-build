# ruby-build

ruby-buildはコマンドラインツールです。
Unixライクなシステムで、Rubyのバージョンをソースからインストールする手順を単純にするものです。

`rbenv install`コマンドにより、[rbenv][]のプラグインとして使えます。
また、`ruby-build`コマンドにより、独立したプログラムとしても使えます。

## インストール

### Homebrewによるパッケージ管理
```sh
brew install ruby-build
```

以下で更新します。
```sh
brew upgrade ruby-build
```

### gitを使ってrbenvプラグインとしてクローンする
```sh
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
```

以下で更新します。
```sh
git -C "$(rbenv root)"/plugins/ruby-build pull
```

### 独立したプログラムとして手動でインストールする

まず、tarballを https://github.com/rbenv/ruby-build/releases/latest からダウンロードします。
それから以下とします。
```sh
tar -xzf ruby-build-*.tar.gz
PREFIX=/usr/local ./ruby-build-*/install.sh
```

## 使い方

### 基本的な使い方

```sh
# 独立としたプログラムとしての使い方
$ ruby-build --list                        # 利用できるRubyのバージョンを一覧にします
$ ruby-build 3.2.2 /opt/rubies/ruby-3.2.2  # Ruby 3.2.2をインストールします
$ ruby-build -d ruby-3.2.2 /opt/rubies     # 上の例の別の形式

# rbenvのプラグインとしての使い方
$ rbenv install 3.2.2  # Ruby 3.2.2を ~/.rbenv/versions/3.2.2 へインストールします
```

> [!WARNING]
> ruby-buildは、Rubyのソースをダウンロードしてコンパイルを試みる前に、ほとんどシステムの依存関係が存在することを検証しません。
> ビルドツールや開発ヘッダといった[全ての必須のライブラリ][build-env]が既にシステムに存在していることをお確かめください。

基本的に、ruby-buildがRubyのバージョンのインストールですることは以下です。
- Rubyのソースコードが含まれる、公式のtarballをダウンロードする
- アーカイブからシステムの一時ディレクトリへ展開する
- ソースコードがある場所で、`./configure --prefix=/path/to/destination`を実行する
- `make install`を走らせてRubyをコンパイルする
- インストールされたRubyが機能するか検証する

状況によって、ruby-buildは上記以外にもすることがあります。
例えば、Rubyに適切なOpenSSLのバージョンをリンクしようとします。
OpenSSL自体をダウンロードしてコンパイルすることも指しています。
Homebrewでインストールされたlibyamlやreadlineといったライブラリを見つけてリンクしようともします。

### 発展的な使い方

#### 独自のビルド定義

ruby-buildで認識されていないRubyのバージョンをインストールするには、Rubyのバージョン番号の場所にある、独自のビルド定義ファイルへのパスを指定します。

[既定のビルド定義][definitions]をご確認いただくと、定義ファイルの書き方の例があります。

#### 独自のビルド構成

ビルド過程は、以下の環境変数を通じて構成できます。

| 変数                        | 機能                                                                                         |
| ------------------------------- | ------------------------------------------------------------------------------------------------ |
| `TMPDIR`                        | 一時ファイルが補完される場所です。                                                                |
| `RUBY_BUILD_BUILD_PATH`         | ソースファイルがダウンロードされ、ビルドされる場所です（既定では`TMPDIR`の時間記録付きの副ディレクトリ）。        |
| `RUBY_BUILD_CACHE_PATH`         | ダウンロードされたパッケージファイルをキャッシュする場所です（rbenvプラグインとして呼び出されたときは既定で`~/.rbenv/cache`です）。  |
| `RUBY_BUILD_HTTP_CLIENT`        | `aria2c`、`curl`、`wget`のいずれかを使ってダウンロードします（既定ではPATHで最初に見つかったものです）。    |
| `RUBY_BUILD_ARIA2_OPTS`         | ダウンロードで`aria2c`に渡す追加のオプションです。                                          |
| `RUBY_BUILD_CURL_OPTS`          | ダウンロードで`curl`に渡す追加のオプションです。                                            |
| `RUBY_BUILD_WGET_OPTS`          | ダウンロードで`wget`に渡す追加のオプションです。                                            |
| `RUBY_BUILD_MIRROR_URL`         | 独自のミラーURLのルートです。                                                                          |
| `RUBY_BUILD_MIRROR_PACKAGE_URL` | 独自の完全なミラーURLです（例：http://mirror.example.com/package-1.0.0.tar.gz）。                |
| `RUBY_BUILD_SKIP_MIRROR`        | ダウンロードミラーを迂回し、全てのパッケージファイルを元のURLから取得します。                 |
| `RUBY_BUILD_TARBALL_OVERRIDE`   | rubyのtarballを取得してくるためのURLを上塗りします。`#checksum`を付けられます。             |
| `RUBY_BUILD_DEFINITIONS`        | コロン区切りのパスのリストであり、ビルド定義ファイルを探索する場所です。                              |
| `RUBY_BUILD_ROOT`               | ビルド定義ファイルを探索するパスの接頭辞です。*廃止されました：*`RUBY_BUILD_DEFINITIONS`をお使いください|
| `CC`                            | Cコンパイラへのパスです。                                                                          |
| `RUBY_CFLAGS`                   | `CFLAGS`への追加オプションです（*例*として`-O3`を上塗りできます）。                                         |
| `CONFIGURE_OPTS`                | `./configure`の追加オプションです。                                                                |
| `MAKE`                          | 独自の`make`コマンドです（*例*として`gmake`）。                                                         |
| `MAKE_OPTS` / `MAKEOPTS`        | `make`の追加オプションです。                                                                       |
| `MAKE_INSTALL_OPTS`             | `make install`の追加オプションです。                                                               |
| `RUBY_CONFIGURE_OPTS`           | `./configure`の追加オプションです（Rubyのソースにのみ適用されます）。                                  |
| `RUBY_MAKE_OPTS`                | `make`の追加オプションです（Rubyのソースにのみ適用されます）。                                         |
| `RUBY_MAKE_INSTALL_OPTS`        | `make install`の追加オプションです（Rubyのソースにのみ適用されます）。                                 |
| `NO_COLOR`                      | 出力でANSIの彩色を無効にします。既定では端末に接続しているときの出力で色彩を使います。  |
| `CLICOLOR_FORCE`                | 端末に接続していないときでも、出力でANSIの色彩を使います。                                 |

#### パッチをあてる

コマンド`rbenv
install`と`ruby-build`は共に`-p/--patch`フラグに対応しており、ビルド前にRubyのソースコードにパッチをあてられます。
パッチは標準入力から読み取られます。

```sh
# 単一のパッチをあてます
$ rbenv install --patch 1.9.3-p429 < /path/to/ruby.patch

# HTTPからパッチをあてます
$ rbenv install --patch 1.9.3-p429 < <(curl -sSL http://git.io/ruby.patch)

# 複数のパッチをあてます
$ cat fix1.patch fix2.patch | rbenv install --patch 1.9.3-p429
```

#### チェックサムの検証

ruby-buildに付属する全てのRubyの定義ファイルにはパッケージのチェックサムが含まれています。
つまり、全ての外部からダウンロードされたパッケージは取得された後、自動で真正性が検査されます。

チェックサムを施す方法についての詳細は、次節をご参照ください。

#### パッケージのミラー

ダウンロードを高速にするため、ruby-buildはパッケージファイルをAmazon CloudFrontでホストされているミラーから取得します。
この高速化を享受するため、パッケージにはチェックサムを指定しなければなりません。

```sh
# 例：
install_package "ruby-2.6.5" "https://ruby-lang.org/ruby-2.6.5.tgz#<SHA2>"
```

ruby-buildではまず、このパッケージを`$RUBY_BUILD_MIRROR_URL/<SHA2>`から取得することを試みます（補足：これは完全なURLです）。
ここで`<SHA2>`はファイルのチェックサムです。
以下の場合は元の場所からパッケージをダウンロードするようにフォールバックします。
- パッケージがミラーで見つからなったとき
- ミラーがダウンしているとき
- ダウンロードしたものが壊れているとき。つまりファイルのチェックサムが合わないとき
- チェックサムを計算できるツールがないとき
- `RUBY_BUILD_SKIP_MIRROR`が有効のとき

`RUBY_BUILD_MIRROR_URL`を設定して独自のミラーを指定できます。

ミラーサイトが上記のURLの形式に準拠していないとき、`RUBY_BUILD_MIRROR_PACKAGE_URL`を設定して完全なURLを指定できます。
完全なURLである点を除き、`RUBY_BUILD_MIRROR_URL`と同じはたらきをします。

既定のruby-buildのダウンロードミラーは[Basecamp](https://basecamp.com/)の支援を受けています。

#### インストール後もビルドディレクトリを保持する

`ruby-build`と`rbenv install`は共にフラグ`-k`ないし`--keep`を受け付けます。
このフラグはruby-buildにインストール後も、ダウンロードしたソースを保持するように伝えるものです。
Rubyで`gdb`や`memprof`を使う必要があるときは役に立つことがあるかもしれません。

`rbenv install`コマンドで`--keep`を使うと、ソースコードは`~/.rbenv/sources`に保持されます。
`ruby-build`で`--keep`を使うときは、`RUBY_BUILD_BUILD_PATH`でソースコードの場所を指定するべきです。

## 困ったときは

よくある問題への解決策については[ruby-buildのウィキ][wiki]をご参照ください。

ウィキで答えが見つからなかったときは、[イシュートラッカー][issue tracker]でイシューを開いてください。
必ず、ビルドで失敗したときのビルドログの全文を含めてください。


  [rbenv]: https://github.com/gemmaro/rbenv/tree/ja/translation/ja#readme
  [definitions]: https://github.com/rbenv/ruby-build/tree/master/share/ruby-build
  [wiki]: https://github.com/rbenv/ruby-build/wiki
  [build-env]: https://github.com/rbenv/ruby-build/wiki#suggested-build-environment
  [issue tracker]: https://github.com/rbenv/ruby-build/issues

## 日本語訳について

この日本語訳の原文は[ruby-buildのreadme](https://github.com/rbenv/ruby-build#readme)です。

先行訳として[Ruby STUDIO/ruby-build](https://ruby.studio-kingdom.com/rbenv/ruby_build/)がありますが、本訳の作成にあたって参照していません。
