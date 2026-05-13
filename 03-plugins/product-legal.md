---
title: product-legal
parent: 03 Plugins
nav_order: 3
---

# product-legal — Product Counsel

> "Reviews product launches against your risk calibration, answers 'is this a problem?' questions in minutes, checks marketing copy for claims that need substantiation, and flags upcoming launches that need legal eyes before anyone asks."
> — `product-legal/.claude-plugin/plugin.json`

## 1. บทนำ

`product-legal` เป็น plugin สำหรับ **product counsel** — ทนายในบริษัท tech / consumer product ที่ต้องดู feature launch, marketing copy, dark pattern risk, claim substantiation ก่อนที่สินค้าจะออกตลาด

ลักษณะงานของ product counsel:

- **launch review** ก่อน feature ใหม่ปล่อย — ดูทั้ง legal risk, marketing claim, regulatory issue
- **"is this a problem?"** — คำถามรวดเร็วจาก PM ใน Slack ที่ต้องการคำตอบในนาที ไม่ใช่ memo 5 หน้า
- **marketing claims review** — โฆษณา/landing page/email มี claim ไหนต้องการ substantiation
- **launch radar** — เฝ้า launch tracker (Jira/Linear) ว่ามีอะไรกำลังจะออกแล้วอาจต้องการ legal eyes

plugin นี้พึ่งพา **risk calibration** ที่เรียนรู้จาก past review ของลูกค้าใน cold-start interview — ทุกครั้งที่ออก verdict จะ pattern-match กับ calibration นั้น

**เหมาะกับ**: in-house product counsel, marketing counsel, consumer product legal, growth team legal

## 2. `.claude-plugin/plugin.json`

```json
{
  "name": "product-legal",
  "version": "1.0.2",
  "description": "Reviews product launches against your risk calibration, answers 'is this a problem?' questions in minutes, checks marketing copy for claims that need substantiation, and flags upcoming launches that need legal eyes before anyone asks.",
  "author": {
    "name": "Anthropic"
  }
}
```

| ฟิลด์ | ค่า | ความหมาย |
|------|----|---------|
| `name` | `product-legal` | namespace ของคำสั่ง (เช่น `/product-legal:launch-review`) |
| `version` | `1.0.2` | semver |
| `description` | (ยาว) | คำอธิบายแสดงใน marketplace |
| `author.name` | `Anthropic` | upstream maintainer |

## 3. โครงสร้างโฟลเดอร์

```text
product-legal/
├── .claude-plugin/
│   └── plugin.json
├── .gitignore
├── .mcp.json                    # Slack, Drive, Linear, Atlassian (Jira/Confluence), Asana
├── CLAUDE.md                    # practice profile template (38.6 KB)
├── README.md                    # (5.7 KB)
├── agents/
│   └── launch-watcher.md        # scheduled — monitor launch tracker
├── hooks/
│   └── hooks.json               # {} placeholder
├── references/
│   └── currency-watch.md        # staleness check สำหรับ product/consumer law (2.8 KB)
└── skills/                      # 7 skills
    ├── cold-start-interview/
    ├── customize/
    ├── feature-risk-assessment/
    ├── is-this-a-problem/
    ├── launch-review/
    ├── marketing-claims-review/
    └── matter-workspace/
```

## 4. Skills ทั้งหมด

7 skill — ทุกตัวเรียกได้โดยตรง

| Skill | Path | คำสั่ง | สำหรับ |
|-------|------|--------|---------|
| `launch-review` | `skills/launch-review/` | `/product-legal:launch-review` | full launch review ตาม framework + risk calibration — ออกเป็น memo จัดหมวด |
| `is-this-a-problem` | `skills/is-this-a-problem/` | `/product-legal:is-this-a-problem` | fast answer — "fine / needs a look / hold" สำหรับ Slack-question ที่ต้องการคำตอบเดี๋ยวนั้น |
| `marketing-claims-review` | `skills/marketing-claims-review/` | `/product-legal:marketing-claims-review` | review copy — claim ไหนต้อง substantiation, ไหน puffery, ไหน reframe |
| `feature-risk-assessment` | `skills/feature-risk-assessment/` | `/product-legal:feature-risk-assessment` | deep risk assessment เมื่อ launch-review เจอ issue ที่ต้อง analyze ลึกกว่า line item |
| `cold-start-interview` | `skills/cold-start-interview/` | `/product-legal:cold-start-interview` | สัมภาษณ์ครั้งแรก — เรียน launch tracker, past review, risk calibration |
| `customize` | `skills/customize/` | `/product-legal:customize` | ปรับ profile — risk calibration, escalation contact, launch framework, marketing posture |
| `matter-workspace` | `skills/matter-workspace/` | `/product-legal:matter-workspace` | จัดการ matter workspace สำหรับ multi-client / outside counsel |

