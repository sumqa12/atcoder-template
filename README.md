# AtCoder用C++開発環境 (CLion + Docker)

このリポジトリは、AtCoder競技プログラミング用のC++開発環境を提供します。CLionとDockerを使用して、統一された開発環境を構築できます。

## 概要

この開発環境は以下の特徴を持ちます：

- **Dockerコンテナベース**: 環境の統一性を保証（CLionの「Dockerツールチェーン」でビルド・実行・デバッグ）
- **CMakeによる自動ターゲット登録**: `.cpp`ファイルを置くだけで実行構成が作られる
- **C++23対応**: 最新のC++標準に対応（`-std=gnu++23`）
- **AtCoder Library統合**: ACLを簡単に使用可能
- **デバッグ機能**: GDBによるデバッグサポート
- **テスト一括実行**: `test_<ターゲット名>`でサンプルケースを`oj`で一括判定
- **コードフォーマット**: `.clang-format`によるフォーマット

## 環境要件

### 必須ソフトウェア

- **Docker Desktop** (またはDocker Engine)
  - Windows/Mac: [Docker Desktop](https://www.docker.com/products/docker-desktop/)
  - Linux: Docker Engine
- **CLion**
  - [CLion公式サイト](https://www.jetbrains.com/clion/)

## セットアップ

### 1. リポジトリのクローン

```bash
git clone git@github.com:sumqa12/atcoder-template.git
cd atcoder-template
```

### 2. Dockerイメージをビルド

初回は数分かかります。

```bash
docker build -t atcoder-template .devcontainer
```

### 3. CLionにDockerツールチェーンを設定

1. CLionでプロジェクトのフォルダを開く
2. `File | Settings | Build, Execution, Deployment | Toolchains` を開く
3. `+` → `Docker` を選び、以下を設定
   - **Image**: `atcoder-template:latest`
   - **CMake**: `Docker CMake`（コンテナ内の`cmake`）
   - **Build Tool**: コンテナ内の`ninja`（自動検出されない場合は`/usr/bin/ninja`）
   - **C Compiler**: `/usr/bin/gcc-13`
   - **C++ Compiler**: `/usr/bin/g++`
   - **Debugger**: `/usr/bin/gdb`
4. ▲ボタンでDockerツールチェーンを一番上（デフォルト）に移動
5. `Build, Execution, Deployment | CMake` で、プロファイルの **Toolchain** が `Docker` になっていることを確認
   - 必要に応じて **Generator** を `Ninja` にします

> **別の方法: Dev Container**  
> `.devcontainer/devcontainer.json`はCLionのDev Containers機能にも対応しています。
> `File | Remote Development | Dev Containers`から開くと、コンテナ内でCLionのバックエンドが動きます（メモリを多く使います）。

### 4. 動作確認

1. ルートに`hello.cpp`というファイルを作成して、以下のサンプルコードを書きます：

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello World" << endl;
    return 0;
}
```

2. `Tools | CMake | Reload CMake Project` を実行します
3. 右上の実行構成で`hello`を選び、▶（`Shift+F10`）で実行します
4. 実行ウィンドウに「Hello World」と表示されれば成功です。

## プロジェクト構成

```
atcoder-template/
├── .devcontainer/         # コンテナ設定
│   ├── Dockerfile       # Dockerイメージ定義（CLionのDockerツールチェーンで使用）
│   └── devcontainer.json # Dev Container設定ファイル
├── .idea/                 # CLionのプロジェクト設定（個人設定は除外）
├── sample/                # 便利関数のサンプル集（mainなし）
├── .clang-format          # コードフォーマット設定
├── .clangd                # clangd設定
├── CMakeLists.txt         # ビルド設定（.cppを自動でターゲット登録）
├── template.cpp           # C++テンプレートファイル
└── README.md              # このテンプレートの説明
```

## 開発環境の詳細

### インストール済みツール

**コンパイラ・開発ツール**
- `g++-13`: GNU C++コンパイラ（C++23対応）**※ 実行用**
- `clang`: LLVM C++コンパイラ **※ リント・補完する用**
- `clangd`: Language Server Protocol実装
- `gdb`: デバッガー

**ユーティリティ**
- `zsh`: Z Shell（デフォルトシェル）
- `time`: 実行時間測定
- `tree`: ディレクトリ構造表示
- `git`: バージョン管理
- `curl`: HTTPクライアント

**AtCoder関連ツール**
- **AtCoder Library (ACL)**: `/lib/ac-library`にインストール済み
- **atcoder-cli (acc)**: コンテスト管理ツール。コンテスト用ディレクトリの作成や提出を自動化できます。詳細は[公式インストールガイド](http://tatamo.81.la/blog/2018/12/07/atcoder-cli-installation-guide/)を参照してください。
- **online-judge-tools (oj)**: テストケースの実行や提出を自動化するツール。`oj t`コマンドでテストケースを一括実行できます。詳細は[公式GitHub](https://github.com/online-judge-tools/oj)を参照してください。
- **atcs**: AtCoder用のURLを指定して問題文を取得するシンプルなスクレイピングツール。復習目的での問題文をAIに読み込ませる際に便利なツールです。詳細は [AtCoder シンプルなスクレイピングツール「atcs」の紹介](https://qiita.com/tomyou666/items/6acf1f048b81a24da58b) を参照してください。

**その他**
- `nodejs` / `npm`: JavaScript実行環境（atcoder-cli実行用）
- `python3.13` / `pip`: Python実行環境（online-judge-tools実行用）

### シンボリックリンク

- `g++` → `g++-13`: デフォルトでg++-13を使用

## 使用方法

### コンテナのシェルを開く

`acc`・`oj`・`atcs`はコンテナ内にインストールされています。CLionのターミナル（`Alt+F12`）でプロジェクトのルートから以下を実行し、コンテナのシェルに入ってから使ってください。

```bash
docker run -it --rm -v "${PWD}:/work" -w /work atcoder-template zsh
```

### 新しい問題の作成
[atcoder-cli](http://tatamo.81.la/blog/2018/12/07/atcoder-cli-installation-guide/) を利用して問題をダウンロードします。

1. コンテナのシェルで`acc new <contest-id>`コマンドを実行してコンテスト問題をダウンロード
```bash
acc new abc999
```

> ここでログイン情報を求められますがcli経由で提出をしない場合は必要ないので何も入力せずにEnterを押下します。

2. `abc<コンテスト番号>`フォルダが作成されます。a問題を解く場合は、`abc<コンテスト番号>/a`フォルダに`a.cpp`を作成して問題を解いてください。また必要に応じて `template.cpp`からマクロなどをコピーしてください。

3. CLionで`Tools | CMake | Reload CMake Project`を実行すると、`abc999_a_a`（実行用）と`test_abc999_a_a`（サンプル一括判定用）のターゲットが作られます。


```
abc999
|-- a
|   `-- tests
|       |-- sample-1.in
|       |-- sample-1.out
|       |-- sample-2.in
|       |-- sample-2.out
|       |-- sample-3.in
|       `-- sample-3.out
|-- b
|   `-- tests
|       |-- sample-1.in
|       |-- sample-1.out
|       |-- sample-2.in
|       |-- sample-2.out
|       |-- sample-3.in
|       `-- sample-3.out
|-- c
|   `-- tests
|       |-- sample-1.in
|       |-- sample-1.out
|       |-- sample-2.in
|       |-- sample-2.out
|       |-- sample-3.in
|       `-- sample-3.out
|-- contest.acc.json
|-- d
|   `-- tests
|       |-- sample-1.in
|       |-- sample-1.out
|       |-- sample-2.in
|       `-- sample-2.out
|-- e
|   `-- tests
|       |-- sample-1.in
|       |-- sample-1.out
|       |-- sample-2.in
|       `-- sample-2.out
|-- f
|   `-- tests
|       |-- sample-1.in
|       |-- sample-1.out
|       |-- sample-2.in
|       `-- sample-2.out
`-- g
    `-- tests
        |-- sample-1.in
        |-- sample-1.out
        |-- sample-2.in
        |-- sample-2.out
        |-- sample-3.in
        `-- sample-3.out
```

### コードの実行

`CMakeLists.txt`がプロジェクト内の`.cpp`ファイル（`sample/`を除く）を探し、1ファイルにつき1つの実行ターゲットを作ります。
ターゲット名はパスの`/`や`.`を`_`に置き換えたものです（例: `abc999/a/a.cpp` → `abc999_a_a`）。

> 新しい`.cpp`を作ったら`Tools | CMake | Reload CMake Project`を実行してください。実行構成の一覧に追加されます。

#### 方法1: 普通に実行

1. 右上の実行構成で対象のターゲット（例: `abc999_a_a`）を選択
2. ▶ボタン（`Shift+F10`）で実行
3. 実行ウィンドウに標準入力を打ち込めます

サンプル入力をファイルから流したい場合は、`Run | Edit Configurations...` で対象の構成を開き、**Redirect input from** に`abc999/a/tests/sample-1.in`などを指定します。

#### 方法2: デバッグモードで実行（ブレークポイントを利用したい場合）

1. 行番号の右側をクリックしてブレークポイントを設定
2. 🐞ボタン（`Shift+F9`）でデバッグ開始
3. 変数のホバー、ウォッチ式、コールスタックなどを確認可能

#### 方法3: テストケース一括実行（`test_<ターゲット名>`）

`tests/`ディレクトリがある問題には、`oj t`でサンプルを一括判定するターゲット`test_<ターゲット名>`が作られます。

1. 右上の実行構成で`test_abc999_a_a`を選択
2. 🔨ボタン（`Ctrl+F9`）でビルド
3. 解答がビルドされたあと`oj t`が実行され、結果（AC/WAなど）がビルドウィンドウに表示されます

### ビルドタイプ

- **Debug**（デフォルト）: `-g`付き。デバッグ向け
- **Release**: `-O3`付き。実行時間を確認したいときに使います
  - `Settings | Build, Execution, Deployment | CMake`で`+`を押すとReleaseプロファイルを追加できます

### コードのフォーマット

- CLionはプロジェクトのルートに`.clang-format`があると自動でClangFormatを使うため、`Ctrl+Alt+L`で`.clang-format`の設定どおりにフォーマットされます
- 保存時に自動フォーマットしたい場合は`Settings | Tools | Actions on Save`で **Reformat code** を有効にします

## 設定ファイルの説明

### CMakeLists.txt

- **C++標準**: `-std=gnu++23`
- **警告**: `-Wall -Wextra -Wshadow`
- **AtCoder Library**: `/lib/ac-library`をインクルードパスに追加（`-DAC_LIBRARY_DIR=<path>`で変更可能）
- **実行ターゲット**: `.cpp`ごとに自動作成（`sample/`、`cmake-build-*/`、隠しディレクトリは除外）
- **テストターゲット**: `tests/`がある問題に`test_<ターゲット名>`を作成（`oj`が見つかる場合のみ）
- **sample/**: `main`がないため、コード解析用のオブジェクトライブラリ`samples`として登録（通常のビルドには含まれません）

### Dockerfile

Dockerイメージの定義ファイル。以下の処理を行います：

1. **ベースイメージ**: Ubuntu 25.10
2. **タイムゾーン設定**: Asia/Tokyo
3. **基本ツールインストール**: zsh, git, curl等
4. **C++環境構築**: g++-13, clang, clangd, gdb, cmake, ninja, make（CLionのDockerツールチェーン用）
5. **Python環境構築**: python3.13, pip（online-judge-tools実行用）
6. **AtCoder Libraryインストール**: `/lib/ac-library`
7. **atcoder-cliインストール**: npm経由
8. **online-judge-toolsインストール**: pip経由
9. **atcsインストール**: アーキテクチャに応じて自動選択

### .devcontainer/devcontainer.json

Dev Containerの設定ファイル（CLionのDev Containers機能で使う場合）：

- **ビルドコンテキスト**: プロジェクトルート
- **Dockerfile**: `.devcontainer/Dockerfile`を参照
- **IDEバックエンド**: CLion

### .idea/

CLionが生成するプロジェクト設定です。`workspace.xml`やMaterial Themeプラグインの設定（`material_theme_project_new.xml`）などの個人設定は`.idea/.gitignore`で除外しています。

## 便利な機能

### テンプレートファイル

`template.cpp`には以下の機能が含まれています：

- **型エイリアス**: `ll`（long long）、`ld`（long double）、`vi`（vector<int>）、`vl`（vector<ll>）、`vvl`（2次元配列）、`vvvl`（3次元配列）、`vb`（vector<bool>）、`pl`（pair<ll,ll>）など
- **マクロ**:
  - `rep(i, n)`: `for (int i = 0; i < n; i++)`の短縮
  - `repn(i, num, n)`: `for (int i = num; i < n; i++)`の短縮
  - `per(i, n)`: 逆順のfor文
  - `pern(i, num, n)`: 逆順範囲のfor文
  - `all(v)`: `v.begin(), v.end()`の短縮
  - `pb`: `push_back`の短縮
  - `fi`, `se`: `first`, `second`の短縮
  - `LB(v, val)`, `UB(v, val)`, `BS(v, val)`: lower_bound、upper_bound、binary_searchの短縮
  - `MINE(v)`, `MAXE(v)`: min_element、max_elementの短縮
- **ユーティリティ関数**:
  - `yes()`, `YES()`, `Yes()`: 真偽値の出力（"yes"/"no"など）
  - `possible()`, `Possible()`, `POSSIBLE()`: 可能性の出力
  - `fast_io()`: 高速入出力の設定
  - `cinarr(v)`: 配列の一括入力
  - `print(...)`: 可変長引数での出力
  - `print_ld(x, precision)`: 固定小数点での出力
  - `printarr(v, isReverse)`: 配列の出力
  - `addv(v, loc, val)`: 配列要素の追加
- **その他の便利機能**:
  - `_pq<T>`: 最小値優先のpriority_queue
  - `chmin(a, b)`, `chmax(a, b)`: 最小値・最大値の更新
  - `So(v)`, `Sore(v)`: ソート関数
  - `binary_count(a)`: ビットカウント

### AtCoder Library (ACL)の使用

ACLは自動的にインクルードパスに含まれています：

```cpp
#include "atcoder/modint.hpp"
using mint = atcoder::modint998244353;
```

### atcoder-cli (acc)の使用

コンテスト管理ツール：

```bash
# コンテストの開始（abc432フォルダが作成される）
acc new abc432

# a問題のディレクトリに移動
cd abc432/a

# 問題の提出（a.cppを提出）
acc submit a.cpp

# テスト実行（abc432/aディレクトリ内で実行）
acc test
```

詳細は[公式インストールガイド](http://tatamo.81.la/blog/2018/12/07/atcoder-cli-installation-guide/)を参照してください。

### online-judge-tools (oj)の使用

テストケース実行ツール：

```bash
# abc432/aディレクトリに移動
cd abc432/a

# テストケースを実行（tests/ディレクトリ内のテストケースを一括実行）
g++ -std=gnu++23 a.cpp && oj t -d tests/

# または、CLionで「test_abc432_a_a」ターゲットをビルド（Ctrl+F9）する
```

詳細は[公式GitHub](https://github.com/online-judge-tools/oj)を参照してください。

### atcsの使用

問題スクレイピングツール：

```bash
# 問題を取得
atcs https://atcoder.jp/contests/abc123/tasks/abc123_a
```

詳細は [AtCoder シンプルなスクレイピングツール「atcs」の紹介](https://qiita.com/tomyou666/items/6acf1f048b81a24da58b) を参照してください。

## ライセンス

このプロジェクトは [MIT License](LICENSE) の下で公開されています。

## 貢献

バグ報告や機能要望は、Issueでお知らせください。プルリクエストも歓迎します。

---

**Happy Coding! 🚀**
