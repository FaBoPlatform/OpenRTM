# Robot Controller RTC


## rtc.conf

# RTC Code

```python
# coding: utf-8
#!/usr/bin/env python
# -*- Python -*-
import sys
import time
import RTC
import OpenRTM_aist
import RPi.GPIO as GPIO
import robotcar

robot_spec = ["implementation_id", "RobotController",
              "type_name",         "RobotControlerComponent",
              "description",       "Robot Controller Component",
              "version",           "1.0",
              "vendor",            "GClue, Inc.",
              "category",          "robot",
              "activity_type",     "DataFlowComponent",
              "max_instance",      "10",
              "language",          "Python",
              "lang_type",         "script",
              ""]

class RobotController(OpenRTM_aist.DataFlowComponentBase):
    
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
    
    def drive(self, power, handle):
        if power < -5:
            self._robot.car_back(-power)
        elif power > 5:
            self._robot.car_forward(power)
        else:
            self._robot.car_stop()
        
        if handle < 8.9 or handle > 9.1:
            self._robot.handle_move(handle)
        else:
            self._robot.handle_move(9)
        
    
    def __init__(self, manager):
        print "__init__"
        OpenRTM_aist.DataFlowComponentBase.__init__(self, manager)
        
        return

    def onInitialize(self):
        print "onInitialize"
        
        # Init robot
        self._robot = robotcar.Robot()
        self._robot.handle_init()
        
        # init port
        self._vector  = RTC.TimedFloatSeq(RTC.Time(0,0),[])
        self._analog  = RTC.TimedDoubleSeq(RTC.Time(0,0),[])
        self._button  = RTC.TimedLongSeq(RTC.Time(0,0),[]) 
        
        self._vectorIn  = OpenRTM_aist.InPort("Vector", self._vector)
        self._analogIn  = OpenRTM_aist.InPort("Analog", self._analog)
        self._buttonIn  = OpenRTM_aist.InPort("Button", self._button)

        # Set InPort 
        self.addInPort("Vector", self._vectorIn)
        self.addInPort("Analog", self._analogIn)
        self.addInPort("Button", self._buttonIn)

        return RTC.RTC_OK
    
    def onExecute(self, ec_id):
        
        vectors_  = self._vectorIn.read()        
        vectorsSize_  = len(vectors_.data)
        buttons_  = self._buttonIn.read()        
        buttonsSize_  = len(buttons_.data)
        analogs_  = self._analogIn.read()        
        analogsSize_  = len(analogs_.data)
        
        power = 0
        handle = 9
        if analogsSize_ > 0:
            sys.stdout.write("\r%10.8s  %10.8s   " % (analogs_.data[2], analogs_.data[5]))
            sys.stdout.flush()
            
            if analogs_.data[2] != 0:
                power = -int(analogs_.data[2])*100
            if analogs_.data[5] != 0:
                handle = self.map(analogs_.data[5]*100, -100, 100, 7.5, 10.5)
                
        if buttonsSize_ > 0:
            if buttons_.data[1] != 0:
                power = self.map(buttons_.data[1], 0, 1, 0, -100)
            elif buttons_.data[3] != 0:
                power = self.map(buttons_.data[3], 0, 1, 0, 100)
                
        if vectorsSize_ > 0:
            power = int(vectors_.data[1])
            handle = int(vectors_.data[0])
            
            power = self.map(power, -150, 150, -100, 100)
            handle = self.map(handle, -150, 150, 7.5, 10.5)
            
        #if power > 0 or handle > 0:
        self.drive(power, handle)
                    
        return RTC.RTC_OK


def RobotControllerInit(manager):
    print "RobotControllerInit"
    profile = OpenRTM_aist.Properties(defaults_str=robot_spec)
    manager.registerFactory(profile,
                          RobotController,
                          OpenRTM_aist.Delete)

def MyModuleInit(manager):
    print "MyModuleInit"
    RobotControllerInit(manager)

    # Create a component
    comp = manager.createComponent("RobotController")

def main():
    print "run"
    mgr = OpenRTM_aist.Manager.init(sys.argv)
    mgr.setModuleInitProc(MyModuleInit)
    mgr.activateManager()
    mgr.runManager()
    
if __name__ == "__main__":
    main()
```

## robotcar.py

