# 리더보드 보안 설정 (중요)

게임 코드는 클라이언트에서 점수를 직접 Firebase에 기록합니다. 클라이언트 검증(닉네임 정리·점수 0~2000 클램프)은 추가했지만, 이것만으로는 브라우저 콘솔로 우회해 가짜 점수를 넣을 수 있습니다. **전시용 공개 랭킹이라면 아래 서버 규칙을 반드시 적용**하세요.

## 적용 방법
Firebase 콘솔 → Realtime Database → **규칙(Rules)** 탭 → 아래 내용으로 교체 → 게시.

```json
{
  "rules": {
    "scores": {
      ".read": true,
      "$id": {
        ".write": "!data.exists() && newData.exists()",
        ".validate": "newData.hasChildren(['nickname','score','grade','timestamp','date']) && newData.child('score').isNumber() && newData.child('score').val() >= 0 && newData.child('score').val() <= 2000 && newData.child('nickname').isString() && newData.child('nickname').val().length <= 8 && newData.child('grade').isString() && newData.child('grade').val().length == 1"
      }
    }
  }
}
```

## 이 규칙이 막아주는 것
- **읽기**: 누구나 가능 (랭킹 표시에 필요)
- **쓰기**: 새 기록 추가만 허용, 기존 기록 수정·삭제 차단 (`!data.exists()`)
- **점수 범위**: 0~2000 밖이면 거부 (비정상 고득점 차단)
- **닉네임**: 8자 이하 문자열만 허용
- **필수 필드**: nickname·score·grade·timestamp·date 모두 있어야 통과

## 한계와 추가 권장
- 규칙만으로는 "정상 범위 내의 가짜 점수" 반복 등록은 막지 못합니다. 전시 기간이 짧다면 충분하지만, 더 엄격히 하려면 Firebase App Check(앱 무결성 검증) 추가를 검토하세요.
- 닉네임 욕설 필터가 필요하면 클라이언트 `saveScore`의 `safeNick` 처리부에 금칙어 목록을 추가할 수 있습니다 (요청 시 작업해 드립니다).
