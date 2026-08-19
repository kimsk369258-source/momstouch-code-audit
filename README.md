# 상품코드 대조 대시보드

일별 매출 데이터를 상품코드 기준표와 자동 대조하고, 이상 유무와 이력을 저장하는 팀 내부용 대시보드입니다.

## 아키텍처

빌드 과정이나 별도 백엔드 서버 코드가 없습니다. `index.html` 안에 모든 화면 로직(JS)이 들어있고,
데이터 저장/인증만 Firebase가 대신 처리합니다.

```
팀원 브라우저
   │  1) https://<계정>.github.io/<저장소>/ 접속
   ▼
GitHub Pages  ──────────────▶  index.html + JS 를 그대로 전달 (정적 파일 호스팅만 담당)
   │
   │  2) 브라우저가 Firebase SDK로 직접 통신 (HTTPS)
   ▼
Firebase Authentication  ──▶  팀 공용 비밀번호 로그인 확인
Firebase Firestore       ──▶  기준코드표 / 일별 대조 이력 저장·조회
```

- **GitHub Pages**: 파일을 그대로 서빙만 함 (서버 코드 실행 없음)
- **Firebase Authentication**: 팀 공용 이메일/비밀번호 로그인 게이트
- **Firebase Firestore**: 실제 데이터가 저장되는 곳 (기준표 1개 문서 + 날짜별 대조결과 문서들)

## 폴더 구조

```
.
├── index.html         # 전체 애플리케이션 (GitHub Pages는 이 파일을 루트(/)에서 자동으로 서빙)
├── firestore.rules     # Firestore 보안 규칙 (Firebase 콘솔에도 동일하게 붙여넣어야 적용됨)
├── .nojekyll            # GitHub Pages가 Jekyll 처리 없이 파일을 그대로 서빙하도록 하는 빈 파일
└── README.md
```

## 1. Firebase 설정 (최초 1회)

1. https://console.firebase.google.com 에서 새 프로젝트 생성 (무료 Spark 요금제로 충분)
2. 왼쪽 메뉴 **Firestore Database** → 데이터베이스 만들기 (프로덕션 모드로 시작)
3. **Firestore → 규칙** 탭에 이 저장소의 `firestore.rules` 내용을 그대로 붙여넣고 **게시**
4. 왼쪽 메뉴 **Authentication** → 시작하기 → 로그인 방법에서 **이메일/비밀번호** 사용 설정
5. **Authentication → Users** 탭 → 사용자 추가 → 이메일(아무 형식, 예: `team@momstouch.local`)과
   비밀번호 하나를 등록 → 이게 팀 공용 로그인 비밀번호가 됩니다
6. 프로젝트 설정(⚙️) → 일반 → "내 앱" → 웹 앱 추가(`</>` 아이콘) → 나오는 `firebaseConfig` 값을 복사

## 2. index.html에 설정값 채우기

`index.html`을 열어 상단 `<script type="module">` 안의 아래 두 값을 채웁니다.

```js
const FIREBASE_CONFIG = {
  apiKey: "...",          // 여기부터
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "...",           // 여기까지, 6번에서 복사한 값 그대로
};
const TEAM_EMAIL = "team@momstouch.local"; // 5번에서 등록한 이메일
```

## 3. GitHub에 올리기

### 처음 배포하는 경우

```bash
# 1) GitHub 웹사이트에서 새 저장소 생성 (Public 또는 Private 둘 다 Pages 가능)
#    https://github.com/new

# 2) 이 폴더에서 git 초기화 후 푸시
git init
git add .
git commit -m "상품코드 대조 대시보드 초기 배포"
git branch -M main
git remote add origin https://github.com/<본인계정>/<저장소이름>.git
git push -u origin main
```

### 이후 업데이트할 때

```bash
git add .
git commit -m "수정 내용 설명"
git push
```

## 4. GitHub Pages 켜기 (최초 1회)

1. 저장소 → **Settings → Pages**
2. **Source**: `Deploy from a branch` 선택
3. **Branch**: `main` / `/ (root)` 선택 → Save
4. 1~2분 후 `https://<본인계정>.github.io/<저장소이름>/` 로 접속 가능

## 5. 확인

배포된 링크로 접속 → 팀 비밀번호로 로그인 → ① 기준코드 관리에서 `상품코드정리.xlsx` 업로드 →
② 일별 대조에서 POS 합치기 툴 결과 파일 업로드 → ③ 이력에서 저장된 기록 확인.

## 보안 관련 참고

- 팀 공용 비밀번호 1개 방식이라 "누가 언제 저장했는지" 개인별 접근 기록은 남지 않습니다.
- `firestore.rules`가 "로그인한 사람만" 읽기/쓰기를 허용하므로, 로그인 없이는 Firestore 데이터에
  직접 접근할 수 없습니다.
- `index.html`에 들어가는 `FIREBASE_CONFIG`는 브라우저에 그대로 노출되는 값입니다(정상 동작 방식).
  실제 접근 제어는 이 설정값이 아니라 Firestore 규칙(3단계)이 담당합니다.
- 저장소를 Private으로 만들어도 GitHub Pages로 배포되는 결과물(`index.html`)은 URL을 아는 누구나
  열람 가능합니다. 데이터 자체는 로그인 게이트로 보호되지만, 링크 자체를 필요한 사람에게만 공유하세요.
