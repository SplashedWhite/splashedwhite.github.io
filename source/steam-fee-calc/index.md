---
title: Steam 市场手续费计算器
date: 2026-06-13 18:05:03
banner_img: /img/关于背景.jpg
comments: false
sidebar: false
---

{% raw %}
<style>
.steam-calc-wrap {
  max-width: 1000px;
  margin: 2rem auto;
  padding: 0 16px;
}

.steam-calc-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

.calc-card {
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  padding: 28px;
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}

.calc-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 28px rgba(0, 0, 0, 0.12);
}

.calc-card-title {
  margin: 0 0 20px 0;
  font-size: 1.25rem;
  font-weight: 600;
  color: #1b2838;
  border-left: 4px solid #66c0f4;
  padding-left: 12px;
}

.input-box {
  margin-bottom: 24px;
}

.input-box label {
  display: block;
  margin-bottom: 8px;
  color: #555;
  font-size: 0.95rem;
}

.input-box input {
  width: 100%;
  padding: 12px 14px;
  border: 2px solid #e8e8e8;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
  box-sizing: border-box;
  outline: none;
  font-family: inherit;
}

.input-box input:focus {
  border-color: #66c0f4;
  box-shadow: 0 0 0 3px rgba(102, 192, 244, 0.15);
}

.result-panel {
  background: #f8fafc;
  border-radius: 12px;
  padding: 18px;
}

.result-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  font-size: 0.95rem;
}

.result-row.main {
  font-size: 1.1rem;
  font-weight: 500;
  padding-bottom: 12px;
}

.result-label {
  color: #666;
}

.result-value {
  font-weight: 600;
  color: #1b2838;
}

.result-row.main .result-value {
  font-size: 1.45rem;
  color: #2da44e;
}

.result-value.fee {
  color: #e67e22;
}

.result-divider {
  height: 1px;
  background: #e2e8f0;
  margin: 8px 0 12px 0;
}

.calc-tip {
  margin-top: 16px;
  font-size: 0.85rem;
  color: #999;
  line-height: 1.6;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .steam-calc-grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }
  .calc-card {
    padding: 20px;
  }
  .result-row.main .result-value {
    font-size: 1.25rem;
  }
}
</style>

<div class="steam-calc-wrap">
  <div class="steam-calc-grid">
    <!-- 左侧：挂牌价算实得 -->
    <div class="calc-card">
      <h3 class="calc-card-title">已知挂牌价 → 算实际到手</h3>
      <div class="input-box">
        <label>挂牌售价（元）</label>
        <input type="number" id="listPrice" placeholder="请输入市场挂牌价格" step="0.01" min="0">
      </div>
      <div class="result-panel">
        <div class="result-row main">
          <span class="result-label">实际到手金额</span>
          <span class="result-value" id="receiveAmount">0.00</span>
        </div>
        <div class="result-divider"></div>
        <div class="result-row">
          <span class="result-label">总手续费（15%）</span>
          <span class="result-value fee" id="totalFee">0.00</span>
        </div>
        <div class="result-row">
          <span class="result-label">游戏开发商分成（10%）</span>
          <span class="result-value" id="devFee">0.00</span>
        </div>
        <div class="result-row">
          <span class="result-label">Steam 平台分成（5%）</span>
          <span class="result-value" id="valveFee">0.00</span>
        </div>
      </div>
      <div class="calc-tip">
        规则说明：手续费基于实得金额计算，总费率 15%<br>
        公式：实得 = 挂牌价 ÷ 1.15
      </div>
    </div>

    <!-- 右侧：实得反推挂牌价 -->
    <div class="calc-card">
      <h3 class="calc-card-title">已知目标实得 → 反推挂牌价</h3>
      <div class="input-box">
        <label>期望到手金额（元）</label>
        <input type="number" id="targetReceive" placeholder="请输入你想拿到的金额" step="0.01" min="0">
      </div>
      <div class="result-panel">
        <div class="result-row main">
          <span class="result-label">需设置的挂牌价</span>
          <span class="result-value" id="needListPrice">0.00</span>
        </div>
        <div class="result-divider"></div>
        <div class="result-row">
          <span class="result-label">总手续费（15%）</span>
          <span class="result-value fee" id="targetTotalFee">0.00</span>
        </div>
        <div class="result-row">
          <span class="result-label">游戏开发商分成（10%）</span>
          <span class="result-value" id="targetDevFee">0.00</span>
        </div>
        <div class="result-row">
          <span class="result-label">Steam 平台分成（5%）</span>
          <span class="result-value" id="targetValveFee">0.00</span>
        </div>
      </div>
      <div class="calc-tip">
        自动补齐手续费，确保你最终拿到指定金额<br>
        公式：挂牌价 = 目标实得 × 1.15
      </div>
    </div>
  </div>
</div>

<script>
// 金额格式化，保留两位小数
function formatMoney(num) {
  if (isNaN(num) || !isFinite(num) || num <= 0) return '0.00';
  return num.toFixed(2);
}

// 正向计算：挂牌价 → 实得
function calcByListPrice() {
  const listPrice = parseFloat(document.getElementById('listPrice').value);
  const receive = listPrice / 1.15;
  const totalFee = listPrice - receive;
  const devFee = receive * 0.1;
  const valveFee = receive * 0.05;

  document.getElementById('receiveAmount').textContent = formatMoney(receive);
  document.getElementById('totalFee').textContent = formatMoney(totalFee);
  document.getElementById('devFee').textContent = formatMoney(devFee);
  document.getElementById('valveFee').textContent = formatMoney(valveFee);
}

// 反向计算：实得 → 挂牌价
function calcByReceive() {
  const targetReceive = parseFloat(document.getElementById('targetReceive').value);
  const listPrice = targetReceive * 1.15;
  const totalFee = listPrice - targetReceive;
  const devFee = targetReceive * 0.1;
  const valveFee = targetReceive * 0.05;

  document.getElementById('needListPrice').textContent = formatMoney(listPrice);
  document.getElementById('targetTotalFee').textContent = formatMoney(totalFee);
  document.getElementById('targetDevFee').textContent = formatMoney(devFee);
  document.getElementById('targetValveFee').textContent = formatMoney(valveFee);
}

// 绑定实时计算事件
document.getElementById('listPrice').addEventListener('input', calcByListPrice);
document.getElementById('targetReceive').addEventListener('input', calcByReceive);
</script>
{% endraw %}