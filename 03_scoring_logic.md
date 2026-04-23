# SBTI 人格测试 — 计算公式与逻辑逆向还原

---

## 1. 维度体系

测试使用 **15 个维度**，分为 5 个模型组，每组 3 个维度：

| 模型组 | 维度编号 | 维度名称 | 对应题号 |
|--------|---------|---------|---------|
| 自我模型 (S) | S1 | 自尊自信 | q1, q2 |
| 自我模型 (S) | S2 | 自我清晰度 | q3, q4 |
| 自我模型 (S) | S3 | 核心价值 | q5, q6 |
| 情感模型 (E) | E1 | 依恋安全感 | q7, q8 |
| 情感模型 (E) | E2 | 情感投入度 | q9, q10 |
| 情感模型 (E) | E3 | 边界与依赖 | q11, q12 |
| 态度模型 (A) | A1 | 世界观倾向 | q13, q14 |
| 态度模型 (A) | A2 | 规则与灵活度 | q15, q16 |
| 态度模型 (A) | A3 | 人生意义感 | q17, q18 |
| 行动驱力模型 (Ac) | Ac1 | 动机导向 | q19, q20 |
| 行动驱力模型 (Ac) | Ac2 | 决策风格 | q21, q22 |
| 行动驱力模型 (Ac) | Ac3 | 执行模式 | q23, q24 |
| 社交模型 (So) | So1 | 社交主动性 | q25, q26 |
| 社交模型 (So) | So2 | 人际边界感 | q27, q28 |
| 社交模型 (So) | So3 | 表达与真实度 | q29, q30 |

---

## 2. 原始分数计算

每个维度有 2 道题，每道题的选项值为 1/2/3（少数题目选项值反序）。

```javascript
// 将同一维度下的两道题的答案值相加
rawScores[dim] = answers[q1_of_dim] + answers[q2_of_dim];
// 范围: 2 ~ 6
```

---

## 3. 维度分级（Level 映射）

```javascript
function sumToLevel(score) {
  if (score <= 3) return 'L';  // 低
  if (score === 4) return 'M'; // 中
  return 'H';                  // 高 (score >= 5)
}
```

| 两题总分 | Level | 数值编码 |
|---------|-------|---------|
| 2, 3 | L (Low) | 1 |
| 4 | M (Medium) | 2 |
| 5, 6 | H (High) | 3 |

---

## 4. 人格匹配模式（Pattern）

每个人格类型定义了一个 15 位 pattern 字符串，对应 15 个维度的 L/M/H 期望值：

```javascript
// pattern 格式: "HHH-HMH-MHH-HHH-MHM"
// 对应顺序:      S1S2S3-E1E2E3-A1A2A3-Ac1Ac2Ac3-So1So2So3
// 去掉横杠后为 15 个字符的 L/M/H 序列
```

**25 种常规人格的 pattern 列表：**

| 人格 | Pattern |
|------|---------|
| CTRL | `HHH-HMH-MHH-HHH-MHM` |
| ATM-er | `HHH-HHM-HHH-HMH-MHL` |
| Dior-s | `MHM-MMH-MHM-HMH-LHL` |
| BOSS | `HHH-HMH-MMH-HHH-LHL` |
| THAN-K | `MHM-HMM-HHM-MMH-MHL` |
| OH-NO | `HHL-LMH-LHH-HHM-LHL` |
| GOGO | `HHM-HMH-MMH-HHH-MHM` |
| SEXY | `HMH-HHL-HMM-HMM-HLH` |
| LOVE-R | `MLH-LHL-HLH-MLM-MLH` |
| MUM | `MMH-MHL-HMM-LMM-HLL` |
| FAKE | `HLM-MML-MLM-MLM-HLH` |
| OJBK | `MMH-MMM-HML-LMM-MML` |
| MALO | `MLH-MHM-MLH-MLH-LMH` |
| JOKE-R | `LLH-LHL-LML-LLL-MLM` |
| WOC! | `HHL-HMH-MMH-HHM-LHH` |
| THIN-K | `HHL-HMH-MLH-MHM-LHH` |
| SHIT | `HHL-HLH-LMM-HHM-LHH` |
| ZZZZ | `MHL-MLH-LML-MML-LHM` |
| POOR | `HHL-MLH-LMH-HHH-LHL` |
| MONK | `HHL-LLH-LLM-MML-LHM` |
| IMSB | `LLM-LMM-LLL-LLL-MLM` |
| SOLO | `LML-LLH-LHL-LML-LHM` |
| FUCK | `MLL-LHL-LLM-MLL-HLH` |
| DEAD | `LLL-LLM-LML-LLL-LHM` |
| IMFW | `LLH-LHL-LML-LLL-MLL` |

