# playground-python-mecab

Pythonでmecabを利用するサンプルコードです。
よく利用される辞書の mecab-ipadic-neologd も使ってみます。

mecabはGit submoduleで参照しています。mecabのアップストリームが更新された場合でも、このリポジトリで参照されるコミット変更されないため、
更新したい場合はGit submoduleの作法で更新してください。

## 前提

以下を前提にしています。

- Python3系がインストールされていること
- C++コンパイラ(gccなど)がインストールされていること
- swigがインストールされていること

[devcontainer](https://code.visualstudio.com/docs/remote/containers)を利用すれば、これらが整ったコンテナで実行できます。

## インストール

### チェックアウト

#### Option 1: (推奨) devcontainer

> devcontainerを使うには、VSCodeとRemote Containers拡張機能が必要なので、事前にインストールしておいてください。使い方は、[公式ドキュメント](https://code.visualstudio.com/docs/remote/containers)を参照してください。

VSCode Remote Containersを使って、このリポジトリをコンテナボリュームにクローンするのがもっとも簡単です。

既にローカル端末にクローンしている場合は、それを利用することもできますが、この場合はパフォーマンスが低下します。私の環境ではターミナルでのコマンド操作をためらうレベルでおそくなったので、コンテナボリュームへのクローンをお勧めします。

初回は、コンテナイメージをビルドするため起動までに時間がかかります。2回目以降はビルドされたコンテナを利用するので早いです。

VSCodeが開いたらターミナルで、以下のコマンドを実行し、submoduleを取得してください。

```shell
git submodule update --init
```


#### Option 2: ローカル

ローカル端末に直接インストールする場合は、単純にgit cloneするだけです。
playground-python-mecabをサブモジュール含めてチェックアウトします。

```shell
git clone https://github.com/fuji44/playground-python-mecab.git
cd playground-python-mecab
git submodule update --init
```

後の手順を実行するには、前提に記述したソフトウェア類が必要なため、別途準備する必要があることに注意してください。

### MeCabのビルド・インストール

mecabと辞書(ipadic)、Pythonバインディングパッケージをビルド

```shell
# mecab
cd ./mecab/mecab
./configure
make
make check
sudo make install
sudo ldconfig

# mecab-ipadic
cd ../mecab-ipadic
./configure --with-charset=utf8
make
sudo make install

# mecab python bind
cd ../mecab/swig
make python
cd ../python
python3 setup.py build
sudo python3 setup.py install
```

#### Option: mecab-ipadic-NEologd

カスタム辞書 mecab-ipadic-NEologd を使いたいならこちらも実行します。
辞書を更新する場合も同じらしいです。

```shell
# mecab-ipadic-NEologd
cd mecab-ipadic-neologd
./bin/install-mecab-ipadic-neologd -n
```

### 確認

mecabコマンドが正しく動作することを確認します。

```shell
$ echo "本日は晴天なり。" | mecab
本日    名詞,副詞可能,*,*,*,*,本日,ホンジツ,ホンジツ
は      助詞,係助詞,*,*,*,*,は,ハ,ワ
晴天    名詞,一般,*,*,*,*,晴天,セイテン,セイテン
なり    助動詞,*,*,*,文語・ナリ,基本形,なり,ナリ,ナリ
。      記号,句点,*,*,*,*,。,。,。
EOS
```

python3で正しく動作することを確認します。
以下の例ではインタプリタで確認しています。

```python
>>> import MeCab
>>> m = MeCab.Tagger()
>>> print(m.parse("本日は晴天なり。"))
本日    名詞,副詞可能,*,*,*,*,本日,ホンジツ,ホンジツ
は      助詞,係助詞,*,*,*,*,は,ハ,ワ
晴天    名詞,一般,*,*,*,*,晴天,セイテン,セイテン
なり    助動詞,*,*,*,文語・ナリ,基本形,なり,ナリ,ナリ
。      記号,句点,*,*,*,*,。,。,。
EOS
```


#### Option: mecab-ipadic-NEologd

カスタム辞書 mecab-ipadic-NEologd を正しく使えるか確認します。

コマンド

```shell
$ echo "本日は晴天なり。" | mecab -d /usr/local/lib/mecab/dic/mecab-ipadic-neologd
本日は晴天なり  名詞,固有名詞,人名,一般,*,*,本日は晴天なり,ホンジツハセイテンナリ,ホンジツハセイテンナリ
。      記号,句点,*,*,*,*,。,。,。
EOS
```

python3

```python
>>> import MeCab
>>> m = MeCab.Tagger("-d /usr/local/lib/mecab/dic/mecab-ipadic-neologd")
>>> print(m.parse("本日は晴天なり。"))
本日は晴天なり  名詞,固有名詞,人名,一般,*,*,本日は晴天なり,ホンジツハセイテンナリ,ホンジツハセイテンナリ
。      記号,句点,*,*,*,*,。,。,。
EOS
```


## 開発メモ

### submoduleを更新する

流れは簡単。

- サブモジュールの任意のコミットをチェックアウトする。
- メインモジュールでサブモジュールの変更をコミットする。

```shell
cd mecab
git checkout d717d499afe17b6d8e3899f8c037d41f3a6c5d8a
cd ..
git add mecab
git commit "update mecab"
```
