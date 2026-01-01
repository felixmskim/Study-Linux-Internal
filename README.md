# 🐧Study-Linux-Internal
리눅스 커널의 내부 구조를 소스코드 레벨에서 분석하고, 빌드 및 커스텀 커널 부팅을 통해 시스템 최적화 및 성능 확장을 학습하기 위한 repository입니다.

# Introduction: 
- **Dynnamic Analysis:** 커널 소스 수정 후 직접 빌드 및 실행을 통한 매커니즘 검증.
- **System Optimization:** 커널 파라미터 및 스케줄링 알고리즘 수정을 통한 성능 변화 분석.
- **Deep Dive:** 프로세스 관리, 메모리 계층, 스토리지 시스템의 소스코드 레벨 하부.

# 🧰 Analysis Environment: 
## OS & Kernel
- Host OS: Windows 11
- Guest OS: Ubuntu 22.04 LTS (on WSL2)
- Analysis kernel: Linux Kernel v6.18.2 (Stable)
- Builde Method: Out-of-tree build(소스와 빌드 결과물 분리)
## Hardware Optimization (`.wslconfig`)
커널 빌드는 고부하 작업이므로, 빌드 시에는 자원을 확장하여 사용하고 분석 시에는 호스트 성능을 제한하는 가변적 자원 관리를 권장합니다.
```Ini, TOML
# %USERPROFILE%\.wslconfig
[wsl2]
memory=6GB      # 가상 머신 메모리 제한
processors=6    # 논리 프로세서 할당 제한
swap=12GB
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
- [Main Guide] https://velog.io/@mythos/series/Linux-Tutorial
- [Roadmap] https://medium.com/@mukulkathpalia/my-journey-into-linux-kernel-internals-a-beginners-roadmap-60350eccdae6
- [Kernel Archive] https://www.kernel.org/
