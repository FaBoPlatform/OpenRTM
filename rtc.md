# OpenRTMのインストール

## OpenRTMのRepositoryを追加

`/etc/apt/sources.list`の最後に、OpenRTMのRepoを追加する。

```shell
 deb http://openrtm.org/pub/Linux/raspbian/ jessie main
```

```shell
$ apt-get update
```

## 必要なパッケージのインストール

```
$ apt-get -y --force-yes install gcc g++ make uuid-dev
$ apt-get -y --force-yes install openrtm-aist openrtm-aist-dev openrtm-aist-example
$ apt-get -y --force-yes install openrtm-aist-python openrtm-aist-python-example
```

## 動作確認

Jupyerを起動して、下記コマンドで、omniorbが起動している事を確認する。

```
!ps ax | grep omni 
```

