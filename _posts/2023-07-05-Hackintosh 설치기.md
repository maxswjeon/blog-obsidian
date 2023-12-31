---
title: Hackintosh 설치기
date: 2023-07-05 22:00:00 +0900
categories: [Development, Etc]
tags: [hackintosh, macos]
---
지금까지는 macOS에 대한 필요성을 잘 느끼지 못했다. 일단 나는 앱 개발을 직접 하는 사람도 아니었고, 그리고 실제 기기를 만지는 것 보다는 브라우저 상에서 어떻게 예쁘게, 어떻게 잘 돌아가게 만들지 고민하는 게 더 중요한 사람이었다.  
이 모든 건 엄마가 "앱 하나만 만들어 줘"라는 거에서 시작했다. (이걸 알면 엄마가 많이 후회하긴 할 것이다)  
간단한 앱이었다. 네이버 블로그씨처럼 하루에 하나씩 주제를 던져주고, 이걸 기록할 수 있게 하는 앱이었다. 다만 서버도 있어야 되고 (벡엔드가 있어야 된다는 것이었다) 그런 상황이었을 뿐이다.  
결정적으로 "웹사이트로 만들어주면 돼?"라는 대답에 "앱으로 나오면 더 좋고"라는 대답을 한 게 문제였다. React Native를 배워보고 싶은 생각은 있었지만 실제로 어떤 앱을 만들어야 할 지를 잘 몰랐고, 지금까지는 앱이 필요한 순간에 웹으로 떼울 수 있다고 생각했기 떄문이었다.  
가장 먼저 생각난 건 T3 Stack이었다. 최근에 T3 Stack, 그 중 tRPC에 대해 알게 되었고 나중에 한 번 써 보고 싶다는 생각을 해 보았다. (Backend API와 Frontend API를 잇는 것은 어려운 일이다) 찾다 보니, expo라는 Framework와 함께 T3 Stack을 이용하여 Web과 Native 환경을 둘 다 구축할 수 있게 해주는 template이 있었다.  
여기서부터 문제가 발생한다.  
기본적으로 나는 Windows 개발 환경에서 작업하던 사람이었다. 일단 내가 산 게 Windows / Linux 용으로 산 ThinkPad X4 Carbon이었고, 지금까지 Windows 노트북으로 쓰면서 매우 만족하고 있었다. 다만 Windows의 가장 맘에 안드는 것은 개발 환경이었는데, Line Break가 꼬인다던가 (그놈의 CRLF) 아니면 Permission이 꼬인다던지, 아니면 Shell이 느리다던지 등의 이유로 WSL 또는 Linux 서버를 집에 설치해 놓고 쓰고 있었다.  
근데 React Native를 개발하려고 보니 이게 안되는게 많았다. 일단 서버에서 개발을 시작한다 하면 서버에다가 내 핸드폰을 꽂아야 하는 말도 안되는 상황이 나오고, 그렇다고 Virtual Machine이 잘 돌아가는 것도 아니고 (안다 KVM이라는 게 있다는 거... 근데 그게 ADB가 안되잖아?) 그렇다고 VNC를 설치하는 짓도 하고 싶지 않았다. 애초에 서버에 X11을 까는 것은 죄다. (너무 꼰대같은 발상인가. 암튼.)  
결정적으로 iOS 빌드를 하려면 맥이 필요했다.  
그렇게, 난 해킨토시를 하기로 마음 먹었다.
# Proxmox에 macOS 깔아보기
해킨토시를 바로 시작하려고 했던 것은 아니다. Proxmox가 이미 있었기 때문에, 대충 서버 몇 대를 정리하고, 데이터를 백업하고, macOS를 깔아 보았다.  
[Nicholas Sherlock님의 Installing macOS 13 Ventura on Proxmox 7.2](https://www.nicksherlock.com/2022/10/installing-macos-13-ventura-on-proxmox/)라는 블로그 글이 있다. 되게 도움 많이 받았다. 중간에 막혔던 부분은 설명에 나온 대로 Ventura-recovery.img를 만들면 설치 중간에 에러가 나면서 설치가 중단된다. 이 문제는 놀랍게도 애플이 설치 미디어에 장난질을 쳐 놔서 중요한 파일이 설치 미디어에 포함되어 있지 않은 버전이 현재 배포되고 있다는 것이었다.  
이를 해결한 방법은 Monterey VMware Disk Image를 받아서 Ventura Full image를 만든 후 이걸로 설치하는 방법이었다. 이렇게 되면 온라인에서 다운을 받지 않고 오프라인 설치가 되기 때문에 잘 설치할 수 있었다.
# Proxmox에서 GPU Passthrough 하기
잘 설치하고, EFI 폴더를 옮기고, 사용하다보니 GUI가 너무 느렸다. 사용하기 불편할 정도였다. 당연히 GPU Passthrough를 하면 된다는 걸 알고 있었지만, 지금까지는 이게 하드웨어 가속처럼 GPU만 붙여 놓으면 외부에서도 VNC나 그런걸로 볼 때 부드럽게 넘어갈 줄 알았다. 하지만 아니었다. Passthrough는 말 그대로 PCI Device 자체를 VM에 붙여 버리는 거라, 메인 디스플레이가 GPU를 통해 나오는 것이었다(!)  
일단 가지고 있는 GPU가 없어서, RX 560 4G 모델을 당근마켓에서 5만원 정도에 구해 와서 Passthrough 시키고, 작동해 보았다. 다행이 모니터에 HDMI 단자가 하나 더 있어서, 그걸로 연결하고 보니 너무 부드럽게 잘 작동했다.  
하지만 역시 불안정한 건 마찬가지였다. 여기서 macOS를 깔아서 해킨토시를 쓰자는 결심을 하게 된다.
# 해킨토시를 만들기로 결심하다
해킨토시를 만들어 보는 건 처음이었다. macOS에 반감이 있는 건 아니었지만, 그래도 비싼 애플 하드웨어를 내 돈 주고 사기는 너무 힘들었다. 내 첫 맥북도 회사에서 나오는 온보딩 장비구매 비용으로 구입했었던 것이니...  
맥 자체의 사용성은 너무 만족했던 기억이 난다. 지금도 쓰고 싶지만 돈이 없어서 못 쓰는 거니... 만약 맥을 서버로 쓴다던지 할 수 있다면 참 좋겠다는 생각이었다.
# 해킨토시 설치 USB 만들기
해킨토시 설치를 해 본 적은 없었다. 지금까지 VMware에 깔아서 써 보기만 해 봤지, 실제로 잘 쓴 적은 없었던 것 같다. 이런 경험을 깨게 된 건 이전에 다니던 직장에서 리얼 맥인 맥북 프로를 써 봐서 그랬던 것 같다.  
사실 해킨토시에 대한 무서움이 있었다. ACPI를 건든다던지, kext를 어떻게 구해서 쓴다던지 이런 것에 대한 무서움과 "다 안될 거야"라고 생각했던 것들이었는데, 이 모든게 "나한테는 컴퓨터가 두 대 있어!"라는 걸로 모두 안심이 되었다.  
모든 과정은 [Dortania Guide](https://dortania.github.io/OpenCore-Install-Guide/)를 따라가면서 만들었다. 이게 최신인줄 알고 보다가 나중에 알고보니 최신 버전은 0.9.3이고 이건 0.8.8 기반으로 만들어져 있더라. 그래도 따라가는 데에 문제가 없었다.  
설치 USB를 만드는 자체가 되게 오래 걸렸다. Kext를 모으는데 한 시간, config.plist를 수정하는 데 한 시간, 실제로 설치하는 데 2시간 정도 걸렸던 것 같다.  
실제로 가이드를 잘 따라가면 설치 USB를 만들고 실제 설치까지 걸리는 데 2시간에서 3시간이면 충분했다. 설치도 바로 잘 됐다. 너무 좋아서, 이제부터 삽질이 시작이라는 것을 몰랐을 뿐이다.
# 해킨토시 설치 후 삽질하기
**설치 중인 하드웨어 정보**  

| **Component**     | **Model** |
| ------------- | ---------------------------------- |
| CPU.          | AMD Ryzen 5600X |
| Motherboard | ASUS B550M Pro4 |
| GPU | AMD Radeon 560 4GB |
| RAM | Samsung DDR4-3200 32GB x 4 (128GB) |
| Audio Chipset | ALC1200 |
| OS Disk | SK Hynix P31 1TB |    

해킨토시가 잘 설치되고, 다 됐다는 생각에 너무 안심했는지 모르겠지만 너무 좋았다. 하지만 여기서부터 문제가 시작되었다.
## EFI 파티션으로 OpenCore 파일을 옮기면 부팅이 안 됨
나중에 알게 된 건데, 이건 EFI 파티션 문제가 아니라 USB 문제였다.
## 재시작 하면 메인보드에 Error LED가 켜지며 부팅이 안됨
이거 때문에 되게 고생했는데, RTC 에러라고 하는 경우가 많아서 많이 삽질했다. 알고 보니 LED Controller가 USB로 붙어 있는데, 이게 하드웨어에 직접 연동이 되어 있는 거라 문제가 있는 듯 했다. USBMap을 이용해서 제외해주니 잘 되었다.
## MacPro7,1 SMBIOS를 쓰면 설치가 안됨
어떤 사람이 iGPU(Integrated GPU, 내장그래픽)이 없는데 왜 iMac18,1 SMBIOS를 쓰냐고 물어봐서 어? 해서 MacPro7,1 SMBIOS로 다시 세팅하고 포맷을 했었다. 무시했어야 했다. 이 시스템에서는 MacPro7,1 SMBIOS를 쓰면 GPU에러로 추정되는 IOG Flags 0x3 에러가 난다.
# 서버 고민
하드웨어는 언제나 고민이었다. 내가 사랑하던(?) 서버를 없애고 이걸 쓰게 됐는데, 묘하게 서버가 필요하다고 생각이 들 때가 있었다. GitLab은 포기한지 꽤 됐지만 (어쩌피 설치해놔도 잘 안 쓰게 되더라) 최근에 나온 Misskey 같은 걸 설치하고 싶어지는 마음이 드는 것도 사실이었다.  
또한 서버를 통합하게 되면서 ZFS가 들어있는 하드 디스크들을 다 옮기게 됐는데 (정확히는 서버 하드웨어에 설치한 거라 옮긴 건 아니다) 이게 NAS에 들어가 있던 때가 그립달까?  
결정적으로 원래 개발하던 습관이 데스크톱에서 우리 집에 있는 서버에 접속해서 개발하다가 나가서 노트북으로도 우리 집에 있는 서버에 접속해서 개발하는 그런 습관을 가지고 있었는데, 이제는 Git을 쓰는 방법밖에 없는 것 같아서 되게 심란했다.  
지금 글로 써 보니 원래 하고 있던 습관이 되게 구식인 방법이라는 걸 느끼고 있다(...)
# 하드웨어 고민
현재 내가 가지고 있는 하드 디스크가 5개인데, 메인보드에 SATA 포트가 6개라 6개를 쓸 수 있을 줄 알았더니 NVMe 디스크를 2개를 넣으니 마지막 2개 포트가 먹통이 되었다. 설마 하며 메뉴얼을 찾아보니 PCIe Lane을 같이 쓰고 있어 그렇댄다.  
Above 4G Decoding이 Mac에서는 필요하니까 Bifurcation이 될 거라고 생각하고 일단 x16을 x4 4개로 바꿔주는 걸 알리익스프레스에서 구매했는데, 잘 될지 모르겠다. 가장 희망적으로 보고 있는 건 Bifurcation이 되어서 NVMe 2번을 안쓰고 2개의 SATA 포트를 살리고, NVMe 4포트를 얻는 것이 가장 좋은 일 아닐까 생각하고 있다. (이러면 나중에 10Gbit 달고 싶을 때 못 달긴 하는데 그정도로 고민할 때는 내가 돈이 많아서 Mac을 사거나 컴퓨터를 맞추지 않을까 생각한다.)  
이렇게 4일간의 삽질로 이루어낸 해킨토시 삽질기를 마친다.