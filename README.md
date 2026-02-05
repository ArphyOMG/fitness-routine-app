# fitness-routine-app
운동 동작 추천 &amp; 기록 웹앱

📦 프로젝트 파일 구조 & 역할 정의 (Vanilla + Tailwind + Firebase)
0) 목표
    멀티 페이지(HTML) 기반으로 선택 → 추천 → 실행/기록 → 히스토리 흐름을 구현한다.
    초기 MVP는 룰 기반 추천 + 기록 락인에 집중한다.
    코드 구조는 **“UI / 비즈니스 로직 / DB 접근 분리”**를 강제한다.

1) 디렉터리 구조 (권장)
fitness-routine-app/
├─ public/
│  ├─ index.html
│  ├─ body-select.html
│  ├─ count-select.html
│  ├─ recommend.html
│  ├─ workout.html
│  └─ history.html
│
├─ src/
│  ├─ css/
│  │  └─ input.css
│  │
│  ├─ js/
│  │  ├─ app-config.js
│  │  ├─ firebase-init.js
│  │  ├─ auth.js
│  │  ├─ db.js
│  │  ├─ state.js
│  │  ├─ recommend.js
│  │  ├─ workout.js
│  │  ├─ history.js
│  │  └─ ui.js
│  │
│  └─ assets/
│     └─ (icons, images)
│
├─ .gitignore
├─ README.md
├─ package.json
├─ tailwind.config.js
└─ postcss.config.js

2) public/ (화면 파일) — “각 페이지는 UI 뼈대만 가진다”

    원칙: 페이지 파일에는 “DOM 컨테이너 + 스크립트 로드”만,
    로직은 src/js/에 둔다.

    1. public/index.html

            홈/진입점
            “오늘 운동 시작” CTA
            최근 기록 요약(가능하면) 또는 더미 요약

        로드 스크립트

            firebase-init.js (추후)
            auth.js (추후)
            ui.js (공통 UI)
            (선택) history.js에서 최근 기록 일부만 재사용 가능

    2. public/body-select.html

            운동 부위 선택(복수)
            “다음” 버튼

        출력

            selectedBodyParts를 state.js를 통해 저장

        로드 스크립트

            state.js
            ui.js

    3. public/count-select.html

            오늘 운동 개수 선택 (4/5/6/8)
            “다음” 버튼

        출력

            selectedCount 저장

        로드 스크립트

            state.js
            ui.js

    4. public/recommend.html

            추천 루틴 리스트 출력
            “운동 교체” 버튼(각 항목)
            “오늘 루틴 확정” 버튼

        출력

            todayRoutineDraft(운동 id 배열) 저장
            확정 시 routines 저장(파이어베이스 연결 후)

        로드 스크립트
            recommend.js
            state.js
            db.js(파이어베이스 연결 후)
            ui.js

    5. public/workout.html

            운동 실행 + 세트 기록 UI
            운동별 세트 테이블
            세트 저장(추후 Firestore)
            휴식 타이머

        출력

            세트 단위 로그 저장 → routineSets

        로드 스크립트

            workout.js
            state.js
            db.js
            ui.js

    6. public/history.html

            날짜별 루틴 카드 리스트
            클릭 시 상세 보기(옵션: 모달/별도 화면)

        데이터

            routines + routineSets 조회

        로드 스크립트

            history.js
            db.js
            ui.js

3) src/css/ — Tailwind 빌드 입력
    src/css/input.css

            Tailwind base/components/utilities 포함
            커스텀 유틸/컴포넌트 클래스 정의 가능

        빌드 결과물은 보통 public/style.css로 생성되도록 구성한다.

