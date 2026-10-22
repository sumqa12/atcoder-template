# Ubuntuの公式コンテナを軸に環境構築
FROM ubuntu:25.10

# インタラクティブモードにならないようにする
ARG DEBIAN_FRONTEND=noninteractive
# タイムゾーンを日本に設定
ENV TZ=Asia/Tokyo
RUN ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

# aptのミラーを理研に変更 (archive.ubuntu.com / security.ubuntu.com が遅いため)
ARG UBUNTU_MIRROR=http://ftp.riken.jp/Linux/ubuntu
RUN sed -i -E "s@http://(archive|security)\.ubuntu\.com/ubuntu@${UBUNTU_MIRROR}@g" /etc/apt/sources.list.d/ubuntu.sources

# インフラを整備
RUN apt-get update && \
    apt-get install -y zsh time tzdata tree git curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# デフォルトシェルをZ shellにする
SHELL ["/bin/zsh", "-c"]

# C++, Python3, PyPy3の3つの環境想定
# cmake / ninja-build / make / gdb は CLion の Docker ツールチェーンで使用
RUN apt-get update && \
    apt-get install -y \
        g++-15 \
        clang clangd \
        gdb \
        cmake ninja-build make \
        python3.13 python3-pip \
        nodejs npm && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 一般的なコマンドで使えるように設定
# e.g. python3.8 main.py => python main.py
RUN update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-15 30 && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-15 30 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python3.13 30 && \
    update-alternatives --install /usr/bin/pip pip /usr/bin/pip3 30

# C++でAtCoder Library(ACL)を使えるようにする
RUN git clone --depth 1 https://github.com/atcoder/ac-library.git /lib/ac-library
ENV CPLUS_INCLUDE_PATH=/lib/ac-library

# コンテスト補助アプリケーションをインストール
RUN npm install -g atcoder-cli
# acc new で毎回すべての問題を取得する (既定値の inquire だと選択が必要)
RUN acc config default-task-choice all
RUN pip install \
    setuptools \
    online-judge-tools \
    --break-system-packages

# スクレイピングツール「atcs」をインストール (アーキテクチャごとにURLを変更)
RUN ARCH=$([ "$(uname -m)" = "aarch64" ] && echo "arm64" || echo "amd64") && \
    curl -L "https://github.com/tomyou666/atcoder-scraper/releases/latest/download/atcs-linux-${ARCH}" -o atcs && \
    chmod +x atcs && \
    mv atcs /usr/local/bin/atcs

