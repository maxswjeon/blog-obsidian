---
title: ST-Link와 J-Link
date: 2023-06-20 18:00:00 +0900
categories: [Development, Hardware]
tags: [st-link, j-link]
---
ST-Link V2가 도착했다. 이걸 [J-Link로 대신 쓸 수 있다](https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/)고 해서 만 오천원이나 주고 샀고, 설명서대로 드라이버 깔고, 프로그램 깔고, 툴을 돌렸더니 "Unsupported ST-LINK hardware variant"라는 오류가 났다. 알고 보니 ST-Link V2에도 종류가 있다더라. [독일 포럼 글](https://www.mikrocontroller.net/topic/450099)을 번역해서 읽어 보고서야 알았다. 64KB짜리 Flash가 들어간 칩이랑 128KB짜리 플래시 (64KB 내부 플래시 + 64KB 외부 플래시 - 아마 SPI? 인가보다)짜리 Flash가 들어간 칩으로 만들어진 V2.1이 있다고 하더라. 일단 중국에서 오는 DAPLink랑 ST-Link OB(On Board)를 테스트 해볼까 했지만, 그것도 왠지 안 될 거 같아서 그냥 J-Link v9을 $17에 샀다.  
어떻게 되려나...