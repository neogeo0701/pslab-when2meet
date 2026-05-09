# PSLAB 회의 시간 조율

PSLAB 공동창업진 4명(고대현 · 김호민 · 유준환 · 전동훈)이 모일 수 있는 시간을 모으는 When2meet 스타일 단일 HTML 웹앱입니다. 별도의 빌드 단계 없이 `index.html` 하나로 동작하며, Firebase Realtime Database로 실시간 동기화됩니다.

## 🌐 배포 사이트

**[https://neogeo0701.github.io/pslab-when2meet/](https://neogeo0701.github.io/pslab-when2meet/)**

GitHub Pages로 자동 배포되며, `main` 브랜치에 push하면 1~2분 후 반영됩니다.

## 사용법

1. 첫 화면에서 본인 이름을 선택합니다.
2. 그리드(요일 × 시간, 30분 단위)에서 가능한 시간을 **드래그**로 칠합니다.
   - 이미 칠해진 셀 위에서 다시 드래그하면 **해제**됩니다.
   - PC는 마우스 드래그, 모바일은 손가락 드래그로 동작합니다.
3. 변경 사항은 자동으로 Firebase에 저장되며, 다른 멤버가 동시에 수정해도 실시간 반영됩니다.
4. **전체 보기**를 선택하면 4명의 결과가 합쳐져 색의 농도로 표시됩니다 (1명 → 옅음, 4명 → 가장 진함). 셀에 마우스를 올리거나 탭하면 누가 가능한지 이름이 표시됩니다.

## Firebase 설정

1. [Firebase 콘솔](https://console.firebase.google.com)에서 새 프로젝트를 생성합니다.
2. 좌측 메뉴의 **Build → Realtime Database**에서 데이터베이스를 만들고 위치를 선택합니다 (예: `asia-southeast1`).
3. **규칙(Rules)** 탭에서 아래와 같이 설정합니다 — 4명만 쓰는 비공개 도메인이라면 데모 규칙으로 충분합니다. 공개 배포라면 Auth 기반으로 강화하세요.
   ```json
   {
     "rules": {
       "availability": {
         ".read": true,
         ".write": true
       }
     }
   }
   ```
4. **프로젝트 설정 → 일반 → 내 앱**에서 웹 앱(`</>` 아이콘)을 추가하고, 표시되는 `firebaseConfig` 값을 복사합니다.
5. `index.html` 상단의 `firebaseConfig` 객체를 복사한 값으로 교체합니다 (`// TODO` 주석이 붙어 있는 부분).
   ```js
   const firebaseConfig = {
     apiKey: "...",
     authDomain: "...",
     databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
     projectId: "...",
     storageBucket: "...",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
   > Realtime Database를 쓰므로 **`databaseURL`이 반드시 포함**되어야 합니다. Firestore 설정과 혼동하지 않도록 주의하세요.

## 데이터 구조

```
/availability
  /고대현
    "mon-09:00": true
    "mon-09:30": true
    ...
  /김호민
    ...
  /유준환
    ...
  /전동훈
    ...
```

- 셀 키는 `{day}-{HH:MM}` 형식입니다 (`day` ∈ `mon|tue|wed|thu|fri|sat|sun`).
- 시간 범위는 `09:00`부터 `22:30`까지 30분 단위 28개 슬롯입니다.
- 가능한 셀만 `true`로 저장하고, 해제 시에는 키 자체를 삭제합니다.

## GitHub Pages 배포

1. 새 GitHub 리포지토리를 만들고 `index.html`, `README.md`를 푸시합니다.
   ```bash
   git init
   git add index.html README.md
   git commit -m "Initial PSLAB when2meet"
   git branch -M main
   git remote add origin https://github.com/<username>/<repo>.git
   git push -u origin main
   ```
2. 리포지토리 **Settings → Pages**로 이동합니다.
3. **Source**를 `Deploy from a branch`로 두고 **Branch**를 `main` → `/ (root)`로 선택한 뒤 저장합니다.
4. 1~2분 후 `https://<username>.github.io/<repo>/`에서 사이트가 열립니다. 멤버들에게 이 링크만 공유하면 됩니다.

## 보안 메모

- Firebase 웹 SDK 설정 값(`firebaseConfig`)은 클라이언트에 노출되어도 그 자체로는 위험하지 않습니다. 실제 접근 제어는 **Realtime Database 규칙**이 담당합니다.
- 위 데모 규칙은 누구나 읽고 쓸 수 있으므로, URL이 외부에 알려지면 데이터를 수정/삭제 당할 수 있습니다.
- 더 안전하게 하려면 Firebase Auth(예: Google 로그인)를 추가하고 `auth != null` 또는 화이트리스트 이메일 기반으로 `.write` 규칙을 좁히세요.

## 빌드

빌드 단계가 없습니다. `index.html` 하나로 동작합니다. Firebase SDK는 CDN(`gstatic.com`)에서 ES 모듈로 직접 로드됩니다.