**ลำดับ skill ตาม timeline ของ launch**:
1. (ก่อน launch ปลาย funnel) `launch-watcher` agent flag launch
2. (review) `launch-review` ทำ full review
3. (เจอ issue ลึก) `feature-risk-assessment` deep dive
4. (marketing material) `marketing-claims-review`
5. (Slack question รายวัน) `is-this-a-problem`

## 5. Agents

1 agent

| Agent | Description | Trigger |
|-------|-------------|---------|
| `launch-watcher` | monitor launch tracker (Jira/Linear) — flag upcoming launch ที่น่าจะต้องการ legal review ก่อนที่ product counsel จะถูกถาม | daily, หรือสั่ง "what launches are coming", "what should I know about", "launch radar" |

Tools ที่ระบุใน frontmatter: `mcp__jira__*`, `mcp__linear__*`, `mcp__*__slack_send_message` — agent ทำงานข้าม launch tracker หลายระบบ และ post alert ลง Slack

แนวคิด: **shift-left** — แทนที่จะรอ PM มาถาม ทนาย product ได้ list ก่อนล่วงหน้า

## 6. Hooks

```text
product-legal/hooks/hooks.json
```

เนื้อหา: `{ "hooks": {} }` — placeholder เปล่า เหมือน plugin อื่นใน Group 1

## 7. References

**`references/currency-watch.md`** — staleness check สำหรับ product/consumer protection law

ประเด็นที่ครอบคลุม:

- **Children's online safety**: COPPA 2025 (deadline เม.ย. 2026), UK AADC, California AADC (เนื่องจาก *NetChoice v. Bonta* ต้อง verify), KOSA
- **Platform & intermediary liability**: DSA enforcement (€120M X fine ธ.ค. 2025, TikTok/adult platform preliminary finding เม.ย. 2026, Snapchat investigation มี.ค. 2026), Section 230 (SCOTUS cases pending)
- **FTC §5 & dark patterns**: Humor Rainbow/OkCupid (มี.ค. 2026 — undisclosed AI training data sharing เป็น §5 violation), "click to cancel" rule, endorsement guides (updated 2023), **AMG Capital (2021)** — §13(b) ไม่ให้ monetary relief ต้องใช้ §19
- **AI transparency & synthetic content**: EU AI Act transparency (deadline ธ.ค. 2026 หลัง Digital Omnibus), NE LB 525, CA AB 2013 (training data disclosure)

หลักการ: "**stale watch list ดูเหมือนทันสมัยทั้งที่ผิด**" — ทุก skill เมื่ออ้าง rule ต้องบอก "verify at [source]"

## 8. Connectors (MCP)

จาก `.mcp.json` — **5 servers** (เน้น project tracker):

| MCP server | URL | สำหรับ |
|-----------|-----|--------|
| Slack | `mcp.slack.com/mcp` | search message — "is this a problem?" มักมาจาก Slack |
| Google Drive | `drivemcp.googleapis.com/mcp/v1` | search/read/fetch — PRD, design doc, marketing copy |
| Linear | `mcp.linear.app/mcp` | issue tracking — สำหรับ tech-forward company |
| Atlassian | `mcp.atlassian.com/v1/sse` | Jira issue + Confluence page |
| Asana | `mcp.asana.com/sse` | task + project tracking |

`recommendedCategories`: `project-management`, `issue-tracking`, `documents`, `chat`, `email`, `outside-counsel-network`

