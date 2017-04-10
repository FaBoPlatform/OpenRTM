# 104 Angle

## AngleをRTC化してADCを試す

## rtc.confの作成。

まず、`/home/pi/OpenRTM/104/`フォルダを作成し、ここにrtc.confを生成する。

`/home/pi/OpenRTM/104/rtc.conf`

```shell
corba.nameservers: localhost
exec_cxt.periodic.rate:1000.0
naming.formats: %n.rtc
logger.enable: YES
logger.file_name: rtc.log
```

この設定で、起動履歴がrtc.logに保存される。エラーやうまく動かない場合は、ここを参照する。

## Jupyterのコード

```python
#!/usr/bin/env python
# coding: utf-8
# Python 

import spidev
import time
import sys
import RTC
import OpenRTM_aist

DEFAULT_ANGLE_PIN = 0

fabo_angle_spec = ["implementation_id", "FaBoAngle",
              "type_name",      "FaBoAngleComponent",
              "description",      "FaBo Angle component",
              "version",            "1.0",
              "vendor",            "GClue, Inc.",
              "category",          "fabo",
              "activity_type",     "DataFlowComponent",
              "max_instance",   "10",
              "language",          "Python",
              "lang_type",         "script",
              "conf.default.angle_pin", str(DEFAULT_ANGLE_PIN),
              ""]

class FaBoAngle(OpenRTM_aist.DataFlowComponentBase):
    
    def readadc(self, channel):
        """
        Analog Data Converterの値を読み込む
        @channel チャンネル番号
        """
        adc = self._spi.xfer2([1,(8+channel)<<4,0])
        data = ((adc[1]&3) << 8) + adc[2]
        return data

    def map(self, x, in_min, in_max, out_min, out_max):
        """
        map関数
        @x 変換したい値
        @in_min 変換前の最小値
        @in_max 変換前の最大値
        @out_min 変換後の最小
        @out_max 変換後の最大値
        @return 変換された値
        """
        return (x - in_min) * (out_max - out_min) // (in_max - in_min) + out_min

    def __init__(self, manager):
        print "__init__"
        OpenRTM_aist.DataFlowComponentBase.__init__(self, manager)
        
        # Set default pin.
        self._angle_pin =  [DEFAULT_ANGLE_PIN]

        return

    def onInitialize(self):
        print "onInitialize"
        
        # Bind parameter.
        self.bindParameter("angle_pin", self._angle_pin, str(DEFAULT_ANGLE_PIN))
        
        # Add port.
        self._angle     = RTC.TimedShort(RTC.Time(0,0),0)
        self._angleOut    = OpenRTM_aist.OutPort("ANGLE", self._angle)
        self.addOutPort("ANGLE", self._angleOut)

        # Init SPI.
        self._spi = spidev.SpiDev()
        self._spi.open(0,0)
        
        return RTC.RTC_OK
    
    def onActivated(self, ec_id):
        print "onActivated"
        print self._angle_pin[0]
        return RTC.RTC_OK
    
    def onExecute(self, ec_id):
        #print "onExcute"
        
        # Read pin
        data = self.readadc(self._angle_pin[0])
        value = self.map(data, 0, 1023, 0, 100)
        sys.stdout.write("\rch = %d, value = %d  " % (self._angle_pin[0], value))
        sys.stdout.flush()
    
        # Write value to port.
        self._angle.data = value
        self._angleOut.write()
            
        return RTC.RTC_OK

def FaBoAngleInit(manager):
    print "FaBoAngleInit"
    
    profile = OpenRTM_aist.Properties(defaults_str=fabo_angle_spec)
    manager.registerFactory(profile,
                            FaBoAngle,
                            OpenRTM_aist.Delete)    

def MyModuleInit(manager):
    print "MyModuleInit"
    
    FaBoAngleInit(manager)
    comp = manager.createComponent("FaBoAngle")

def main():
    print "main"
    mgr = OpenRTM_aist.Manager.init(sys.argv)
    mgr.setModuleInitProc(MyModuleInit)
    mgr.activateManager()
    mgr.runManager()
    
if __name__ == "__main__":
    main()
```

## 動作確認

JuputerでShift + Enterで実行すれば、RTCのインスタンスが生成される。

ローカルのMacのEclipseでシステム・エディタを立ち上げておく。

ローカルのMacのExampleからSeqIOのフォルダからSeqIn.pyを起動する。

```shell
$ python2 SeqIn.python2
```

configurationのパラメーターを変更する。0がDefaultなので、1に変更してみる。

![](/img/104_001.png)

![](/img/104_002.png)

![](/img/104_003.png)

2回適用ボタンを押さないと反映されない。

![](/img/104_004.png)

![](/img/104_005.png)

RTC間を接続。

![](/img/104_006.png)

Activate。

![](/img/104_007.png)

Mac側のコンソールでログをチェック。

![](/img/104_008.png)

## Jupyter Code

[https://github.com/FaBoPlatform/OpenRTM/blob/master/jupyter/104/104_Angle.ipynb](https://github.com/FaBoPlatform/OpenRTM/blob/master/jupyter/104/104_Angle.ipynb)