---

## 5. 匹配算法（Manhattan 距离）

```javascript
function computeResult() {
  // Step 1: 计算用户 15 维度向量（数值形式 L=1, M=2, H=3）
  const userVector = dimensionOrder.map(dim => levelNum(levels[dim]));
  // 例: [3, 2, 3, 2, 3, 3, 3, 2, 3, 3, 2, 3, 2, 3, 2]

  // Step 2: 对每个人格类型计算距离
  const ranked = NORMAL_TYPES.map(type => {
    const vector = parsePattern(type.pattern).map(levelNum);

    let distance = 0;  // Manhattan 距离
    let exact = 0;     // 完全匹配的维度数

    for (let i = 0; i < vector.length; i++) {
      const diff = Math.abs(userVector[i] - vector[i]);
      distance += diff;
      if (diff === 0) exact += 1;
    }

    // Step 3: 计算相似度百分比
    const similarity = Math.max(0, Math.round((1 - distance / 30) * 100));

    return { ...type, ...TYPE_LIBRARY[type.code], distance, exact, similarity };
  });

  // Step 4: 排序 — 距离升序 > 精确匹配降序 > 相似度降序
  ranked.sort((a, b) => {
    if (a.distance !== b.distance) return a.distance - b.distance;
    if (b.exact !== a.exact) return b.exact - a.exact;
    return b.similarity - a.similarity;
  });

  return ranked[0]; // 最优匹配
}
```

**关键公式：**
- `distance` = 用户向量与类型 pattern 向量的曼哈顿距离（绝对差之和）
- `exact` = 维度完全匹配的个数（0~15）
- `similarity` = `max(0, round((1 - distance / 30) × 100))`
  - 最大距离 = 15 × 2 = 30（每一维度最大差值为 H(3)-L(1)=2）

---

## 6. 特殊人格判定逻辑

```javascript
// 隐藏人格: DRUNK 酒鬼
if (app.answers['drink_gate_q2'] === 2) {
  finalType = TYPE_LIBRARY.DRUNK;
  // 强制覆盖，跳过常规匹配
}

// 兜底人格: HHHH 傻乐者
if (bestNormal.similarity < 60) {
  finalType = TYPE_LIBRARY.HHHH;
  // 标准库最高匹配率不足 60% 时触发
}
```

**判定优先级：**
1. **DRUNK（酒鬼）** — 最高优先级，酒精触发题判定
2. **HHHH（傻乐者）** — 次高优先级，匹配率 < 60% 时的兜底
3. **常规 25 种人格** — 按距离排序取最优

---

## 7. 结果页展示逻辑

```javascript
// 匹配度徽章
badge = `匹配度 ${similarity}% · 精准命中 ${exact}/15 维`;

// 维度评分展示
// 每个维度显示: Level + 原始分数
// 例: "H / 5分"
```

---

## 8. 完整计算流程图

```
用户作答 30 题 + 门控题
        ↓
按维度汇总原始分数 (每维 2~6 分)
        ↓
sumToLevel() → 15 位 L/M/H 向量
        ↓
判定 DRUNK 触发? ──Yes──→ 强制返回 DRUNK
        ↓ No
与 25 种人格 pattern 计算 Manhattan 距离
        ↓
排序: distance ↑ → exact ↓ → similarity ↓
        ↓
最优匹配 similarity < 60%? ──Yes──→ 强制返回 HHHH
        ↓ No
返回最优匹配人格
```
