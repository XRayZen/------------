# VSCode+DockerでGoの開発環境を構築
>[参考記事](https://pythonchan.hatenablog.com/entry/2022/04/18/092904)
>[VSCodeとDockerでMacにGolangの開発環境を作成する](https://dev.classmethod.jp/articles/vscode-remote-containers-golang/)
>[VSCode devcontainerでローカルを汚さずに、快適なGo言語の開発環境を整える](https://zenn.dev/bun913/articles/f0a6c6177a4716)

Macの開発環境を汚さずにGoの開発環境を構築する
- チームメンバーがVSCodeを活用している場合、チーム全体で同じ環境を共有できる
  - ライブラリのバージョン違い等で悩む必要がなくなる

## 1. VSCodeの拡張機能をインストール
- VSCodeのExtensionであるRemote Containersをインストールする。
  - Dockerを動かしつつ、VS codeで開発していくのに必要なものです。

![Alt text](https://cdn-ak.f.st-hatena.com/images/fotolife/p/pythonchan/20220417/20220417150813.png)

## 2. Remote-Containersの設定をする
- Window左下のマークをクリック
![Alt text](https://d1tlzifd8jdoy4.cloudfront.net/wp-content/uploads/2019/07/f02d50335b6d25536ed0492a4e631276.png)
- `Remote-Containers: Add Development Container Configuration Files...`を選択
- 何の開発環境を作るのか聞かれるのでgoと入力し、Go golang:1を選択します。
- この状態で、右下のReopen in Containerボタンを押すか、左下のアイコンをクリックしてRemote-Containers: Reopen Folder in Containerを選択すると、VSCodeがリロードされ、開発用のコンテナの作成等が行われます。
![Alt text](https://d1tlzifd8jdoy4.cloudfront.net/wp-content/uploads/2019/07/641828279a85cfc8aca8cf6c345708ed.png)
- 作成が終了すると、ファイル一覧に先程の設定ファイルが見えるようになる
- ターミナルを開いて、go versionを実行すると、golangのバージョンが表示される
# .devcontainers.json
環境については、プロジェクト直下の「.devcontainer」フォルダに格納されています。

devcontainer.jsonにはリモート先のVS Codeの拡張機能や設定、Dockerfileにはコンテナの定義が記載されています。
このファイルを元にして、環境をカスタマイズするのも良いでしょう。

Dockerファイルを修正したらF1キーを押しコマンドパレットから、「Rebuild Container」でコンテナの再作成ができます。
```json
// See https://aka.ms/vscode-remote/devcontainer.json for format details.
{
	"name": "Go",
	"dockerFile": "Dockerfile",
	"runArgs": [
		"--cap-add=SYS_PTRACE",
		"--security-opt", "seccomp=unconfined"
	],
	
	// Uncomment the next line if you want to publish any ports.
	// "appPort": [],

	// Uncomment the next line to run commands after the container is created.
	// "postCreateCommand": "go version",

	"extensions": [
		"ms-vscode.go"
	],
	"settings": {
		"go.gopath": "/go"
	}
}
```
([リファレンス](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference))

例えば、extensionsという項目でコンテナ内のVSCodeに最初からインストールするExtensionを指定することができます。

またデフォルトでは、Dockerfileを使って開発用のコンテナを作成していますが、docker-composeをコンテナの作成をすることもできます。





