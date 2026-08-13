# 정보이론 2026 — 숙제 제출·AI 채점·통보 파이프라인 설계

## 구성 요소
| 구성 | 위치 | 공개성 |
|---|---|---|
| 강의 홈페이지 (interactive) | `lincolnkerry/Information-Theory-2026-Fall` → GitHub Pages | 공개 |
| AI TA Q&A | 같은 repo의 Discussions/Issues + `ai-ta.yml` (Claude) | 공개 |
| 숙제 제출 | GitHub Classroom — 과제마다 학생별 **private repo** (`hwN-<username>`) | 학생별 비공개 |
| 채점 엔진 | org 내 **private grader repo** (`grader/grade_all.py` + cron Actions) | 교수 전용 |
| 성적·통보 | feedback을 학생 repo에 commit + 학생 이메일(SMTP) 발송, gradebook CSV | 학생별 비공개 |

## 데이터 흐름 (HW 1건 기준)
1. 교수: Classroom에서 과제 생성(템플릿 = `hw-template` + `problems.pdf`), invite 링크를 홈페이지 `HW[]`에 게시.
2. 학생: invite 수락 → 자동 생성된 private repo에 `submission/hwN.pdf` push (마감 23:59).
3. 마감 직후 cron: grader가 org의 `hwN-*` repo 전수 수집 → Claude API에 제출물(PDF/이미지/md) + rubric + 교수 솔루션을 주고 채점 →
   `{score, per_problem[], summary, feedback_md}` JSON 수신.
4. grader가 각 학생 repo에 `feedback/hwN-feedback.md` commit, `gradebook/hwN.csv` 갱신, roster 기반으로 학생 이메일 발송.
5. 학생: repo에서 correction 확인, 이의는 7일 내 repo issue → 교수는 Actions에서 `--only <username>` 재채점.

## 핵심 설계 판단
1. **채점을 학생 repo의 Actions가 아니라 중앙 grader repo에서 실행.**
   학생 repo에서 workflow를 돌리면 학생이 workflow 파일을 수정해 secrets(API key, SMTP)를 탈취할 수 있음.
   중앙 실행이면 secrets·rubric·솔루션이 학생 손이 닿지 않는 private repo에만 존재.
2. **채점 결과의 비공개성**: 결과는 (a) 해당 학생만 보는 private repo commit, (b) 본인 이메일 — 두 경로뿐.
3. **PDF 원본 채점**: Claude API는 PDF/이미지를 직접 읽으므로 손글씨 스캔 제출도 OCR 없이 채점 가능.
4. **Hybrid 전환 여지**: `--dry-run`으로 점수·피드백을 gradebook에만 쌓고 검토 후 발송하는 운용도 가능(학기 초 권장).
5. **Argus/Hepha 연동(선택)**: grader는 단순 Python CLI이므로, GitHub Actions 대신 교수님 에이전트가
   같은 스크립트를 스케줄 실행해도 동작 동일. 엔진 교체 비용 ≈ 0.

## 비용·운영 추정
- 채점 1건(제출물 PDF ~10p + rubric + 솔루션): Claude Sonnet 기준 수십 원 수준 → 수강생 40명 × HW 8회 ≈ 커피 몇 잔.
- Q&A TA: 질문당 유사 비용. org·Classroom·Pages·Actions는 무료.

## 남는 수동 작업
- 학기 초 roster.csv 작성(학생 GitHub 계정 수집 — 첫 주 HW#0 "repo 만들기"로 겸하면 자동 수집됨).
- 주간: `CURRENT_HW` 숫자 갱신(또는 manual dispatch), 채점 결과 훑어보기.
