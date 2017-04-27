# Jupyterでの実行

JupyterでOpenRTMを使用できるようにするために何箇所か修正が必要となる。

## Configファイルが上書きされる修正

JupyterとOpenRTMのConfigファイルの場所を保存している変数の名前がかぶっている。
self._configFileという名の変数が、JupyterのConfigファイルの場所で上書きされているので、臨時措置として、328-332行目をコメントアウトし、JupyterのConfigファイルで上書きされたself._configFileを参照しにいかないようにする。

`/usr/lib/python2.7/dist-packages/OpenRTM_aist/ManagerConfig.py`

```python
def findConfigFile(self):
    print "findConfigFile"
    print self._configFile
    #if self._configFile != "":
    #  if not self.fileExist(self._configFile):
    #    print OpenRTM_aist.Logger.print_exception()
    #    return False
    #  return True
    env = os.getenv(self.config_file_env)
    print env
    if env:
      if self.fileExist(env):
        self._configFile = env
        return True

    i = 0
    while (self.config_file_path[i]):
      if self.fileExist(self.config_file_path[i]):
        self._configFile = self.config_file_path[i]
        return True
      i += 1

    return False
```

OpenRTMでは、下記の優先順位で、configファイルを参照しにいくルールになっている。

* コマンドラインオプション "-f"
* 環境変数 "RTC_MANAGER_CONFIG"
* デフォルト設定ファイル "./rtc.conf"
* デフォルト設定ファイル "/etc/rtc.conf"
* デフォルト設定ファイル "/etc/rtc/rtc.conf"
* デフォルト設定ファイル "/usr/local/etc/rtc.conf"
* デフォルト設定ファイル "/usr/local/etc/rtc/rtc.conf"
* 埋め込みコンフィギュレーション値

(クラス OpenRTM_aist.ManagerConfig.ManagerConfig)[http://www.openrtm.org/OpenRTM-aist/documents/current/python/classreference_ja/class_open_r_t_m__aist_1_1_manager_config_1_1_manager_config.html]

Jupyter上では、環境変数 "RTC_MANAGER_CONFIG"を用いてConfigファイルの場所を指定するのが良いと思われる。

## 改行コードの修正

改行コードがWindows向け(\n\r)になっているので、766行目を\nのみに修正する。

`/usr/lib/python2.7/dist-packages/OpenRTM_aist/Properties.py`

```python
def load(self, inStream):
    pline = ""
    for readStr in inStream:
      print readStr
      if not readStr:
        continue
      tmp = [readStr]
      OpenRTM_aist.eraseHeadBlank(tmp)
      _str = tmp[0]
      if _str[0] == "#" or _str[0] == "!" or _str[0] == "\n":
        continue
      # _str = _str.rstrip('\n\r') 改行コードを\nだけに変更する
      _str = _str.rstrip('\n')
      if _str[len(_str)-1] == "\\" and not OpenRTM_aist.isEscaped(_str, len(_str)-1):
        tmp = [_str[0:len(_str)-1]]
        OpenRTM_aist.eraseTailBlank(tmp)
        pline += tmp[0]
        continue
      pline += _str
      if pline == "":
        continue

      key = []
      value = []
      self.splitKeyValue(pline, key, value)
      key[0] = OpenRTM_aist.unescape(key)
      OpenRTM_aist.eraseHeadBlank(key)
      OpenRTM_aist.eraseTailBlank(key)

      value[0] = OpenRTM_aist.unescape(value)
      OpenRTM_aist.eraseHeadBlank(value)
      OpenRTM_aist.eraseTailBlank(value)

      self.setProperty(key[0], value[0])
      pline = ""
```
