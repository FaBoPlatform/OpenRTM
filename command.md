# I2C関連

I2Cの認識済みデバイス一覧
> sudo i2cdetect -y 1

# パス関連

環境変数
> export RTC_MANAGER_CONFIG="/home/pi/OpenRTM/rtc.conf"

# Ominiorb

Omniorbの生存確認
> ps ax | grep omniorb

Omniorbの再起動
> sudo /etc/init.d/omniorb4-nameserver restart

