# cf. https://qiita.com/termoshtt/items/539541c180dfc40a1189

# CMakeのバージョン指定
cmake_minimum_required(VERSION 3.20) # CMakeLists.txtをCMake 3.1の仕様で解析するように設定

# プロジェクトの設定
# `project`は、cmakeプロジェクトの名称。プロジェクト名とプロジェクトで使用するプログラム言語を指定
# ここで指定した名前は、`PROJECT NAME`にセットされる
project(test C) # 「testというCプロジェクトの設定ファイル」と宣言

# 実行ファイルの作成
# <Main>という名前の実行ファイルを作成
add_executable(Main 
    src/main.c
    src/lib/lib_test.c
)

