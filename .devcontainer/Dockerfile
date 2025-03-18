# Mac環境/GitHub Codespaces 用 Ubuntuベースイメージ
FROM --platform=linux/amd64 mcr.microsoft.com/devcontainers/base:ubuntu

# 必須パッケージのインストール
RUN apt-get update
RUN apt-get install -y curl git wget unzip libgconf-2-4 gdb libstdc++6 libglu1-mesa fonts-droid-fallback lib32stdc++6 python3 cmake ninja-build aptitude clang pkg-config libgtk-3-dev
RUN apt-get install -y openjdk-17-jdk android-tools-adb
RUN apt-get clean

# Android SDK 用ディレクトリの作成と環境変数の設定
ENV ANDROID_SDK_ROOT=/usr/local/android-sdk
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
RUN mkdir -p $ANDROID_SDK_ROOT
RUN wget -O cmdline-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip && \
  mkdir temp && unzip cmdline-tools.zip -d temp && rm cmdline-tools.zip && \
  mkdir -p $ANDROID_SDK_ROOT/cmdline-tools && \
  mv temp/cmdline-tools $ANDROID_SDK_ROOT/cmdline-tools/latest && \
  rm -rf temp
ENV PATH="$PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools"
RUN yes | $ANDROID_SDK_ROOT/cmdline-tools/latest/bin/sdkmanager --licenses && \
  $ANDROID_SDK_ROOT/cmdline-tools/latest/bin/sdkmanager "build-tools;30.0.3" "platform-tools" "platforms;android-29" "sources;android-29"
RUN sdkmanager --install "ndk;26.3.11579264"
RUN chown -R vscode:vscode /usr/local/android-sdk
#ENV ADB_SERVER_SOCKET=tcp:host.docker.internal:5037

# asdf のインストール
ENV HOME=/home/vscode
ENV ASDF_DIR=$HOME/.asdf
ENV PATH="$ASDF_DIR/bin:$ASDF_DIR/shims:$PATH:$HOME/.pub-cache/bin"
RUN git clone https://github.com/asdf-vm/asdf.git --branch v0.14.0 $ASDF_DIR && chown -R vscode:vscode $ASDF_DIR

# .tool-versions をコンテナへコピー(ビルド時のバージョン指定用)
COPY .tool-versions /workspace/.tool-versions
WORKDIR /workspace

# asdf 管理に Flutter プラグインを追加し、.tool-versions に基づいて Flutter をインストール
RUN . $ASDF_DIR/asdf.sh && asdf plugin-add flutter https://github.com/oae/asdf-flutter.git
RUN . $ASDF_DIR/asdf.sh && asdf install flutter
RUN sudo chown -R vscode:vscode $ASDF_DIR
RUN git config --global --add safe.directory $ASDF_DIR

# Flutter のインストール確認
USER vscode
RUN flutter doctor

# Flutter Web の有効化
RUN flutter config --enable-web

# VSCode の Flutter Extension 用に実際のPATHを通す
ENV PATH="$ASDF_DIR/installs/flutter/$(grep flutter .tool-versions | cut -d ' ' -f 2)/bin:$PATH"

# Dart パッケージのインストール
#RUN dart pub global activate build_runner
#RUN dart pub global activate melos
