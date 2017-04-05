# OpenRTMのOSXへのインストール

## 必要な条件

* 2.x系のPython
* usc2でBuildしたPython(一般のインストールパッケージはusc4でBuildされている)

## OpenRTM-aistのインストール

http://sugarsweetrobotics.com/?page_id=111のページより、

* OpenRTM-aist-C++-1.1.2-OSX10.9.dmg

をダウロードしてくる。
dmgをクリックし、OpenRTM-aist-C++-1.1.2-Release-OSX10.9.pkg をインストールする。


`/usr/local/lib/python2.7/site-packages` に

* omniORB
* omniORB.pth
* omniidl
* omniidl_be

などなどがインストールされる。

また、namingサーバもインストールされる。

* /usr/local/bin/rtm-naming
* /usr/local/bin/omniNames

rtm-naming コマンドで、namingサーバを起動する。

```shell
$ rtm-naming

Starting omniORB omniNames: Dev-2.local:2809
omniNames: (0) 2017-04-04 20:17:48.386611: Data file: /Users/sasakiakira/omninames-Dev-2.local.dat.
omniNames: (0) 2017-04-04 20:17:48.386810: Starting omniNames for the first time.
omniNames: (0) 2017-04-04 20:17:48.387177: Wrote initial data file /Users/sasakiakira/omninames-Dev-2.local.dat.
omniNames: (0) 2017-04-04 20:17:48.387589: Read data file /Users/sasakiakira/omninames-Dev-2.local.dat successfully.
omniNames: (0) 2017-04-04 20:17:48.387670: Root context is IOR:010000002b00000049444c3a6f6d672e6f72672f436f734e616d696e672f4e616d696e67436f6e746578744578743a312e300000010000000000000074000000010102000f00000031302e3230322e3136362e3133320000f90a00000b0000004e616d6553657276696365000300000000000000080000000100000000545441010000001c000000010000000100010001000000010001050901010001000000090101000354544108000000dc80e3580100a190
omniNames: (0) 2017-04-04 20:17:48.387756: Checkpointing Phase 1: Prepare.
omniNames: (0) 2017-04-04 20:17:48.387898: Checkpointing Phase 2: Commit.
omniNames: (0) 2017-04-04 20:17:48.388306: Checkpointing completed.
omniNames properly started
```

Eclipseの<RT System Editor>パースペクティブを表示し、localhostを追加する。

![](/img/dev101.png)

![](/img/dev102.png)

![](/img/dev103.png)


## OpenRTM-aist-Pythonのインストール

Python版のOpenRTM-aistをインストールする。

2.7系のPythonが必須である。usc4でBuildされたPythonではエラーがでる。その場合は、usc2でBuildし直す。

usc2でBuildしたPythonの例

```
$ PYTHON_CONFIGURE_OPTS="--enable-unicode=ucs2" pyenv install 2.7.10
```

```shell
$ wget http://openrtm.org/pub/OpenRTM-aist/python/1.1.2/OpenRTM-aist-Python-1.1.2.tar.gz
$ tar xvfz OpenRTM-aist-Python-1.1.2.tar.gz
$ cd OpenRTM-aist-Python-1.1.
$ pyenv shell --unset
$ pyenv local 2.7.10
$ python setup.py build_core
$ python setup.py install_core
$ python setup.py build_example
$ python setup.py build_example
```


## サンプルコードを実行する

`/Users/ユーザ名/.pyenv/versions/2.7.10/share/openrtm-1.1/example/`

あたりに、サンプルも生成されるので、サンプルを実行してみる。

パスを通す
```shell
$ export PYTHONPATH=$PYTHONPATH:/usr/local/lib/python2.7/site-packages/:/Users/ユーザ名/.pyenv/versions/2.7.10/lib/python2.7/site-packages/
```


```shell
$ cd /Users/ユーザ名/.pyenv/versions/2.7.10/share/openrtm-1.1/example/python/SeqIO
$ python SeqIn.py
```

Eclipseの<RT System Editor>パースペクティブを表示し、localhostに起動したRTCが追加された事を確認する。

![](/img/dev104.png)

システムダイアグラムを表示し、RTCを貼り付ける。

![](/img/dev105.png)

![](/img/dev106.png)


## 参考

* [MacOSX + OpenRTM-aist](http://ysuga.net/?p=206)
* [OpenRTM-aistをMac OS X Mavericksにインストールする](http://qiita.com/switchback_sus4/items/25a969fcc30da2cdff3b)
