---
title: validate
parent: 06 Scripts
nav_order: 5
---

# validate.py — JSON Schema validator สำหรับ subagent output

> *"Harness-side schema validation for managed-agent worker output."*
> — docstring บรรทัดแรก

`scripts/validate.py` คือ **structural validator** สำหรับ output ของ reader subagent — ใช้คู่กับ `output_schema:` ใน YAML manifest เพื่อให้แน่ใจว่า subagent ส่ง JSON ที่ถูกต้องตามสัญญาก่อนที่ orchestrator จะ consume

ไฟล์นี้ **สั้นที่สุดในชุด** (44 บรรทัด) แต่เป็นชั้นที่ทำให้ "structured output" ใช้งานได้จริงในขณะที่ Managed Agents API ปัจจุบันยังไม่ enforce structured output ฝั่ง server

## ทำไมต้องมี validator แยก

ตามที่ docstring อธิบาย:

> *"The CMA API does not enforce structured output today, so the deploy harness runs this between a reader subagent and the orchestrator. Schemas live in each subagent yaml under `output_schema:` — the deploy script extracts them."*

ปัจจุบัน (เมื่อ `claude-for-legal` ถูกเขียน) **Managed Agents API ยังไม่ enforce structured output ฝั่ง server** — แต่ pattern ของ orchestrator → reader leaf → orchestrator ต้องการ contract ที่เชื่อถือได้

วิธีแก้ของ Anthropic:

1. **เก็บ schema ใน `agent.yaml`** ของ reader leaf ใต้ field `output_schema:`
2. **`deploy-managed-agent.sh` ลบ `output_schema`** ก่อน POST (เพราะ API ไม่รู้จัก field)
3. **Runtime harness** (ไม่ใช่ orchestrate.py — แต่เป็น harness ฝั่ง deploy ของผู้ใช้) เรียก `validate.py` ระหว่าง subagent output → orchestrator input

ดังนั้น `validate.py` คือ **shim** ที่จำลอง "structured output enforcement" ที่ API ยังไม่มี

## CLI

```bash
validate.py <output.json> <schema.json|schema.yaml>
```

| Argument | ความหมาย |
|---|---|
| `<output.json>` | ไฟล์ JSON ที่ subagent produce (อาจเป็น `.json` หรือ `.yaml`) |
| `<schema.json\|schema.yaml>` | JSON Schema ที่ extract จาก `output_schema:` ของ subagent manifest |

| Exit | ความหมาย |
|---|---|
| `0` | Valid — print `OK` ออก stdout |
| `1` | Invalid — print `INVALID: <message> at <path>` ออก stderr |
| `2` | Argument count ผิด (ไม่ใช่ 2 args) — print docstring เป็น help |

## Code reading

```python
"""Harness-side schema validation for managed-agent worker output.

Usage: validate.py <output.json> <schema.json|schema.yaml>
Exits 0 on valid, 1 on invalid (message to stderr).
"""
import json
import sys
from pathlib import Path
import jsonschema


def _load(path: Path):
    text = path.read_text()
    if path.suffix in (".yaml", ".yml"):
        import yaml
        return yaml.safe_load(text)
    return json.loads(text)


def main() -> int:
    if len(sys.argv) != 3:
        print(__doc__, file=sys.stderr)
        return 2
    instance = _load(Path(sys.argv[1]))
    schema = _load(Path(sys.argv[2]))
    try:
        jsonschema.validate(instance=instance, schema=schema)
    except jsonschema.ValidationError as e:
        print(f"INVALID: {e.message} at {'/'.join(str(p) for p in e.absolute_path)}",
              file=sys.stderr)
        return 1
    print("OK")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

จุดสังเกตเล็ก ๆ ที่ดี:

1. **รองรับทั้ง JSON และ YAML** — schema เก็บใน YAML manifest, output ของ subagent มักเป็น JSON
2. **Late-binding `import yaml`** — ใน `_load()` import yaml เฉพาะตอนต้องใช้ — ลด cold start ถ้า input เป็น `.json` อยู่แล้ว
3. **Error message รวม path** — `at finding/3/severity` ช่วย debug schema mismatch ได้ทันที — ไม่ใช่ generic error
4. **`pathlib.Path` แทน `os.path`** — modern Python convention

## Algorithm — pseudocode ภาษาไทย

```text
function main():
    if len(argv) != 3:
        print docstring → stderr
        return 2

    instance = _load(argv[1])    # output.json จาก subagent
    schema   = _load(argv[2])    # schema.yaml จาก output_schema:

    try:
        jsonschema.validate(instance, schema)
    except ValidationError as e:
        print "INVALID: {e.message} at {/path/to/field}" → stderr
        return 1

    print "OK"
    return 0

