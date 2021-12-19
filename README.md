# playground-python-mecab

Pythonでmecabを利用するサンプルコードです。
よく利用される辞書の mecab-ipadic-neologd も使ってみます。

mecabはGit submoduleで参照しています。mecabのアップストリームが更新された場合でも、このリポジトリで参照されるコミット変更されないため、
更新したい場合はGit submoduleの作法で更新してください。

## インストール

playground-python-mecabをサブモジュール含めてチェックアウトします。

```shell
git clone https://github.com/fuji44/playground-python-mecab.git
cd playground-python-mecab
git submodule update --init
```

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
```

```shell
# mecab-ipadic
cd ./mecab/mecab-ipadic
./configure --with-charset=utf8
make
sudo make install
```


```shell
# mecab python bind
cd ./mecab/mecab/swig
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
