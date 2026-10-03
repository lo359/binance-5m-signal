# binance-5m-signal
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>币安K线 · 5分钟事件合约信号</title>
  <style>
    :root {
      --bg: #08111f;
      --panel: #101d31;
      --panel2: #0b1729;
      --line: #263c5d;
      --text: #edf5ff;
      --muted: #90a7c8;
      --blue: #3baaff;
      --green: #16d39a;
      --red: #ff5679;
      --yellow: #ffc34d;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      color: var(--text);
      background: radial-gradient(circle at top, #193963 0%, var(--bg) 52%);
      font-family: -apple-system, BlinkMacSystemFont, "PingFang SC",
        "Microsoft YaHei", Arial, sans-serif;
      padding: 16px;
    }

    .app { max-width: 860px; margin: auto; }

    h1 { margin: 3px 0 7px; font-size: 24px; }

    .subtitle {
      margin: 0 0 15px;
      color: var(--muted);
      line-height: 1.6;
      font-size: 13px;
    }

    .card {
      margin-bottom: 12px;
      padding: 15px;
      border: 1px solid var(--line);
      border-radius: 16px;
      background: rgba(16, 29, 49, .95);
      box-shadow: 0 12px 36px rgba(0, 0, 0, .22);
    }

    .controls {
      display: grid;
      grid-template-columns: 1fr 1fr 1fr;
      gap: 10px;
    }

    label {
      display: block;
      margin-bottom: 6px;
      color: var(--muted);
      font-size: 12px;
    }

    select, input, button {
      width: 100%;
      height: 42px;
      padding: 0 11px;
      border: 1px solid #3a5478;
      border-radius: 10px;
      outline: none;
      color: var(--text);
      background: #091526;
      font-size: 14px;
    }

    button {
      margin-top: 18px;
      cursor: pointer;
      border: 0;
      color: #07111e;
      background: var(--blue);
      font-weight: 800;
    }

    button:hover { filter: brightness(1.08); }

    .manual {
      display: flex;
      align-items: center;
      gap: 9px;
      margin-top: 13px;
      padding-top: 12px;
      border-top: 1px solid var(--line);
    }

    .manual input[type="checkbox"] {
      width: 18px;
      height: 18px;
      accent-color: var(--blue);
    }

    .manual-text {
      flex: 1;
      color: #cfdef2;
      font-size: 13px;
    }

    .manual input[type="number"] { max-width: 185px; }

    .status {
      margin-top: 12px;
      color: var(--muted);
      font-size: 13px;
    }

    .signal-panel {
      padding: 24px 12px;
      border: 1px solid var(--line);
      border-radius: 14px;
      background: var(--panel2);
      text-align: center;
    }

    .signal-title {
      margin-bottom: 7px;
      color: var(--muted);
      font-size: 13px;
    }

    #signal {
      font-size: clamp(34px, 9vw, 50px);
      font-weight: 900;
      letter-spacing: 1px;
    }

    #confidence {
      margin-top: 10px;
      color: var(--muted);
      font-size: 14px;
    }

    .up { color: var(--green); }
    .down { color: var(--red); }
    .wait { color: var(--yellow); }

    .grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 10px;
    }

    .metric {
      padding: 12px;
      border: 1px solid #263d5c;
      border-radius: 12px;
      background: var(--panel2);
    }

    .metric-name {
      margin-bottom: 7px;
      color: var(--muted);
      font-size: 12px;
    }

    .metric-value {
      overflow-wrap: anywhere;
      font-size: 17px;
      font-weight: 750;
    }

    .reason-title { font-weight: 800; }

    ul {
      margin: 9px 0 0;
      padding-left: 21px;
      color: #cfdef2;
      font-size: 14px;
      line-height: 1.75;
    }

    .notice {
      color: #c3d3e8;
      font-size: 13px;
      line-height: 1.72;
    }

    .notice strong { color: var(--yellow); }

    @media (max-width: 570px) {
      .controls { grid-template-columns: 1fr; }
      button { margin-top: 0; }
      .manual { flex-wrap: wrap; }
      .manual input[type="number"] { max-width: 100%; }
    }
  </style>
</head>

