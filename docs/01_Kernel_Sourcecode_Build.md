# 0. 실행 환경
다음은 필자의 컴퓨터 사양 등 실행환경 정보이다.
```bash
# 2026-01-04 기준

# [Host Machine]
## 시스템 정보: `Windows 키 + I` -> `시스템` -> `정보` (CPU, RAM, OS 정보 확인)
## 또는 powershell에서 `systeminfo`입력
장치 이름	KMS_LAPTOP (삼성전자 갤럭시북2 NT750XEE-X71A)
OS Windows 11 (24H2)
프로세서	12th Gen Intel(R) Core(TM) i5-1240P(1.70 GHz)
설치된 RAM	8.00GB(7.71GB 사용 가능)
장치 ID	AF4BBFED-CB27-4A9C-BA68-262A0DC95B56
제품 ID	00342-20901-43034-AAOEM
시스템 종류	64비트 운영 체제, x64 기반 프로세서
펜 및 터치	이 디스플레이에 사용할 수 있는 펜 또는 터치식 입력이 없습니다.

# [Guest OS (WSL2)]
## $ lsb_release -a
Distro: Ubuntu 24.04.3 LTS
## $ uname -r
Current Kernel: 6.6.87.2-microsoft-standard-WSL2
compiler: gcc 13.3.0
```
<br></br>

