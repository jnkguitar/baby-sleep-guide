# 수면의식 순서도 (Sleep Routine Flowchart) — v1.3

잠잠(JamJam)이 부모에게 안내하는 재우기 의사결정 흐름이다.
`index.html`의 상태머신(`MACHINE`)과 1:1로 대응하며, 각 분기·타이머 값은
코드의 타이밍 상수(`T`)를 그대로 따른다. 상태 단위의 정밀한 정의는
[상태머신 문서](./state-machine.md)를 참고할 것.

- 대상: 생후 4~12주
- 시간 값: 트림 10분(게워냄이 잦은 아기 20분) · 칭얼거림 무해 한도 20분 ·
  잠듦 확인 주기 10분 · 트림 재시도 5분 · 재수유 게이트(수유 후) 90분 ·
  트림 재시도 게이트(수유 후) 30분 · 진정계단 칸당 2분 · 교대 기준 누적
  강성울음 30분 · 응급 기준 연속 울음 2시간
- 전역 버튼: 활성 상태 어디서나 **[위험신호 확인]**·**[부모 한계]**·**[처음부터]**
  접근 가능 (아래 순서도에는 생략)

```mermaid
flowchart TD
    Start([시작]) --> Mode{시작 방식}
    Mode -->|처음부터 시작| Bath[준비 1 · 목욕]
    Mode -->|바로 눕히기부터| FeedWhen[마지막 수유 시각 입력]

    Bath --> Dress[준비 2 · 기저귀·수면복]
    Dress --> Move[준비 3 · 수면 공간 이동]
    Move --> Feed[준비 4 · 마지막 수유]
    Feed --> Burp[준비 5 · 세워 안고 트림<br/>10분·게워냄 20분]
    Dress -.이전 단계로.-> Bath
    Move -.이전 단계로.-> Dress
    Feed -.이전 단계로.-> Move
    Burp -.이전 단계로.-> Feed
    Burp --> Settle
    FeedWhen --> Settle[졸리지만 깨어 있는 상태로 눕히기]

    Settle --> Obs{아기 상태?}
    Obs -->|조용함| Quiet[조용함 — 지켜보기]
    Obs -->|칭얼거림| Fuss[칭얼거림 — 무개입 관찰<br/>최대 20분]
    Obs -->|강성울음| Red[위험신호 확인]
    Obs -.딸꾹질.-> Settle

    Quiet -->|잠들었음| Sleep([잠듦 · 완료])
    Quiet -->|10분 무액션| SleepCheck{아기가 잠들었나요?}
    Quiet -->|칭얼거림| Fuss
    Quiet -->|강성울음| Red
    SleepCheck -->|잠들었음| Sleep
    SleepCheck -->|잠들지 않음| Quiet

    Fuss -->|조용해짐| Quiet
    Fuss -->|20분 경과| Cause
    Fuss -->|강성울음| Red

    Red -->|위험신호 있음| Emerg([응급 · 병원])
    Red -->|해당 없음| Cause[원인 체크<br/>기저귀·온습도·게워냄·옷·기타]

    Cause -->|원인 찾음| Fix[원인 해결]
    Fix --> Settle
    Cause -->|못 찾음| BurpGate{수유 30분 이내<br/>& 트림 미시도?}

    BurpGate -->|예| BurpRetry[트림 재시도 · 5분]
    BurpRetry --> Settle
    BurpGate -->|아니오| HungerGate{마지막 수유<br/>90분 이상?}

    HungerGate -->|예| Refeed[추가 수유]
    HungerGate -->|아니오| Hunger{배고픔 신호?}
    Hunger -->|있음| Refeed
    Hunger -->|없음| Ladder

    Refeed --> RefeedBurp[추가 수유 후 트림 · 5분]
    RefeedBurp --> Settle

    Ladder[진정계단 7칸<br/>존재→목소리→쪽쪽이→터칭<br/>→눕힌 채→안아서→수유] -->|진정됨| Settle
    Ladder -.4칸 완료 · 원인 재확인.-> Cause
    Ladder -->|7칸·수유해도 진정 안 됨| LimitGate{누적 강성울음<br/>30분 이상?}
    LimitGate -->|예| Handoff[교대·휴식]
    LimitGate -->|아니오| Settle

    Handoff -->|진정됨| Settle
    Handoff -->|연속 울음 2시간| Emerg
```

## 분기 근거 요약

| 분기 | 조건 | 근거 |
| --- | --- | --- |
| 칭얼거림 → 원인 체크 | 20분 경과 | AAP(미국소아과학회): 잠들기 전 15~20분 울음 무해 |
| 트림 재시도 진입 | 마지막 수유 후 30분 이내 & 세션당 미사용 | 먹은 지 30분 안의 울음은 가스 가능성 |
| 배고픔 → 추가 수유 자동 | 마지막 수유 후 90분 이상 | 재수유 판단 기준 |
| 진정계단 → 교대 | 누적 강성울음 30분 이상 | 부모 소진 방지 |
| 교대 → 응급 | 연속 울음 2시간 | Seattle Children's(미국 시애틀 아동병원) 병원 기준 |
