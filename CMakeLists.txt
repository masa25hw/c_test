# CMakeのバージョン指定
cmake_minimum_required(VERSION 3.1) # CMakeLists.txtをCMake 3.1の仕様で解析するように設定

# プロジェクトの設定
# プロジェクト名とプロジェクトで使用するプログラム言語を指定
project(hello_world CXX) # 「hello_worldというC++プロジェクトの設定ファイル」と宣言

# 実行ファイルの作成
add_executable(main_app main.cpp) # main.cppを使ってmain_appという名前の実行ファイルを作成

