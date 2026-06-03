# 5+2轻断食 — 断食日休息对齐设计

## 目标
将用户选择的练几休几模式与碳循环训练部位结合，并自动将休息日对齐到断食日。

## 核心逻辑

### 1. 训练部位映射（与碳循环一致）
按训练模式对应不同的部位序列：

| 模式 | 训练部位序列（7天） |
|------|-------------------|
| 练1休1 | 胸+三头+腹, 休息, 背+二头+有氧, 休息, 腿+肩, 休息, 胸+三头+腹 |
| 练2休1 | 胸+三头+腹, 背+二头+有氧, 休息, 腿+肩, 胸+三头+腹, 休息, 背+二头+有氧 |
| 练3休1 | 胸+三头+腹, 背+二头+有氧, 腿+肩, 休息, 胸+三头+腹, 背+二头+有氧, 腿+肩 |
| 练4休1 | 胸+三头+腹, 背+二头+有氧, 腿+肩, 胸+三头+腹, 休息, 背+二头+有氧, 腿+肩 |

### 2. 最佳偏移量计算
```
function findBestOffset(fastingDays, trainingPattern):
  bestOffset = 0, bestScore = -1
  for offset in 0..6:
    score = 0
    for day in 0..6:
      if pattern[(day+offset)%7] == "休息" and day in fastingDays:
        score++  // 休息日匹配断食日+1分
    if score > bestScore:
      bestScore = score, bestOffset = offset
  return bestOffset
```

### 3. 28天展开
按最佳偏移量展开28天，每天有：
- `bodyPart`: 训练部位或"休息"
- `isTraining`: 是否为训练日（有部位=true，休息=false）
- 断食日强制覆盖为休息（即使偏移后仍有冲突）

### 4. 展示变化
- 训练卡片下方显示实际排期（对齐后），包含部位名称
- 食谱表格新增"训练部位"行（在早餐行之前）
- 宏量营养表"训练"列改为"训练部位"显示
- Excel导出也包含训练部位

## 文件修改

### D:\CCd\饮食计划\health-tools\5+2轻断食.html

**修改区域**：
1. 新增 `bodyPartPlans` 对象，存储各模式的7天部位序列
2. 修改训练逻辑：使用最佳偏移量展开28天
3. 修改 `updateTrainingSchedule()`：显示对齐后的部位排期
4. 修改 `generateMonthRecipe()`：新增训练部位行
5. 修改 `downloadFivetwo()`：Excel导出包含部位
6. 修改 `renderMacroTable()`：训练列改为部位显示