<body>
  <main class="app">
    <h1>币安K线 · 5分钟事件合约信号</h1>
    <p class="subtitle">
      基于 Binance 实时1分钟K线计算短线涨跌倾向。此页面仅提供信号，不会自动交易。
    </p>

    <section class="card">
      <div class="controls">
        <div>
          <label for="symbol">交易对</label>
          <select id="symbol">
            <option value="ETHUSDT">ETHUSDT</option>
            <option value="BTCUSDT">BTCUSDT</option>
            <option value="SOLUSDT">SOLUSDT</option>
          </select>
        </div>

        <div>
          <label for="market">币安行情市场</label>
          <select id="market">
            <option value="futures">U本位永续合约</option>
            <option value="spot">现货</option>
          </select>
        </div>

        <button id="restart">重新连接</button>
      </div>

      <div class="manual">
        <input id="manualMode" type="checkbox" />
        <div class="manual-text">使用事件合约平台显示的本轮结算参考价</div>
        <input id="manualPrice" type="number" step="0.01" placeholder="例如：2674.50" disabled />
      </div>

      <div class="status" id="status">正在加载币安K线数据…</div>
    </section>

    <section class="card">
      <div class="signal-panel">
        <div class="signal-title">当前5分钟事件方向信号</div>
        <div id="signal" class="wait">加载中</div>
        <div id="confidence">置信度：--</div>
      </div>
    </section>

    <section class="card">
      <div class="grid">
        <div class="metric">
          <div class="metric-name">当前价格</div>
          <div class="metric-value" id="price">--</div>
        </div>
        <div class="metric">
          <div class="metric-name">本轮5分钟参考价</div>
          <div class="metric-value" id="reference">--</div>
        </div>
        <div class="metric">
          <div class="metric-name">距本轮结算</div>
          <div class="metric-value wait" id="countdown">--:--</div>
        </div>
        <div class="metric">
          <div class="metric-name">相对参考价</div>
          <div class="metric-value" id="eventMove">--</div>
        </div>
        <div class="metric">
          <div class="metric-name">EMA 9 / EMA 21</div>
          <div class="metric-value" id="ema">--</div>
        </div>
        <div class="metric">
          <div class="metric-name">布林带：上 / 中 / 下</div>
          <div class="metric-value" id="bb">--</div>
        </div>
        <div class="metric">
          <div class="metric-name">RSI(14)</div>
          <div class="metric-value" id="rsi">--</div>
        </div>
        <div class="metric">
          <div class="metric-name">近3分钟动量</div>
          <div class="metric-value" id="momentum">--</div>
        </div>
      </div>
    </section>

    <section class="card">
      <div class="reason-title">信号依据</div>
      <ul id="reasons"><li>等待行情数据。</li></ul>
    </section>

    <section class="card notice">
      <strong>使用建议：</strong>仅在“偏上涨”或“偏下跌”且置信度达到 <strong>70%</strong> 时再考虑参与。
      当剩余时间少于60秒、价格又紧贴参考价时，页面会强制提示观望。事件合约为二元结果，单笔可能损失全部投入金额。
    </section>
  </main>

  <script>
    let ws = null;
    let candles = [];
    let latestPrice = 0;
    let autoReference = 0;
    let reconnectTimer = null;

    const $ = id => document.getElementById(id);

    function formatPrice(value, digits = 2) {
      if (!Number.isFinite(value)) return "--";
      return value.[$undefined](token://undefined/undefined/undefined)("en-US", {
        minimumFractionDigits: digits,
        maximumFractionDigits: digits
      });
    }

    function apiEndpoints() {
      if ($("market").value === "spot") {
        return {
          rest: "https://api.binance.com/api/v3/klines",
          socket: "wss://stream.binance.com:9443/ws"
        };
      }

      return {
        rest: "https://fapi.binance.com/fapi/v1/klines",
        socket: "wss://fstream.binance.com/ws"
      };
    }

    function sma(values, period) {
      if (values.length < period) return null;
      return values.slice(-period).reduce((sum, x) => sum + x, 0) / period;
    }

    function standardDeviation(values) {
      const average = values.reduce((sum, x) => sum + x, 0) / values.length;
      return Math.sqrt(
        values.reduce((sum, x) => sum + (x - average) ** 2, 0) / values.length
      );
    }

    function ema(values, period) {
      if (values.length < period) return null;

      const multiplier = 2 / (period + 1);
      let result = values.slice(0, period).reduce((sum, x) => sum + x, 0) / period;

      for (let i = period; i < values.length; i++) {
        result = values[i] * multiplier + result * (1 - multiplier);
      }
      return result;
    }

    function rsi(values, period = 14) {
      if (values.length < period + 1) return null;

      let gains = 0;
      let losses = 0;

      for (let i = values.length - period; i < values.length; i++) {
        const difference = values[i] - values[i - 1];
        if (difference >= 0) gains += difference;
        else losses += Math.abs(difference);
      }

      if (losses === 0) return 100;

      const rs = (gains / period) / (losses / period);
      return 100 - 100 / (1 + rs);
    }

    function secondsUntilFiveMinuteClose() {
      const period = 5 * 60 * 1000;
      return Math.ceil((period - (Date.now() % period)) / 1000);
    }

    function activeReferencePrice() {
      if ($("manualMode").checked) {
        const manual = Number($("manualPrice").value);
        if (manual > 0) return manual;
      }
      return autoReference;
    }

    async function loadInitialData() {
      const symbol = $("symbol").value;
      const { rest } = apiEndpoints();

      const [oneMinuteResponse, fiveMinuteResponse] = await Promise.all([
        fetch(`${rest}?symbol=${symbol}&interval=1m&limit=120`),
        fetch(`${rest}?symbol=${symbol}&interval=5m&limit=2`)
      ]);

      if (!oneMinuteResponse.ok || !fiveMinuteResponse.ok) {
        throw new Error("币安接口请求失败");
      }

      const oneMinuteData = await oneMinuteResponse.json();
      const fiveMinuteData = await fiveMinuteResponse.json();

      candles = oneMinuteData.map(row => ({
        openTime: Number(row[0]),
        open: Number(row[1]),
        close: Number(row[4])
      }));

      latestPrice = candles[candles.length - 1].close;

      // 当前未完成的5分钟K线开盘价，作为事件周期参考价代理
      autoReference = Number(fiveMinuteData[fiveMinuteData.length - 1][1]);
    }

    function connectWebSocket() {
      if (ws) {
        ws.onclose = null;
        ws.close();
      }

      clearTimeout(reconnectTimer);

      const symbol = $("symbol").value.toLowerCase();
      const { socket } = apiEndpoints();

      ws = new WebSocket(`${socket}/${symbol}@kline_1m`);

      ws.onopen = () => {
        const marketName = $("market").value === "spot" ? "Binance现货" : "Binance U本位永续";
        $("status").textContent = `已连接：${marketName} · ${$("symbol").value} · 1分钟K线`;
      };

      ws.onmessage = event => {
        const message = JSON.parse(event.data);
        const kline = message.k;
        if (!kline) return;

        const incoming = {
          openTime: Number(kline.t),
          open: Number(kline.o),
          close: Number(kline.c)
        };

        latestPrice = incoming.close;

        const last = candles[candles.length - 1];
        if (last && last.openTime === incoming.openTime) {
          candles[candles.length - 1] = incoming;
        } else {
          candles.push(incoming);
          if (candles.length > 150) candles.shift();
        }

        // 进入新的5分钟周期时，记录该周期1分钟K线的开盘价
        if (incoming.openTime % 300000 === 0) {
          autoReference = incoming.open;
        }

        render();
      };

      ws.onerror = () => {
        $("status").textContent = "WebSocket连接异常，请重新连接或切换现货/合约。";
      };

      ws.onclose = () => {
        $("status").textContent = "连接断开，3秒后自动重连…";
        reconnectTimer = setTimeout(connectWebSocket, 3000);
      };
    }

    function render() {
      const reference = activeReferencePrice();
      if (candles.length < 30 || !latestPrice || !reference) return;

      const closes = candles.map(item => item.close);
      const price = latestPrice;
      const ema9 = ema(closes, 9);
      const ema21 = ema(closes, 21);
      const basis = sma(closes, 20);
      const deviation = standardDeviation(closes.slice(-20));
      const upper = basis + deviation * 2;
      const lower = basis - deviation * 2;
      const currentRsi = rsi(closes, 14);
      const momentum3 = ((price - closes[closes.length - 4]) / closes[closes.length - 4]) * 100;
      const eventMove = ((price - reference) / reference) * 100;
      const secondsLeft = secondsUntilFiveMinuteClose();

      $("price").textContent = formatPrice(price);
      $("reference").textContent = formatPrice(reference);
      $("countdown").textContent =
        `${String(Math.floor(secondsLeft / 60)).padStart(2, "0")}:${String(secondsLeft % 60).padStart(2, "0")}`;

      $("eventMove").textContent = `${eventMove >= 0 ? "+" : ""}${eventMove.toFixed(3)}%`;
      $("eventMove").className = `metric-value ${eventMove > 0 ? "up" : eventMove < 0 ? "down" : "wait"}`;

      $("ema").textContent = `${formatPrice(ema9)} / ${formatPrice(ema21)}`;
      $("bb").textContent = `${formatPrice(upper)} / ${formatPrice(basis)} / ${formatPrice(lower)}`;
      $("rsi").textContent = currentRsi.toFixed(1);

      $("momentum").textContent = `${momentum3 >= 0 ? "+" : ""}${momentum3.toFixed(3)}%`;
      $("momentum").className = `metric-value ${momentum3 > 0 ? "up" : momentum3 < 0 ? "down" : "wait"}`;

      calculateSignal({
        price, reference, ema9, ema21, basis, upper, lower,
        currentRsi, momentum3, eventMove, secondsLeft
      });
    }

    function calculateSignal(data) {
      let upScore = 0;
      let downScore = 0;
      const reasons = [];

      const {
        price, reference, ema9, ema21, basis, upper, lower,
        currentRsi, momentum3, eventMove, secondsLeft
      } = data;

      if (price > reference) {
        upScore += 2;
        reasons.push(`当前价格高于本轮参考价 ${formatPrice(reference)}，结算暂偏“上涨”。`);
      } else {
        downScore += 2;
        reasons.push(`当前价格低于本轮参考价 ${formatPrice(reference)}，结算暂偏“下跌”。`);
      }

      if (ema9 > ema21) {
        upScore += 2;
        reasons.push("EMA9 高于 EMA21，1分钟趋势偏多。");
      } else {
        downScore += 2;
        reasons.push("EMA9 低于 EMA21，1分钟趋势偏空。");
      }

      if (price > basis && price < upper) {
        upScore += 1;
        reasons.push("价格在布林中轨上方，反弹结构仍在。");
      } else if (price < basis && price > lower) {
        downScore += 1;
        reasons.push("价格在布林中轨下方，短线结构偏弱。");
      } else if (price >= upper) {
        downScore += 1;
        reasons.push("价格接近布林上轨，存在冲高回落风险。");
      } else if (price <= lower) {
        upScore += 1;
        reasons.push("价格接近布林下轨，存在超跌反弹风险。");
      }

      if (currentRsi >= 52 && currentRsi <= 72) {
        upScore += 1;
        reasons.push(`RSI ${currentRsi.toFixed(1)}，多头动能尚可。`);
      } else if (currentRsi <= 48 && currentRsi >= 28) {
        downScore += 1;
        reasons.push(`RSI ${currentRsi.toFixed(1)}，空头动能偏强。`);
      } else if (currentRsi > 75) {
        downScore += 1;
        reasons.push(`RSI ${currentRsi.toFixed(1)} 偏高，追涨风险增加。`);
      } else if (currentRsi < 25) {
        upScore += 1;
        reasons.push(`RSI ${currentRsi.toFixed(1)} 偏低，追跌风险增加。`);
      }

      if (momentum3 > 0.035) {
        upScore += 1;
        reasons.push("近3分钟价格动量为正。");
      } else if (momentum3 < -0.035) {
        downScore += 1;
        reasons.push("近3分钟价格动量为负。");
      } else {
        reasons.push("近3分钟动量不明显，市场偏震荡。");
      }

      let signal = "观望";
      let signalClass = "wait";
      let confidence;

      // 临近结算又贴近参考价时，避免“抛硬币”行情
      if (secondsLeft <= 60 && Math.abs(eventMove) < 0.025) {
        confidence = 50;
        reasons.unshift("距结算不足60秒且价格贴近参考价，随机性较大：强制观望。");
      } else if (upScore - downScore >= 2 && upScore >= 5) {
        signal = "偏上涨";
        signalClass = "up";
        confidence = Math.min(88, 55 + upScore * 4 + (upScore - downScore) * 3);
      } else if (downScore - upScore >= 2 && downScore >= 5) {
        signal = "偏下跌";
        signalClass = "down";
        confidence = Math.min(88, 55 + downScore * 4 + (downScore - upScore) * 3);
      } else {
        confidence = Math.min(65, 45 + Math.max(upScore, downScore) * 3);
        reasons.unshift("多空指标没有形成一致信号，优先观望。");
      }

      $("signal").textContent = signal;
      $("signal").className = signalClass;
      $("confidence").textContent =
        `置信度：${Math.round(confidence)}% ｜ 多头 ${upScore} 分 / 空头 ${downScore} 分`;

      $("reasons").innerHTML = reasons
        .slice(0, 6)
        .map(text => `<li>${text}</li>`)
        .join("");
    }

    async function start() {
      $("signal").textContent = "加载中";
      $("signal").className = "wait";
      $("status").textContent = "正在获取历史K线及本轮5分钟参考价…";

      try {
        await loadInitialData();
        connectWebSocket();
        render();
      } catch (error) {
        console.error(error);
        $("status").textContent =
          "加载失败：请检查网络；如当前地区无法访问币安接口，可切换网络后重试。";
      }
    }

    $("restart").addEventListener("click", start);

    $("manualMode").addEventListener("change", event => {
      $("manualPrice").disabled = !event.target.checked;
      render();
    });

    $("manualPrice").addEventListener("input", render);
    setInterval(render, 1000);

    start();
  </script>
</body>
</html>
