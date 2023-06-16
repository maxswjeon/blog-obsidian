---
title: Visual Studio Code에서 nRF52811 개발하기
date: 2023-06-15 18:00:00 +0900
categories: [Develpment, Hardware]
tags: [nRF52811, bluetooth, ble]
---
nRF52811 기반의 개발 보드 및 모듈인 [Ebyte사의 E104-BT5011A 모듈](https://www.cdebyte.com/products/E104-BT5011A) 5개와 개발 보드 5개를 구매했다. Bluetooth 5.1 버전부터 들어간 AoA(Angle of Arrival)과 AoD(Angle of Departure)을 이용하여 더 정확한 Position Finding이 가능할까 싶고, 3개 이상의 BLE Device (Sender이든 Reciever이든)가 필요했던 기존 삼각측량 방법이 아닌 1개의 BLE Device로 해볼 수 있다는 점이 매력적이라 구매하였다.
조금 걱정되는 것은 내 핸드폰이 Note 20 Ultra로 Bluetooth 5.0을 지원하는데, AoA와 AoD는 5.1부터 지원된다는 것이다. 이론적으로는 계산하는 쪽에서만 5.1 이상을 지원하면 되는데, 그래도 불안한 건 여전하다...
# AoA와 AoD 개념 설명
## AoA (Angle of Arrival)
신호는