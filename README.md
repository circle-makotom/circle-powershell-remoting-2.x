# CircleCI + Windows + PowerShell remoting (WinRM)

このデモでは、 CircleCI の Windows ビルドを使用して, リモート環境の Windows Server にファイルをコピーの上, 任意のコマンドをリモート実行する実装例を紹介します.
ファイルのコピーおよびコマンドのリモート実行には [WinRM を使用した PowerShell のリモート実行](https://docs.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_remote?view=powershell-5.1) を使用します.

このデモは Windows Server 2016 で検証の上作成していますが, 技術的には Windows Server 2019 でも同様に動作することが期待されています (ただし未検証).

このデモは CircleCI server 2.x を想定して作成されています. それ以外の CircleCI 環境では想定通り動作しない場合があります.

このデモでは CircleCI server の Windows ビルドが事前に有効化されていることが想定されています. Windows ビルドの有効化については[こちらに参考情報があります](https://gist.github.com/makotom/2aa76b7b68f12357357c672f9349321e).

## 事前設定 1: CircleCI の build-agent の最新化

セキュリティ上の制約に対処するため, 下記手順により最新の build-agent を使用するべく CircleCI の設定を変更する必要があります.

注意: 本手順による CircleCI の完全停止は基本的には発生しません. ただし, 手順実行中はジョブの開始が最大 5 分程度遅延する可能性があり, かつ設定不備があった場合はジョブを一切実行できない状態になるため, 基本的にはメンテナンス期間を設けたうえで当該期間中に下記手順を実施することをおすすめいたします.

1.  CircleCI のサービス ボックスに SSH でログインします.
2.  開いた SSH シェル上で下記 3 行のコマンドを連続して実行します.

        sudo mkdir -p /etc/circleconfig/picard-dispatcher
        echo "export CIRCLE_BUILD_AGENT=circleci/picard:1.0.54207-5704a5fc" | sudo tee -a /etc/circleconfig/picard-dispatcher/customizations
        sudo docker restart picard-dispatcher

3.  `picard-dispatcher` コンテナが再起動するのを待ちます. (標準で 2 分ほどかかります.)

    起動が完了すると, `sudo docker logs picard-dispatcher` コマンドの出力に `picard-dispatcher.init Still running...` というメッセージやジョブを処理したことを示すイベントが表示されるようになります.

## 事前設定 2: デプロイ先での PowerShell remoting の有効化

デプロイ先に PowerShell remoting でアクセスできるよう構成する必要があります.

確認している必要要件は下記のとおりです:

* 5986/tcp ポートを使用できること.
* SSL を使用できること.
* NTLM を使用したパスワード ベースの認証ができること.

なお, 参考までに, 本稿執筆に際しては [こちらのサンプル スクリプト](https://gist.github.com/makotom/70fa58701fd06a383411bed3f8236767) にて PowerShell remoting が有効にできることを確認しています.

## 実際にデモ プログラムを動かす

`config.yml` にハードコードされているホスト名・ユーザー名・パスワードを適宜書き換え, CircleCI 上でプロジェクトをセットアップし, ジョブを実行します.
