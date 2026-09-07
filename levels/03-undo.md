# Lv.3 — 되돌리기 ✅

> Git이 무서운 이유는 "잘못 건드리면 못 돌아갈까 봐"입니다.
> 이 레벨을 끝내면 **되돌릴 수 있다는 확신**이 생깁니다.

---

## 🎯 목표

실수한 지점이 **어느 구역인지**에 따라 다른 명령을 쓴다는 것을 익힌다.

## 🧭 되돌리기 지도

```mermaid
flowchart RL
    C["🗄️ 저장소<br/>(커밋됨)"] -->|"git commit --amend<br/>방금 커밋 고치기"| C
    B["📦 스테이지<br/>(add 됨)"] -->|"git restore --staged<br/>add 취소"| A["📝 작업 디렉토리<br/>(수정만 함)"]
    A -->|"git restore<br/>수정 버리기"| Z["🗑️ 원래대로"]
```

**질문 하나로 결정됩니다: "어디까지 갔지?"**

| 실수한 지점 | 명령 | 결과 |
|:--|:--|:--|
| 파일만 수정함 | `git restore <파일>` | 수정 내용이 **사라짐** ⚠️ |
| `add`까지 함 | `git restore --staged <파일>` | 스테이지에서만 내려옴, 수정은 **남음** |
| `commit`까지 함 | `git commit --amend` | 방금 커밋을 새로 씀 |

---

## 📖 실습 시나리오

### (1) 아직 `add` 안 한 수정 버리기

```bash
# 파일을 아무렇게나 망쳐놓고 저장한 뒤
git status                 # "Changes not staged" 확인
git diff                   # 뭘 망쳤는지 확인
git restore 파일명          # 되돌리기
git status                 # 깨끗해짐
```

> ⚠️ **주의: 이건 진짜로 사라집니다.** 커밋 안 한 내용은 Git이 기억하지 못해요.
> `restore` 전에 `git diff`로 뭘 버리는지 꼭 확인하세요.

### (2) `add` 취소하기

```bash
git add 파일명
git status                       # "Changes to be committed"
git restore --staged 파일명       # 스테이지에서 내리기
git status                       # "Changes not staged" 로 돌아옴
```

**수정 내용은 그대로 남습니다.** 카메라 앞에 세웠던 사람을 다시 자리로 돌려보낸 것뿐.

### (3) 방금 한 커밋 고치기

```bash
git commit -m "오타가 잇는 메시지"
git log --oneline
git commit --amend -m "오타가 있는 메시지"
git log --oneline               # 해시가 바뀐 것에 주목!
```

> ⚠️ `--amend`는 커밋을 **수정**하는 게 아니라 **새로 만들어 갈아끼우는** 것입니다.
> 그래서 해시가 바뀝니다. **남과 공유한(push한) 커밋에는 쓰지 마세요.**

---

## 📖 `.gitignore` — 애초에 추적 안 하기

비밀번호, 로그, 빌드 결과물처럼 **버전 관리하면 안 되는 파일**을 제외합니다.

```bash
echo 비밀번호123 > secret.txt
git status                       # secret.txt 가 뜸

echo secret.txt > .gitignore
git status                       # 사라짐! 대신 .gitignore 가 뜸

git add .gitignore
git commit -m ".gitignore 추가"
```

### 자주 쓰는 패턴

```gitignore
secret.txt        # 특정 파일
*.log             # 확장자 전체
node_modules/     # 폴더 통째로
.env              # 환경변수 (API 키 등) — 가장 중요!
```

> 🔥 **실무에서 제일 중요한 파일 중 하나입니다.**
> API 키를 GitHub에 올렸다가 사고 나는 경우가 정말 많아요.
> ⚠️ 단, **이미 커밋된 파일은 `.gitignore`에 넣어도 계속 추적됩니다.**
> 그땐 `git rm --cached 파일명`으로 추적을 먼저 끊어야 합니다.

---

## ⚠️ 헷갈리기 쉬운 것

| 명령 | 하는 일 |
|:--|:--|
| `git restore 파일` | 수정 **버리기** (위험) |
| `git restore --staged 파일` | `add`만 **취소** (안전) |
| `git rm --cached 파일` | Git 추적만 끊기, 파일은 남음 |
| `git rm 파일` | 파일 **삭제** + 추적 끊기 |

---

## 🧪 실습 기록

### 되돌리기 3종 실제 결과

| 단계 | 명령 | 결과 |
|:--|:--|:--|
| 파일 파괴 | `echo 망했다 > practice.txt` | 3줄이 1줄로 덮어써짐 |
| 복구 | `git restore practice.txt` | 사과·바나나·포도 복귀 ✅ |
| `add` 취소 | `git restore --staged practice.txt` | 딸기 🍓 **살아남음** ✅ |
| 메시지 수정 | `git commit --amend -m "딸기 추가"` | `2a8bb6f` → `51608b7` **해시 교체** |

### 세 칸을 직접 들여다본 결과 (`--staged` 직후)

```
📝 작업 디렉토리 : 사과 바나나 포도 딸기 🍓   ← 딸기 있음
📦 스테이지      : 사과 바나나 포도          ← 되돌려짐
🗄️ 저장소        : 사과 바나나 포도
```

> 확인 명령: `git show :파일명` (스테이지), `git show HEAD:파일명` (저장소)

### 💡 가장 헷갈렸던 지점

**파일은 하나가 아니라 세 개의 사본으로 존재한다.**

이걸 모르면 `restore`와 `restore --staged`가 왜 다른지 절대 이해가 안 됩니다.
두 명령 다 "오른쪽 칸을 왼쪽 칸으로 복사해서 덮어쓰기"이고, **어느 칸에서 어느 칸으로**만 다릅니다.

```
git restore --staged 파일   :  저장소 ──> 스테이지      (작업 파일 안 건드림)
git restore 파일            :  스테이지 ──> 작업 디렉토리 (내 수정 증발)
```

### 부수적으로 배운 것

- `git log oneline` → `fatal: ambiguous argument` : **`--` 두 개가 "옵션"이라는 표시.** 없으면 Git이 브랜치/파일 이름으로 해석함
- `git log --oneline`의 `(origin/main)` 라벨 : **GitHub이 아는 최신 위치.** 그 위로 쌓인 커밋 = 아직 안 올린 것
- `.gitignore`로 제외한 파일이 있어도 `working tree clean`으로 나옴 : 삭제가 아니라 **Git의 시야에서만 사라진 것**

---
⬅️ 이전: [Lv.2 — 변경 추적](02-diff.md)   ·   ➡️ 다음: [Lv.4 — 브랜치](04-branch.md)
