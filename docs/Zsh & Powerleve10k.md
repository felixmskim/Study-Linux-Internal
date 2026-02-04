효율적인 개발과 가독성을 위해 밋밋한 Bash 쉘 대신, 커스터마이징이 가능한 Zsh와 Powerlevel10k 테마를 적용해 보자.

# 1. 개요
기본 터미널 환경은 가독성이 떨어져서 재미가 없다. Oh-my-zsh와 Powerlevel10k 테마를 사용하면 현재 디렉토리 상태, Git 브랜치 정보, OS 로고 등을 직관적인 아이콘으로 확인할 수 있다.

before. 허접 bash

![before](./assets/images/Zsh%20&%20Powerleve10k/bash.png)

after. 쌈@뽕한 Zsh

![after](./assets/images/Zsh%20&%20Powerleve10k/zsh%20설정%20완료.png)

# 2. 사전 준비: 필수 패키지 및 폰트 설치
터미널에서 특수 아이콘(집 모양, 로고 등)이 깨지지 않게 하려면 **Nerd Font** 설치가 필수적이다.

## 2.1 기본 패키지 설치
```Bash
sudo apt update
sudo apt install zsh git curl -y
```

## 2.2 폰트 설정 (Windows Terminal 기준)

1. [MesloLGS NF](https://github.com/romkatv/powerlevel10k/blob/master/README.md#manual-font-installation) 폰트 4종을 다운로드하여 윈도우에 설치한다. 다운로드 폴더 가서 우클릭하여 **[설치]** 버튼을 누른다.

![MesloLGS NF](./assets/images/Zsh%20&%20Powerleve10k/MesloLGS%20NG.png)

2. 윈도우 터미널 설정에서 `[Ubuntu 프로필] -> [모양] -> [글꼴]`을 반드시 MesloLGS NF로 변경하자.

### 만약에 목록에 안 뜬다면? (수동 입력법!)
1. 설정 화면 왼쪽 아래에 **[설정 파일 열기(Open JSON file)]** 를 누르자. 메모장이나 VS Code로 열린다

![Json 파일 열기](./assets/images/Zsh%20&%20Powerleve10k/Json파일%20설정.png)

2. `profile` -> `list`에서 **default** 또는 `profile` -> `list` 항목 안에서 **Ubuntu** 에 해당하는 부분을 찾자

3. 그 안에 `font` 설정을 다음과 같이 직접 타이핑해서 넣거나 수정하자.

![font](./assets/images/Zsh%20&%20Powerleve10k/font.png)

# 3. Oh-my-zsh 및 테마 설치

## 3.1 Oh-my-zsh 설치
아래 명령어를 통해 Oh-my-zsh를 설치하고 기본 쉘을 zsh로 변경하자.

```Bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 3.2 Powerlevel10k 테마 설치
```Bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

# 4. 상세 설정 및 가독성 최적화

## 4.1 테마 및 플러그인 활성화

`~/.zshrc` 파일을 열어 테마와 유용한 플러그인들을 등록하자. 가독성을 해치는 기본 별칭을 정리하고 `l`과 같은 단축어를 추가한다.

```Bash
# ~/.zshrc 수정
ZSH_THEME="powerlevel10k/powerlevel10k"

# 가독성을 위한 ls 별칭 설정
alias l='ls --color=auto --group-directories-first'

# 플러그인 추가 (자동 완성 및 문법 강조)
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

## 4.2 Powerlevel10k 구성 (아이콘 설정 핵심)
터미널에서 `p10k configure`를 입력하여 마법사를 실행한다. 이때 아이콘(로고, 집 모양 등)을 제대로 활성화하는 것이 핵심이다.

- **Unicode 체크:** 설정 초기 단계에서 다이아몬드(💎)나 자물쇠 모양이 제대로 보이는지 묻는 질문에 정확히 답해야 한다. **Unicode 관련 질문에 모두 Yes로 답해야만 이후 아이콘 선택지가 나타난다.**

- **Many Icons 선택:** 질문 중 아이콘 개수를 묻는 곳에서 반드시 `(2) Many icons`를 선택하자. 그래야 홈 디렉토리의 집 모양(🏠)과 OS 로고가 정상적으로 출력된다.

![example](./assets/images/Zsh%20&%20Powerleve10k/p10k%20configure%20예시.png)

## 🔗 참고 자료
- [Blog] [Oh-my-zsh와 테마(powerlevel10k) 설치하기](https://velog.io/@yuja/Oh-my-zsh%EA%B3%BC-%ED%85%8C%EB%A7%88powerlevel10k-%EB%B0%8F-%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)