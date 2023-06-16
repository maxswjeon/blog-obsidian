---
title: Visual Studio Code에서 nRF52811 개발하기
date: 2023-06-15 18:00:00 +0900
categories: [Develpment, Hardware]
tags: [nRF52811, bluetooth, ble]
---
nRF52811 기반의 개발 보드 및 모듈인 [Ebyte사의 E104-BT5011A 모듈](https://www.cdebyte.com/products/E104-BT5011A) 5개와 개발 보드 5개를 구매했다. Bluetooth 5.1 버전부터 들어간 AoA(Angle of Arrival)과 AoD(Angle of Departure)을 이용하여 더 정확한 Position Finding이 가능할까 싶고, 3개 이상의 BLE Device (Sender이든 Reciever이든)가 필요했던 기존 삼각측량 방법이 아닌 1개의 BLE Device로 해볼 수 있다는 점이 매력적이라 구매하였다.  
조금 걱정되는 것은 내 핸드폰이 Note 20 Ultra로 Bluetooth 5.0을 지원하는데, AoA와 AoD는 5.1부터 지원된다는 것이다. 이론적으로는 계산하는 쪽에서만 5.1 이상을 지원하면 되는데, 그래도 불안한 건 여전하다...
# AoA와 AoD
## AoA (Angle of Arrival)
![AoA Method](Pasted%20image%2020230616155915.png)  
여러 개의 안테나를 이용하여 신호가 도착하는 각도를 계산하는 방법이다.
## AoD (Angle of Departure)
![AoD Method](Pasted%20image%2020230616160405.png)  
여러 개의 안테나를 이용하여 신호가 출발하는 각도를 계산하는 방법이다.
# Ebyte E104-BT5011A
```embed
title: 'E104-BT5011A_nRF528**_BLE_Module_Chengdu Ebyte Electronic Technology Co., Ltd (1)'
image: 'https://www.cdebyte.com/products/img/3.jpg'
description: 'E104-BT5011A is a serial to BLE Bluetooth master-slave integrated module based on Bluetooth protocol version 5.1. It is small in size and low in power consumption.'
url: 'https://www.cdebyte.com/products/E104-BT5011A'
```
nRF52811이라는 
# nRF Connect SDK 설치
![](Pasted%20image%2020230616165134.png)
![](Pasted%20image%2020230616165144.png)
설치에 꽤 많은 시간이 걸린다. 30분 정도? 시간을 넉넉하게 가지고 가는 것이 좋을 듯.

**Visual Studio Code를 켜놓고 설치하면 안된다.** Path를 쓰기 때문에 설치 후에 Visual Studio Code를 한 번 껐다 켜 주어야 Toolchain을 정상적으로 인식한다. nrfjprog를 못찾는 경우도 있는데, 이런 경우 따로 설치해도 된다. (따로 설치할 필요는 없는 것 같긴 하다)
```embed
title: 'nRF Command Line Tools - Downloads'
image: 'https://www.nordicsemi.com/-/media/Images/Logos/Nordic-Semiconductor_logo.png'
description: 'Nordic Semiconductor'
url: 'https://www.nordicsemi.com/Products/Development-tools/nrf-command-line-tools/download'
```
