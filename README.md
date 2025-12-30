# 🐧Study-Linux-Internal
리눅스 커널의 내부 구조를 소스코드 레벨에서 분석하고, 시스템 최적화 및 성능 확장을 학습하기 위한 repository입니다.

# Introduction: 
공부의 목적 (학부연구생으로서 커널 구조 파악 및 성능 최적화 이해).

# 🧰 Analysis Environment: 
## OS & Kernel
- Host OS: Windows 11
- Guest OS: Ubuntu 22.04 LTS (on WSL2)
- Analysis kernel: Linux Kernel v6.19.0-rc2 (Mainline)
  - Note: ```Cloned with --depth 1``` for effecient storage and indexing
## IDE & Extensions
- Editor: Visual Studio Code
- Remote Development: VSCode Remote - WSL
- IntelliSense: Clangd
## Hardware Optimization (`.wslconfig`)
WSL2의 과도한 자원 점유로 인한 하드웨어 Throttling 방지 및 호스트 성능 유지를 위해 아래와 같이 자원을 제한하여 운영 중입니다.
```Ini, TOML
# %USERPROFILE%\.wslconfig
[wsl2]
memory=4GB      # 가상 머신 메모리 제한
processors=2    # 논리 프로세서 할당 제한
```

# 📂 Directory Structure
```plaintext
Study-Linux-Internal/
├── docs/           # 아키텍처 이론 및 시스템 개념 정리
├── code/           # 실습용 커널 모듈 및 분석 코드
├── daily/          # 학습 진행 상황 및 트러블슈팅 기록 (Daily Log)
└── references/     # 참고 논문 및 기술 문서 링크
```

# Curriculum: 
- Scalability: 다중 코어 환경에서의 자원 경합 및 확장성 분석.
- Scheduler: CFS(Completely Fair Scheduler)의 prio 및 vruntime 메커니즘.
- Performance: 시스템 콜 오버헤드 및 컨텍스트 스위칭 최적화.

# Topics: 
주제별 학습 내용 링크.

# Reference
https://medium.com/@mukulkathpalia/my-journey-into-linux-kernel-internals-a-beginners-roadmap-60350eccdae6