**สังเกต**: product-legal มี **3 project tracker** (Linear + Jira + Asana) — เพราะ product team แต่ละองค์กรใช้คนละตัว plugin ต้องครอบคลุมทั้งหมด

## 9. Workflow ตัวอย่าง

### Case 1 — "Quick Slack question" (ที่พบบ่อยที่สุด)

1. PM ทักทนาย product ใน Slack: "เราอยากทำ promo 'first month free' แต่ user ต้อง add credit card — ok ไหม?"
2. ทนาย copy คำถามเข้า Claude: `/product-legal:is-this-a-problem "first month free promo with required credit card"`
3. skill pattern-match กับ risk calibration → verdict (ยาว 2–3 บรรทัด):
   - **needs a look** — "click-to-cancel" rule กำลังจะมีผล อาจมี state law ระบุ negative option ต้องการ explicit consent + ง่ายในการ cancel
   - **next step**: ดู cancellation flow + disclosure language ของ promo
4. ทนายตอบ PM ใน Slack ภายในนาที — ถ้า "needs a look" ก็นัด deep dive

### Case 2 — Full launch review ก่อน feature ปล่อย

1. PM share PRD link (Confluence) สำหรับ feature "AI-generated profile photo"
2. ทนาย: `/product-legal:launch-review "https://confluence/.../ai-profile-photo"`
3. skill อ่าน PRD ผ่าน Atlassian MCP → review ตาม category (privacy, IP, marketing claim, dark pattern, AI transparency, ...) → memo:
   - **Privacy**: training data sourcing? — flag for `privacy-legal:pia-generation` handoff
   - **IP**: ใช้ third-party model? — license review
   - **AI transparency**: EU AI Act compliance (synthetic content disclosure) — verify at `currency-watch.md`
   - **Marketing**: ห้ามใช้คำว่า "photorealistic" without disclaimer
4. memo + risk tag ส่งให้ PM + escalate ตาม matrix ใน `CLAUDE.md`
5. ถ้าเจอ issue ใหญ่: เปิด `/product-legal:feature-risk-assessment` deep dive

### Case 3 — Launch radar (proactive)

1. `launch-watcher` agent (daily) crawl Jira → เห็น ticket "AI Customer Service Bot — Launch Q3" ที่ไม่ได้ flag legal review
2. agent post Slack #product-legal: "launch ที่อาจต้อง legal review: AI Customer Service Bot (target Q3) — keyword 'AI' + 'customer-facing' ตรง pattern"
3. ทนายเปิด ticket → ทำ `/product-legal:launch-review` proactive

## 10. ข้อควรระวัง

- **"is this a problem?" output ไม่ใช่ legal opinion** — เป็น pattern-match กับ risk calibration ของลูกค้าเอง ถ้า calibration ผิดหรือ outdated verdict ก็ผิดตาม ทนายต้อง verify เสมอ
- **Risk calibration ต้อง update ตามเวลา** — ใช้ `customize` ปรับเมื่อ company posture เปลี่ยน หรือเมื่อมี regulatory shift
- **Launch radar ไม่ใช่ "ครอบคลุมทุก launch"** — agent flag ตาม keyword pattern เท่านั้น ถ้า PM ตั้งชื่อ ticket ที่ไม่ตรง pattern (เช่น project code name) จะ slip past
- **Marketing claims review ต้อง verify substantiation** — skill บอกว่า claim ไหนต้อง substantiation แต่ไม่ verify ว่า substantiation ที่มีอยู่ดีพอ — ทนายต้องดูเอง
- **`currency-watch.md` stale ได้** — DSA enforcement, FTC posture, AI Act เปลี่ยนแทบทุกไตรมาส verify ทุกครั้ง
- **ทุก output ต้องผ่าน attorney review** — ตามข้อ disclaimer ใน `README.md` ของ source

> "Pattern matches against your calibration" — ประโยคจาก description ของ `is-this-a-problem` — ย้ำว่า plugin ทำงานบน calibration ของลูกค้า ไม่ใช่ legal absolute

---

**Source**: [`anthropics/claude-for-legal/product-legal/`](https://github.com/anthropics/claude-for-legal/tree/main/product-legal)
**Version**: 1.0.2
**Last reviewed**: 2026-05-13
