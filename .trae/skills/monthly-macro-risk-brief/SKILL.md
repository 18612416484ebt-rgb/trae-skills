---
name: "monthly-macro-risk-brief"
description: "Generates the monthly macro and investment environment analysis for Zhongyou Capital's risk brief. Invoke when user needs to write the monthly risk report, asks to prepare macro analysis for a new month, or uploads DM reference texts for the report. Automatically fetches data from iFinD, cross-verifies facts, and outputs a DOCX matching the established template style."
---

# Monthly Macro Risk Brief Generator

## Overview

This skill generates the "宏观与投资环境分析" section for 中油资本月度风险简报. It enforces data accuracy through iFinD cross-verification and absolute style consistency with the established template.

## Trigger Conditions

Invoke this skill when:
- User asks to write the monthly risk brief for a new month
- User mentions preparing the macro/investment environment analysis
- User uploads DM reference texts for the monthly report
- User says "做这个月的风控简报" or similar

## Input Requirements

**Required from user:**
- Target month and year (e.g., "2026年7月")
- Whether to include the company portfolio risk section (用户可能说"这段不用输出")

**Optional from user:**
- DM research report texts (华创固收/华创宏观/华泰宏观等)
- Specific data points they have already collected

## Workflow

### Step 1: Data Collection via iFinD MCP

Use the following MCP tools to fetch monthly data. Always call `get_edb_data` for macro indicators and `index_data`/`bond_market_data`/`stock_highfreq_quotes` for market data.

**Required quantitative data points:**

| Category | Data Points | MCP Tool |
|----------|-------------|----------|
| **PMI** | Manufacturing PMI, Non-manufacturing PMI, new orders, new export orders, production, raw materials price, output price | `get_edb_data` |
| **Money Market** | DR001, DR007 monthly avg, MLF operation amount & maturity, reverse repo net injection, treasury bond trading net injection | `get_edb_data` |
| **Interest Rate Bonds** | 2Y/5Y/10Y/30Y treasury yields (month-end vs prior month-end, daily average range), 30Y-10Y spread | `get_edb_data` (EDB) |
| **Credit Bonds** | Credit spreads (AAA), rating spreads, issuance volume trends | `bond_market_data` + web search |
| **Equity (A-share)** | Shanghai Composite, Shenzhen Component, ChiNext, STAR 50 (monthly change %, closing price, monthly high/low) | `index_data` |
| **Equity (HK)** | HSI, Hang Seng Tech (monthly change %, closing price, monthly high/low) | `index_data` |
| **HK Flows** | Southbound capital monthly net inflow (南向资金), sector allocation changes | `stock_highfreq_quotes` or web search |

**Cross-verification rules:**
- Verify index closing prices and monthly changes with iFinD
- Verify treasury yield bp changes with EDB data (extract both month-end values)
- Flag any data point differing from public sources by >0.5bp or >0.1%
- For HSI, always double-check monthly decline magnitude (was -9.14% in June, not -1.75%)

### Step 2: DM Research Integration (Optional)

**If user provides DM reports:**
1. Extract qualitative strategy views for each section
2. Integrate into corresponding outlook paragraphs
3. Add to references list as items 12-14

**Key DM value-add areas:**
- Credit bond strategy (理财规模, 利差中枢, 分期限策略)
- Macro forward-looking judgments (GDP, PPI top判断)
- PMI interpretation frameworks

**If no DM input:**
- Generate using iFinD data + public sources only
- Credit bond outlook will be data-driven but less strategy-forward
- Still acceptable for base-level brief

### Step 3: Document Structure (CRITICAL)

**The output MUST follow this exact paragraph structure:**

```
[Paragraph 1: PMI macro - NO TITLE, direct start with date]

[Paragraph 2: 资金市场方面，description of monthly funding market]
[Paragraph 3: 预计[Next month] outlook for funding market]

[Paragraph 4: 利率债市场呈现出 description of bond market]
[Paragraph 5: 展望后市，bond market outlook]

[Paragraph 6: 信用债方面，description of credit bond market]
[Paragraph 7: 后市展望，credit bond outlook]

[Paragraph 8: 权益市场方面，description of equity market]
[Paragraph 9: 展望后市，equity market outlook]

[Paragraph 10: Company portfolio risk - if user requests]
```

