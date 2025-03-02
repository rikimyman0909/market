# market
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>CoinMarketCap Enhanced with Live Global Data</title>
  <!-- Include Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css" />
  <style>
    /* Global Styles */
    html { scroll-behavior: smooth; }
    :root {
      --primary-bg: rgb(20, 22, 24);
      --secondary-bg: #2a3239;
      --tertiary-bg: #454547;
      --text-primary: #ffffff;
      --text-secondary: #f6db79;
      --accent-green: #2ed573;
      --accent-red: #ff4757;
      --accent-blue: #434546;
      --border-color: #222222;
      --card-shadow: 0 4px 6px rgba(151, 155, 154, 0.1);
      --flash-duration: 800ms;
      --accent-theme: #71777f;
    }
    body.light-theme {
      --primary-bg: #e2dfdf;
      --secondary-bg: #f0f0f0;
      --tertiary-bg: #e0e0e0;
      --text-primary: #000000;
      --text-secondary: #333333;
      --accent-blue: #c8f351;
    }
    * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Inter', sans-serif; }
    body { background-color: var(--primary-bg); color: var(--text-primary); }

    /* Header Styles */
    header { padding: 0.5rem; border-bottom: 1px solid var(--border-color); }
    .header-container {
      max-width: 1700px; margin: 0 auto; display: flex;
      justify-content: space-between; align-items: center;
    }
    .header-left, .header-center, .header-right { display: flex; align-items: center; gap: 1rem; }
    .header-left .brand { display: flex; align-items: center; gap: 0.5rem; }
    .brand-logo { width: 45px; height: 45px; border-radius: 90px }
    .header-center nav a {
      margin: 0 0.75rem; color: var(--text-secondary); text-decoration: none;
      transition: color 0.3s; font-weight: bold;
    }
    .header-center nav a:hover { color: var(--accent-blue); }
    .button {
      background: var(--accent-theme); color: var(--text-secondary); padding: 0.5rem 1.5rem;
      border-radius: 20px; text-decoration: none; font-weight: bold;
      transition: background 0.3s, transform 0.3s; border: 2px solid transparent;
    }
    .button:hover { background: #2a2d30; transform: translateY(-3px); border-color: #ffffff; }
    .search-bar {
      padding: 0.5rem 1rem; border-radius: 20px; background: var(--tertiary-bg);
      border: none; color: var(--text-primary); outline: none;
    }
    .header-right { position: relative; gap: 1rem; }

    /* Settings Button & Menu */
    .settings-btn {
      background: transparent; border: none; color: var(--text-secondary);
      font-size: 1.5rem; cursor: pointer; transition: transform 0.3s, color 0.3s;
    }
    .settings-btn:hover { color: var(--accent-blue); }
    .settings-btn.active { transform: rotate(45deg); }
    .settings-menu {
      position: absolute; top: 120%; right: 0; background: var(--tertiary-bg);
      border: 1px solid var(--border-color); box-shadow: var(--card-shadow);
      border-radius: 5px; padding: 0.5rem; display: none; flex-direction: column;
      z-index: 10; animation: fadeIn 0.3s ease-out; min-width: 120px;
    }
    .settings-menu button {
      background: transparent; border: none; color: var(--text-secondary);
      padding: 0.5rem 1rem; text-align: left; cursor: pointer; font-size: 1rem;
      transition: background 0.3s;
    }
    .settings-menu button:hover { background: var(--secondary-bg); color: var(--accent-blue); }
    
    /* Theme Switch (Toggle) Styles */
    .theme-switch {
      display: inline-block;
      position: relative;
      width: 80px;
      height: 40px;
      margin-left: 1rem;
    }
    .theme-switch input {
      position: absolute;
      opacity: 0;
      width: 0;
      height: 0;
    }
    .switch-track {
      display: flex;
      align-items: center;
      justify-content: space-between;
      background-color: #2a3239;
      border-radius: 34px;
      width: 100%;
      height: 100%;
      padding: 12px;
      transition: background-color 0.4s;
      font-size: 0.9rem;
      color: #4b4f4d;
      position: relative;
    }
    .switch-option {
      z-index: 1;
      display: flex;
      align-items: center;
      gap: 5px;
    }
    /* Initially, assume dark mode is active */
    .option-dark { opacity: 1; }
    .option-light { opacity: 0.4; }
    .switch-thumb {
      position: absolute;
      top: 3px;
      left: 3px;
      width: 34px;
      height: 34px;
      background-color: #ffffff;
      border-radius: 50%;
      transition: transform 0.5s;
    }
    /* When checked (light theme active) */
    .theme-switch input:checked + .switch-track {
      background-color: var(--accent-theme);
    }
    .theme-switch input:checked + .switch-track .switch-thumb {
      transform: translateX(40px);
    }
    .theme-switch input:checked + .switch-track .option-dark {
      opacity: 0.4;
    }
    .theme-switch input:checked + .switch-track .option-light {
      opacity: 1;
    }
    /* Main Container */
    .container { max-width: 1200px; margin: 0 auto; padding: 1.5rem; }

    /* Global Stats */
    .market-stats {
      display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 1.5rem; margin: 2rem 0;
    }
    .stat-card {
      padding: 1.5rem; border-radius: 12px;
      text-align: center;
    }
    .stat-card h3 { color: var(--text-secondary); margin-bottom: 0.5rem; }
    .stat-value { font-size: 1.2rem; font-weight: 600; }
    
    /* Loading Animation for Stat Cards */
    .stat-loading {
      display: flex;
      justify-content: center;
      gap: 5px;
    }
    .stat-loading .dot {
      width: 6px;
      height: 6px;
      background: var(--text-primary);
      border-radius: 50%;
      animation: blink 1.4s infinite both;
    }
    .stat-loading .dot:nth-child(2) {
      animation-delay: 0.2s;
    }
    .stat-loading .dot:nth-child(3) {
      animation-delay: 0.4s;
    }
    @keyframes blink {
      0%, 80%, 100% { opacity: 0.2; }
      40% { opacity: 1; }
    }
    
    .fear-greed {
      background: linear-gradient(135deg, #cfdc1b 0%, #e74c3c 100%);
      -webkit-background-clip: text; -webkit-text-fill-color: transparent;
    }

    /* Trending Cards */
    .trending { margin: 2rem 0; }
    .trending-container {
      display: grid; grid-template-columns: repeat(auto-fit, minmax(190px, 1fr));
      gap: 1.5rem;
    }
    .trending-card {
      background: var(--secondary-bg); padding: 0.8rem; border-radius: 40px;
      box-shadow: var(--card-shadow); cursor: pointer; transition: transform 0.2s;
      position: relative;
    }
    .trending-card:hover { transform: translateY(-5px); }
    .trending-header { display: flex; align-items: center; gap: 0.5rem; margin-bottom: 0.5rem; }
    .trending-header img { width: 32px; height: 32px; border-radius: 50%; }
    .trending-details { font-size: 0.9rem; margin-bottom: 0.5rem; }
    .trending-chart canvas { width: 200px !important; height: 80px !important; }

    /* ================================
       UPDATED COIN DETAIL WINDOW UI
       ================================ */
    .coin-details-section {
      display: none;
      background: rgb(30, 32, 34);
      border-radius: 12px;
      padding: 1.5rem;
      margin: 2rem 0;
      position: relative;
      box-shadow: 0 8px 16px rgba(0,0,0,0.25);
      animation: fadeIn 0.4s ease-out;
    }
    .close-details {
      position: absolute;
      top: 1rem; right: 1rem;
      cursor: pointer;
      font-size: 1.5rem;
      color: var(--text-secondary);
    }
    /* Settings button & panel inside the detail window */
    #detailSettingsBtn {
      position: absolute; top: 0.3rem; left: 0.3rem;
      transition: transform 0.3s;
      background: transparent;
      border: none;
      color: var(--text-secondary);
      font-size: 1.5rem;
      cursor: pointer;
    }
    #detailSettingsBtn.rotated { transform: rotate(45deg); }
    #detailSettingsPanel {
      display: none;
      position: absolute; top: 3rem; left: 1rem;
      background: var(--tertiary-bg);
      border: 1px solid var(--border-color);
      padding: 1rem;
      border-radius: 8px;
      z-index: 20;
      box-shadow: var(--card-shadow);
    }
    #detailSettingsPanel h3 { margin-bottom: 0.5rem; font-size: 1.1rem; }
    
    /* Header: Coin Info & Buyers/Sellers Widget */
    .coin-details-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      border-bottom: 0px solid var(--border-color);
      padding-bottom: 1rem;
      margin-bottom: 1rem;
    }
    .price-info {
      display: flex;
      align-items: center;
    }
    .price-info img {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      margin-right: 1rem;
      border: 2px solid var(--accent-blue);
    }
    .price-info h2 {
      font-size: 1.5rem;
      margin: 0;
    }
    .current-price {
      font-size: 1.2rem;
      margin-top: 0.3rem;
      color: var(--accent-green);
    }
    /* Buyers/Sellers Widget – UPDATED with Lighter, Captivating Colors */
    .buyers-sellers-widget {
      width: 200px;
      border-radius: 34px;
      overflow: hidden;
      background: #f0f0f0;
      box-shadow: inset 0 2px 4px rgba(249, 179, 0, 0.15);
      font-size: 0.8rem;
      position: absolute;
      left: 40%;
      margin-top: 10px;
    }
    .ratio-container {
      display: flex;
      width: 100%;
      height: 30px;
    }
    .ratio-buyers,
    .ratio-sellers {
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
      transition: width 0.4s ease;
    }
    .ratio-buyers {
      background: linear-gradient(45deg, #d1f7c4, #a3d9a5);
      color: #2c662d;
    }
    .ratio-sellers {
      background: linear-gradient(45deg, #fbd3d3, #f7c6c7);
      color: #a94442;
    }
    .ratio-buyers span,
    .ratio-sellers span {
      width: 100%;
      text-align: center;
    }
    
    /* Coin Stats Box */
    .coin-stats-box {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
      gap: 1rem;
      background: var(--tertiary-bg);
      border-radius: 8px;
      padding: 1rem;
      margin-bottom: 1.5rem;
    }
    .coin-stats-box .stat-item { text-align: center; }
    .coin-stats-box .stat-item span {
      display: block;
      font-size: 0.8rem;
      color: var(--text-secondary);
    }
    .coin-stats-box .stat-item div {
      font-size: 1rem;
      font-weight: bold;
      margin-top: 5px;
    }
    
    /* Timeframe Selector and Chart */
    .timeframe-selector {
      display: flex;
      gap: 1rem;
      margin-bottom: 1.5rem;
      justify-content: center;
    }
    .timeframe-btn {
      background: var(--tertiary-bg);
      color: var(--text-secondary);
      border: none;
      padding: 0.5rem 1rem;
      border-radius: 8px;
      cursor: pointer;
      transition: background 0.3s;
    }
    .timeframe-btn.active {
      background: var(--accent-blue);
      color: #fff;
    }
    .detail-chart-container {
      height: 200px;
      position: relative;
    }
    #detailChart {
      width: 100%;
      height: 100%;
    }
    .expanding-logo {
      width: 0;
      opacity: 0;
      animation: expandLogo 1.2s backwards;
      margin-bottom: 20px;
    }
    @keyframes expandLogo {
      0% { width: 0; opacity: 0; }
      100% { width: 150px; opacity: 1; }
    }
    .spinner {
      border: 4px solid rgba(255, 255, 255, 0.3);
      border-top: 4px solid #fff;
      border-radius: 50%;
      width: 40px;
      height: 40px;
      animation: spin 1s linear infinite;
      margin-top: 10px;
    }
    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
    .cube {
      width: 1080px;
      height: 95px;
      background-color: rgba(128, 128, 128, 0.5); /* Gray background with transparency */
      border-radius: 15px;
      position: absolute; /* Allow positioning */
      top: 20%; /* Adjust this value to position it where you want */
      left: 18.5%; /* Adjust this value to position it where you want */
      z-index: -1; /* Position the cube below the text */
    }
    .small-loader {
      display: flex;
      justify-content: center;
      gap: 5px;
      margin-top: 10px;
    }
    .small-loader .dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: #fff;
      animation: blink 1.4s infinite both;
    }
    .small-loader .dot:nth-child(2) {
      animation-delay: 0.2s;
    }
    .small-loader .dot:nth-child(3) {
      animation-delay: 0.1s;
    }
    .section-heading {
      text-align: left;       /* Changed from centered to left-aligned */
      margin-bottom: 1rem;
      font-size: 1.5rem;      /* Smaller font size for a cleaner look */
      font-weight: bold;
    }
    @keyframes blink {
      0% { opacity: 0.2; }
      40% { opacity: 1; }
      80%, 100% { opacity: 0.2; }
    }
    
    @keyframes shake {
      0% { transform: translate(1px, 1px) rotate(0deg); }
      10% { transform: translate(-1px, -2px) rotate(-1deg); }
      20% { transform: translate(-3px, 0px) rotate(1deg); }
      30% { transform: translate(3px, 2px) rotate(0deg); }
      40% { transform: translate(1px, -1px) rotate(1deg); }
      50% { transform: translate(-1px, 2px) rotate(-1deg); }
      60% { transform: translate(-3px, 1px) rotate(0deg); }
      70% { transform: translate(3px, 1px) rotate(-1deg); }
      80% { transform: translate(-1px, -1px) rotate(1deg); }
      90% { transform: translate(1px, 2px) rotate(0deg); }
      100% { transform: translate(1px, -2px) rotate(-1deg); }
    }
    .shaking { animation: shake 0.5s infinite; }
    .plus-button {
      position: absolute;
      top: 5px;
      right: 5px;
      background: #fff;
      border-radius: 50%;
      width: 20px;
      height: 20px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 16px;
      cursor: pointer;
      display: none;
      color: black;
      border: 1px solid #ccc;
    }
    .card-container { position: relative; }
    .trending-card {
      background: rgb(45, 47, 50) ;
      color: rgb(230, 230, 230);
      padding: 10px;
      width: 209px;
      border-radius: px;
      margin: 10px;
    }
    /* Fee Tracker Section Styles */
    #fee-tracker {
      width: fit-content;
      align-items: center;
      font-size: 12px;
      font-weight: boldrem;
      font-style: italic;
      color: var(--text-primary);
      border: 1px solid var(--text-primary);
      border-radius:345px;
      padding: 0.5rem 1.5rem;
      margin: 1rem 2rem;
    }
    #fee-tracker span {
      margin-right: 10px;
    }
    .separator {
      margin: 0 10px;
      color: var(--text-secondary);
    }
    
    /* Flash Update Animations */
    .flash-green { animation: flashGreen var(--flash-duration); }
    .flash-red { animation: flashRed var(--flash-duration); }
    @keyframes flashGreen { 0% { background-color: transparent; } 50% { background-color: #2ed57333; } 100% { background-color: transparent; } }
    @keyframes flashRed { 0% { background-color: transparent; } 50% { background-color: #ff475733; } 100% { background-color: transparent; } }
    @keyframes fadeIn { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }
  </style>
</head>
<body>
  <div id="preloader" style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; backdrop-filter: blur(10px); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 9999;">
    <img src="logo.png" alt="Logo" class="expanding-logo">
    <div class="spinner"></div>
    <div class="small-loader"></div>
  </div>
  <header>
    <div class="header-container">
      <div class="header-left">
        <div class="brand">
          <img src="logo.png" alt="Platone Logo" class="brand-logo" />
          <div style="font-weight:bold">Platone </div>
        </div>
      </div>
      <div class="header-center">
        <nav>
          <a href="#trending" class="button">Cryptocurrencies</a>
          <a href="#" class="button">DeepCharts</a>
          <a href="#" class="button">ETF</a>
          <a href="#" class="button">NEWS</a>
          <a href="#" class="button">Products</a>
        </nav>
      </div>
      <div class="header-right">
        <input type="text" placeholder="Search..." class="search-bar" />
        <a href="#" class="button">Log In</a>
        <!-- Theme Toggle Switch with Moon/Sun icons -->
        <label class="theme-switch">
          <input type="checkbox" id="themeSwitch">
          <div class="switch-track">
            <span class="switch-option option-dark">
              <i class="fas fa-moon"></i>
            </span>
            <span class="switch-option option-light">
              <i class="fas fa-sun"></i>
            </span>
            <div class="switch-thumb"></div>
          </div>
        </label>
        <button class="settings-btn" id="settingsBtn"><i class="fas fa-cog"></i></button>
        <div class="settings-menu" id="settingsMenu">
          <button id="profileBtn">Profile</button>
          <button id="resetBtn">Reset</button>
        </div>
      </div>
    </div>
  </header>
  <div id="fee-tracker">
    <span>⛽ BTC: $<span id="fee-btc">--</span></span>
    <span class="separator">|</span>
    <span>ETH Gas: $<span id="fee-eth">--</span></span>
  </div>
  <div class="cube"></div>
  <main class="container">
    <!-- Global Stats -->
    <div class="market-stats">
      <div class="stat-card" id="global-marketcap">
        <h3>Global Market Cap</h3>
        <div class="stat-loading">
          <span class="dot"></span>
          <span class="dot"></span>
          <span class="dot"></span>
        </div>
        <p class="stat-value" style="display:none;"></p>
      </div>
      <div class="stat-card" id="global-volume">
        <h3>24h Volume</h3>
        <div class="stat-loading">
          <span class="dot"></span>
          <span class="dot"></span>
          <span class="dot"></span>
        </div>
        <p class="stat-value" style="display:none;"></p>
      </div>
      <div class="stat-card" id="btc-dominance">
        <h3>Bitcoin Dominance</h3>
        <div class="stat-loading">
          <span class="dot"></span>
          <span class="dot"></span>
          <span class="dot"></span>
        </div>
        <p class="stat-value" style="display:none;"></p>
      </div>
      <div class="stat-card" id="fear-greed">
        <h3>Fear & Greed Index</h3>
        <div class="stat-loading">
          <span class="dot"></span>
          <span class="dot"></span>
          <span class="dot"></span>
        </div>
        <p class="stat-value fear-greed" style="display:none;"></p>
      </div>
      
    </div>
    <!-- Fee Tracker Section (Left side of Main Body) -->
    <!-- (Additional content can be added here) -->
    <!-- ========== UPDATED COIN DETAIL WINDOW ========= -->
    <div class="coin-details-section" id="coinDetails">
      <span class="close-details" onclick="closeDetails()">×</span>
      <!-- Detail Settings -->
      <button id="detailSettingsBtn" class="settings-btn"><i class="fas fa-cog"></i></button>
      <div id="detailSettingsPanel">
        <h3>Customized</h3>
        <label for="graphColorPicker">Graph Color:</label>
        <input type="color" id="graphColorPicker" value="#2ed573" />
      </div>
      <!-- Header: Coin Info & Buyers/Sellers Widget -->
      <div class="coin-details-header">
        <div class="price-info">
          <img id="detailCoinImg" src="https://s2.coinmarketcap.com/static/img/coins/64x64/1.png" alt="Coin Icon" />
          <div>
            <h2 id="detailCoinName">Bitcoin</h2>
            <div class="current-price" id="detailPrice">$--</div>
          </div>
        </div>
        <!-- Buyers/Sellers Widget -->
        <div class="buyers-sellers-widget">
          <div class="ratio-container">
            <div class="ratio-buyers" id="buyersBar">
              <span id="buyersPercentText">Buyers: --%</span>
            </div>
            <div class="ratio-sellers" id="sellersBar">
              <span id="sellersPercentText">Sellers: --%</span>
            </div>
          </div>
        </div>
      </div>
      <!-- Coin Stats Box -->
      <div class="coin-stats-box">
        <div class="stat-item">
          <span>Market Cap</span>
          <div id="detailMarketCap">Loading...</div>
        </div>
        <div class="stat-item">
          <span>Volume (24h)</span>
          <div id="detailVolume">Loading...</div>
        </div>
        <div class="stat-item">
          <span>FDV</span>
          <div id="detailFDV">Loading...</div>
        </div>
        <div class="stat-item">
          <span>Vol/Mkt Cap (24h)</span>
          <div id="detailVolMktCap">Loading...</div>
        </div>
        <div class="stat-item">
          <span>Total supply</span>
          <div id="detailTotalSupply">Loading...</div>
        </div>
        <div class="stat-item">
          <span>Max. supply</span>
          <div id="detailMaxSupply">Loading...</div>
        </div>
      </div>
      <!-- Timeframe Selector -->
      <div class="timeframe-selector">
        <button class="timeframe-btn active" data-hours="24">24H</button>
        <button class="timeframe-btn" data-hours="168">7D</button>
        <button class="timeframe-btn" data-hours="720">1M</button>
        <button class="timeframe-btn" data-hours="2160">3M</button>
      </div>
      <!-- Chart Container -->
      <div class="detail-chart-container">
        <canvas id="detailChart"></canvas>
      </div>
      <!-- NERD Button -->
      <button class="button" id="nerdBtn" data-symbol="">NERD</button>
    </div>

    <!-- Trending Coins Section -->
    <div class="trending" id="trending">
      <h2 style="section-heading">Crypto</h2>
      <div class="trending-container" id="trending-container">
        <!-- Trending coin cards will be injected here -->
      </div>
    </div>
  </main>
  <script>
    /* Global Variables */
    const TIME_LIMIT = 168;
    const circulatingSupply = {
      Bitcoin: 19600000, Ethereum: 120200000, XRP: 54045766079,
      Tether: 83016246102, BNB: 155852912, Solana: 428116287,
      'USD Coin': 25057892805, Dogecoin: 142897336384, Cardano: 35045020830,
      TRON: 88737645165, Sui: 1000000000, Aptos: 1000000000,
      Avalanche: 720000000, Chainlink: 1000000000, Stellar: 50001806812,
      Litecoin: 84000000, Toncoin: 5000000000, ShibaInu: 549063278876301,
      Hedera: 50000000000, Bittensor: 1000000000, Polkadot: 1100000000,
      Mantra: 5000000000, Dai: 5340000000, Uniswap: 753766667,
      Monero: 1038218400, Nearprotocol: 1000000000, Pepe: 420690000000000,
      AAVE: 14720000
    };

    const trendingCoins = [
      { name: 'Bitcoin', id: 1, symbol: 'BTCUSDT' },
      { name: 'Ethereum', id: 1027, symbol: 'ETHUSDT' },
      { name: 'XRP', id: 52, symbol: 'XRPUSDT' },
      { name: 'BNB', id: 1839, symbol: 'BNBUSDT' },
      { name: 'Solana', id: 5426, symbol: 'SOLUSDT' },
      { name: 'Cardano', id: 2010, symbol: 'ADAUSDT' },
      { name: 'Dogecoin', id: 74, symbol: 'DOGEUSDT' },
      { name: 'Sui', id: 20947, symbol: 'SUIUSDT' },
      { name: 'Aptos', id: 21794, symbol: 'APTUSDT' },
      { name: 'Avalanche', id: 5805, symbol: 'AVAXUSDT' },
      { name: 'Tron', id: 1958, symbol: 'TRXUSDT' },
      { name: 'Chainlink', id: 1975, symbol: 'LINKUSDT' },
      { name: 'Stellar', id: 512, symbol: 'XLMUSDT' },
      { name: 'Litecoin', id: 2, symbol: 'LTCUSDT' },
      { name: 'Toncoin', id: 11419, symbol: 'TONUSDT' },
      { name: 'ShibaInu', id: 5994, symbol: 'SHIBUSDT' },
      { name: 'Hedera', id: 4642, symbol: 'HBARUSDT' },
      { name: 'Bittensor', id: 22974, symbol: 'TAOUSDT' },
      { name: 'Polkadot', id: 6636, symbol: 'DOTUSDT' },
      { name: 'Mantra', id: 6536, symbol: 'OMUSDT' },
      { name: 'Dai', id: 4943, symbol: 'DAIUSDT' },
      { name: 'Uniswap', id: 7083, symbol: 'UNIUSDT' },
      { name: 'Monero', id: 328, symbol: 'XMRUSDT' },
      { name: 'Nearprotocol', id: 6535, symbol: 'NEARUSDT' },
      { name: 'Pepe', id: 24478, symbol: 'PEPEUSDT' },
      { name: 'AAVE', id: 7278, symbol: 'AAVEUSDT' }
    ];

    const trendingContainer = document.getElementById('trending-container');
    const trendingCardsMap = {};
    let currentDetailCoin = null;
    let detailChartInstance = null;
    let currentTimeframe = 168; // default 7 days
    let detailChartColor = "#2ed573";
    let currentCoinPrice = 0;

    // Initialize Trending Coin Cards
    trendingCoins.forEach((coin, index) => {
      const card = document.createElement('div');
      let longPressTriggered = false;
      let pressTimer;

      card.addEventListener('mousedown', () => {
        longPressTriggered = false;
        pressTimer = setTimeout(() => {
          longPressTriggered = true;
          card.classList.add('shaking');
          plus.style.display = 'flex';
        }, 300);
      });

      card.addEventListener('mouseup', () => {
        clearTimeout(pressTimer);
        setTimeout(() => { longPressTriggered = false; }, 50);
      });

      card.className = 'trending-card';
      card.setAttribute('data-coin', coin.name);
      card.innerHTML = `
        <div class="trending-header">
          <img src="https://s2.coinmarketcap.com/static/img/coins/64x64/${coin.id}.png" alt="${coin.name}">
          <span>${coin.name}</span>
        </div>
        <div class="trending-details">
          <div>Price: <strong class="trending-price" data-prev="0">--</strong></div>
          <div>7d: <strong class="trending-7d" data-prev="0">--</strong></div>
          <div>Market Cap: <strong class="trending-marketcap" data-prev="0">--</strong></div>
        </div>
        <div class="trending-chart">
          <canvas id="trending-chart-${index}" width="200" height="100"></canvas>
        </div>
      `;
      trendingContainer.appendChild(card);
      trendingCardsMap[coin.name] = card;
      card.addEventListener('click', function(e) {
        if (longPressTriggered) {
          e.preventDefault();
          return;
        }
        openDetails(coin.name);
      });
      
      const ctx = document.getElementById(`trending-chart-${index}`).getContext('2d');
      fetchHistoricalData(coin.symbol, 168).then(data => {
        const color = data.length && data[data.length - 1] >= data[0] ? '#2ed573' : '#ff4757';
        initChart(ctx, data, color);
      });
    });

    function formatCurrency(value) {
      if (value >= 1e12) return '$' + (value / 1e12).toFixed(2) + 'T';
      if (value >= 1e9) return '$' + (value / 1e9).toFixed(2) + 'B';
      if (value >= 1e6) return '$' + (value / 1e6).toFixed(2) + 'M';
      return '$' + value.toFixed(2);
    }

    function flashUpdate(element, newVal, oldVal) {
      if (newVal > oldVal) { element.classList.add('flash-green'); }
      else if (newVal < oldVal) { element.classList.add('flash-red'); }
      setTimeout(() => { element.classList.remove('flash-green', 'flash-red'); }, 800);
    }

    async function fetchHistoricalData(symbol, hours = 168) {
      const interval = '2h';
      const limit = Math.ceil(hours / 2);
      const endpoint = `https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=${interval}&limit=${limit}`;
      try {
        const response = await fetch(endpoint);
        const data = await response.json();
        return data.map(entry => parseFloat(entry[4]));
      } catch (error) {
        console.error('Error fetching historical data:', error);
        return [];
      }
    }

    function initChart(ctx, data, color) {
      return new Chart(ctx, {
        type: 'line',
        data: {
          labels: data.map((_, i) => i),
          datasets: [{
            data: data,
            borderColor: color,
            borderWidth: 2,
            pointRadius: 0,
            tension: 0.2,
            fill: false
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: { x: { display: false }, y: { display: false } },
          plugins: { legend: { display: false }, tooltip: { enabled: false } }
        }
      });
    }
    
    async function updateDetailChart(hours) {
      if (!currentDetailCoin) return;
      const coin = trendingCoins.find(c => c.name === currentDetailCoin);
      if (!coin) return;

      document.querySelectorAll('.timeframe-btn').forEach(btn => {
        btn.classList.remove('active');
        if (btn.dataset.hours == hours) btn.classList.add('active');
      });

      const data = await fetchHistoricalData(coin.symbol, hours);
      if (!data.length) return;

      const ctx = document.getElementById('detailChart').getContext('2d');
      if (detailChartInstance) detailChartInstance.destroy();

      detailChartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels: data.map((_, i) => i),
          datasets: [{
            data: data,
            borderColor: detailChartColor,
            borderWidth: 2,
            pointRadius: 0,
            tension: 0.2,
            fill: false
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: { x: { display: false }, y: { display: false } },
          plugins: { legend: { display: false }, tooltip: { enabled: false } }
        }
      });
      
      updateBuyersSellers(data);
    }

    // Calculate and update buyers/sellers percentages
    function updateBuyersSellers(data) {
      if (!data || data.length < 2) return;
      const firstPrice = data[0];
      const lastPrice = data[data.length - 1];
      const diff = ((lastPrice - firstPrice) / firstPrice) * 100;
      const factor = 0.5;
      let buyers = 50 + diff * factor;
      buyers = Math.max(0, Math.min(100, buyers));
      const sellers = 100 - buyers;
      updateBuyersSellersUI(buyers, sellers);
    }

    function updateBuyersSellersUI(buyers, sellers) {
      const buyersBar = document.getElementById('buyersBar');
      const sellersBar = document.getElementById('sellersBar');
      buyersBar.style.width = buyers + "%";
      sellersBar.style.width = sellers + "%";
      document.getElementById('buyersPercentText').textContent = "Buyers: " + buyers.toFixed(0) + "%";
      document.getElementById('sellersPercentText').textContent = "Sellers: " + sellers.toFixed(0) + "%";
    }

    // Update coin stats in the stats box.
    function updateCoinStats(coin, price) {
      const supply = circulatingSupply[coin] || 1000000;
      const marketCap = price * supply;
      document.getElementById('detailMarketCap').textContent = formatCurrency(marketCap);
      document.getElementById('detailVolume').textContent = "N/A";
      document.getElementById('detailFDV').textContent = "N/A";
      document.getElementById('detailVolMktCap').textContent = "N/A";
      document.getElementById('detailTotalSupply').textContent = supply.toLocaleString();
      document.getElementById('detailMaxSupply').textContent = "N/A";
    }

    async function openDetails(coinName) {
      const detailSection = document.getElementById('coinDetails');
      detailSection.style.display = 'block';
      currentDetailCoin = coinName;
      
      // Update coin image and NERD button symbol
      const coinData = trendingCoins.find(c => c.name === coinName);
      if(coinData) {
        document.getElementById('detailCoinImg').src = `https://s2.coinmarketcap.com/static/img/coins/64x64/${coinData.id}.png`;
        document.getElementById('nerdBtn').setAttribute('data-symbol', coinData.symbol);
      }
      
      document.getElementById('detailCoinName').textContent = coinName;
      detailSection.scrollIntoView({ behavior: 'smooth' });
      updateDetailChart(currentTimeframe);
    }
    function closeDetails() {
      document.getElementById('coinDetails').style.display = 'none';
      currentDetailCoin = null;
      if (detailChartInstance) {
        detailChartInstance.destroy();
        detailChartInstance = null;
      }
    }

    document.querySelectorAll('.timeframe-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        currentTimeframe = parseInt(btn.dataset.hours);
        updateDetailChart(currentTimeframe);
      });
    });

    // Settings button functionality for coin details
    document.getElementById('detailSettingsBtn').addEventListener('click', function(e) {
      e.stopPropagation();
      this.classList.toggle('rotated');
      const panel = document.getElementById('detailSettingsPanel');
      panel.style.display = (panel.style.display === "block") ? "none" : "block";
    });

    // NERD button redirect functionality
    document.getElementById('nerdBtn').addEventListener('click', function() {
      const symbol = this.getAttribute('data-symbol');
      if (symbol) {
          window.location.href = `nerd.html?symbol=${symbol}`;
      } else {
          window.location.href = 'nerd.html';
      }
    });

    document.getElementById('graphColorPicker').addEventListener('input', function() {
      detailChartColor = this.value;
      if (detailChartInstance) {
        detailChartInstance.data.datasets[0].borderColor = detailChartColor;
        detailChartInstance.update();
      }
    });

    // Theme switch functionality (toggle between dark and light)
    document.getElementById('themeSwitch').addEventListener('change', function() {
      if (this.checked) {
        document.body.classList.add('light-theme');
      } else {
        document.body.classList.remove('light-theme');
      }
    });

    const ws = new WebSocket('wss://stream.binance.com:9443/ws/!ticker@arr');
    ws.onmessage = function (event) {
      const data = JSON.parse(event.data);
      data.forEach((ticker) => {
        const symbol = ticker.s;
        const coin = trendingCoins.find((c) => c.symbol === symbol)?.name;
        if (coin && trendingCardsMap[coin]) {
          const card = trendingCardsMap[coin];
          const price = parseFloat(ticker.c);
          const change7d = parseFloat(ticker.P);
          const marketCap = price * (circulatingSupply[coin] || 1000000);

          const priceEl = card.querySelector('.trending-price');
          const oldPrice = parseFloat(priceEl.getAttribute('data-prev')) || 0;
          priceEl.textContent = formatCurrency(price);
          flashUpdate(priceEl, price, oldPrice);
          priceEl.setAttribute('data-prev', price);

          const changeEl = card.querySelector('.trending-7d');
          changeEl.textContent = `${change7d >= 0 ? '+' : ''}${change7d.toFixed(2)}%`;
          changeEl.className = `trending-7d ${change7d >= 0 ? 'positive' : 'negative'}`;

          const marketCapEl = card.querySelector('.trending-marketcap');
          marketCapEl.textContent = formatCurrency(marketCap);

          if (currentDetailCoin === coin) {
            document.getElementById('detailPrice').textContent = formatCurrency(price);
            currentCoinPrice = price;
            updateCoinStats(coin, price);
          }
        }
      });
    };

    window.addEventListener('load', function() {
      setTimeout(function() {
        document.getElementById('preloader').style.display = 'none';
      }, 1000);
    });

    async function updateGlobalStats() {
      try {
        const response = await fetch('https://api.coingecko.com/api/v3/global');
        const data = await response.json();
        const marketCapUsd = data.data.total_market_cap.usd;
        const volumeUsd = data.data.total_volume.usd;
        const btcDominance = data.data.market_cap_percentage.btc;
        
        // Global Market Cap
        const marketCapCard = document.getElementById('global-marketcap');
        marketCapCard.querySelector('.stat-loading').style.display = 'none';
        const marketCapValue = marketCapCard.querySelector('.stat-value');
        marketCapValue.textContent = formatCurrency(marketCapUsd);
        marketCapValue.style.display = 'block';
        
        // 24h Volume
        const volumeCard = document.getElementById('global-volume');
        volumeCard.querySelector('.stat-loading').style.display = 'none';
        const volumeValue = volumeCard.querySelector('.stat-value');
        volumeValue.textContent = formatCurrency(volumeUsd);
        volumeValue.style.display = 'block';
        
        // Bitcoin Dominance
        const btcCard = document.getElementById('btc-dominance');
        btcCard.querySelector('.stat-loading').style.display = 'none';
        const btcValue = btcCard.querySelector('.stat-value');
        btcValue.textContent = btcDominance.toFixed(2) + '%';
        btcValue.style.display = 'block';
        
      } catch (error) {
        console.error('Error updating global market data:', error);
      }
    }

    async function updateFearGreed() {
      try {
        const response = await fetch('https://api.alternative.me/fng/?limit=1');
        const data = await response.json();
        if (data.data && data.data.length > 0) {
          const indexData = data.data[0];
          const value = indexData.value;
          const classification = indexData.value_classification;
          
          const fgCard = document.getElementById('fear-greed');
          fgCard.querySelector('.stat-loading').style.display = 'none';
          const fgValue = fgCard.querySelector('.stat-value');
          fgValue.innerHTML = `${value} - ${classification}`;
          fgValue.style.display = 'block';
        }
      } catch (error) {
        console.error('Error updating Fear & Greed index:', error);
      }
    }
    
    updateGlobalStats();
    updateFearGreed();
    setInterval(() => { updateGlobalStats(); updateFearGreed(); }, 60000);

    const settingsBtn = document.getElementById('settingsBtn');
    const settingsMenu = document.getElementById('settingsMenu');

    settingsBtn.addEventListener('click', function (e) {
      e.stopPropagation();
      settingsBtn.classList.toggle('active');
      settingsMenu.style.display = (settingsMenu.style.display === 'flex') ? 'none' : 'flex';
    });

    window.addEventListener('click', function (e) {
      if (!settingsMenu.contains(e.target) && !settingsBtn.contains(e.target)) {
        settingsMenu.style.display = 'none';
        settingsBtn.classList.remove('active');
      }
    });

    /* shaking animation */
    document.querySelectorAll('.trending-card').forEach(card => {
      let pressTimer;
      card.classList.add('card-container');
      const plus = document.createElement('div');
      plus.className = 'plus-button';
      plus.textContent = '+';
      card.appendChild(plus);
      card.addEventListener('mousedown', function(e) {
        pressTimer = setTimeout(() => {
          card.classList.add('shaking');
          plus.style.display = 'flex';
        }, 600);
      });
      card.addEventListener('mouseup', function(e) { clearTimeout(pressTimer); });
      card.addEventListener('mouseleave', function(e) { clearTimeout(pressTimer); });
      plus.addEventListener('click', function(e) {
        card.classList.remove('shaking');
        plus.style.display = 'none';
        e.stopPropagation();
      });
    });
    
    async function fetchFees() {
      // --- BTC Fee ---
      let btcFeeData;
      try {
        const response = await fetch('https://mempool.space/api/v1/fees/recommended');
        btcFeeData = await response.json();
      } catch (error) {
        btcFeeData = { fastestFee: 10 }; // fallback in sat/vB
      }
      const btcFeeSat = btcFeeData.fastestFee || 10;
      
      // --- ETH Fee ---
      let ethFeeData;
      try {
        const response = await fetch('https://www.etherchain.org/api/gasPriceOracle');
        ethFeeData = await response.json();
      } catch (error) {
        ethFeeData = { standard: 50 }; // fallback in Gwei
      }
      const ethGasGwei = ethFeeData.standard || 50;
      
      // --- Fetch Live Prices from CoinGecko for BTC and ETH ---
      let priceData;
      try {
        const response = await fetch("https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum&vs_currencies=usd");
        priceData = await response.json();
      } catch (error) {
        priceData = {
          bitcoin: { usd: 85000 },
          ethereum: { usd: 2400 }
        };
      }
      const btcPrice = priceData.bitcoin.usd;
      const ethPrice = priceData.ethereum.usd;
      
      // --- Fee Calculations in USD ---
      const btcFeeBTC = (btcFeeSat * 250) / 100000000;
      const btcFeeUSD = btcFeeBTC * btcPrice;
      const ethFeeETH = (ethGasGwei * 21000) / 1e9;
      const ethFeeUSD = ethFeeETH * ethPrice;
      
      return {
        BTC: btcFeeUSD.toFixed(2),
        ETH: ethFeeUSD.toFixed(2)
      };
    }
    
    async function updateFeeTracker() {
      const fees = await fetchFees();
      document.getElementById("fee-btc").textContent = fees.BTC;
      document.getElementById("fee-eth").textContent = fees.ETH;
    }
    
    updateFeeTracker();
    setInterval(updateFeeTracker, 60000);
    
  </script>
</body>
</html>
