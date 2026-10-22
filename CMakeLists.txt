cmake_minimum_required(VERSION 3.20)
project(atcoder CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON) # -std=gnu++23 (AtCoderと同じ)

# ビルドタイプ未指定ならDebug（ブレークポイントを使えるように）
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif()

add_compile_options(-Wall -Wextra -Wshadow -Wno-sign-compare)

# AtCoder Library (Dockerコンテナでは /lib/ac-library にインストール済み)
set(AC_LIBRARY_DIR "/lib/ac-library" CACHE PATH "AtCoder Libraryのパス")
if(EXISTS "${AC_LIBRARY_DIR}/atcoder")
  include_directories(SYSTEM "${AC_LIBRARY_DIR}")
else()
  message(WARNING "AtCoder Library not found: ${AC_LIBRARY_DIR} (-DAC_LIBRARY_DIR=<path> で指定してください)")
endif()

find_program(OJ_EXECUTABLE oj)

# 解答ファイルを1ファイル=1実行ファイルとして登録する
#   template.cpp        -> template
#   abc999/a/a.cpp      -> abc999_a_a
# ファイルを追加したら「CMakeプロジェクトの再読み込み」をすると実行構成に現れる
file(GLOB_RECURSE SOLUTION_SOURCES CONFIGURE_DEPENDS
  RELATIVE "${CMAKE_SOURCE_DIR}"
  "${CMAKE_SOURCE_DIR}/*.cpp")
list(FILTER SOLUTION_SOURCES EXCLUDE REGEX "^(sample|cmake-build-[^/]*|build|\\.[^/]*)/")

foreach(src IN LISTS SOLUTION_SOURCES)
  string(REGEX REPLACE "\\.cpp$" "" name "${src}")
  string(REGEX REPLACE "[^A-Za-z0-9_]" "_" name "${name}")
  if(name MATCHES "^(all|clean|help|install|package|test|depend|edit_cache|rebuild_cache|samples)$")
    set(name "${name}_") # CMakeの予約済みターゲット名との衝突を避ける
  endif()
  add_executable(${name} "${src}")

  # tests/ があれば oj でサンプルを一括実行するターゲット test_<name> を作る
  get_filename_component(dir "${CMAKE_SOURCE_DIR}/${src}" DIRECTORY)
  if(OJ_EXECUTABLE AND IS_DIRECTORY "${dir}/tests")
    add_custom_target(test_${name}
      COMMAND "${OJ_EXECUTABLE}" t -c "$<TARGET_FILE:${name}>" -d "${dir}/tests"
      WORKING_DIRECTORY "${dir}"
      DEPENDS ${name}
      USES_TERMINAL
      VERBATIM)
  endif()
endforeach()

# sample/ は main を持たないスニペット集なので、補完・解析用にオブジェクトライブラリとして登録
file(GLOB SAMPLE_SOURCES CONFIGURE_DEPENDS "${CMAKE_SOURCE_DIR}/sample/*.cpp")
if(SAMPLE_SOURCES)
  add_library(samples OBJECT ${SAMPLE_SOURCES})
  set_target_properties(samples PROPERTIES EXCLUDE_FROM_ALL ON)
endif()