function _load(path):
    text = read path
    if path เป็น .yaml/.yml:
        return yaml.safe_load(text)
    else:
        return json.loads(text)
```

## ตัวอย่างใช้งาน

### Validate output ของ reader subagent

สมมุติว่ามี `output_schema` ใน `reg-monitor/reader.yaml`:

```yaml
output_schema:
  type: object
  required: ["findings", "summary"]
  properties:
    findings:
      type: array
      items:
        type: object
        required: ["rule_id", "severity"]
        properties:
          rule_id: {type: string}
          severity: {enum: ["high", "medium", "low"]}
    summary: {type: string}
```

deploy harness extract schema นี้ไปเป็น `./out/reader-schema.yaml` แล้ว reader subagent run จน produce `./out/reader-output.json`:

```json
{
  "findings": [
    {"rule_id": "SEC-17a-4", "severity": "high"}
  ],
  "summary": "1 high-severity rule diff detected."
}
```

จากนั้น harness call:

```bash
python3 scripts/validate.py ./out/reader-output.json ./out/reader-schema.yaml
```

ถ้า valid → `OK` + exit 0 → orchestrator consume ต่อได้

### Output ตอน invalid

สมมติ reader produce:

```json
{
  "findings": [
    {"rule_id": "SEC-17a-4", "severity": "critical"}
  ]
}
```

(เพราะ "critical" ไม่อยู่ใน enum + ขาด `summary`)

```bash
python3 scripts/validate.py output.json schema.yaml
# stderr:
# INVALID: 'critical' is not one of ['high', 'medium', 'low'] at findings/0/severity
# exit 1
```

หรือ:

```
INVALID: 'summary' is a required property at
```

(path เป็น empty string ถ้า error ที่ root)

### Usage ใน shell pipeline

```bash
if ! python3 scripts/validate.py ./out/reader-output.json ./out/reader-schema.yaml; then
  echo "reader subagent produced malformed output — abort handoff" >&2
  exit 1
fi
# ถ้ามาถึงตรงนี้ = valid, ส่งต่อให้ orchestrator
```

## กรณีใช้งานจริง

| สถานการณ์ | วิธีใช้ |
|---|---|
| **Runtime gate** ระหว่าง subagent → orchestrator | call ระหว่างทุก handoff ใน workflow engine |
| **Unit test** ของ subagent | run บน fixture output + expected schema |
| **Regression test** หลังแก้ subagent system prompt | ensure output shape ไม่เปลี่ยน |
| **Debug** schema mismatch | run interactively บน production output |
| **CI pre-deploy** | combine กับ `test-cookbooks.sh` + sample inputs |

## ตำแหน่งใน pipeline

```text
Reader Subagent
       │
       │ produces output.json
       ▼
┌──────────────┐
│ validate.py  │  ← shim ที่จำลอง "structured output enforcement"
└──────┬───────┘
       │ OK
       ▼
Orchestrator (consume valid output)
       │
       │ emit handoff_request → ส่งต่อให้ subagent อื่น
       ▼
   (orchestrate.py 5-layer gate)
```

## หลักการที่ encode ในไฟล์นี้

1. **Schema-first contract between agents** — ไม่พึ่ง free text หรือ "ก็เชื่อเขาเถอะ"
2. **Same validator for runtime + test** — `jsonschema.validate()` ใช้ตัวเดียวกัน ลดความเสี่ยงของ schema drift
3. **Error message มี path** — debug ได้เร็ว
4. **Stdout/stderr separation** — `OK` ออก stdout, error ออก stderr → pipe-friendly
5. **Exit code 0/1/2** — Unix convention: 0=ok, 1=app error, 2=usage error

## Dependencies

- `python3`
- `jsonschema`
- `pyyaml` (เฉพาะตอน load `.yaml` schema)

## ข้อสังเกตจากการอ่าน source

- ไฟล์ใช้ `from __future__ import` **ไม่ใช้** — เพราะ target Python 3.10+ และไม่ใช้ type hint แบบใหม่
- ไม่มี logging — script ตั้งใจให้ "เงียบเมื่อสำเร็จ" (Unix philosophy: silence is golden)
- ไม่ accept stdin — design เลือกใช้ file path argument เพื่อให้ debug ง่าย (re-run บน fixture เดิมได้)

อ่านคู่กับ [orchestrate](orchestrate.html) (เห็น `jsonschema.validate()` pattern เดียวกันใช้ใน 5 ชั้น handoff gate) — `validate.py` คือเวอร์ชั่น standalone ของ logic เดียวกันที่ใช้ใน harness ฝั่ง deploy
