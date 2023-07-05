---
title: Vercel에서 canvas (Node Canvas) 사용하기
date: 2023-06-08 18:37:59 +0900
categories: [Development, Web]
tags: [canvas, node-canvas, vercel]
---
# 개요
인증서 관리 웹사이트를 만들고 있었다. 다음과 같은 기능을 지원하는 웹사이트였다.

- 유저 정보 관리
- 유저에게 인증서 발급
	- 인증서 발급 시 유저 이름, 발급 일자, 출력 일자 등이 자동으료 표기되어야 함
	- 자동으로 인증서 진위여부 인증용 QR코드를 생성해서 발급해줄 수 있어야 함

따라서 2D 이미지를 동적으로 만들어야 하는 상황이 생겼고, 이때 Canvas가 떠올라서 Node 환경에서 Canvas를 사용할 수 있는 라이브러리를 찾고 있었다.  
일단 Canvas를 직접 다루는 것은 어렵기 때문에 [Fabric.js](http://fabricjs.com/)를 사용하려고 했다. 브라우저에서는 잘 동작했고, 이를 Node에서도 동작할 수 있게 하고 싶었다.  
그러는 과정에서 [canvas (node-canvas)](https://github.com/Automattic/node-canvas)라는 라이브러리를 찾게 되었고 (정확히는 Fabric.js가 지원하는 Node 환경이 node-canvas였지만), 이를 사용하여 프로젝트를 완성한 후 Vercel에 배포하려고 했다.
# 배포 과정에서 만난 오류들
## Error: /lib64/libz.so.1: version `ZLIB_1.2.9` not found
해당 오류는 vercel의 zlib 버전과 패키지의 zlib 버전이 맞지 않아서 생기는 일이다. patchelf라는 툴로 패치해주면 된다.
```shell
yum install wget

wget https://github.com/NixOS/patchelf/archive/refs/tags/0.17.0.tar.gz
tar -xf 0.17.0.tar.gz
cd patchelf-0.17.0
./bootstrap.sh
./configure
make
make install
cd ..

wget https://zlib.net/fossils/zlib-1.2.9.tar.gz
tar -xf zlib-1.2.9.tar.gz
cd zlib-1.2.9
sh configure
make
cp libz.so.1.2.9 ../node_modules/canvas/build/Release/libz.so.X
cd..

patchelf --replace-needed /lib64/libz.so.1 libz.so.X ./node_modules/canvas/build/Release/libpng16.so.16
patchelf --replace-needed libz.so.1 libz.so.X ./node_modules/canvas/build/Release/libpng16.so.16
```
patchelf를 빌드해서 설치하고 (yum으로 설치하면 실행이 되질 않는다 - 왜 그런지는 모르겠다) zlib-1.2.9를 빌드하여 링크 해주면 된다.
## NOW_SANDBOX_WORKER_MAX_LAMBDA_SIZE: The Serverless Function is which exceeds the maximum size limit of 50mb.
이 글을 쓰게 된 가장 큰 이유다. Vercel은 AWS Lambda를 이용하는 것으로 보인다. Lambda는 Serverless Function Size에 대한 Hard Limit이 있는데, canvas의 패키지 사이즈가 너무 커서 이러한 문제가 일어난다.
```
Large Dependencies                                                               Uncompressed size  Compressed size
node_modules/.pnpm/canvas@2.11.2_encoding@0.1.13                                         179.14 MB         45.81 MB
node_modules/.pnpm/@prisma+client@4.15.0_prisma@4.15.0                                    16.44 MB          7.29 MB
node_modules/.pnpm/next@13.4.4_@babel+core@7.22.1_react-dom@18.2.0_react@18.2.0           19.73 MB          4.89 MB
.next/server/chunks                                                                       12.11 MB          1.86 MB
data/Pretendard-Regular.otf                                                                1.58 MB          1.06 MB
node_modules/.pnpm/react-dom@18.2.0_react@18.2.0                                           1.64 MB        405.36 KB
node_modules/.pnpm/caniuse-lite@1.0.30001491                                             882.21 KB         316.6 KB

All dependencies                                                                         212.94 MB         61.82 MB
```
이 문제를 해결하는 것이 가장 컸는데, 결국 커스텀 빌드를 하는 것으로 마무리 되었다.  
[Automatic/node-canvas](https://github.com/Automattic/node-canvas) 레포지토리는 신기한 방법으로 빌드되는데, prebuilds 브랜치에서 모든 빌드 스크립트를 관리하고, main 브랜치에 있는 빌드 스크립트는 안 쓰는 것이다. (되게 놀라웠다 발상의 전환...) 이때 GitHub Actions에서는 main 브랜치 내용을 불러 오면 최신 빌드가 되는 것이다. 그래서 main 브랜치의 빌드 스크립트는 현재 outdated된 상태고, prebuilds 브랜치에 있는 것이 빌드 파일들 기준으로 최신인 것이다. (정확히는 Tag 기반으로 Pull 해 오게 된다)  
prebuild/Linux/binding.gyp에서 JPEG과 SVG 지원을 끄고 관련 라이브러리 복사를 제외해 준 다음 빌드 타겟 Node 버전과 Debain stretch의 apt 리스트만 수정해 주면 빌드가 된다. 빌드 전 태그에 맞는 Release를 생성해 주어야 한다. (참고: [maxswjeon/node-canvas](https://github.com/maxswjeon/node-canvas/tree/prebuilds))  
이렇게 빌드되면 npm에 패키지를 올리고 이를 사용하면 된다. 패키지 매니저로 npm을 썼다면 빌드 스크립트만 수정해서 옵션을 따로 주면 됐을 텐데, pnpm을 쓰는 바람에 pnpm은 해당 옵션이 없어서 (build-from-source였나 아무튼) 그래서 직접 빌드해서 올리게 되었다. 지금 와서 생각해 보면 빌드 할 필요 없이 prebuild를 아예 삭제해서 올렸으면 build from source를 했지 않았을까 생각하는데...
# 회고
생각해보면 LD_LIBRARY_PATH에 /tmp를 넣어 놓고 Lazy Loading을 하는 것이 더 낫지 않았나 싶다. 생각해보고, 나중에 다시 한 번 해봐야겠다.