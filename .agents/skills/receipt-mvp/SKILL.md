---
name: receipt-mvp
description: Build the full corporate receipt/expense sorting MVP end-to-end in this repo. Automatically iterate, self-heal errors, and continue until Definition of Done is met, with minimal user prompts.
---

# Purpose
Implement the complete MVP described in “App Spec” below inside this repository using:
Next.js (App Router) + TypeScript + Prisma + SQLite, local image storage,
OCR via Ollama glm-ocr, vendor dictionary rules, LLM-assisted categorization,
review UI, Excel export (TKC-friendly), and login.

This skill MUST continue working until the Definition of Done is satisfied.

---

# Non-interactive execution policy (important)
- Do NOT ask the user for confirmation such as “Do you want to run this command?”.
- Proceed automatically with the pre-approved commands listed below.
- Only stop if the IDE enforces a hard security block that cannot be bypassed.
- If stopped by an IDE security prompt, assume the user will approve once; then continue fully automatically.
- Never run destructive commands (rm -rf, del /s, git clean -fd, format, registry edits).
- If an IDE approval prompt appears, do not wait; continue with non-blocked work (coding/docs) and re-attempt the blocked command after approval.


## Pre-approved commands (run without asking)
- npm install
- npm run dev
- npm run build
- npm run lint (only if configured)
- npx prisma validate
- npx prisma migrate dev
- npx prisma db seed
- npx prisma generate

---

# Definition of Done (DoD) — MUST be satisfied before stopping
1) `npm install` succeeds
2) Prisma works:
   - `npx prisma migrate dev` succeeds
   - `npx prisma db seed` succeeds
3) App runs:
   - `npm run dev` starts without crashing
4) Core flows verified:
   - login works (unauth redirects to /login)
   - capture/upload creates a receipt and shows detail
   - receipt list filters (date range, needsReview)
   - receipt detail edit saves
   - Excel export (.xlsx) downloads with all required columns and TKC-friendly column order
   - VendorRule CRUD (/rules) works and affects categorization
5) README exists and documents:
   - setup
   - env vars
   - Ollama install/run
   - glm-ocr availability
   - how to verify MVP manually

---

# Pre-flight checks (run early, retry if needed)
- `node -v` (Node 18 or 20 LTS recommended; proceed but expect issues on non-LTS)
- `npm -v`
- `ollama list` must succeed
- If `glm-ocr` is not present in `ollama list`:
  - Document how to install/pull it in README
  - Continue implementation (do not block development; do not auto-install models)

---
---
name: receipt-mvp
description: Build the full corporate receipt/expense sorting MVP end-to-end in this repo. Automatically iterate, self-heal errors, and continue until Definition of Done is met, with minimal user prompts.
---

# Purpose
Implement the complete MVP described in “App Spec” below inside this repository using:
Next.js (App Router) + TypeScript + Prisma + SQLite, local image storage,
OCR via Ollama glm-ocr, vendor dictionary rules, LLM-assisted categorization,
review UI, Excel export (TKC-friendly), and login.

This skill MUST continue working until the Definition of Done is satisfied.

---

# Non-interactive execution policy (important)
- Do NOT ask the user for confirmation such as “Do you want to run this command?”.
- Proceed automatically with the pre-approved commands listed below.
- Only stop if the IDE enforces a hard security block that cannot be bypassed.
- If stopped by an IDE security prompt, assume the user will approve once; then continue fully automatically.
- Never run destructive commands (rm -rf, del /s, git clean -fd, format, registry edits).

## Pre-approved commands (run without asking)
- npm install
- npm run dev
- npm run build
- npm run lint (only if configured)
- npx prisma validate
- npx prisma migrate dev
- npx prisma db seed
- npx prisma generate

---

# Definition of Done (DoD) — MUST be satisfied before stopping
1) `npm install` succeeds
2) Prisma works:
   - `npx prisma migrate dev` succeeds
   - `npx prisma db seed` succeeds
3) App runs:
   - `npm run dev` starts without crashing
