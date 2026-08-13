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
# 独立としたプログラムとして
$ ruby-build --list                        # 各Rubyの最新安定リリースを一覧化
$ ruby-build --definitions                 # 全定義を一覧化、旧版を含みます
$ ruby-build 3.4.9 ~/.rubies/ruby-3.4.9    # Ruby 3.4.9をインストール
$ ruby-build -d ruby-3.4.9 ~/.rubies       # 上の例の別の形式
$ ruby-build -d ruby-3.4 ~/.rubies         # 最新のRuby 3.4.xをインストール

# rbenvのプラグインとして
$ rbenv install 3.4.9  # Ruby 3.4.9を ~/.rbenv/versions/3.4.9 へインストール
$ rbenv install 3      # 最新のRuby 3.xをインストール
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

### Rubyの版

「最新の」Rubyの版を一覧するとき、つまり`ruby-build
--list`の出力においては、ruby-buildはこのプロジェクトに付属するRubyの版しか知りません。
それが意味するのは新しいRubyの版が出たとき、ruby-buildは直ちにはそれを知らないということです――その新しいRubyの版をインストールして使えるようにする前にruby-buildを更新せねばなりません。
これはruby-buildが各個別のRubyの版について[定義ファイル](#custom-build-definitions)を付属しているからです。

最新のRubyの版の一覧をダウンロードすべく（ツール自体を更新する必要なく）インストーラが常に遠隔の資源をあたることが重要であれば、[ruby-install][]をruby-buildの代替としてご確認ください。

### Rubyの実装

ruby-buildは以下のRubyの実装のための定義を備えており、それらの実装は`ruby-build
--list`の出力では版の接頭辞に記されています。

- [CRuby][]：ruby-buildでは`X.Y.Z`の形式で接頭辞のない版の番号として一覧されます。
  これはほとんどの人々が使う主要なRubyの実装であり歴史的に「MRI」として知られるものでもあります。
  ruby-buildでは他の版の管理器との互換性のためCRubyの版の番号に`ruby-`の接頭辞を加えることを許容します。

- `jruby`：[JRuby][]は高性能なRubyの実装でJava仮想機械 (Java Virtual Machine; JVM)
  を土台に構築された本物のスレッドが付いています。

- `mruby`：は軽量で、組み込み可能なマイクロコントローラ用のRubyの実装です。

- `picoruby`：[PicoRuby][]はワンチップのマイクロコントーラ用の代替のmrubyの実装です。

- `truffleruby`：[TruffleRuby][]のネイティブの独立した配布物で、GraalVMのTruffleフレームワークを土台にするRubyの実装です。

- `truffleruby+graalvm`：JVMの独立したTruffleRubyの配布物です。

### 発展的な使い方

#### 独自のビルド定義

ruby-buildで使えないRubyのバージョンをインストールするには、該当するRubyのバージョン番号の場所にある、独自のビルド定義ファイルへのパスを指定します。

```sh
# 独立したプログラムとして
$ ruby-build -d /path/to/3.4-custom /opt/rubies  # /opt/rubies/3.4-custom にインストール

# rbenvのプラグインとして
$ rbenv install /path/to/3.4-custom              # $(rbenv root)/versions/3.4-custom にインストール
```

独自のビルド定義ファイルの _ディレクトリ_ を与えることもできます。
ruby-buildに付属する `share/ruby-build/`
ディレクトリと共に、パスが探されます（もしかすると、サードパーティのビルド定義の集まりがgitリポジトリとして公開されたり、組織の独自のビルド定義が組織内部で配布されたりするかもしれません）。

```sh
# 独立したプログラムとして
$ RUBY_BUILD_DEFINITIONS=/path/to/custom/defs ruby-build --definitions              # 使用できる全てのRubyのバージョンを一覧にします。独自の定義も含みます
$ RUBY_BUILD_DEFINITIONS=/path/to/custom/defs ruby-build -d 3.5-custom /opt/rubies  # /opt/rubies/3.5-custom にインストール

# rbenvのプラグインとして
$ RUBY_BUILD_DEFINITIONS=/path/to/custom/defs rbenv install --list                  # 使用できる全てのRubyのバージョンを一覧にします。独自の定義も含みます
$ RUBY_BUILD_DEFINITIONS=/path/to/custom/defs rbenv install 3.5-custom              # $(rbenv root)/versions/3.5-custom にインストール
```

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
| `RUBY_BUILD_TARBALL_OVERRIDE`   | rubyのtarballを取得してくるためのURLを上塗りします。随意で`#checksum`を後に続けます。             |
| `RUBY_BUILD_DEFINITIONS`        | コロン区切りのパスのリストであり、ビルド定義ファイルを探索する場所です。                              |
| `RUBY_BUILD_ROOT`               | ビルド定義ファイルを探索するパスの接頭辞です。*廃止済：*`RUBY_BUILD_DEFINITIONS`をお使いください|
| `RUBY_BUILD_VENDOR_OPENSSL`     | システムのopensslに互換性があったとしても、opensslをビルドしてそれを取り入れます                                |
| `CC`                            | Cコンパイラへのパスです。                                                                          |
| `RUBY_CFLAGS`                   | `CFLAGS`への追加オプションです（ *例* ：`-O3`を上塗り）。                                         |
| `CONFIGURE_OPTS`                | `./configure`の追加オプションです。                                                                |
| `MAKE`                          | 独自の`make`コマンドです（ *例* ：`gmake`）。                                                         |
| `MAKE_OPTS` / `MAKEOPTS`        | `make`の追加オプションです。                                                                       |
| `MAKE_INSTALL_OPTS`             | `make install`の追加オプションです。                                                               |
| `RUBY_CONFIGURE_OPTS`           | `./configure`の追加オプションです（Rubyのソースにのみ適用されます）。                                  |
| `RUBY_MAKE_OPTS`                | `make`の追加オプションです（Rubyのソースにのみ適用されます）。                                         |
| `RUBY_MAKE_INSTALL_OPTS`        | `make install`の追加オプションです（Rubyのソースにのみ適用されます）。                                 |
| `NO_COLOR`                      | 出力でANSIの彩色を無効にします。既定では端末に接続しているときの出力で色彩を使います。  |
| `CLICOLOR_FORCE`                | 端末に接続していないときでも、出力でANSIの色彩を使います。                                 |
| `RUBY_REPO`                     | `ruby-dev`を構築するときに使うgitリポジトリのURL |
| `RUBY_REF`                      | `ruby-dev`を構築するときに使うgitブランチ（またはリビジョン）、例：`some-branch@af12decf` |

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

`RUBY_BUILD_MIRROR_URL`を設定して、独自のミラーを指定できます。
設定したときは、まずミラーからパッケージのダウンロードを試み、それから元のURLにフォールバックします。

```sh
# 例：
export RUBY_BUILD_MIRROR_URL="https://my-mirror.example.com"
install_package "ruby-2.6.5" "https://ruby-lang.org/ruby-2.6.5.tgz#<SHA2>"
# こうすると、まず https://my-mirror.example.com/<SHA2> を試します
```

ruby-buildではまず、このパッケージを`$RUBY_BUILD_MIRROR_URL/<SHA2>`から取得することを試みます（補足：これは完全なURLです）。
ここで`<SHA2>`はファイルのチェックサムです。
以下の場合は元の場所からパッケージをダウンロードするようにフォールバックします。
- パッケージがミラーで見つからなったとき
- ミラーがダウンしているとき
- ダウンロードしたものが壊れているとき。つまりファイルのチェックサムが合わないとき
- チェックサムを計算できるツールがないとき
- `RUBY_BUILD_SKIP_MIRROR`が有効のとき

ミラーサイトが上記のURLの形式に準拠していないとき、`RUBY_BUILD_MIRROR_PACKAGE_URL`を設定して完全なURLを指定できます。
完全なURLである点を除き、`RUBY_BUILD_MIRROR_URL`と同じはたらきをします。

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
  [cruby]: https://www.ruby-lang.org/
  [truffleruby]: https://truffleruby.dev/
  [picoruby]: https://github.com/picoruby/picoruby#readme
  [mruby]: https://mruby.org/
  [jruby]: https://www.jruby.org/
  [ruby-install]: https://github.com/postmodern/ruby-install#readme

## 日本語訳について

この日本語訳の原文は[ruby-buildのreadme](https://github.com/rbenv/ruby-build#readme)です。
