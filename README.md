<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HVAC 暖通空调与防排烟综合计算小程序</title>
  <style>
    :root {
      --primary-color: #1677ff;
      --bg-color: #f5f7fa;
      --card-bg: #ffffff;
      --text-main: #262626;
      --text-muted: #595959;
      --border-color: #d9d9d9;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
      background-color: var(--bg-color);
      color: var(--text-main);
      padding: 20px;
      line-height: 1.5;
    }

    .container {
      max-width: 900px;
      margin: 0 auto;
      background: var(--card-bg);
      border-radius: 12px;
      box-shadow: 0 4px 16px rgba(0,0,0,0.08);
      padding: 28px;
    }

    h1 {
      text-align: center;
      font-size: 24px;
      margin-bottom: 24px;
      color: #1f2329;
    }

    .tabs {
      display: flex;
      border-bottom: 2px solid #e5e6eb;
      margin-bottom: 24px;
    }

    .tab-btn {
      padding: 10px 20px;
      font-size: 15px;
      font-weight: 600;
      background: none;
      border: none;
      cursor: pointer;
      color: var(--text-muted);
      border-bottom: 2px solid transparent;
      margin-bottom: -2px;
      transition: all 0.2s;
    }

    .tab-btn.active {
      color: var(--primary-color);
      border-bottom-color: var(--primary-color);
    }

    .tab-content {
      display: none;
    }

    .tab-content.active {
      display: block;
    }

    .section-title {
      font-size: 15px;
      font-weight: bold;
      color: var(--primary-color);
      margin: 16px 0 10px 0;
      padding-left: 8px;
      border-left: 4px solid var(--primary-color);
    }

    .form-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
      gap: 16px;
      margin-bottom: 16px;
    }

    .form-group {
      display: flex;
      flex-direction: column;
    }

    .form-group label {
      font-size: 13px;
      font-weight: 600;
      margin-bottom: 6px;
      color: #333;
    }

    .form-group input, .form-group select {
      padding: 9px 12px;
      border: 1px solid var(--border-color);
      border-radius: 6px;
      font-size: 14px;
      outline: none;
      transition: border-color 0.2s;
    }

    .form-group input:focus, .form-group select:focus {
      border-color: var(--primary-color);
    }

    .btn-submit {
      width: 100%;
      background: var(--primary-color);
      color: #fff;
      border: none;
      padding: 12px;
      border-radius: 6px;
      font-size: 15px;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.2s;
      margin-top: 10px;
    }

    .btn-submit:hover {
      background: #0958d9;
    }

    .result-card {
      margin-top: 24px;
      padding: 20px;
      background-color: #fafafa;
      border: 1px solid #f0f0f0;
      border-radius: 8px;
    }

    .result-card h2 {
      font-size: 16px;
      border-bottom: 1px solid #e8e8e8;
      padding-bottom: 10px;
      margin-bottom: 16px;
      color: #1f2329;
    }

    .result-item {
      margin-bottom: 10px;
      font-size: 14px;
    }

    .result-item span.highlight {
      color: #d9363e;
      font-weight: bold;
    }

    .result-item span.highlight-blue {
      color: var(--primary-color);
      font-weight: bold;
    }

    .status-badge {
      display: inline-block;
      padding: 2px 8px;
      border-radius: 4px;
      font-size: 12px;
      font-weight: bold;
      margin-left: 8px;
    }

    .status-pass {
      background-color: #f6ffed;
      color: #52c41a;
      border: 1px solid #b7eb8f;
    }

    .status-warn {
      background-color: #fff2e8;
      color: #fa541c;
      border: 1px solid #ffbb96;
    }

    .notice-box {
      margin-top: 16px;
      padding: 12px;
      background-color: #fffbe6;
      border: 1px solid #ffe58f;
      border-radius: 6px;
      font-size: 13px;
      color: #8c6b00;
    }

    .notice-box ul {
      margin-left: 18px;
      margin-top: 6px;
    }

    .notice-box li {
      margin-bottom: 4px;
    }
  </style>