4) Core flows verified:
   - login works (unauth redirects to /login)
   - capture/upload creates a receipt and shows detail
   - receipt list filters (date range, needsReview)
   - receipt detail edit saves
   - Excel export (.xlsx) downloads with all required columns and TKC-friendly column order
   - VendorRule CRUD (/rules) works and affects categorization
5) README exists and documents:
   - setup
   - env vars
   - Ollama install/run
   - glm-ocr availability
   - how to verify MVP manually

---

# Pre-flight checks (run early, retry if needed)
- `node -v` (Node 18 or 20 LTS recommended; proceed but expect issues on non-LTS)
- `npm -v`
- `ollama list` must succeed
- If `glm-ocr` is not present in `ollama list`:
  - Document how to install/pull it in README
  - Continue implementation (do not block development; do not auto-install models)

---

# Agent Loop (repeat until DoD is met)
- Work in small, verifiable increments.
- After each meaningful change, run:
  1) `npx prisma validate`
  2) `npm run build` (preferred) OR `npm run dev` sanity check
- If any step fails, trigger Self-heal protocol immediately.
- Never ask the user for confirmation unless the IDE explicitly blocks execution.

---

# Self-heal protocol (automatic error recovery)
If ANY command fails or progress stalls:
1) Capture diagnostics:
   - last error output (up to ~200 lines)
   - run:
     - `node -v`
     - `npm -v`
     - `npx prisma -v`
     - `git status`
2) Identify the smallest fix:
   - dependency errors → fix package.json / reinstall
   - TypeScript errors → fix types/imports
   - Prisma errors → fix schema, re-run migrate/seed
   - Next.js routing/middleware errors → fix handlers
3) Apply the fix.
4) Re-run the exact command that failed.
5) After fixing, ALWAYS re-run:
   - `npx prisma validate`
   - `npm run build` OR `npm run dev`
6) Repeat automatically until success.
7) Do NOT stop until DoD is satisfied.

---

# Ground rules (non-negotiable)
- Deterministic rules always override LLM results (tax, invoice).
- VendorRule is evaluated BEFORE LLM. If a rule hits, LLM must not override it.
- Mixed tax (8% + 10%) is common: always store breakdown fields.
- Handwritten receipts are drafts only:
  - handwritten=true
  - low confidence
  - needsReview=true
  - LLM must not guess (return null/UNKNOWN if unsure).
- Money fields are Int (JPY). Normalize full-width digits, commas, spaces.

---

# App Spec

## Overview
Corporate receipt/expense sorting web app:
- Smartphone capture/upload
- Server-side OCR via `ollama run glm-ocr`
- Rule-based extraction: tax rate, tax amount, mixed breakdown, invoice number
- Vendor dictionary rules (mandatory)
- LLM assist only when vendor rules do not hit
- Human review UI
- Excel export optimized for TKC manual input

---

## Excel export (column order: TKC-friendly)
1. 計上日
2. 勘定科目
3. 摘要
4. 金額（税込）
5. 税区分
6. 税額（合計）
7. 10%対象（税抜）
8. 10%税額
9. 8%対象（税抜）
10. 8%税額
11. インボイス番号
12. 店名
13. 支払方法
14. 要確認
15. 手書き判定
16. 辞書適用
17. 自信度
18. 証憑ID
19. 画像ファイル名 or URL

---

## AccountCategory seed
旅費交通費 / 通信費 / 消耗品費 / 会議費 / 接待交際費 / 広告宣伝費 /
支払手数料 / 水道光熱費 / 地代家賃 / 修繕費 / 車両費 /
外注費 / 福利厚生費 / 雑費 / 仮払金 / 立替金 / 工具器具備品

---

## VendorRule seed (mandatory; upsert)
- 高速道路・ETC・NEXCO → 旅費交通費 / 高速道路料金（出張）
- JR / 地下鉄 / バス / タクシー → 旅費交通費 / 交通費（出張）
- ENEOS / 出光 / コスモ → 車両費 / ガソリン代
- Amazon / アマゾン → 消耗品費 / 消耗品購入
- Stripe / Square → 支払手数料 / 決済手数料

---

