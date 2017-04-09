# 103 Button

## ButtonをRTC化してLEDを点灯する

```python
#!/usr/bin/env python
# coding: utf-8
# Python 

import RPi.GPIO as GPIO
import time
import sys
import RTC
import OpenRTM_aist

fabobutton_spec = ["implementation_id", "FaBoButton",
              "type_name",      "FaBoButtonComponent",
              "description",      "FaBo Button component",
              "version",            "1.0",
              "vendor",            "GClue, Inc.",
              "category",          "fabo",
              "activity_type",     "DataFlowComponent",
              "max_instance",   "10",
              "language",          "Python",
              "lang_type",         "script",
              ""]
BUTTONPIN = 5

class FaBoButton(OpenRTM_aist.DataFlowComponentBase):
    
    def __init__(self, manager):
        print "__init__"
        OpenRTM_aist.DataFlowComponentBase.__init__(self, manager)
        
        self._button     = RTC.TimedBoolean(RTC.Time(0,0),False)
        self._buttonOut    = OpenRTM_aist.OutPort("BUTTON", self._button)
        self.addOutPort("BUTTON", self._buttonOut)
        
        return

    def onInitialize(self):
        print "onInitialize"
        
        GPIO.setwarnings(False)
        GPIO.setmode( GPIO.BCM )
        GPIO.setup(BUTTONPIN, GPIO.IN )
        return RTC.RTC_OK
    
    def onExecute(self, ec_id):
        print "onExcute"
        
        if( GPIO.input(BUTTONPIN)):
            self._button.data = True
            self._buttonOut.write()
        else:
            self._button.data = False
            self._buttonOut.write()
        
        return RTC.RTC_OK

def FaBoButtonInit(manager):
    print "FaBoButtonInit"
    profile = OpenRTM_aist.Properties(defaults_str=fabobutton_spec)
    manager.registerFactory(profile,
                            FaBoButton,
                            OpenRTM_aist.Delete)    

def MyModuleInit(manager):
    print "MyModuleInit"
    FaBoButtonInit(manager)

    # Create a component
    comp = manager.createComponent("FaBoButton")

def main():
    print "main"
    mgr = OpenRTM_aist.Manager.init(sys.argv)
    mgr.setModuleInitProc(MyModuleInit)
    mgr.activateManager()
    mgr.runManager()
    
if __name__ == "__main__":
    main()
```