</head>
<body>

<div class="container">
  <h1>HVAC 暖通空调与防排烟综合计算小程序</h1>

  <div class="tabs">
    <button class="tab-btn active" onclick="switchTab('hvac')">空调冷量/风量负荷及管径计算</button>
    <button class="tab-btn" onclick="switchTab('smoke')">排烟风管与风口尺寸校核与选型</button>
    <button class="tab-btn" onclick="switchTab('cooling')">根据冷量选选风管与水/冷媒管径</button>
  </div>

  <!-- 选项卡 1：暖通负荷及水/冷媒管径计算 -->
  <div id="hvac" class="tab-content active">
    <div class="form-grid">
      <div class="form-group">
        <label>建筑/房间面积 (m²):</label>
        <input type="number" id="hvacArea" value="300" placeholder="请输入面积">
      </div>
      <div class="form-group">
        <label>房间类型 (预设冷指标 W/m²):</label>
        <select id="roomType" onchange="updateCustomLoad()">
          <option value="150">普通办公室 (150 W/m²)</option>
          <option value="200">商业商场/商铺 (200 W/m²)</option>
          <option value="250">餐厅/宴会厅 (250 W/m²)</option>
          <option value="300">数据中心/机房 (300 W/m²)</option>
          <option value="custom" selected>-- 自定义冷负荷指标 --</option>
        </select>
      </div>
      <div class="form-group">
        <label>冷负荷指标 (W/m²):</label>
        <input type="number" id="customLoad" value="350">
      </div>
    </div>
    <button class="btn-submit" onclick="calcHVAC()">一键计算空调负荷及管径建议</button>

    <div class="result-card" id="hvacResult" style="display:none;">
      <h2>一、空调负荷计算结果</h2>
      <div class="result-item">设计总冷量: <span class="highlight" id="totalCooling">0</span> kW</div>
      <div class="result-item">空调总匹数建议: <span class="highlight-blue" id="totalHP">0</span> 匹 (HP)</div>
      <div class="result-item">全空气系统风量建议: <span class="highlight" id="airVolume">0</span> m³/h</div>

      <hr style="margin:14px 0; border:0; border-top:1px dashed #d9d9d9;">

      <h2>二、水系统管径建议 (水系统机组)</h2>
      <div class="result-item">冷冻水流量 (温差 5℃): <span id="chilledWaterFlow">0</span> m³/h</div>
      <div class="result-item">冷冻水管径建议: <span class="highlight-blue" id="chilledWaterPipe">--</span> (控制流速 &le; 1.5 m/s)</div>
      
      <div class="result-item" style="margin-top:8px;">冷却水流量 (温差 5℃，含热回收): <span id="coolingWaterFlow">0</span> m³/h</div>
      <div class="result-item">冷却水管径建议: <span class="highlight-blue" id="coolingWaterPipe">--</span> (控制流速 &le; 2.0 m/s)</div>

      <div class="result-item" style="margin-top:8px;">冷凝水管径建议: <span class="highlight" id="condensatePipe">--</span> (按冷量及无压坡度排水推荐)</div>

      <hr style="margin:14px 0; border:0; border-top:1px dashed #d9d9d9;">

      <h2>三、冷媒铜管管径建议 (VRF氟系统机组)</h2>
      <div class="result-item">冷媒气管建议 (低压管): <span class="highlight-blue" id="refrigGasPipe">--</span></div>
      <div class="result-item">冷媒液管建议 (高压管): <span class="highlight-blue" id="refrigLiquidPipe">--</span></div>
    </div>
  </div>

  <!-- 选项卡 2：防排烟管道与风口尺寸全面选型 (保持不变) -->
  <div id="smoke" class="tab-content">
    
    <div class="section-title">1. 防烟分区及基本参数</div>
    <div class="form-grid">
      <div class="form-group">
        <label>防烟分区面积 (m²):</label>
        <input type="number" id="smokeArea" value="400">
      </div>
      <div class="form-group">
        <label>空间净高 H (m):</label>
        <input type="number" id="spaceHeight" value="3.8" step="0.1">
      </div>
      <div class="form-group">
        <label>自动灭火系统（喷淋）:</label>
        <select id="hasSprinkler">
          <option value="yes" selected>有喷淋 (烟层薄，单口风量需控制)</option>
          <option value="no">无喷淋</option>
        </select>
      </div>
    </div>

    <div class="section-title">2. 排烟风口尺寸及数量设置</div>
    <div class="form-grid">
      <div class="form-group">
        <label>风口尺寸设置模式:</label>
        <select id="portSizeMode" onchange="togglePortInput()">
          <option value="manual" selected>手动自定义排烟风口尺寸</option>
          <option value="auto">系统自动推荐标准尺寸</option>
        </select>
      </div>
      <div class="form-group" id="portCustomWidthGroup">
        <label>自定义风口宽度 W (mm):</label>
        <input type="number" id="portWidth" value="1000" step="50">
      </div>
      <div class="form-group" id="portCustomHeightGroup">
        <label>自定义风口高度 H (mm):</label>
        <input type="number" id="portHeight" value="630" step="50">
      </div>
      <div class="form-group">
        <label>有效开孔率 (%):</label>
        <input type="number" id="openRatio" value="75" min="50" max="100">
      </div>
    </div>

    <div class="section-title">3. 排烟管道/风管尺寸设置</div>
    <div class="form-grid">
      <div class="form-group">
        <label>风管尺寸设置模式:</label>
        <select id="ductSizeMode" onchange="toggleDuctInput()">
          <option value="manual" selected>手动填写风管尺寸 (宽×高)</option>
          <option value="auto">自动计算风管推荐尺寸</option>
        </select>
      </div>
      <div class="form-group" id="ductWidthGroup">
        <label>排烟风管宽度 A (mm):</label>
        <input type="number" id="ductWidth" value="1250" step="50">
      </div>
      <div class="form-group" id="ductHeightGroup">
        <label>排烟风管高度 B (mm):</label>
        <input type="number" id="ductHeight" value="500" step="50">
      </div>
    </div>

    <button class="btn-submit" onclick="calcSmoke()">一键校核排烟风管与风口计算</button>

    <div class="result-card" id="smokeResult" style="display:none;">
      <h2>二、排烟系统风管与风口校核结果</h2>
      <div class="result-item">防烟分区最小总排烟量: <span class="highlight" id="totalSmokeVol">0</span> m³/h</div>
      <div class="result-item">单口防“吸穿”最大允许排烟量 (Q_max): <span class="highlight-blue" id="qMaxLimit">0</span> m³/h</div>
      <div class="result-item">设计所需排烟风口最少数量: <span class="highlight" id="minPortCount">0</span> 个</div>
      
      <hr style="margin:12px 0; border:0; border-top:1px dashed #d9d9d9;">

      <div class="result-item">
        <strong>风口尺寸与风速校验:</strong> 
        <span id="portSizeDisplay">--</span>
        <span id="portVelocityBadge" class="status-badge">--</span>
      </div>
      <div class="result-item">单风口计算过风风速: <span class="highlight-blue" id="portVelocity">0</span> m/s (规范要求 &le; 10 m/s)</div>

      <hr style="margin:12px 0; border:0; border-top:1px dashed #d9d9d9;">

      <div class="result-item">
        <strong>排烟风管尺寸与风速校验:</strong> 
        <span id="ductSizeDisplay">--</span>
        <span id="ductVelocityBadge" class="status-badge">--</span>
      </div>
      <div class="result-item">排烟管道实际截面风速: <span class="highlight" id="ductVelocity">0</span> m/s (规范要求: 金属风管 &le; 20 m/s，非金属 &le; 15 m/s)</div>

      <div class="notice-box">
        <strong>⚠️ 距墙门及布局规范复核提示：</strong>
        <ul>
          <li><strong>距安全出口距离：</strong> 排烟口边缘与疏散门洞边缘的水平距离<span style="color:#d9363e;font-weight:bold;">不得小于 1.5m</span>。</li>
          <li><strong>防烟分区服务半径：</strong> 排烟口最远点距防烟分区边缘任意一点水平距离<span style="color:#d9363e;font-weight:bold;">不应大于 30m</span>。</li>
          <li><strong>墙面安装高度：</strong> 若采用墙面排烟口，其上边缘距室内地坪高度<span style="color:#d9363e;font-weight:bold;">不应小于 2.0m</span>。</li>
        </ul>
      </div>
    </div>
  </div>

  <!-- 选项卡 3：根据冷量直接推荐风管及管径 -->
  <div id="cooling" class="tab-content">
    <div class="form-grid">
      <div class="form-group">
        <label>已知系统冷量 Q (kW):</label>
        <input type="number" id="inputCoolingKw" value="100" placeholder="请输入冷量 kW">
      </div>
      <div class="form-group">
        <label>全空气系统风管风速限制 (m/s):</label>
        <input type="number" id="airVelocity" value="6.0" step="0.5" placeholder="主风管推荐 5.0 - 8.0 m/s">
      </div>
    </div>
    <button class="btn-submit" onclick="calcByCooling()">根据冷量推荐风管与各类管径</button>

    <div class="result-card" id="coolingResult" style="display:none;">
      <h2>一、系统基础风量推算</h2>
      <div class="result-item">对应折算匹数: <span class="highlight-blue" id="cTotalHP">0</span> 匹 (HP)</div>
      <div class="result-item">全空气系统建议风量: <span class="highlight" id="cAirVolume">0</span> m³/h</div>

      <hr style="margin:14px 0; border:0; border-top:1px dashed #d9d9d9;">

      <h2>二、全空气系统风管尺寸建议</h2>
      <div class="result-item">风管推荐规格 (宽 × 高): <span class="highlight-blue" id="cAirDuctSize">--</span></div>

      <hr style="margin:14px 0; border:0; border-top:1px dashed #d9d9d9;">

      <h2>三、水系统管径建议 (水系统机组)</h2>
      <div class="result-item">冷冻水流量 (ΔT=5℃): <span id="cChilledFlow">0</span> m³/h</div>
      <div class="result-item">冷冻水管径建议: <span class="highlight-blue" id="cChilledPipe">--</span> (控制流速 &le; 1.5 m/s)</div>
      
      <div class="result-item" style="margin-top:8px;">冷却水流量 (ΔT=5℃，含热回收): <span id="cCoolingFlow">0</span> m³/h</div>
      <div class="result-item">冷却水管径建议: <span class="highlight-blue" id="cCoolingPipe">--</span> (控制流速 &le; 2.0 m/s)</div>

      <div class="result-item" style="margin-top:8px;">冷凝水管径建议: <span class="highlight" id="cCondensatePipe">--</span> (按无压坡度排水推荐)</div>

      <hr style="margin:14px 0; border:0; border-top:1px dashed #d9d9d9;">

      <h2>四、冷媒铜管管径建议 (VRF氟系统机组)</h2>
      <div class="result-item">冷媒气管建议 (低压管): <span class="highlight-blue" id="cRefrigGasPipe">--</span></div>
      <div class="result-item">冷媒液管建议 (高压管): <span class="highlight-blue" id="cRefrigLiquidPipe">--</span></div>
    </div>
  </div>
