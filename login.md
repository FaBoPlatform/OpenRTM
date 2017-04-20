# RaspberryPIへのログインとテスト

RaspberryPIへのログインは、RaspberryPIをHost Access Pointにしてログインする。　
事前に、下記コマンドを実行し、Host Access Pointのモードになっている事を前提に説明する。

```shell
$ sudo /opt/fabo/bin/wifi-switch --mode ap
```

## アクセスポイント

* SSID: PI3-AP#####
* PASS: raspberry

のスポットに接続する。#####はMacアドレスの4桁が割り振られる。

## Jupyterでのログイン

Browserを起動し、Jupyterで接続する。

* http://172.31.0.1:8888/

Tokenは、'fabo'。

## MotorShield Libのインストル　

```shell
$ sudo pip install git+https://github.com/FaBoPlatform/MotorShield
```

## Test Code

RobotCarのTestをおこう。
下記フォルダにアクセスし、上から順にテストコードを実行する。

* Documents/OpenRTM/Test/RobotCarTest.ipynb

## OpenRTM Test

### RaspberryPI側

RTCのRobotControllerをJupyter上で動かしてみる。

* Documents/OpenRTM/RobotController/RobotController.ipynb

### Localマシン側

```shell
$ rtm-naming
```

PS4Controllerを実行

```shell
$ python main.py
```

![](/img/test001.png)

![](/img/test002.png)

![](/img/test003.png)

![](/img/test101.png)

![](/img/test102.png)

![](/img/test201.png)




