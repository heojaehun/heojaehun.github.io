---
title: "Fedora 40에 nimf 설치 방법"
date: 2024-09-03T05:59:17+09:00
draft: false
---

리눅스에서 한글을 입력해야 한다면 nimf를 사용하는 것이 가장 좋은 선택일 것입니다.
최근 사용하던 PC에 Fedora 40을 설치하고 nimf도 함께 과정에서 문제를 겪었습니다.
어떤 문제가 있었는지, 어떻게 해결하면 되는지 기록하였습니다.

## 어떤 문제가 있나?

- [nimf github](httpis://github.com/hamonikr/nimf?tab=readme-ov-file)에 설명된 방법을 따라 설치했다.
- nimf-settings도 문제없이 실행되고 입력기 설정도 nimf로 설정 됐다.
- 그러나 아무리해도 한글 입력이 안된다.
- 자세히 살펴보니 nimf-settings의 설정 항목 중에 이상이 있음을 알게됐다.
- Input method에는 'Dubeolsik(두벌식)'이라는 설정이 보여야하는데 아무것도 없는 상태이다.

## nimf 설치 방법

nimf 프로젝트 github에 설명된 방법되로 RPM Package를 직접 빌드한 뒤 설치하면 된다.
다만, 2024년 9월 6일자 기준으로 설명된 방법대로 하면 오류가 나타나니 아래처럼 설치하면 된다.

> 아래 설명은 [Building RPM Package](https://github.com/hamonikr/nimf/blob/master/BUILD.md#rpm-package) 페이지를 참고하며 볼 것

1. 의존선 패키지 설치
    - dnf를 이용해 빌드할 때 필요한 의존 패키지를 모두 설치한다.

```sh
    sudo dnf install anthy-devel gcc-c++ glib2-devel gtk-doc gtk2-devel \ 
         gtk3-devel intltool libappindicator-gtk3-devel libhangul-devel \
         librime-devel librsvg2-tools libtool libxkbcommon-devel \
         libxklavier-devel m17n-db-devel m17n-lib-devel qt5-qtbase-devel \
         qt5-qtbase-private-devel qt6-qtbase-devel qt6-qtbase-private-devel \
         wayland-devel naver-nanum-fonts-all wayland-protocols-devel \
         expat expat-devel libayatana-appindicator-gtk3-devel.x86_64 \
         im-chooser rpm-build rpmdevtools
```

2. RPM 패키지 빌드 준비
    - 아래에 적어놓은 순서까지만 우선 진행한다.

```sh
rpmdev-setuptree
cd ~/
wget https://github.com/hamonikr/nimf/releases/download/v1.3.8/nimf-1.3.8-1.fc40.src.rpm
rpm -ivh nimf-1.3.8-1.fc40.src.rpm
```

3. RPM 패키지 빌드 스크립트 수정
    - `rpmbuild/SPECS/nimf.spec` 파일을 열어 `git submodule update...` 라고 적힌 줄을 지우거나 주석처리 한다.

```sh
(...)

%prep
%setup -q -n nimf
autoreconf -ivf
# Clone and build libhangul
# git submodule update --init --recursive   <<< 이 줄을 주석처리
cd libhangul

(...)
```

4. RPM 패키지 빌드와 설치
    - 남은 과정을 진행한다.
    - 만약 3번 과정을 진행하느라 현재 디렉토리 경로를 변경했다면 원래 위치로 돌아간다.
    - 원래 위치는 이전 과정을 정확히 따라했다면 사용자 home 디렉토리이다. (이동 하려면 `cd ~` 입력)

```sh
rpmbuild -ba rpmbuild/SPECS/nimf.spec
sudo rpm -ivh rpmbuild/RPMS/x86_64/nimf-*.rpm
```

5. 설치 끝
    - 이제 ibus 대신 nimf를 입력기로 사용하도록 설정한다.

## 입력기 설정 방법

이 설정은 nimf를 설치하기 전에 해도되고 뒤에 해도 상관 없다.

1. Settings를 실행하고 Keyboard의 Input Sources에 Korean만 남긴다.
    - Korean(Hangul)은 ibus 입력기이므로 사용하면 안된다.

2. nimf-settings를 실행하여 한/영 전환 키를 설정이나 그 외 필요한 설정을 마무리한다.
3. 설정 끝
    - 사용자 계정을 로그아웃한 뒤 다시 로그인하면 한글 입력이 가능하다.

## 뒷 이야기

Ubuntu 22.04를 사용하던 PC를 Ubuntu 24.04.1로 업그레이드 하는 과정에 오류가 났습니다.
다행히 콘솔로 로그인은 할 수 있어서 부랴부랴 필요한 파일을 구할 수는 있었죠.
그런 다음 Ubuntu 24.04를 다시 설치하려다 이전부터 사용하고 싶었던 Fedora로 갈아타기로 했습니다.

Fedora 40을 별 문제없이 설치하고 일단 사용해보던 중 역시나 ibus 한글 입력기가 말썽이었습니다.
(Fedora는 ibus를 기본 한글 입력기로 사용합니다.)
이전부터 한글 입력기로 nimf를 사용했고 별 다른 문제가 없었기 때문에 Fedora에서도 사용하기로 했습니다.

사실 Fedora에 nimf를 설치해보는 게 처음은 아닙니다.
이전에 잠시 Fedora 38를 사용해볼 때 nimf를 설치해본 적이 있습니다.
그때는 <손에 잡히는 Vim>을 쓰신 김선영님의 [블로그](https://sunyzero.tistory.com/273)에 소개된 방법을 참고 했습니다.
nimf는 문제없이 잘 설치됐고 사용도 했었습니다만, 다른 이유로 Fedora 사용을 그만두었습니다.
이번에도 그 글을 참고해서 nimf를 설치했는데, 알 수 없는 문제가 있다는 것을 알게되었고 앞서 소개한 방법대로 문제를 해결하였습니다.