</div>

<script>
  function switchTab(tabId) {
    document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(content => content.classList.remove('active'));
    
    if (tabId === 'hvac') {
      document.querySelectorAll('.tab-btn')[0].classList.add('active');
      document.getElementById('hvac').classList.add('active');
    } else if (tabId === 'smoke') {
      document.querySelectorAll('.tab-btn')[1].classList.add('active');
      document.getElementById('smoke').classList.add('active');
    } else {
      document.querySelectorAll('.tab-btn')[2].classList.add('active');
      document.getElementById('cooling').classList.add('active');
    }
  }

  function updateCustomLoad() {
    const typeVal = document.getElementById('roomType').value;
    if (typeVal !== 'custom') {
      document.getElementById('customLoad').value = typeVal;
    }
  }

  function togglePortInput() {
    const mode = document.getElementById('portSizeMode').value;
    const displayStyle = mode === 'manual' ? 'flex' : 'none';
    document.getElementById('portCustomWidthGroup').style.display = displayStyle;
    document.getElementById('portCustomHeightGroup').style.display = displayStyle;
  }

  function toggleDuctInput() {
    const mode = document.getElementById('ductSizeMode').value;
    const displayStyle = mode === 'manual' ? 'flex' : 'none';
    document.getElementById('ductWidthGroup').style.display = displayStyle;
    document.getElementById('ductHeightGroup').style.display = displayStyle;
  }

  // --- 第一页：暖通负荷与水/冷媒管径计算逻辑 ---
  function calcHVAC() {
    const area = parseFloat(document.getElementById('hvacArea').value) || 0;
    const loadPerM2 = parseFloat(document.getElementById('customLoad').value) || 0;

    // 1. 基本冷量与匹数计算
    const totalKw = (area * loadPerM2) / 1000; // kW
    const totalHP = totalKw / 2.5; // 匹数估算 (1匹约 2.5kW)
    const airVol = (totalKw * 3600) / (1.2 * 1.005 * 9); // 全空气系统送风量 (m³/h)

    // 2. 冷冻水流量与管径推荐 (ΔT=5℃)
    const chilledFlow = (totalKw * 3600) / (4.187 * 1000 * 5); // m³/h
    const chilledPipeStr = recommendPipeSize(chilledFlow, 1.2); 

    // 3. 冷却水流量与管径推荐 (冷量*1.25放热系数, ΔT=5℃)
    const coolingFlow = chilledFlow * 1.25; // m³/h
    const coolingPipeStr = recommendPipeSize(coolingFlow, 1.5); 

    // 4. 冷凝水管径推荐
    const condensatePipeStr = recommendCondensatePipe(totalKw);

    // 5. 冷媒铜管管径推荐 (VRF/多联机系统)
    const refrigPipes = recommendRefrigerantPipe(totalKw);

    // 渲染结果
    document.getElementById('totalCooling').innerText = totalKw.toFixed(2);
    document.getElementById('totalHP').innerText = totalHP.toFixed(1);
    document.getElementById('airVolume').innerText = Math.round(airVol);

    document.getElementById('chilledWaterFlow').innerText = chilledFlow.toFixed(2);
    document.getElementById('chilledWaterPipe').innerText = chilledPipeStr;

    document.getElementById('coolingWaterFlow').innerText = coolingFlow.toFixed(2);
    document.getElementById('coolingWaterPipe').innerText = coolingPipeStr;

    document.getElementById('condensatePipe').innerText = condensatePipeStr;

    document.getElementById('refrigGasPipe').innerText = refrigPipes.gas;
    document.getElementById('refrigLiquidPipe').innerText = refrigPipes.liquid;

    document.getElementById('hvacResult').style.display = 'block';
  }

  // 根据流量与推荐流速估算水管公称管径 DN
  function recommendPipeSize(flowM3h, targetVelocity) {
    if (flowM3h <= 0) return "--";
    const flowM3s = flowM3h / 3600;
    const reqDiameterM = Math.sqrt((4 * flowM3s) / (Math.PI * targetVelocity));
    const reqDiameterMm = reqDiameterM * 1000;

    const standardPipes = [
      { dn: "DN25", inner: 25 },
      { dn: "DN32", inner: 32 },
      { dn: "DN40", inner: 40 },
      { dn: "DN50", inner: 50 },
      { dn: "DN65", inner: 65 },
      { dn: "DN80", inner: 80 },
      { dn: "DN100", inner: 100 },
      { dn: "DN125", inner: 125 },
      { dn: "DN150", inner: 150 },
      { dn: "DN200", inner: 200 },
      { dn: "DN250", inner: 250 },
      { dn: "DN300", inner: 300 }
    ];

    for (let pipe of standardPipes) {
      if (pipe.inner >= reqDiameterMm) {
        let actualV = flowM3s / (Math.PI * Math.pow(pipe.inner/1000, 2) / 4);
        return `${pipe.dn} (实际流速约 ${actualV.toFixed(2)} m/s)`;
      }
    }
    return "DN350 以上主干管";
  }

  // 根据冷量推荐冷凝水排水管径
  function recommendCondensatePipe(coolingKw) {
    if (coolingKw <= 20) return "DN25 (无压排水坡度 ≥ 0.01)";
    if (coolingKw <= 40) return "DN32 (无压排水坡度 ≥ 0.01)";
    if (coolingKw <= 80) return "DN40 (无压排水坡度 ≥ 0.008)";
    if (coolingKw <= 160) return "DN50 (无压排水坡度 ≥ 0.008)";
    if (coolingKw <= 300) return "DN65 (无压排水坡度 ≥ 0.005)";
    if (coolingKw <= 600) return "DN80 (无压排水坡度 ≥ 0.005)";
    return "DN100 (无压排水坡度 ≥ 0.005)";
  }

  // 根据冷量(kW)推荐冷媒铜管管径 (气管/液管)
  function recommendRefrigerantPipe(coolingKw) {
    if (coolingKw <= 5.6) {
      return { gas: "Φ12.7 mm (1/2\")", liquid: "Φ6.35 mm (1/4\")" };
    } else if (coolingKw <= 16) {
      return { gas: "Φ15.88 mm (5/8\")", liquid: "Φ9.52 mm (3/8\")" };
    } else if (coolingKw <= 22.4) {
      return { gas: "Φ19.05 mm (3/4\")", liquid: "Φ9.52 mm (3/8\")" };
    } else if (coolingKw <= 33.5) {
      return { gas: "Φ22.2 mm (7/8\")", liquid: "Φ9.52 mm (3/8\")" };
    } else if (coolingKw <= 45) {
      return { gas: "Φ28.58 mm (1-1/8\")", liquid: "Φ12.7 mm (1/2\")" };
    } else if (coolingKw <= 67) {
      return { gas: "Φ28.58 mm (1-1/8\")", liquid: "Φ15.88 mm (5/8\")" };
    } else if (coolingKw <= 96) {
      return { gas: "Φ34.9 mm (1-3/8\")", liquid: "Φ19.05 mm (3/4\")" };
    } else if (coolingKw <= 135) {
      return { gas: "Φ38.1 mm (1-1/2\")", liquid: "Φ19.05 mm (3/4\")" };
    } else {
      return { gas: "Φ41.3 mm 以上或多路分流", liquid: "Φ22.2 mm 以上或多路分流" };
    }
  }

  // 根据风量和设定风速，选定标准全空气矩形风管尺寸
  function recommendAirDuct(airVolM3h, targetV) {
    if (airVolM3h <= 0) return "--";
    const reqArea = airVolM3h / (3600 * targetV); // 所需截面积 m²

    const standardDucts = [
      { name: "320 × 200 mm", area: 0.064 },
      { name: "400 × 250 mm", area: 0.100 },
      { name: "500 × 320 mm", area: 0.160 },
      { name: "630 × 320 mm", area: 0.2016 },
      { name: "630 × 400 mm", area: 0.252 },
      { name: "800 × 400 mm", area: 0.320 },
      { name: "800 × 500 mm", area: 0.400 },
      { name: "1000 × 500 mm", area: 0.500 },
      { name: "1000 × 630 mm", area: 0.630 },
      { name: "1250 × 630 mm", area: 0.7875 },
      { name: "1600 × 630 mm", area: 1.008 },
      { name: "1600 × 800 mm", area: 1.280 },
      { name: "2000 × 800 mm", area: 1.600 },
      { name: "2000 × 1000 mm", area: 2.000 }
    ];

    for (let duct of standardDucts) {
      if (duct.area >= reqArea) {
        let actualV = airVolM3h / (3600 * duct.area);
        return `${duct.name} (截面积 ${duct.area.toFixed(2)} m²，实际风速约 ${actualV.toFixed(2)} m/s)`;
      }
    }
    return `截面积需 ≥ ${reqArea.toFixed(2)} m² (建议加大尺寸或分多路输送)`;
  }

  // --- 第三页：直接根据冷量计算各系统管径逻辑 ---
  function calcByCooling() {
    const totalKw = parseFloat(document.getElementById('inputCoolingKw').value) || 0;
    const targetAirV = parseFloat(document.getElementById('airVelocity').value) || 6.0;

    // 1. 基本风量与匹数计算
    const totalHP = totalKw / 2.5; // 匹数估算
    const airVol = (totalKw * 3600) / (1.2 * 1.005 * 9); // 全空气风量

    // 2. 矩形风管管径推荐
    const airDuctStr = recommendAirDuct(airVol, targetAirV);

    // 3. 冷冻水流量与管径推荐 (ΔT=5℃)
    const chilledFlow = (totalKw * 3600) / (4.187 * 1000 * 5);
    const chilledPipeStr = recommendPipeSize(chilledFlow, 1.2);

    // 4. 冷却水流量与管径推荐
    const coolingFlow = chilledFlow * 1.25;
    const coolingPipeStr = recommendPipeSize(coolingFlow, 1.5);

    // 5. 冷凝水管径推荐
    const condensatePipeStr = recommendCondensatePipe(totalKw);

    // 6. 冷媒铜管推荐
    const refrigPipes = recommendRefrigerantPipe(totalKw);

    // 渲染结果
    document.getElementById('cTotalHP').innerText = totalHP.toFixed(1);
    document.getElementById('cAirVolume').innerText = Math.round(airVol);
    document.getElementById('cAirDuctSize').innerText = airDuctStr;

    document.getElementById('cChilledFlow').innerText = chilledFlow.toFixed(2);
    document.getElementById('cChilledPipe').innerText = chilledPipeStr;

    document.getElementById('cCoolingFlow').innerText = coolingFlow.toFixed(2);
    document.getElementById('cCoolingPipe').innerText = coolingPipeStr;

    document.getElementById('cCondensatePipe').innerText = condensatePipeStr;

    document.getElementById('cRefrigGasPipe').innerText = refrigPipes.gas;
    document.getElementById('cRefrigLiquidPipe').innerText = refrigPipes.liquid;

    document.getElementById('coolingResult').style.display = 'block';
  }

  // --- 第二页：防排烟系统计算逻辑 (保持不变) ---
  function calcSmoke() {
    const area = parseFloat(document.getElementById('smokeArea').value) || 0;
    const height = parseFloat(document.getElementById('spaceHeight').value) || 3;
    const hasSprinkler = document.getElementById('hasSprinkler').value === 'yes';
    const ratio = (parseFloat(document.getElementById('openRatio').value) || 75) / 100;

    let totalSmokeVol = Math.max(area * 60, 15000);
    let qMax = hasSprinkler ? 12000 : 25000; 

    let minPortCount = Math.ceil(totalSmokeVol / qMax);
    if (minPortCount < 1) minPortCount = 1;

    let singlePortVol = totalSmokeVol / minPortCount;

    const portMode = document.getElementById('portSizeMode').value;
    let portW = 0, portH = 0, portEffArea = 0, portV = 0;
    let portDisplayStr = "";

    if (portMode === 'manual') {
      portW = (parseFloat(document.getElementById('portWidth').value) || 1000) / 1000;
      portH = (parseFloat(document.getElementById('portHeight').value) || 630) / 1000;
      let grossArea = portW * portH;
      portEffArea = grossArea * ratio;
      portV = singlePortVol / (3600 * portEffArea);
      portDisplayStr = `${(portW*1000)} × ${(portH*1000)} mm (外框 ${grossArea.toFixed(2)} m², 有效 ${portEffArea.toFixed(2)} m²)`;
    } else {
      let reqEffArea = singlePortVol / (3600 * 10);
      let reqGrossArea = reqEffArea / ratio;
      portDisplayStr = matchStandardSize(reqGrossArea);
      portV = 10.0;
    }

    const portBadge = document.getElementById('portVelocityBadge');
    if (portV <= 10.0) {
      portBadge.className = "status-badge status-pass";
      portBadge.innerText = "风速合格 (≤10m/s)";
    } else {
      portBadge.className = "status-badge status-warn";
      portBadge.innerText = "风速超标! (>10m/s)";
    }

    const ductMode = document.getElementById('ductSizeMode').value;
    let ductW = 0, ductH = 0, ductArea = 0, ductV = 0;
    let ductDisplayStr = "";

    if (ductMode === 'manual') {
      ductW = (parseFloat(document.getElementById('ductWidth').value) || 1250) / 1000;
      ductH = (parseFloat(document.getElementById('ductHeight').value) || 500) / 1000;
      ductArea = ductW * ductH;
      ductV = totalSmokeVol / (3600 * ductArea);
      ductDisplayStr = `${(ductW*1000)} × ${(ductH*1000)} mm (截面积 ${ductArea.toFixed(2)} m²)`;
    } else {
      let reqDuctArea = totalSmokeVol / (3600 * 15);
      ductDisplayStr = matchStandardDuct(reqDuctArea);
      ductV = 15.0;
    }

    const ductBadge = document.getElementById('ductVelocityBadge');
    if (ductV <= 20.0) {
      ductBadge.className = "status-badge status-pass";
      ductBadge.innerText = "风管风速合格 (≤20m/s)";
    } else {
      ductBadge.className = "status-badge status-warn";
      ductBadge.innerText = "风管风速偏高! (>20m/s)";
    }

    document.getElementById('totalSmokeVol').innerText = Math.round(totalSmokeVol);
    document.getElementById('qMaxLimit').innerText = qMax;
    document.getElementById('minPortCount').innerText = minPortCount;
    
    document.getElementById('portSizeDisplay').innerText = portDisplayStr;
    document.getElementById('portVelocity').innerText = portV.toFixed(2);

    document.getElementById('ductSizeDisplay').innerText = ductDisplayStr;
    document.getElementById('ductVelocity').innerText = ductV.toFixed(2);

    document.getElementById('smokeResult').style.display = 'block';
  }

  function matchStandardSize(area) {
    const standardSizes = [
      { name: "630 × 400 mm", area: 0.252 },
      { name: "630 × 630 mm", area: 0.396 },
      { name: "800 × 500 mm", area: 0.400 },
      { name: "800 × 630 mm", area: 0.504 },
      { name: "1000 × 500 mm", area: 0.500 },
      { name: "1000 × 630 mm", area: 0.630 },
      { name: "1250 × 630 mm", area: 0.787 },
      { name: "1500 × 800 mm", area: 1.200 }
    ];
    for (let size of standardSizes) {
      if (size.area >= area) {
        return `${size.name} (推荐尺寸)`;
      }
    }
    return `需定制大尺寸风口 (≥ ${area.toFixed(2)} m²)`;
  }

  function matchStandardDuct(area) {
    const standardDucts = [
      { name: "800 × 400 mm", area: 0.32 },
      { name: "1000 × 500 mm", area: 0.50 },
      { name: "1250 × 500 mm", area: 0.625 },
      { name: "1250 × 630 mm", area: 0.787 },
      { name: "1600 × 630 mm", area: 1.008 },
      { name: "2000 × 800 mm", area: 1.60 }
    ];
    for (let duct of standardDucts) {
      if (duct.area >= area) {
        return `${duct.name} (推荐尺寸)`;
      }
    }
    return `建议增大管道截面 (≥ ${area.toFixed(2)} m²)`;
  }
</script>

</body>
</html>