4) src/js/ (핵심 로직)
    1. src/js/app-config.js

            앱 전역 상수/설정
            예: 부위 목록, 운동 개수 옵션, 난이도 라벨, 타이머 기본값 등

        예시 책임

            BODY_PARTS = ["가슴","등","하체","어깨","팔","코어"]
            COUNT_OPTIONS = [4,5,6,8]

    2. src/js/firebase-init.js

            Firebase 초기화
            Firestore/Auth 인스턴스 export

        원칙

            이 파일만 Firebase SDK를 직접 다룬다.
            나머지는 db.js, auth.js가 호출한다.

    3. src/js/auth.js

            로그인/로그아웃/현재 사용자 상태
            MVP는 익명 로그인 or Google 로그인 중 택1 가능

        책임

            getCurrentUser()
            requireAuthOrRedirect()

    4. src/js/db.js ✅ (중요)

            Firestore 접근 “단일 창구”.

        원칙

            페이지/로직 파일에서 firebase/firestore를 직접 호출하지 않는다.
            DB 변경이 필요하면 이 파일만 수정한다.

        대표 함수 예시

            fetchExercisesByBodyParts(bodyParts)
            saveRoutine({ userId, date, bodyParts, exerciseIds })
            saveRoutineSet({ routineId, exerciseId, setNo, weight, reps })
            fetchRoutineHistory(userId, limit)
            fetchRoutineDetail(routineId)

    5. src/js/state.js ✅ (중요)

            페이지 간 상태 저장/복원.

        저장소

            MVP는 localStorage 기반
            나중에 필요하면 sessionStorage/Firestore로 확장

        대표 키

            selectedBodyParts
            selectedCount
            todayRoutineDraft
            todayRoutineId (확정 후)

        대표 함수

            setSelectedBodyParts(parts)
            getSelectedBodyParts()
            setSelectedCount(n)
            getSelectedCount()
            setTodayRoutineDraft(ids)
            getTodayRoutineDraft()

    6. src/js/recommend.js

            추천 로직 + 추천 화면 바인딩

        역할

            (1) exercises 로드
            (2) 룰 기반 추천 결과 생성
            (3) UI 렌더/교체 이벤트 처리
            (4) “확정” 시 routine 저장

        추천 알고리즘 책임

            generateRoutine(exercises, count, recentHistory)

    7. src/js/workout.js

            운동 실행/기록 UX

        역할

            오늘 루틴의 운동 리스트 렌더

            세트 추가/수정/저장

            휴식 타이머 시작/정지/리셋

        원칙

            “저장”은 db.js 함수 호출로만 한다.

    8. src/js/history.js

            히스토리 화면 렌더

        역할

            날짜별 루틴 리스트 조회/렌더

            루틴 클릭 시 상세(운동/세트) 보여주기

    9. src/js/ui.js

            공통 UI 유틸

        예시

            renderHeader(activeTab)
            toast(message)
            formatDate(date)
            createExerciseCard(exercise)
            createSetRow(set)

        Vanilla로 갈수록 “UI 생성 함수”가 중요해진다.
        공통 컴포넌트는 여기로 모아 중복을 줄인다.

5) 루틴 저장/조회 데이터 흐름 (파일 관점)
    
    추천 페이지에서

        state.js : 선택값(부위/개수) 읽음
        db.js : exercises 조회
        recommend.js : 추천 생성 → state.js에 draft 저장
        “확정” → db.js.saveRoutine() 호출 → routineId 저장

    운동 실행 페이지에서

        state.js : todayRoutineId / draft 읽음
        db.js : routine 상세 / sets 저장
        workout.js : 세트 입력/타이머/저장

    히스토리 페이지에서

        db.js : routines 리스트 + 상세/세트 조회
        history.js : 카드 렌더

6) 개발 운영 규칙 (협업/리뷰 기준)

    화면은 public/*.html
    로직은 src/js/*.js
    DB 접근은 무조건 db.js를 통해서만
    상태는 무조건 state.js를 통해서만

    이 규칙만 지키면 프로젝트가 커져도 “망하지” 않는다.

7) MVP 단계별 파일 생성 순서 (추천)

    1. public/index.html, public/body-select.html
    2. src/js/state.js, src/js/ui.js, src/js/app-config.js
    3. public/count-select.html, public/recommend.html + src/js/recommend.js
    4. public/workout.html + src/js/workout.js
    5. public/history.html + src/js/history.js
    6. 마지막에 Firebase (firebase-init.js, auth.js, db.js) 연결