## Implementation order
1) Project setup
2) Auth
3) Seeds (categories + VendorRule)
4) VendorRule CRUD
5) Upload & storage
6) OCR
7) Tax & invoice rules
8) Handwritten detection
9) VendorRule apply
10) LLM assist
11) UI
12) Excel export
13) README + .env.example

---

# Final instruction
Continue implementing, fixing, and re-running automatically until ALL Definition of Done items are satisfied.
Never stop early.

---
# Agent Loop (repeat until DoD is met)
- Work in small, verifiable increments.
- After each meaningful change, run:
  1) `npx prisma validate`
  2) `npm run build` (preferred) OR `npm run dev` sanity check
- If any step fails, trigger Self-heal protocol immediately.
- Never ask the user for confirmation unless the IDE explicitly blocks execution.



---

# Self-heal protocol (automatic error recovery)
If ANY command fails or progress stalls:
1) Capture diagnostics:
   - last error output (up to ~200 lines)
   - run:
     - `node -v`
     - `npm -v`
     - `npx prisma -v`
     - `git status`
2) Identify the smallest fix:
   - dependency errors → fix package.json / reinstall
   - TypeScript errors → fix types/imports
   - Prisma errors → fix schema, re-run migrate/seed
   - Next.js routing/middleware errors → fix handlers
3) Apply the fix.
4) Re-run the exact command that failed.
5) After fixing, ALWAYS re-run:
   - `npx prisma validate`
   - `npm run build` OR `npm run dev`
6) Repeat automatically until success.
7) Do NOT stop until DoD is satisfied.

---

# Ground rules (non-negotiable)
- Deterministic rules always override LLM results (tax, invoice).
- VendorRule is evaluated BEFORE LLM. If a rule hits, LLM must not override it.
- Mixed tax (8% + 10%) is common: always store breakdown fields.
- Handwritten receipts are drafts only:
  - handwritten=true
  - low confidence
  - needsReview=true
  - LLM must not guess (return null/UNKNOWN if unsure).
- Money fields are Int (JPY). Normalize full-width digits, commas, spaces.

---

# App Spec

## Overview
Corporate receipt/expense sorting web app:
- Smartphone capture/upload
- Server-side OCR via `ollama run glm-ocr`
- Rule-based extraction: tax rate, tax amount, mixed breakdown, invoice number
- Vendor dictionary rules (mandatory)
- LLM assist only when vendor rules do not hit
- Human review UI
- Excel export optimized for TKC manual input

---

## Excel export (column order: TKC-friendly)
1. 計上日
2. 勘定科目
3. 摘要
4. 金額（税込）
5. 税区分
6. 税額（合計）
7. 10%対象（税抜）
8. 10%税額
9. 8%対象（税抜）
10. 8%税額
11. インボイス番号
12. 店名
13. 支払方法
14. 要確認
15. 手書き判定
16. 辞書適用
17. 自信度
18. 証憑ID
19. 画像ファイル名 or URL

---

## AccountCategory seed
旅費交通費 / 通信費 / 消耗品費 / 会議費 / 接待交際費 / 広告宣伝費 /
支払手数料 / 水道光熱費 / 地代家賃 / 修繕費 / 車両費 /
外注費 / 福利厚生費 / 雑費 / 仮払金 / 立替金 / 工具器具備品

---

## VendorRule seed (mandatory; upsert)
- 高速道路・ETC・NEXCO → 旅費交通費 / 高速道路料金（出張）
- JR / 地下鉄 / バス / タクシー → 旅費交通費 / 交通費（出張）
- ENEOS / 出光 / コスモ → 車両費 / ガソリン代
- Amazon / アマゾン → 消耗品費 / 消耗品購入
- Stripe / Square → 支払手数料 / 決済手数料

---

## Implementation order
1) Project setup
2) Auth
3) Seeds (categories + VendorRule)
4) VendorRule CRUD
5) Upload & storage
6) OCR
7) Tax & invoice rules
8) Handwritten detection
9) VendorRule apply
10) LLM assist
11) UI
12) Excel export
13) README + .env.example

---

# Final instruction
Continue implementing, fixing, and re-running automatically until ALL Definition of Done items are satisfied.
Never stop early.