```python
# -*- coding: utf-8 -*- 
import smbus
import time
import threading
import RPi.GPIO as GPIO
import sys

## DRV8830 Default I2C slave address
SLAVE_ADDRESS_LEFT  = 0x64
SLAVE_ADDRESS_RIGHT  = 0x65
## PCA9685 Default I2C slave address
PCA9685_ADDRESS = 0x40

''' DRV8830 Register Addresses '''
## sample rate driver
CONTROL = 0x00

## Value of Lidar
ACQ_COMMAND = 0x00
STATUS = 0x01
ACQ_CONFIG_REG = 0x04
FULL_DELAY_HIGH = 0x0f
FULL_DELAY_LOW = 0x10

## Value motor.
FORWARD = 0x01
BACK = 0x02
STOP = 0x00

## Value of servlo
CONTROL_REG = 0x00
OSC_CLOCK = 25000000

PWM0_ON_L = 0x06
PWM0_ON_H = 0x07
PWM0_OFF_L = 0x08
PWM0_OFF_H = 0x09

ALL_PWM_ON_L = 0xFA
ALL_PWM_ON_H = 0xFB
ALL_PWM_OFF_L = 0xFC
ALL_PWM_OFF_H = 0xFD
PRE_SCALE = 0xFE

SLEEP_BIT = 0x10

#PWMを50Hzに設定
PWM_HZ = 50

## smbus
bus = smbus.SMBus(1)

class Runba(threading.Thread):
    def __init__(self, address_right=SLAVE_ADDRESS_RIGHT, address_left=SLAVE_ADDRESS_LEFT): 
        self.address_right = address_right
        self.address_left = address_left
        threading.Thread.__init__(self)
    
    def map(self, x, in_min, in_max, out_min, out_max):
        return (x - in_min) * (out_max - out_min) // (in_max - in_min) + out_min
    
    def right_forward(self, speed):
        if speed < 0:
            print "value is under 0,  must define 1-100 as speed."
            return
        elif speed > 100:
            print "value is over 100,  must define 1-100 as speed."
            return
        self.direction = FORWARD
        s = self.map(speed, 1, 100, 1, 58)
        sval = FORWARD | ((s+5)<<2) #スピードを設定して送信するデータを1Byte作成
        bus.write_i2c_block_data(self.address_right,CONTROL,[sval]) #生成したデータを送信
    
    # speedは0-100で指定
    def right_back(self, speed):
        if speed < 0:
            print "value is under 0,  must define 1-100 as speed."
            return
        elif speed > 100:
            print "value is over 100,  must define 1-100 as speed."
            return
        self.direction = BACK
        s= self.map(speed, 1, 100, 1, 58)
        sval = BACK| ((s+5)<<2) #スピードを設定して送信するデータを1Byte作成
        bus.write_i2c_block_data(self.address_right,CONTROL,[sval]) #生成したデータを送信
        
    def right_stop(self):
        bus.write_i2c_block_data(self.address_right,CONTROL,[STOP]) #モータへの電力の供給を停止(惰性で動き続ける)
        
    def left_forward(self, speed):
        if speed < 0:
            print "value is under 0,  must define 1-100 as speed."
            return
        elif speed > 100:
            print "value is over 100,  must define 1-100 as speed."
            return
        self.direction = FORWARD
        s = self.map(speed, 1, 100, 1, 58)
        sval = FORWARD | ((s+5)<<2) #スピードを設定して送信するデータを1Byte作成
        bus.write_i2c_block_data(self.address_left,CONTROL,[sval]) #生成したデータを送信
    
    # speedは0-100で指定
    def left_back(self, speed):
        if speed < 0:
            print "value is under 0,  must define 1-100 as speed."
            return
        elif speed > 100:
            print "value is over 100,  must define 1-100 as speed."
            return
        self.direction = BACK
        s= self.map(speed, 1, 100, 1, 58)
        sval = BACK| ((s+5)<<2) #スピードを設定して送信するデータを1Byte作成
        bus.write_i2c_block_data(self.address_left,CONTROL,[sval]) #生成したデータを送信
        
    def left_stop(self):
        bus.write_i2c_block_data(self.address_left,CONTROL,[STOP]) #モータへの電力の供給を停止(惰性で動き続ける)
    
    def right_brake(self):
        bus.write_i2c_block_data(self.address_right,CONTROL,[0x03]) #モータをブレーキさせる
    
    def left_brake(self):
        bus.write_i2c_block_data(self.address_left,CONTROL,[0x03]) #モータをブレーキさせる
    
    
class Robot(threading.Thread):   
    flg = True
    myServo = ""
    
    def __init__(self, address=SLAVE_ADDRESS_LEFT): 
        self.address = address
        self.c = 0
        threading.Thread.__init__(self)
        print "init"
       
        self.flg = True
    
    def run(self):
        print "run"
        self.count()
    
    def refresh(self):
        GPIO.setwarnings(False)
        GPIO.setmode(GPIO.BCM)
        GPIO.setup(HALL_SENSOR, GPIO.IN) #GPIOを入力に設定
        GPIO.setup(SERVO_PIN , GPIO.OUT) #GPIOを入力に設定
        self.myServo = GPIO.PWM(SERVO_PIN ,PWM_HZ) 
        self.myServo.start(10)
    
    def map(self, x, in_min, in_max, out_min, out_max):
        return (x - in_min) * (out_max - out_min) // (in_max - in_min) + out_min
                      
    def car_forward(self, speed):
        if speed < 0:
            print "value is under 0,  must define 1-100 as speed."
            return
        elif speed > 100:
            print "value is over 100,  must define 1-100 as speed."
            return
        self.direction = FORWARD
        s = self.map(speed, 1, 100, 1, 58)
        sval = FORWARD | ((s+5)<<2) #スピードを設定して送信するデータを1Byte作成
        bus.write_i2c_block_data(self.address,CONTROL,[sval]) #生成したデータを送信

    def car_stop(self):
        bus.write_i2c_block_data(self.address,CONTROL,[STOP]) #モータへの電力の供給を停止(惰性で動き続ける)
        
    def car_back(self, speed):
        if speed < 0:
            print "value is under 0,  must define 1-100 as speed."
            return
        elif speed > 100:
            print "value is over 100,  must define 1-100 as speed."
            return
        self.direction = BACK
        s= self.map(speed, 1, 100, 1, 58)
        sval = BACK| ((s+5)<<2) #スピードを設定して送信するデータを1Byte作成
        bus.write_i2c_block_data(self.address,CONTROL,[sval]) #生成したデータを送信

    def car_brake(self):
        bus.write_i2c_block_data(self.address,CONTROL,[0x03]) #モータをブレーキさせる
     
    def handle_init(self):
        self.set_freq(50)
    
    def handle_zero(self):
        self.set_PWM(0, 8.5)
        
    def set_freq(self, hz):
        setval=int(round(OSC_CLOCK/(4096*hz))-1)
        ctrl_dat = bus.read_word_data(PCA9685_ADDRESS,CONTROL_REG)

        #スリープにする
        bus.write_i2c_block_data(PCA9685_ADDRESS,CONTROL_REG,[ctrl_dat | SLEEP_BIT])
        time.sleep(0.01)
        #周波数を設定
        bus.write_i2c_block_data(PCA9685_ADDRESS,PRE_SCALE,[setval])
        time.sleep(0.01)
        #スリープを解除
        bus.write_i2c_block_data(PCA9685_ADDRESS,CONTROL_REG,[ctrl_dat & (~SLEEP_BIT)])
      
    def handle_move(self, direction):
        if direction < 7.5:
            return
        elif direction > 10.5:
            return
        self.set_PWM(0, direction)
        
    def set_PWM(self, pwmpin,value):
        if (value > 100):
            print "Error"
            return
        # 0~100を0~4096に変換
        setval=int(value*4096/100)
        # 最初からオン
        bus.write_i2c_block_data(PCA9685_ADDRESS,PWM0_ON_L+pwmpin*4,[0x00])
        bus.write_i2c_block_data(PCA9685_ADDRESS,PWM0_ON_H+pwmpin*4,[0x00])
        # Value％経過後にオフ
        bus.write_i2c_block_data(PCA9685_ADDRESS,PWM0_OFF_L+pwmpin*4,[setval & 0xff])
        bus.write_i2c_block_data(PCA9685_ADDRESS,PWM0_OFF_H+pwmpin*4,[setval>>8])
        
```