# 1. 커널 소스코드 다운로드
우선 커널 소스코드를 다운로드 받자. 리눅스 커널은 오픈소스이기 때문에 무료로 다운로드 가능하다.
```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
```
`git.kernel.org`에서 안정화된 리눅스 커널을 다운로드 받을 수 있다 [[Here](https://www.kernel.org/)]. 직접 설치가 빠르대요.

![리눅스 커널 소스코드 다운로드](./assets/images/리눅스%20소스코드%20다운로드%20확인.png)

| 폴더/파일명 | 내용 | 폴더/파일명 | 내용 |
| :--- | :--- | :--- | :--- |
| `arch` | CPU 아키텍처 별 소스코드 | `kernel` | 커널 코어 소스코드 |
| `block` | 블록 계층 코어 소스코드 | `MAINTAINERS` | 커널 소스코드 관리자 정보 |
| `certs` | 암호화, 인증 관련 소스코드 | `Makefile` | 커널 소스 빌드 및 버전 정보 관리 |
| `COPYING` | 소스 저작권(GNU GPL) 정보 | `mm` | 메모리 관리 |
| `crypto` | 암호화 라이브러리 | `net` | 네트워크 관련 소스코드 |
| `Documentation` | 커널 문서 (Manual) | `scripts` | 사용자 스크립트 소스코드 |
| `drivers` | 디바이스 드라이버 소스코드 | `security` | 보안 모델 구현 소스코드 |
| `firmware` | 펌웨어 소스코드 | `sound` | 사운드 장치를 위한 드라이버 소스코드 |
| `fs` | 파일 시스템 관련 소스코드 | `tools` | 사용자 도구 소스코드 |
| `include` | 커널 구조체 헤더 파일 | `usr` | initramfs cpio archive 를 위한 소스코드 |
| `init` | 리눅스 초기화 및 부팅 소스코드 | `virt` | 커널 가상화(KVM) 지원 소스코드 |
| `ipc` | 프로세스간 데이터 교환 소스코드 | `Kconfig` | 커널 소스코드 설정 파일 |
| `Kbuild` | 커널 소스코드 빌드 관리 | `README` | 커널 빌드 도움말 |

분석할 내용이 한 보따리다. 생략된 내용도 있음. 나중에 하나씩 분석해보자.
<br></br>

# 2. 빌드에 필요한 패키지 설치하기
리눅스 커널 빌드를 위해선 몇가지 패키지들을 설치해야 한다. 자세한 내용은 [Here](https://www.kernel.org/doc/html/latest/process/changes.html)를 참고하십시오.
```bash
sudo apt-get install git-all fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
```
<br></br>

# 3. 최신 커널 태그 확인 및 Checkout
우선은 로컬 저장소에 최신 버전 정보가 없을 수 있으니, 원격 저장소에서 최신 태그들을 업데이트 한다.
```bash
# 2026-01-04 기준 최신 태그 정보 업데이트
git fetch --tags
```

그 다음 6.x 버전대이 최신 태그들을 확인해보자.
```bash
# 6.x 버전대 태그 목록 확인 (최신순 10개)
git tag -l 'v6.*' | sort -V | tail -n 10
```
필자는 `v6.18.2` 버전을 선택했다. 현재 **Stable**한 버전 중 가장 최신 버전이다.
![리눅스 버전 확인](./assets/images/리눅스%20버전%20확인.png)
추가로 **LTS(장기 지원)**, **Unstable**버전이 있다. LTS버전은 장기 지원 버전으로, 자료가 상대적으로 많고 안정적이다. Unstable 버전은 개발 중인 버전이다. 최첨단 기능을 볼 수 있지만 버그가 있을 수 있다. rc가 붙으면 개발중임 (예. `v6.19-rc3`)

원하는 버전을 정했다면 해당 버전으로 소스 코드를 전환한다. 
```bash
# branc 생성 및 원하는 버전으로 소스 전환
git checkout -b CV6.18.2 v6.18.2 
```

# 4. config 파일 생성하기
빌드에 앞서 빌드에 필요한 커널 설정 파일을 생성해야 한다. `make menuconfig`를 통해 생성하는 방법도 있지만, 기존의 `config`파일을 불러오는 방법을 사용할 것이다. 일반적인 Ubuntu 같은 배포판은 커널을 설치할 때 설정 파일(config)을 `/boot` 폴더에 함께 넣어주지만, WSL2는 Microsoft가 빌드한 전용 커널을 사용한다. 그리고 그 설정 파일을 `/boot`가 아닌 다른 위치에 둔다.
대부분의 WSL2 커널은 현재 실행 중인 커널 설정을 `/proc` 안에 압축해서 가지고 있다. 이를 추출해서 `.config`로 만들 수 있다.

  ```bash
  # 압축된 설정 파일을 풀어서 .config로 저장
  zcat /proc/config.gz > .config
  ``` 

  linux 소스코드와 빌드 파일이 섞이게하고 싶지 않다면, `build`디렉토리를 따로 생성하자.

  ```bash
  # build 디렉터리 이름은 알아서 정하면 됨
mkdir build-for-linux-stable
cd build-for-linux-stable
cp /boot/config-$(uname -r) .config
  ```

위와 같이 입력하여 기존 `config` 파일을 `build` 디렉토리로 복사해올 수 있다.
`.config` 파일 안에 있는 세 라인을 주석처리 해야한다. 각 라인 앞에 `#`을 붙여 주석처리하면 된다. 필자는 `vim` 에디터 사용하여 주석처리 함.
```bash
vim .config
```
다음 라인을 주석처리하지 않으면 정상적으로 빌드가 되지 않으므로 꼭 주석처리해주자.
```bash
# 빌드 시 문제가 되는 라인이다. 각 라인을 주석처리 해야한다.
CONFIG_MODULE_SIG_ALL
CONFIG_MODULE_SIG_KEY
CONFIG_SYSTEM_TRUSTED_KEYS
```

# 5. 빌드하기
지금까지 커널 리눅스 소스코드 빌드를 위한 빌드업이었다. 다시 소스코드 디렉토리 (`linux-stable`)로 들어가서 다음의 명령어를 입력하자.
```bash
# 해석: 현재 시스템의 자원을 최대한 활용하여, 소스 코드가 아닌 별도의 폴더에 결과물을 생성하라!
# -j(jobs): 동시에 실행할 작업(compile)의 개수를 지정
# $(nproc): 현재 컴퓨터의 CPU코어 전체 개수를 자동으로 반환하는 명령어 (e.g. 8코어라면 nproc는 8이 되어 `-j8`과 같은 효과)
# ==> 모든 CPU 코어 풀가동하여 병렬로 컴파일

# O=[빌드 결과물이 저장될 위치]
# 빌드 결과물이 저장된 위치를 저장하는 옵션
# Out-of-tree build: 소스 코드 디렉터리와 빌드 결과물을 분리할 수 있음. 깔끔해보이고 하나의 소스로 여러 가지 설정의 빌드를 동시에 진행할 때 유리함.
# build 디렉터리가 실제로 있어야 오류 안남!
make -j$(nproc) O=../[build 디렉토리 명]/
```

![build success](./assets/images/빌드%20success.png)

필자는 이전에 빌드를 한번 했어서 금방 끝났다. 처음에는 정체불명의 선택지가 마구마구 나올 것이다. 이를 무시하기 `Enter` 키를 꾹 눌러주자. 그럼 특별한 설정 없이 default 값으로 설정된다. 빌드하는데 꽤 시간이 걸리니 잠깐 머리도 식히고 스트레칭도 하고 오는 걸 추천한다. 필자는 대략 40분 정도 소요되었다.

빌드가 끝나면 다음과 같은 파일을 확인할 수 있다.

![빌드 결과](./assets/images/빌드%20결과.png)

다음 파일 중 `arch/x86/boot/bzImage` 는 커널 실행 이미지이다. `vmlinux` 파일은 커널이 빌드될 때의 각종 리스팅 정보와 Section 정보들을 담고 있는 `ELF` 파일이다. 이 파일은 나중에 커널 디버깅에 사용된다.