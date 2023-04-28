---
title: Python與工業儀器-透過Modbus RTU RS485
date: 2023-04-28 23:20:56
tags: 
- [兼職]
- [接案]
- [python]
categories:
- [接案二三事]
---

# 緣起
最近做完一個工業類的案子，案主有一個廢水處理廠，需要隨時隨地監控這個廢水池中溫度、溶氧量、pH等數值。

# 架構
這案子要做的有硬體安裝配線與軟體開發，最後是硬體與軟體的溝通。
## 硬體部分
主要硬體有
* 溫度計
* 溶氧計
* 酸鹼值計
* 馬達
前三個硬體都支援RS485標準，沒太大問題。最後一項馬達，要檢測他處於**On**還是**Off**狀態。在淘寶淘了個電流檢測器，當電流大於某個閾值，即為**On**。
而這些硬體的安裝、配線接線等是其他人負責。我主要負責軟體。

## 軟體
Python中關於RS485的有**serial**和**modbus_tk**，直接拿來用就好，問題不大。
以下是code snippet:
```
def mod(sensor_type, id=1, dc_loc="", PORT="com3"):
    red = []
    alarm = ""
    try:
        master = modbus_rtu.RtuMaster(serial.Serial(port=PORT, baudrate=19200, bytesize=8, parity='N', stopbits=1))
        master.set_timeout(5.0)
        master.set_verbose(True)
        if sensor_type == 'temp':
            # id 03 00 8A 00 01, read PV
            pv_value = master.execute(id, cst.READ_HOLDING_REGISTERS, 138, 1)[0] / 10
            print(f"id: {id}, Current PV: {pv_value}")
            alarm = "正常"
            return pv_value
        elif sensor_type == 'ph' or sensor_type == 'do':
            # id 03 00 35 00 02, read ph
            value = ReadFloat(master.execute(id, cst.READ_HOLDING_REGISTERS, 53, 2))
            print(f"id: {id}, Current {sensor_type} value: {value}")
            alarm = "正常"
            return value

        elif sensor_type == 'motor':
            # 01 03 00 dc_loc 00 02, read ph
            value = master.execute(10, cst.READ_HOLDING_REGISTERS, int(dc_loc,16), 2)
            print(f"id: {int(dc_loc,16)}, Current {sensor_type} value: {value}")
            alarm = "正常"
            return round(value[0]/1000, 2)
        else:
            print("something error!")        
            return alarm

    except Exception as exc:
        print(str(exc))
        alarm = (str(exc))
        return red, alarm
```
基本上只要看儀器的說明書，就能知道要讀取的數值的位置。
比較特別的地方是，pH和溶氧計的返回值需要用**IEEE-754**去decode。
而馬達部分則是讀取DC value，超過一定值便判定馬達**On**

### 後端
後端採用Django，將收到的數值做一些美化和圖表。Django的主題不再本文的討論範圍，有興趣的讀者可自行查閱相關文章。

# 最終結果
## 首頁
![](https://imgur.com/P4lXABi.png)
## 圖表
![](https://imgur.com/tR33KRK.png)
