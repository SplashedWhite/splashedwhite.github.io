---
title: VPS 剩余价值与溢价计算器
date: 2026-06-13 22:23:08
banner_img: /img/关于背景.jpg
comments: false
sidebar: false
---


{% raw %}
<style>
.vps-calc-wrap {
  max-width: 1000px;
  margin: 2rem auto;
  padding: 0 16px;
  display: flex;
  flex-direction: column;
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

.input-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px 20px;
}

.input-box {
  display: flex;
  flex-direction: column;
}

.input-box label {
  margin-bottom: 8px;
  color: #555;
  font-size: 0.95rem;
}

.input-box input,
.input-box select {
  width: 100%;
  padding: 12px 14px;
  border: 2px solid #e8e8e8;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
  box-sizing: border-box;
  outline: none;
  font-family: inherit;
  background: #fff;
}

.input-box input:focus,
.input-box select:focus {
  border-color: #66c0f4;
  box-shadow: 0 0 0 3px rgba(102, 192, 244, 0.15);
}

.btn-secondary {
  padding: 0 14px;
  border: none;
  border-radius: 8px;
  background: #f1f5f9;
  color: #475569;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.2s ease;
  white-space: nowrap;
}

.btn-secondary:hover {
  background: #e2e8f0;
}

.btn-secondary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
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

.result-value.positive {
  color: #e53e3e;
}

.result-value.negative {
  color: #2da44e;
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

.rate-tip {
  margin-top: 6px;
  font-size: 0.8rem;
  color: #94a3b8;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .input-grid {
    grid-template-columns: 1fr;
  }
  .calc-card {
    padding: 20px;
  }
  .result-row.main .result-value {
    font-size: 1.25rem;
  }
}
</style>

<div class="vps-calc-wrap">
  <!-- 基础配置卡片 -->
  <div class="calc-card">
    <h3 class="calc-card-title">基础配置</h3>
    <div class="input-grid">
      <div class="input-box">
        <label>对应周期续费金额</label>
        <input type="number" id="renewAmount" step="0.01" min="0" placeholder="输入所选付款周期的官方续费价">
      </div>
      <div class="input-box">
        <label>货币单位</label>
        <select id="currency">
          <option value="usd">美元 USD</option>
          <option value="eur">欧元 EUR</option>
          <option value="gbp">英镑 GBP</option>
          <option value="jpy">日元 JPY</option>
          <option value="hkd">港币 HKD</option>
          <option value="sgd">新加坡元 SGD</option>
          <option value="cad">加拿大元 CAD</option>
          <option value="twd">新台币 TWD</option>
          <option value="krw">韩元 KRW</option>
          <option value="cny">人民币 CNY</option>
        </select>
      </div>
      <div class="input-box">
        <label>付款周期</label>
        <select id="billingCycle">
          <option value="15">半月付（15天）</option>
          <option value="30" selected>月付（30天）</option>
          <option value="90">季付（90天）</option>
          <option value="182">半年付（182天）</option>
          <option value="365">年付（365天）</option>
          <option value="730">两年付（730天）</option>
          <option value="1095">三年付（1095天）</option>
        </select>
      </div>
      <div class="input-box">
        <label>当前汇率（1 原货币 = ? 人民币）</label>
        <div style="display: flex; gap: 8px;">
          <input type="number" id="exchangeRate" step="0.0001" min="0" placeholder="可手动修改">
          <button id="fetchRateBtn" class="btn-secondary">刷新汇率</button>
        </div>
        <div class="rate-tip">数据来自 apihz.cn 公开汇率接口，每日更新</div>
      </div>
    </div>
  </div>

  <!-- 日期与交易卡片 -->
  <div class="calc-card">
    <h3 class="calc-card-title">日期与交易信息</h3>
    <div class="input-grid">
      <div class="input-box">
        <label>计算基准日期（交易日期）</label>
        <input type="date" id="tradeDate">
      </div>
      <div class="input-box">
        <label>服务到期时间</label>
        <input type="date" id="expireDate">
      </div>
      <div class="input-box" style="grid-column: 1 / -1;">
        <label>交易价格（人民币）</label>
        <input type="number" id="tradePrice" step="0.01" min="0" placeholder="填写实际成交价格，自动计算溢价/折价">
      </div>
    </div>
  </div>

  <!-- 结果卡片 -->
  <div class="calc-card">
    <h3 class="calc-card-title">计算结果</h3>
    <div class="result-panel">
      <div class="result-row main">
        <span class="result-label">剩余价值（人民币）</span>
        <span class="result-value" id="remainValueCny">0.00</span>
      </div>
      <div class="result-divider"></div>
      <div class="result-row">
        <span class="result-label">剩余服务天数</span>
        <span class="result-value" id="remainDays">0 天</span>
      </div>
      <div class="result-row">
        <span class="result-label">剩余价值（原货币）</span>
        <span class="result-value" id="remainValueOrg">0.00</span>
      </div>
      <div class="result-divider"></div>
      <div class="result-row">
        <span class="result-label">溢价/折价金额</span>
        <span class="result-value" id="premiumAmount">0.00</span>
      </div>
      <div class="result-row">
        <span class="result-label">溢价率</span>
        <span class="result-value" id="premiumRate">0.00%</span>
      </div>
    </div>
    <div class="calc-tip">
      计算公式：剩余价值 = 周期续费金额 × (剩余天数 ÷ 付款周期总天数) × 汇率<br>
      溢价率 = (交易价格 - 剩余价值) ÷ 剩余价值 × 100%；正数为溢价（买贵），负数为折价（买赚）<br>
      结果仅为理论参考值，实际交易还需考虑平台Push手续费、线路溢价、传家宝属性等因素
    </div>
  </div>
