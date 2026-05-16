📂 mini-redis-core/ (또는 설정한 리포지토리명)
├── 1. 프로젝트 개요 (Overview)
│   ├── 1.1 미션 목적 및 고성능 인메모리 Key-Value 저장소(Redis) 아키텍처 분석
│   ├── 1.2 "데이터 오버헤드 제어 -> 전역 상태 관리 -> 런타임 클리어" 핵심 메커니즘
│   └── 1.3 차기 고급 알고리즘 및 저수준 메모리 최적화 전이를 위한 엔지니어링 배경
├── 2. 실행 환경 및 제약 조건 (Environment & Policy)
│   ├── 2.1 호스트 개발 및 실행 환경 (Python 3.8+ CLI 인터페이스 표준)
│   ├── 2.2 [엄격 제한] 파이썬 내장 dict, set, collections 라이브러리 사용 전면 차단 원칙 준수
│   └── 2.3 데이터 영속성(Persistence) 및 네트워크 레이어 배제를 통한 자료구조 독점 제어 정책
├── 3. 수행 항목 체크리스트 (Task Checklist)
│   ├── 3.1 단계별 저수준 자료구조 설계 및 REPL 인터페이스 마일스톤 (Step 1 ~ Step 5)
│   └── 3.2 작업 증적(Evidence) 및 Redis 스타일 명령어 출력 매핑 테이블
├── 4. 밑바닥부터 구현하는 3대 독립 자료구조 코어 아키텍처 (Core Data Structures)
│   ├── 4.1 **이중 연결 리스트 (Doubly Linked List) 모듈 설계**
│   │   ├── 4.1.1 prev, next, data 포인터 캡슐화 노드(Node) 객체 구조화
│   │   └── 4.1.2 O(1) 연산 보장 메서드: insert_front/back, remove_front/back, move_to_front
│   ├── 4.2 **체이닝(Chaining) 방식 해시맵 (Hash Map) 모듈 설계**
│   │   ├── 4.2.1 커스텀 소수(Prime Number) 승수 기반 독자적 해시 함수 알고리즘 설계
│   │   ├── 4.2.2 연결 리스트 재사용을 통한 충돌(Collision) 해결 체이닝 파이프라인 수립
│   │   └── 4.2.3 동적 스케일아웃: 로드 팩터(Load Factor) 0.75 초과 시 버킷 테이블 2배 확장(Rehashing)
│   └── 4.3 **우선순위 관리를 위한 최소 힙 (Min Heap) 모듈 설계**
│       ├── 4.3.1 (expire_at, key) 튜플 구조 요소의 트리 인덱스 배치 명세
│       └── 4.3.2 힙 재정렬 무결성 확보: _heapify_up 및 _heapify_down 재귀/반복 변환 제어
├── 5. String 타입 기본 명령어 및 만료 선행 검증 (String Commands Engine)
│   ├── 5.1 데이터 접근 전초기: Lazy Deletion 기반 키 만료 여부 우선 순차 검증 로직
│   ├── 5.2 **SET key value 명령어 핸들링**
│   │   └── 데이터 신규 적재, 기존 키 덮어쓰기 시 TTL 구조 초기화 및 LRU 메모리 업데이팅
│   ├── 5.3 **GET key 명령어 핸들링**
│   │   └── 만료 키 자동 파괴 및 (nil) 반환 정책, 성공 조회 건에 한정한 LRU 인덱스 타겟 갱신
│   ├── 5.4 **DEL / EXISTS / DBSIZE / KEYS 명령어 핸들링**
│   │   └── 데이터 삭제 시 3대 자료구조(해시맵, 리스트, 힙) 동시 인덱싱 파괴 무결성 테스트
│   └── 5.5 Redis 스타일 표준 출력 프로토콜 바인딩 (OK, (nil), (integer) N, (error))
├── 6. 메모리 한계 제한 설정 및 LRU 자동 제거 엔진 (Memory Guard & Eviction)
│   ├── 6.1 실시간 메모리 정밀 산정: 키/값 문자열의 UTF-8 바이트 누적 스캔 알고리즘 [used_memory]
│   ├── 6.2 CONFIG SET maxmemory 명령어를 통한 가용 상한 임계치(Bytes) 제어 정책
│   ├── 6.3 **O(1) 성능의 LRU 캐시 제거 파이프라인 (해시맵 + 이중 연결 리스트 조합 원리)**
│   │   └── used_memory 초과 발생 시 가장 오래 사용되지 않은 테일(Tail) 영역 키 순차 Eviction
│   ├── 6.4 오버플로우 방어: 단일 엔트리 사양이 maxmemory 총량을 능가할 시 OOM 에러 차단 격리
│   └── 6.5 INFO memory 명령 기반 계측 메트릭(used_memory, maxmemory, evicted_keys) 관제
├── 7. 최소 힙 기반 TTL 만료 관리 및 시간 복잡도 해석 (Time-To-Live Hardening)
│   ├── 7.1 EXPIRE key seconds 명령 기반 에포크 타임(Epoch Time) 절댓값 만료 스케줄링
│   ├── 7.2 TTL key 명령 기반 잔여 수명 수치 분석 및 예외 플래그 반환 (-1: 무제한, -2: 미존재)
│   └── 7.3 **자료구조 결합에 따른 시간 복잡도($O$) 수학적 증명 리포트**
│       ├── 7.3.1 해시맵과 이중 연결 리스트 결합을 통한 LRU 추적의 $O(1)$ 연산 원리 규명
│       └── 7.3.2 배열/리스트 대비 최소 힙(Min Heap) 구조가 TTL 만료 정렬에 최적화된 이유 ($O(\log K)$)
├── 8. 에러 처리 표준 프로토콜 및 REPL CLI 인터페이스 (Interactive REPL Engine)
│   ├── 8.1 대화형 프롬프트 인터페이스(`mini-redis>`) 및 exit/quit 인프라 스위칭 시스템
│   ├── 8.2 정규표현식/토큰 파싱 기반 공백 처리 및 큰따옴표 감싸기 문장 인수 분해 엔진
│   └── 8.3 **엄격한 4대 표준 에러 피드백 제어 시스템**
│       ├── 8.3.1 (error) ERR unknown command (정의되지 않은 명령어 유입 차단)
│       ├── 8.3.2 (error) ERR wrong number of arguments (명령어 인수 파라미터 개수 검증 실패)
│       ├── 8.3.3 (error) ERR value is not an integer (정수 변환 및 바운더리 포맷 실패)
│       └── 8.3.4 (error) OOM command not allowed (used_memory 한도 초과 차단 프로텍션)
├── 9. [보너스 영역] 알고리즘 확장 및 가상 메시징 시스템 (Advanced Data Structures)
│   ├── 9.1 가변 공간 제어: capacity 2배 확장 수립용 동적 배열(Dynamic Array) 자체 구현 및 결합
│   ├── 9.2 선형 자료구조 추상화 리포트 수립 (STACK_QUEUE_DEQUE.md 독립 자산화)
│   ├── 9.3 완전 이진 트리 기하 연산 기반 트리 전위/중위/후위/레벨 순회 알고리즘 구현
│   ├── 9.4 키 정렬 및 범위 검색 스펙 확장을 위한 이진 탐색 트리(BST) 레코드 정렬 모듈 빌드
│   └── 9.5 PUBLISH / SUBSCRIBE 명령 구현 및 연결 리스트 기반 구독자 메시지 버퍼 큐(Queue) 연동
├── 10. 트러블슈팅 리포트 (Troubleshooting)
│   ├── 10.1 [Issue #1] 해시맵 버킷 Rehashing 스케일아웃 중 이중 연결 리스트 포인터 꼬임 현상에 따른 무한 루프 에러
│   │   └── (상황 및 현상 → 원인 분석 및 가설 설정 -> 포인터 격리 디버깅 -> 최종 해결 조치 및 무결성 확보)
│   └── 10.2 [Issue #2] Lazy Deletion과 힙 팝(Pop) 타이밍 불일치로 인한 중복 만료 연산 및 used_memory 누수 오류
│       └── (상황 및 현상 → 원인 분석 및 가설 설정 -> 힙 잔여 엔트리 검증 -> 최종 해결 조치 및 방어 코드)
└── 11. 기술적 제언 및 결론 (Insights & Conclusion)
    ├── 11.1 원시 자료구조 튜닝이 소프트웨어 애플리케이션 가용성에 미치는 영향
    └── 11.2 메모리 제한 환경에서의 방어적 캐싱 정책 설계가 가지는 엔지니어링적 시사점