---
title: test-cookbooks
parent: 06 Scripts
nav_order: 4
---

# test-cookbooks.sh — dry-run validator ของ managed-agent cookbooks ทั้งหมด

> *"Dry-run every managed-agent cookbook and assert the resolved `POST /v1/agents` bodies are well-formed."*
> — header comment ของ script

`scripts/test-cookbooks.sh` คือ **CI test harness** ที่ทำหน้าที่:

1. รัน `lint-tool-scope.py` ก่อน (security gate)
2. รัน `deploy-managed-agent.sh <slug> --dry-run` ทุก cookbook
3. ตรวจ resolved `POST /v1/agents` body ทุกอันว่า:
   - เป็น valid JSON
   - **Depth ≤ 1** (subagent ห้ามมี `callable_agents`)
   - มี non-empty `system` prompt
   - ไม่มี `output_schema` หลุดเข้า body

Exit 0 = ทุก cookbook ผ่าน, Exit 1 = มี violation

ไฟล์นี้ **สั้นที่สุดในชุด** (37 บรรทัด) แต่เป็น **gatekeeper สุดท้าย** ก่อน cookbook ออก production

## จุดประสงค์

`test-cookbooks.sh` ไม่ใช่ "unit test" ในความหมายของ pytest — แต่เป็น **shape validation** สำหรับ output ของ deploy script:

```text
agent.yaml → [deploy-managed-agent.sh --dry-run] → JSON array
                                                       │
                                                       ▼
                                            [test-cookbooks.sh]
                                                       │
                                          ┌───────────┴───────────┐
                                          │  เช็ค shape ทั้งหมด:    │
                                          │  - valid JSON          │
                                          │  - depth ≤ 1           │
                                          │  - มี system            │
                                          │  - ไม่มี output_schema  │
                                          └───────────┬───────────┘
                                                       ▼
                                              PASS/FAIL (exit code)
```

## CLI

```bash
scripts/test-cookbooks.sh
```

ไม่มี argument — script วน `managed-agent-cookbooks/*/` เองอัตโนมัติ

## Code reading

```bash
#!/usr/bin/env bash
set -euo pipefail
ROOT="$(cd "$(dirname "$0")/.." && pwd)"
fail=0

# Tool-scope lint: assert orchestrators do not carry MCP toolsets, Write,
# or Slack tools.
if ! python3 "$ROOT/scripts/lint-tool-scope.py"; then
  echo "  ✗ tool-scope lint" >&2
  fail=1
fi

for d in "$ROOT"/managed-agent-cookbooks/*/; do
  slug=$(basename "$d")
  if ! bash "$ROOT/scripts/deploy-managed-agent.sh" "$slug" --dry-run 2>&1 \
       | tail -n +2 \
       | python3 -c "
import json,sys
b=json.load(sys.stdin)
errs=[]
for i,x in enumerate(b):
    if not x.get('system'): errs.append(f'{x.get(\"name\")}: empty system')
    if i<len(b)-1 and x.get('callable_agents'):
        errs.append(f'{x.get(\"name\")}: depth>1 (subagent has callable_agents)')
if 'output_schema' in json.dumps(b):
    errs.append('output_schema leaked into a body')
if errs:
    for e in errs: print(f'      {e}', file=sys.stderr)
    sys.exit(1)
print(f'  ✓ {sys.argv[1]:24s} {len(b)} bodies')
" "$slug"; then
    echo "  ✗ $slug" >&2
    fail=1
  fi
done
exit $fail
```

หลายเรื่องที่ subtle ในนี้:

1. **`tail -n +2`** — ตัด comment บรรทัดแรกของ dry-run output (`# --dry-run: resolved POST...`) ก่อนส่งให้ Python parse JSON
2. **`i<len(b)-1`** — orchestrator (ตัวสุดท้าย) **อนุญาต** ให้มี `callable_agents`; แต่ subagent (ตัวก่อนหน้า) ห้าม
3. **`'output_schema' in json.dumps(b)`** — string-level check (ไม่ใช่ structural) ก็พอ เพราะ false positive ของ string "output_schema" ใน text จริง ๆ ไม่น่ามี
4. **`fail=1` แทนการ exit ทันที** — รันทุก cookbook ต่อจนจบ แล้ว exit ทีเดียวที่ปลาย — ช่วยให้ CI report ปัญหาของทุก cookbook ในรอบเดียว

## Algorithm — pseudocode ภาษาไทย