</div>

<script>
// 金额格式化
function formatMoney(num) {
  if (isNaN(num) || !isFinite(num)) return '0.00';
  return num.toFixed(2);
}

// 修正时区的日期解析，按本地时间计算天数差
function getLocalDaysDiff(dateStartStr, dateEndStr) {
  if (!dateStartStr || !dateEndStr) return 0;
  const [y1, m1, d1] = dateStartStr.split('-').map(Number);
  const [y2, m2, d2] = dateEndStr.split('-').map(Number);
  const start = new Date(y1, m1 - 1, d1);
  const end = new Date(y2, m2 - 1, d2);
  const diffTime = end - start;
  const diffDays = diffTime / (1000 * 60 * 60 * 24);
  return Math.max(0, Math.round(diffDays * 100) / 100);
}

// 核心计算函数
function calculateAll() {
  const renewAmount = parseFloat(document.getElementById('renewAmount').value) || 0;
  const cycleDays = parseInt(document.getElementById('billingCycle').value) || 30;
  const rate = parseFloat(document.getElementById('exchangeRate').value) || 0;
  const tradeDate = document.getElementById('tradeDate').value;
  const expireDate = document.getElementById('expireDate').value;
  const tradePrice = parseFloat(document.getElementById('tradePrice').value) || 0;

  const remainDays = getLocalDaysDiff(tradeDate, expireDate);
  const remainOrg = cycleDays > 0 ? renewAmount * (remainDays / cycleDays) : 0;
  const remainCny = remainOrg * rate;

  let premiumAmount = 0;
  let premiumRate = 0;
  if (remainCny > 0 && tradePrice > 0) {
    premiumAmount = tradePrice - remainCny;
    premiumRate = (premiumAmount / remainCny) * 100;
  }

  document.getElementById('remainDays').textContent = remainDays.toFixed(0) + ' 天';
  document.getElementById('remainValueOrg').textContent = formatMoney(remainOrg);
  document.getElementById('remainValueCny').textContent = formatMoney(remainCny);

  const premiumAmountEl = document.getElementById('premiumAmount');
  const premiumRateEl = document.getElementById('premiumRate');
  
  const amountText = premiumAmount > 0 ? '+' + formatMoney(premiumAmount) : formatMoney(premiumAmount);
  const rateText = premiumRate > 0 ? '+' + formatMoney(premiumRate) + '%' : formatMoney(premiumRate) + '%';
  premiumAmountEl.textContent = amountText;
  premiumRateEl.textContent = rateText;

  premiumAmountEl.classList.remove('positive', 'negative');
  premiumRateEl.classList.remove('positive', 'negative');
  if (premiumAmount > 0) {
    premiumAmountEl.classList.add('positive');
    premiumRateEl.classList.add('positive');
  } else if (premiumAmount < 0) {
    premiumAmountEl.classList.add('negative');
    premiumRateEl.classList.add('negative');
  }
}

// 获取实时汇率（使用 apihz.cn 接口）
async function fetchExchangeRate() {
  const currency = document.getElementById('currency').value;
  const btn = document.getElementById('fetchRateBtn');
  
  // 人民币直接设为1
  if (currency === 'cny') {
    document.getElementById('exchangeRate').value = '1.0000';
    calculateAll();
    return;
  }

  btn.disabled = true;
  btn.textContent = '获取中...';

  try {
    // 接口参数：id/key 固定值，from为原货币（大写），to为CNY，money=1取单单位汇率
    const fromCode = currency.toUpperCase();
    const res = await fetch(`https://cn.apihz.cn/api/jinrong/huilv.php?id=88888888&key=88888888&from=${fromCode}&to=CNY&money=1`);
    const data = await res.json();
    
    if (data.code === 200 && data.rate) {
      document.getElementById('exchangeRate').value = parseFloat(data.rate).toFixed(4);
      calculateAll();
    } else {
      alert('获取汇率失败，请手动输入');
    }
  } catch (e) {
    alert('汇率接口请求失败，请检查网络或手动输入汇率');
  } finally {
    btn.disabled = false;
    btn.textContent = '刷新汇率';
  }
}

// 初始化日期：默认基准日期为今天，到期日期默认30天后
function initDate() {
  const today = new Date();
  const todayStr = `${today.getFullYear()}-${String(today.getMonth()+1).padStart(2,'0')}-${String(today.getDate()).padStart(2,'0')}`;
  document.getElementById('tradeDate').value = todayStr;
  
  const expire = new Date(today);
  expire.setDate(expire.getDate() + 30);
  const expireStr = `${expire.getFullYear()}-${String(expire.getMonth()+1).padStart(2,'0')}-${String(expire.getDate()).padStart(2,'0')}`;
  document.getElementById('expireDate').value = expireStr;
}

// 绑定事件
document.getElementById('renewAmount').addEventListener('input', calculateAll);
document.getElementById('currency').addEventListener('change', function() {
  fetchExchangeRate();
});
document.getElementById('billingCycle').addEventListener('change', calculateAll);
document.getElementById('exchangeRate').addEventListener('input', calculateAll);
document.getElementById('tradeDate').addEventListener('change', calculateAll);
document.getElementById('expireDate').addEventListener('change', calculateAll);
document.getElementById('tradePrice').addEventListener('input', calculateAll);
document.getElementById('fetchRateBtn').addEventListener('click', fetchExchangeRate);

// 页面初始化
initDate();
// 页面加载完成后自动获取一次默认汇率
window.addEventListener('load', function() {
  fetchExchangeRate();
});
</script>
{% endraw %}