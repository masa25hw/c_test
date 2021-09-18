# c_test

![example branch parameter](https://github.com/masa25hw/c_test/actions/workflows/build.yml/badge.svg?main)
![example event parameter](https://github.com/masa25hw/c_test/actions/workflows/build.yml/badge.svg?event=pull_request)

## 目的
- [ ] Cソースファイルのビルド
- [ ] warning/error結果の取得
- [ ] warning/error結果を経てPullRequestのOK/NG判断に活用

## 実施事項
- [ ] Cソースファイルの登録
- [ ] Pull requestへの追加コミットの様子を確認
- [ ] GitHub Actionsで実行した結果の外部出力
    - ローカルなのか、GitHub上のどこかのリポジトリに出力なのか、Pull requestのコメントとして表示されるのか
    - GitHub Actionsの実行結果は何をもってOK/NGと判断させればよいのか（最後にexit 1で処理を抜けたらよいのか？）
- [ ] CMakeの理解
    - [ありきたりなCMakeのプロジェクト作成 for C++](https://qiita.com/yumetodo/items/bd8f556ab56298f19ba8)
- [ ] gccの理解
- [ ] 各ビルドシステムの学習
    - meson?
    - Ninja?
        - [Using GitHub Actions with C++ and CMake](https://cristianadam.eu/20191222/using-github-actions-with-c-plus-plus-and-cmake/)
    - MSbuild
        - [GitHub Actions v2使ってMSBuildを試してみた](https://qiita.com/y428_b/items/f8e0c598238d75f808b1)
            - MSBuild.exeをローカルPCにインストールしていないと実行できない
- [ ] 利用するビルドシステムの決定
- [ ] バリナンバーの渡し方
- [ ] Cソースのビルド
    - [ ] 階層構造で配置したソフトのビルド


## 実現手法の検討
### Dockerを利用して実現
#### Dockerfileの作成
まずは所望の動作を達成するためのDockerファイルを作成

```shell {.line-numbers}
From gcc:latest
RUN apt-get update
RUN apt-get install -y python3-pip
RUN pip3 install meson ninja
COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```
- ビルドシステムにmesonを利用するため、インストール
- 上記Dockerfileの2-4行目は好みのビルドシステムのインストールに置き換え可能
- ここで`RUN`で全ての処理を実行しても良いが、汎用性がなくなるため、ここでは最低限の環境構築にとどめて、メインの処理は別に書いたシェルスクリプト（ここでは`entrypoint.sh`）に記述する
    - 5行目で用意したシェルスクリプトをDockerインスタンス上にコピーして、6行目で開始時に実行するように指示する

#### シェルスクリプトの作成
- [Docker コンテナのアクションを作成する - GitHub Docs](https://docs.github.com/ja/actions/creating-actions/creating-a-docker-container-action#writing-the-action-code)

- 上記ページで解説されているものを参考に記述
    - ただこのページだけ見てもテストしたいリポジトリのクローンをどうするのかが分からない
    - 試してみると別にクローンされてはいないので自分でクローンしてくる必要がある
    - そのために必要な情報を引数として受け取るようにする
    - 公式のActionにはcheckoutというのがありますが、今回はDocker上で実行するため利用できなさそう

```bash {.line-numbers}
#!/bin/sh -l

git clone $2
cd $1
meson build
meson test -C build -v
```
- 第一引数にリポジトリ名（=ディレクトリ名）、第二引数にリポジトリのURLを受けるようにしている
- 5-6行目はmeson特有のものなので、利用するビルドシステムのビルドコマンドと成果物の実行コマンドに置き換えること
- ファイル名は先程Dockerfileに書いたentrypoint.shと合わせること
- `chmod +x entrypoint.sh`で実行可能にすること（Windowsの場合は[Gitでファイルの実行権限を変更する](https://qiita.com/satotka/items/29f1483f8921d2ecfeab)のページが参考になる）

#### Actionファイルの作成
次にActionとして呼ばれたときに何をするかをaction.ymlというファイルに記述する。
```yml
name: 'build & test'

inputs:
  name:
    description: 'Name of target repository'
    required: true
  uri:
    description: 'URI of target repository'
    required: true

runs:
    using: 'docker'
    image: 'Dockerfile'
    args:
    - ${{ inputs.name }}
    - ${{ inputs.uri }}
```

- `inputs`で先程のシェルスクリプトに渡す引数を定義している
- 上から順番に第一引数（`name`）、第二引数（`uri`）で内容は先ほどの通り
- `description`は説明で無くても良い
- `required`は引数が省略可能かを表しており、今回は省略してほしくないので`true`に設定しておく
- 他にも出力を定義できたりする、できることは下記ページを参照
    - [GitHub Actionsのメタデータ構文](https://docs.github.com/ja/actions/creating-actions/metadata-syntax-for-github-actions)

#### Actionリポジトリの作成
ここまで用意したDockerfile、やることを記述したシェルスクリプトentrypoint.sh、Actionを記述したaction.ymlの3つをGithub上の公開リポジトリに保存しておき、リリースタグを打っておくことで、他のリポジトリからActionとして参照できるようになります。

- https://github.com/onihusube/gcc-meson-docker-action

#### テストしたいリポジトリのワークフロー中でActionを呼ぶ
用意した自作Docker ActionをテストしたいリポジトリのGithub Actions ワークフローから呼び出す

```shell {.line-numbers}
name: Test by GCC latest  # ワークフローの名前
on: [push, pull_request]  # 何をトリガーにして実行するか

jobs:
  test:
    runs-on: ubuntu-latest  # Dockerを実行することになるベース環境
    name: run test
    steps:
    - name: run test in docker
      uses: onihusube/gcc-meson-docker-action@v6
      with:
        name: 'harmony'
        uri: 'https://github.com/onihusube/harmony.git'
```

## 参考
**gcc**
- [Github Actions上のDockerコンテナ上のGCCの最新版でテストを走らせたかった](https://onihusube.hatenablog.com/entry/2020/07/24/024223)
- [CMakeの使い方（その１）](https://qiita.com/shohirose/items/45fb49c6b429e8b204ac)
- [CMakeの使い方（その２）](https://qiita.com/shohirose/items/637f4b712893764a7ec1)

**CMake**
- [勝手に作るCMake入門 その1 基本的な使い方](https://kamino.hatenablog.com/entry/cmake_tutorial1)
- [CMake HP](https://cmake.org/documentation/)
- [CMakeの使い方〜覚書〜](https://qiita.com/tchofu/items/69dacfb93908525e5b0b)
- [ごく簡単なcmakeの使い方](https://qiita.com/termoshtt/items/539541c180dfc40a1189)

**make**
- 実は make には色々種類がある. 主なものをあげると以下の通り
    - Microsoft nmake (Windows)
    - Borland make (Windows)
    - GNU make (windows, UNIX 系)
    - Solaris make (Solaris)
- makefileの中ではshの書式が有効

- [make Makefileで階層（サブディレクトリ）を走破しながら依存を解決してコンパイルする](https://giraffydev.hatenablog.com/entry/2016/10/04/101004)
- [シンプルで応用の効くmakefileとその解説](http://urin.github.io/posts/2013/simple-makefile-for-clang)
- [Makefileの書き方メモ](https://qiita.com/nullpo24/items/716bad137f1264b776f5)
- [Autotools ( automake, autoconf, libtool ) 使い方まとめ](http://tamaobject.hatenablog.com/entry/2013/08/01/165119)
## CMake


CMakeは、C, C++, CUDA, Fortran, assemblerなどのプロジェクトのビルドをコンパイラに依存せず自動化するためのツール

- 本来、プログラミング言語の仕様は標準ライブラリとコンパイラの実装に依存するので、開発環境が違えば異なるソースコードを書く必要がある
- しかし、上に挙げたような言語は、古くから標準化が行われているおかげで、同じソースコードを異なるコンパイラでビルドできる
- 例えば、C++にはGCC, MinGW GCC, Clang, MSVCなど複数のコンパイラが存在しているが、標準ライブラリだけを使ったコードであれば、どのコンパイラでもビルドすることができる（もちろんコードを書くときに若干の工夫は必要）
- ソースコードは共有できるが、一方でコンパイラのオプション、つまりビルドに際しての設定項目の仕様やそれを格納する設定ファイルのフォーマットは統一されていない

そのため、異なるコンパイラでビルドするためには、開発環境に合わせてプロジェクトファイルを使い分ける必要がある
- GCC on Linux: GCC用の`Makefile`
- MSVC on Windows: Visual Studioのソリューショファイル`*.sln`
- Clang on macOS: MakefileもしくはXcodeのプロジェクトファイル`*.xcodeproj`

これはなかなか面倒な話である。 実際には、コンパイラやIDEのバージョンごとにプロジェクトファイルの仕様も少しずつ変わるので、ほぼ無数のプロジェクトファイルを用意しなければならず、その全てを管理し続けるのは現実的ではない。

そこで登場したのが**CMake**である。CMakeでは、`CMakeLists.txt`という設定ファイルを作成しておけば、そこからCMakeがサポートする任意のプロジェクトファイルを作成することができる。つまり、どの開発環境でも
1. CMakeで`*.sln`, `*.xcodeproj`, `Makefile`などのプロジェクトファイルを生成する
1. 生成したプロジェクトファイルを使ってビルドする

という2ステップで、プロジェクトをビルドできるようになる。

## Project構成

```
project
├── Makefile.am
└── src
       ├── Makefile.am
       ├── main.c
       └── lib
              ├── Makefile.am
              ├── lib_test.h
              └── lib_test.c
```


## `git ignore`設定
### 実現したいこと
- 個人用で作成したスクリプトやメモなど、個人の開発環境に依存したファイルを管理しないために、`.gitignore`を個別に配置するケースを想定
- 個人依存のファイルを管理しない、かつ個々人で異なる`.gitignore`ファイル自体も管理対象としたくない

### 実現手法
下記構成でトライしたところ、上記ケースを作り出せたので、この方法を採用

#### プロジェクト構成
```
root
│  .gitignore
│  CMakeLists.txt
│  main.cpp
│  README.md
│
├─.github
│  └─workflows
│          blank.yml
│          test.yml
│
└─src
    │  .gitignore
    │  main.c
    │  test1.txt <- 無視したい
    │  test2.txt <- 無視したい
    │
    └─lib
            lib_test.c
            lib_test.h
```

#### 各階層に配置した`.gitignore`ファイル
/root/.gitignore

```git
# gitiginore
/**/.gitignore
```

/root/src/.gitignore

```git
# Personal settings
test*.txt
```

#### テスト結果
##### `/root/.gitignore`に`/**/.gitignore`を記述しなかった場合
ユーザー独自の設定を記述した`/root/src/.gitignore`はgit管理対象となる

```git
$ git status
On branch test_pull_request
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   .gitignore
        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        src/.gitignore

no changes added to commit (use "git add" and/or "git commit -a")
```

##### 上記`/root/.gitignore`および`/root/src/.gitignore`を配置して`git status`を実行した場合

ユーザー独自の設定を記述した`/root/src/.gitignore`はgit管理対象とならず、かつユーザー依存のファイルが管理対象外となっている（**実現したい姿**）

```git
$ git status
On branch test_pull_request
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   .gitignore
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

##### `/root/src/.gitignore`の`test*.txt`の設定をコメントアウトして`git status`を実行した場合

ユーザー独自の設定を記述した`/root/src/.gitignore`はgit管理対象とならず、かつユーザー依存のファイルの設定が都度反映されていることが確認できる（**実現したい姿**）

```git
$ git status
On branch test_pull_request
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   .gitignore
        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        src/test1.txt
        src/test2.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
Sat Sep 18 06:33:49 UTC 2021