```text
1. รัน lint-tool-scope.py
   ถ้า fail → set fail=1 (แต่ไม่หยุด)

2. for ทุก slug ใน managed-agent-cookbooks/*/:
     dry_output = deploy-managed-agent.sh <slug> --dry-run
     bodies = json.load(dry_output ที่ตัดบรรทัดแรก)
     errs = []

     for i, body in enumerate(bodies):
         if body.system ว่าง:
             errs.append("empty system")
         if i ไม่ใช่ตัวสุดท้าย and body.callable_agents:
             errs.append("depth>1 (subagent has callable_agents)")

     if "output_schema" in json.dumps(bodies):
         errs.append("output_schema leaked")

     if errs:
         print errs → stderr
         set fail=1
     else:
         print "✓ {slug} {N} bodies"

3. exit $fail
```

## ตัวอย่างใช้งาน

### Local run

```bash
cd /path/to/claude-for-legal
scripts/test-cookbooks.sh
```

**Output (clean)**:

```
  ✓ diligence-grid           orchestrator tool scope clean
  ✓ docket-watcher           orchestrator tool scope clean
  ✓ launch-radar             orchestrator tool scope clean
  ✓ reg-monitor              orchestrator tool scope clean
  ✓ renewal-watcher          orchestrator tool scope clean
  ✓ diligence-grid           7 bodies
  ✓ docket-watcher           4 bodies
  ✓ launch-radar             5 bodies
  ✓ reg-monitor              3 bodies
  ✓ renewal-watcher          6 bodies
```

(เลข "N bodies" ดูจาก cookbook จริง — root + leaves รวมกัน)

### Output ตอน fail

สมมติเขียน subagent ที่มี `callable_agents` (depth 2):

```
      reader-leaf: depth>1 (subagent has callable_agents)
  ✗ my-bad-cookbook
```

หรือ leak output_schema:

```
      output_schema leaked into a body
  ✗ another-bad-cookbook
```

## กรณีใช้งานจริง

| สถานการณ์ | วิธีใช้ |
|---|---|
| **CI required check** ใน GitHub Actions | run บน PR ทุกตัว ก่อน merge |
| **Pre-deploy verification** ใน production pipeline | gate ก่อน real deploy |
| **Refactor cookbook** | run หลังแก้ system prompt หรือ skill paths |
| **Upgrade `deploy-managed-agent.sh`** | regression test หลังแก้ deploy script |
| **Local smoke test** | run ก่อน push เพื่อหา error เร็ว |

### ตัวอย่าง GitHub Actions integration

```yaml
- name: Test all cookbooks
  run: |
    scripts/test-cookbooks.sh
```

ไม่ต้องตั้ง `ANTHROPIC_API_KEY` เพราะใช้ `--dry-run` ทั้งหมด — เป็น **offline test**

## หลักการที่ encode ในไฟล์นี้

1. **Lint first, then test** — ถ้า security fail ก็ไม่ต้อง test shape — แต่ที่นี่เลือก run ทั้งคู่เพื่อเก็บ error ทุกชนิดในรอบเดียว
2. **Depth-1 policy enforcement** — ไม่ใช่ recommendation แต่เป็น hard fail ใน CI
3. **`output_schema` ไม่หลุด** — เพราะ Managed Agents API ปัจจุบันไม่รับ field นี้, leak = HTTP 400
4. **Non-empty system** — agent ที่ไม่มี system prompt = agent ที่ทำงานไม่ได้
5. **Aggregate errors, don't fail fast** — ดี dev experience: เห็น error ของทุก cookbook ในรอบเดียว

## ข้อสังเกต

- `python3 -c "..."` แบบ inline script เก็บ logic test ใน 10 บรรทัด — readable พอ ๆ กับเขียนไฟล์แยก, แต่ทำให้ shell script standalone ไม่ต้องพึ่งไฟล์ Python อื่น
- `set -euo pipefail` ตอนต้น = strict mode, **ยกเว้น** loop body ที่เรามี `if ! ...; then fail=1; fi` อยู่ดี
- `2>&1` ในไลน์ deploy เพราะอยากให้ stderr (จาก deploy script เอง) มาเป็น stdin ของ Python validator ด้วย — ถ้าไม่มี, error อาจหายไป

## Dependencies

- `bash`
- `python3` (มาตรฐาน, ไม่ต้อง pyyaml ในไฟล์นี้)
- `lint-tool-scope.py` + `deploy-managed-agent.sh` (จาก scripts/ เดียวกัน)

อ่านคู่กับ [deploy-managed-agent](deploy-managed-agent.html) (ตัวที่ทำ `--dry-run`) + [lint-tool-scope](lint-tool-scope.html) (ตัวที่ test-cookbooks เรียกก่อน) เพื่อเห็น chain ทั้งหมด