**Structure rules (non-negotiable):**
- NO numbered section titles (一、二、三、四、五)
- NO standalone outlook sections
- "xxx方面" prefixes must be preserved exactly as inline text
- Outlook paragraphs are standalone, immediately following their description paragraph
- Company portfolio section is optional per user instruction

### Step 4: Style Rules

1. **Objective summary only** - No specific investment guidance or recommendations
2. **No subjective judgment words** like "建议买入", "看好", "推荐", "强烈关注"
3. **Data citation mandatory** - Every statistic must have source URL in references
4. **Tone** - Factual, neutral, institutional
5. **Length** - Moderate adjustment allowed, max 30% increase over template length
6. **Date references** - Use "上月" to refer to prior month, "较5月底" for yield comparisons

### Step 5: DOCX Generation

1. Generate DOCX using docx-js with CJK font support
   - `font: { ascii: "Arial", hAnsi: "Arial", eastAsia: "Microsoft YaHei" }`
2. Use exactly ONE section to avoid blank pages
3. Page size: A4 (11906 x 16838 DXA)
4. Margins: 1440 DXA (1 inch)
5. Body text: size 22, first-line indent 480 DXA
6. After generating, run sanitize.py:
   ```bash
   python "c:\Users\余楠\.trae-cn\builtin\work\hebe\skills\docx\scripts\sanitize.py" "<output>.docx"
   ```
7. Header: "中油资本月度风险简报" (right-aligned, gray #888888, size 18)
8. Footer: page numbers centered with em-dashes

### Step 6: Quality Checklist Before Delivery

- [ ] All data points verified with iFinD
- [ ] No encoding typos (科划50→科创50, 搞置→搁置, 辅5月底→较5月底)
- [ ] No Chinese quotes inside JS strings (use \u201c / \u201d)
- [ ] Structure matches template exactly (no numbered titles)
- [ ] "xxx方面" prefixes present in correct paragraphs
- [ ] Company section only included if explicitly requested
- [ ] References section includes all 14 standard sources (or subset if DM not provided)
- [ ] DOCX sanitized successfully

## Standard Reference Sources

Always include these references at end of document:

1. 国家统计局：[Month]月中国采购经理指数运行情况
2. 券商 PMI 点评研报 (e.g., 光大证券/华泰证券)
3. 普兰金服/公开资金市场总结文章
4. 中国人民银行公开市场业务交易公告
5. 中债信息网/中国财经报：中债收益率曲线日评
6. 同花顺 iFinD-EDB：国债到期收益率曲线
7. 中诚信国际：信用利差周报
8. 同花顺 iFinD：A股及港股指数行情
9. 财联社：港股市场总结
10. 新浪财经：银行间本币市场运行情况
11. 财联社/头条财经：央行流动性投放数据
12. 华创证券固定收益团队：[Period]信用债市场展望 (if DM provided)
13. 华创证券宏观经济研究团队：[Period]经济展望 (if DM provided)
14. 华泰证券宏观研究团队：[Month]PMI数据点评 (if DM provided)

## Known Pitfalls (Critical History)

1. **Never add numbered titles** - Original template has no 一/二/三/四/五 sections
2. **Preserve "xxx方面" prefixes** - These are inline text, NOT section headers
   - Correct: "资金市场方面，2026年6月资金面呈现..."
   - Wrong: "二、资金市场" then "资金市场方面：..."
3. **HSI monthly decline** - Always verify with iFinD (June was -9.14%, not -1.75%)
4. **Treasury yield bp changes** - Verify exact values from iFinD EDB, round to nearest bp
5. **Chinese quotes in JS** - Must use \u201c and \u201d, never literal Chinese quotes in JS strings
6. **Company section** - User may say "这段不用输出" - respect this explicitly
7. **First paragraph** - No title, direct start with "[Year]年[Month]月，制造业采购经理指数..."
8. **Outlook paragraphs** - Each outlook is a standalone paragraph, not a subsection

## Example Invocation

User: "帮我写2026年7月的宏观简报"

Skill workflow:
1. Fetch July 2026 PMI, DR007, treasury yields, equity indices from iFinD
2. Ask user if DM reports available
3. If yes, integrate; if no, proceed with base version
4. Generate structured text following template
5. Create DOCX with proper CJK fonts
6. Run sanitize.py
7. Deliver final document
