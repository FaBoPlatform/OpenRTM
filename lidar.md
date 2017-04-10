
# Lidar RTC

## Lidar Lite V3

Lidar Lite V3はGerminが販売する廉価Lidarである。Lidar Liteは、固定で0x62のSlaveアドレスが割り振られているが、これをActivate時にかえるために、bindParameterを使用する。

また、Lidar Lite V3を最大 3つ同時に使うために3つのインスタンスを起動する。

## Config

`rtc.conf`

```shell
corba.nameservers: localhost
naming.formats: %n.rtc
logger.enable: NO
lidar.RTCLidar0.config_file: RTCLidar0.conf
lidar.RTCLidar1.config_file: RTCLidar1.conf
lidar.RTCLidar2.config_file: RTCLidar2.conf
manager.components.precreate: RTCLidar,RTCLidar
```

`RTCLidar0.conf`

```shell
exec_cxt.periodic.rate:1000.0
naming.formats: %m0.rtc
```

`RTCLidar1.conf`

```shell
exec_cxt.periodic.rate:1000.0
naming.formats: %m1.rtc
```

`RTCLidar2.conf`

```shell
exec_cxt.periodic.rate:1000.0
naming.formats: %m2.rtc
```

## Source


`main.py`

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# -*- Python -*-

import sys
import time

import RTC
import OpenRTM_aist

import lidar
reload(lidar)

lidar_spec = ["implementation_id", "RTCLidar",
              "type_name",         "RTCLidar",
              "description",       "RTCLidar Component",
              "version",           "1.0",
              "vendor",            "GClue, Inc.",
              "category",          "lidar",
              "activity_type",     "DataFlowComponent",
              "max_instance",      "10",
              "language",          "Python",
              "lang_type",         "script",
              "conf.default.i2c_slave_address", "98",
              ""]

class RTCLidar(OpenRTM_aist.DataFlowComponentBase):
    
    def __init__(self, manager):
        print "__init__"
        OpenRTM_aist.DataFlowComponentBase.__init__(self, manager)
        self._i2c_slave_address = [98]
        self.mlidar = lidar.LidarLite()
        return

    def onInitialize(self):
        print "onInitialize"
        self._d_IntOut = RTC.TimedLong(RTC.Time(0,0),0)
        self._IntOutOut = OpenRTM_aist.OutPort("IntOut", self._d_IntOut)
        self.addOutPort("IntOut",self._IntOutOut)
        
        self.bindParameter("i2c_slave_address", self._i2c_slave_address, "98")
        
        return RTC.RTC_OK
    
    def onActivated(self, ec_id):
        print "onActivate"
        print self._i2c_slave_address[0]
        self.mlidar.chageSlaveAddress(int(self._i2c_slave_address[0]))
        print "Finish I2C"
        return RTC.RTC_OK
        
    def onExecute(self, ec_id):
        print "onExcute"
        self._d_IntOut.data = self.mlidar.getDistance()
        OpenRTM_aist.setTimestamp(self._d_IntOut)
        print self._d_IntOut.data
        self._IntOutOut.write()
        return RTC.RTC_OK

def RTCLidarInit(manager):
    print "RTCLidarInit"
    profile = OpenRTM_aist.Properties(defaults_str=lidar_spec )
    manager.registerFactory(profile,
                          RTCLidar,
                          OpenRTM_aist.Delete)

def MyModuleInit(manager):
    print "MyModuleInit"
    RTCLidarInit(manager)

    # Create a component
    comp = manager.createComponent("RTCLidar")

def main():
    print "run"
    mgr = OpenRTM_aist.Manager.init(sys.argv)
    mgr.setModuleInitProc(MyModuleInit)
    mgr.activateManager()
    mgr.runManager()
    
if __name__ == "__main__":
    main()
```

`lidar.py`

```python
#coding: utf-8
import smbus
import time

bus = smbus.SMBus(1)  # use SMBus
SLAVE_ADDRESS = 0x62 # Slave address of Lider

# Resgiter Address
ADDR_ACQ_COMMAND = 0x00 # DevieConnand
ADDR_STATUS = 0x01 # SystemStatus
ADDR_FULL_DELAY_HIGH = 0x0f # Diatance measurement high byte
ADDR_FULL_DELAY_LOW = 0x10 # Diatance measurement low byte
ADDR_UNIT_ID_HIGH = 0x16 # Serial number low byte
ADDR_UNIT_ID_LOW = 0x17 # Serial number high byte
ADDR_I2C_ID_HIGH = 0x18 # Write serial number high byte for I2C address unlock
ADDR_I2C_ID_LOW = 0x19 # Write serial number low byte for I2C address unlock
ADDR_I2C_SEC_ADDR = 0x1a # Write new I2C address after unlock
ADDR_I2C_CONFIG = 0x1e # Default address response control

class LidarLite():

    def __init__(self, address=SLAVE_ADDRESS):
        print "LidarLite__init__"
        self.address = address

    def getDistance(self):
        # 0x00に0x04の内容を書き込む
        bus.write_block_data(self.address, ADDR_ACQ_COMMAND, [0x04])

        # 0x01を読み込んで、最下位bitが0になるまで読み込む
        value = bus.read_byte_data(self.address, ADDR_STATUS)
        while value & 0x01 == 1:
            value = bus.read_byte_data(self.address, ADDR_STATUS)

        # 0x8fから2バイト読み込んで16bitの測定距離をcm単位で取得する
        high = bus.read_byte_data(self.address, ADDR_FULL_DELAY_HIGH)
        low = bus.read_byte_data(self.address, ADDR_FULL_DELAY_LOW)
        val = ( high << 8 ) + low
        dist = val
        #print "Dist = {0} cm , {1} m".format( dist, dist / 100.0 )
        return dist

    def chageSlaveAddress(self, new_address):
        print "LidarLite.chageSlaveAddress"
        if new_address == None:
            return
        elif new_address == self.address:
            return
        else:
            high = bus.read_byte_data(self.address, ADDR_UNIT_ID_HIGH)
            low = bus.read_byte_data(self.address, ADDR_UNIT_ID_LOW)

            bus.write_byte_data(self.address, ADDR_I2C_ID_HIGH, high)
            bus.write_byte_data(self.address, ADDR_I2C_ID_LOW, low)

            bus.write_block_data(self.address, ADDR_I2C_SEC_ADDR, [new_address])
            bus.write_block_data(self.address, ADDR_I2C_CONFIG, [0x08])

            self.address = new_address

```