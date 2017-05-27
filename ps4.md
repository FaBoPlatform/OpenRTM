# PS4 ControllerをRTC化

## PS4のMacBook上での認識

USBをでMacBookにPS4 Wireless Controllerを接続して認識されると、淡いオレンジ色に光始める。

![](/img/ps000.png)

Mac上では、下記のUSBデバイスとして認識される。

![](/img/ps001.png)

![](/img/ps002.png)

![](/img/ps003.png)

## Pygameのインストール

2.7系のPythonでPygameをInstallする。CFLAGSの環境変数も定義して実行。

```shell
$ brew install sdl sdl_image sdl_mixer sdl_ttf portmidi
$ CFLAGS='-I/usr/local/include/SDL' pip install pygame
```

## プログラム

main.py

```python

#!/usr/bin/env python
# coding: utf-8 

import sys
import time
import RTC
import OpenRTM_aist
import pygame
import PS4Controller

DEBUG = True

gamecontroller_spec = [ 
  "implementation_id",  "RTCGameController",
  "type_name",      "GameController",
  "description",      "PS4 Controller for MAC",
  "version",        "1.0.0",
  "vendor",       "GClue, Inc.",
  "category",       "Controller",
    "activity_type",      "DataFlowComponent",
  "max_instance",     "1",
  "language",       "Python",
  "lang_type",      "script"
]

class RTCGameController(OpenRTM_aist.DataFlowComponentBase):

  def __init__(self, manager):
    if DEBUG: print "__init__" 
    self.m_Stick_Gain = [1.0]
    OpenRTM_aist.DataFlowComponentBase.__init__(self, manager)
    return

  def onInitialize(self):
    self.bindParameter("Stick_Gain", self.m_Stick_Gain, "1.0");

    # define variable
    self._analogSeq = RTC.TimedDoubleSeq(RTC.Time(0,0),[])
    self._buttonSeq   = RTC.TimedLongSeq(RTC.Time(0,0),[])
    self._stringSeq = RTC.TimedString(RTC.Time(0,0),"")

    # set port
    self._analogSeqOut = OpenRTM_aist.OutPort("Analog", self._analogSeq)
    self._buttonSeqOut   = OpenRTM_aist.OutPort("Button", self._buttonSeq)
    self._stringSeqOut   = OpenRTM_aist.OutPort("Controller_Type", self._stringSeq)

    # Add outport
    self.addOutPort("Analog", self._analogSeqOut)
    self.addOutPort("Button", self._buttonSeqOut)
    self.addOutPort("Controller_Type", self._stringSeqOut)

    # Write
    self._stringSeq.data = "Wireless Controller"
    self._stringSeqOut.write()

    return RTC.RTC_OK

  def onExecute(self, ec_id):

    analogSeq = []
    buttonSeq = []

    # Analog
    analogSeq.append(float(self.hat_data[0][1]))  # HAT Up Down
    analogSeq.append(float(self.hat_data[0][0]))  # HAT Lefft Right
    analogSeq.append(float(self.axis_data[1] * float(self.m_Stick_Gain[0])))  # Left Stick Up Down
    analogSeq.append(float(self.axis_data[0] * float(self.m_Stick_Gain[0])))  # Left Stick Lefft Right
    analogSeq.append(float(self.axis_data[3] * float(self.m_Stick_Gain[0])))  # Right Stick Up Down
    analogSeq.append(float(self.axis_data[2] * float(self.m_Stick_Gain[0])))  # Right Stick Lefft Right
    analogSeq.append(float(self.axis_data[4]))  # L2 Button
    analogSeq.append(float(self.axis_data[5]))  # R2 Button
    self._analogSeq.data = analogSeq
    
    # Button
    for i in self.button_data:
      buttonSeq.append(long(self.button_data[i]))
    self._buttonSeq.data = buttonSeq
    self._buttonSeqOut.write()

    # Wirte to port
    self._analogSeqOut.write()
    self._buttonSeqOut.write()

    time.sleep(0.1)

    return RTC.RTC_OK

  def set_pos(self, axis_data, button_data, hat_data):
    self.axis_data = axis_data
    self.button_data = button_data
    self.hat_data = hat_data

def main():
  if DEBUG: print "main"

  # Initilize
  mgr = OpenRTM_aist.Manager.init(sys.argv)
  mgr.activateManager()

  # Register component
  profile = OpenRTM_aist.Properties(defaults_str=gamecontroller_spec)
  mgr.registerFactory(profile,
    RTCGameController,
    OpenRTM_aist.Delete)

  # Create a component
  comp = mgr.createComponent("RTCGameController")
  ps4controller = PS4Controller.PS4Controller()
  ps4controller.set_on_update(comp.set_pos)
  mgr.runManager(True)

  # Start thread
  ps4controller.start()

if __name__ == "__main__":
  main()

```

PS4Controller.py

```python

#!/usr/bin/env python
# -*- coding: utf-8 -*-
# -*- Python -*-
import pygame
import os
import time

DEBUG = False

class PS4Controller():

  def __init__(self):
    # Initilize joystick
    pygame.init()
    pygame.joystick.init()
    joysticks = [pygame.joystick.Joystick(x) for x in range(pygame.joystick.get_count())]
    for joystick in joysticks:
      if joystick.get_name() == "Wireless Controller":
        self.ps4 = joystick

    self.ps4.init()

    if DEBUG :
      print "name:" + self.ps4.get_name()
      print "numbuttons: %d" % self.ps4.get_numbuttons()
      print "numaxes:%d" % self.ps4.get_numaxes()
      print "numballs:%d" % self.ps4.get_numballs()
      print "numhats:%d" % self.ps4.get_numhats()

    self.axis_data = {}
    self.button_data = {}
    for i in range(self.ps4.get_numbuttons()):
      self.button_data[i] = 0
    self.hat_data = {}
    for i in range(self.ps4.get_numhats()):
      self.hat_data[i] = (0, 0)
    self.running = False

  def stop(self):
    self.running = False

  def start(self):
    self.running = True
    while self.running:
      for event in pygame.event.get():
        if event.type == pygame.JOYAXISMOTION:
          self.axis_data[event.axis] = round(event.value,2)
          self.update()
          time.sleep(0.2)
        elif event.type == pygame.JOYBUTTONDOWN:
          self.button_data[event.button] = 1
          self.update()
        elif event.type == pygame.JOYBUTTONUP:
          self.button_data[event.button] = 0
          self.update()
        elif event.type == pygame.JOYHATMOTION:
          self.hat_data[event.hat] = event.value
          self.update()
  
  def update(self):
    if self.on_update != None:
      self.on_update(self.axis_data, self.button_data, self.hat_data)

  def set_on_update(self, func):
    self.on_update = func
```