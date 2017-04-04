# HelloRTC

簡単なRTCを作成する。

## 定義

```shell
!export RTC_MANAGER_CONFIG="/home/pi/OpenRTM/rtc.conf"
```

## Import

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# -*- Python -*-

import sys
import time
import RTC
import OpenRTM_aist
import robotcar
```

## コンフィギュレーションパラメータ

```python
seqin_spec = ["implementation_id", "HelloRTC",
              "type_name",         "HelloRTCComponent",
              "description",       "HelloRTC component",
              "version",           "1.0",
              "vendor",            "GClue, Inc.",
              "category",          "robot",
              "activity_type",     "DataFlowComponent",
              "max_instance",      "10",
              "language",          "Python",
              "lang_type",         "script",
              ""]
```

### RTCクラス


```python
class HelloRTC(OpenRTM_aist.DataFlowComponentBase):
    
    def __init__(self, manager):
        print "__init__"
        OpenRTM_aist.DataFlowComponentBase.__init__(self, manager)
        
        return

    def onInitialize(self):
        print "onInitialize"
        return RTC.RTC_OK
    
    def onExecute(self, ec_id):
        print "onExcute"
        return RTC.RTC_OK
```

## RTCクラス呼び出し

```python
def HelloRTCInit(manager):
    print "HelloRTCInit"
    profile = OpenRTM_aist.Properties(defaults_str=seqin_spec)
    manager.registerFactory(profile,
                          HelloRTC,
                          OpenRTM_aist.Delete)

def MyModuleInit(manager):
    print "MyModuleInit"
    HelloRTCInit(manager)

    # Create a component
    comp = manager.createComponent("HelloRTC")

def main():
    print "run"
    mgr = OpenRTM_aist.Manager.init(sys.argv)
    mgr.setModuleInitProc(MyModuleInit)
    mgr.activateManager()
    mgr.runManager()
    
if __name__ == "__main__":
    main()
```

