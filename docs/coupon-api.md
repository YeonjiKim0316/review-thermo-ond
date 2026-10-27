# 쿠폰 API 약속 (화면이 이미 쓰는 주소)

> © 2026 김연지. All rights reserved. 수강생 개인 학습용 · 무단 강의·복제·배포·수정 금지 · [LICENSE.md](../LICENSE.md)

규칙(누가, 얼마, 언제 받는지)은 SPEC_COUPON.md에 직접 정합니다.

화면(`static/`)은 이미 이 약속대로 요청을 보냅니다. 약속과 다르게 만들면 화면이 깨집니다. 그래서 이 문서의 주소, 필드 이름, 상태 코드는 바꾸지 않습니다.

## 1. 엔드포인트

`POST /v1/coupons/issue`

## 2. 요청 (JSON)

| 필드 | 타입 | 규칙 |
|---|---|---|
| `review_id` | 글자 | 리뷰 번호. 앞뒤 공백을 지운 뒤 1~20자 |
| `text` | 글자 | 리뷰 글. 앞뒤 공백을 지운 뒤 1~500자. 공백만 있으면 안 됩니다 |

예:

```json
{"review_id": "r2", "text": "한쪽이 안 들려요"}
```

규칙에 맞지 않거나 필드가 빠지면 422입니다.

## 3. 응답 (JSON, 상태 코드 200)

| 필드 | 타입 | 뜻 |
|---|---|---|
| `issued` | true / false | 쿠폰을 줬는가 |
| `coupon_type` | `"APOLOGY"` / `"THANKS"` / null | 사과 쿠폰 / 감사 쿠폰 / 없음 |
| `amount` | 정수 / null | 금액(원) |
| `code` | 글자 / null | 쿠폰 코드 |

쿠폰을 안 줘도 상태 코드는 200입니다. 이때 `issued`는 false이고 나머지 세 필드는 모두 null입니다.

응답 모양 예(값은 모양을 보여 주는 예일 뿐입니다):

```json
{"issued": true, "coupon_type": "THANKS", "amount": 500, "code": "EXAMPLE-0001"}
{"issued": false, "coupon_type": null, "amount": null, "code": null}
```

## 4. 상태 코드

| 상황 | 상태 코드 |
|---|---|
| 정상 (쿠폰을 줬든 안 줬든) | 200 |
| 입력이 규칙에 맞지 않음 (빈 글, 공백만, 500자 초과, review_id 21자 이상, 필드 없음) | 422 |
| AI 부품이 응답하지 못함 | 503 |

503은 `/v1/reviews/analyze`와 같은 방식입니다. 이미 `app/main.py`에 있는 `get_score`를 쓰면 됩니다.

## 5. 연결 약속

- 긍정/부정은 요청에 들어 있지 않습니다. 서버가 AI 부품으로 직접 판단합니다.
- AI 부품은 `app/deps.py`의 `get_predict`를 FastAPI `Depends`로 받아 씁니다. 테스트가 이 자리에 가짜 AI를 끼웁니다.
- 라벨은 `/v1/reviews/analyze`와 같습니다. AI가 준 점수를 소수점 넷째 자리까지 반올림하고, `thermo.to_label`(`app/thermo.py`)로 `positive` / `negative`를 정합니다.
