# 기능 개발 팀 프롬프트 예제

## 기본 구성: Cross-layer 개발

```
인증 모듈 리팩토링 팀을 만들어줘:
- API 레이어 담당: src/api/ 폴더
- UI 레이어 담당: src/components/ 폴더
- 테스트 담당: tests/ 폴더
각자 plan approval 필요하고, 서로 파일 충돌 없이 진행해줘.
각 teammate에 Sonnet 모델 사용해줘.
```

### 핵심: 파일 소유권 분리

**반드시** 각 teammate가 다른 파일 세트를 소유해야 합니다. 같은 파일을 두 teammate가 수정하면 덮어쓰기가 발생합니다.

## 변형: 4명 모듈 병렬 리팩토링

```
4명의 teammate로 이 모듈들을 병렬 리팩토링해줘:
- Teammate 1: UserService
- Teammate 2: AuthService
- Teammate 3: PaymentService
- Teammate 4: NotificationService
각자 모듈만 수정하고, 공유 인터페이스 변경 시 서로 알려줘.
```

## 변형: Plan Approval + Delegate 모드

```
새 검색 기능을 개발할 팀을 만들어줘.
나는 delegate 모드로 조정만 할게.

Teammate 구성:
- 백엔드: Elasticsearch 쿼리 구현
- 프론트엔드: SearchBar UI + 결과 표시
- 인프라: Docker compose에 Elasticsearch 추가

각 teammate에 plan approval 필수.
"테스트 커버리지 포함된 계획만 승인"해줘.
```

## 팁

- Delegate 모드(Shift+Tab)로 Lead가 코드를 안 건드리게 강제
- Plan approval로 구현 전 방향 검증
- 팀원당 5-6개 Task가 적당 (Lead가 재배치 가능)
- 공유 인터페이스(API contract) 변경 시 broadcast로 전체 알림
