# OpenRTMのインストール

## OpenRTMのRepositoryを追加

`/etc/apt/sources.list`の最後に、OpenRTMのRepoを追加する。

```shell
 deb http://openrtm.org/pub/Linux/raspbian/ jessie main
```

```shell
$ sudo apt-get update
```
## OmniORBのインストール

```
$ sudo apt-get install omniidl-python
```

## OpenRTMのインストール

```
$ sudo apt-get install openrtm-aist openrtm-aist-dev openrtm-aist-example
$ sudo apt-get install openrtm-aist-python openrtm-aist-python-example
```

## 動作確認

Jupyerを起動して、下記コマンドで、omniorbが起動している事を確認する。

```
!ps -aux | grep omni 
```

