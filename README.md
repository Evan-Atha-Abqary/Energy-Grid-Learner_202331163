<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Energy Grid Learner - AI Q-Learning Advanced</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
            animation: backgroundShift 10s ease-in-out infinite alternate;
        }

        @keyframes backgroundShift {
            0% { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
            50% { background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); }
            100% { background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); }
        }

        .container {
            max-width: 1600px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            overflow: hidden;
            backdrop-filter: blur(10px);
        }

        .header {
            background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
            color: white;
            padding: 30px;
            text-align: center;
            position: relative;
            overflow: hidden;
        }

        .header::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
            animation: shine 3s infinite;
        }

        @keyframes shine {
            0% { left: -100%; }
            100% { left: 100%; }
        }

        .header h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .header p {
            font-size: 1.2em;
            opacity: 0.9;
        }

        .main-content {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            padding: 30px;
        }

        .game-panel {
            background: white;
            border-radius: 15px;
            padding: 25px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            border: 2px solid #e3f2fd;
        }

        .controls {
            display: flex;
            gap: 15px;
            margin-bottom: 25px;
            flex-wrap: wrap;
        }

        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: 25px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
            min-width: 120px;
        }

        .btn::before {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 0;
            height: 0;
            background: rgba(255,255,255,0.3);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            transition: all 0.5s ease;
        }

        .btn:hover::before {
            width: 300px;
            height: 300px;
        }

        .btn-train {
            background: linear-gradient(135deg, #ff6b6b, #ee5a52);
            color: white;
            box-shadow: 0 5px 15px rgba(238, 90, 82, 0.4);
        }

        .btn-run {
            background: linear-gradient(135deg, #4ecdc4, #44a08d);
            color: white;
            box-shadow: 0 5px 15px rgba(68, 160, 141, 0.4);
        }

        .btn-reset {
            background: linear-gradient(135deg, #feca57, #ff9ff3);
            color: white;
            box-shadow: 0 5px 15px rgba(254, 202, 87, 0.4);
        }

        .btn-export {
            background: linear-gradient(135deg, #9b59b6, #8e44ad);
            color: white;
            box-shadow: 0 5px 15px rgba(142, 68, 173, 0.4);
        }

        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 7px 20px rgba(0, 0, 0, 0.2);
        }

        .btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }

        .grid-container {
            width: 100%;
            height: 400px;
            border: 3px solid #3498db;
            border-radius: 10px;
            position: relative;
            background: linear-gradient(45deg, #f8f9fa 25%, transparent 25%), 
                        linear-gradient(-45deg, #f8f9fa 25%, transparent 25%), 
                        linear-gradient(45deg, transparent 75%, #f8f9fa 75%), 
                        linear-gradient(-45deg, transparent 75%, #f8f9fa 75%);
            background-size: 20px 20px;
            background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
            overflow: hidden;
        }

        .grid-item {
            position: absolute;
            width: 40px;
            height: 40px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 20px;
            font-weight: bold;
            transition: all 0.3s ease;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }

        .power-plant {
            background: linear-gradient(135deg, #ff7675, #d63031);
            color: white;
            animation: pulse 2s infinite;
        }

        .house {
            background: linear-gradient(135deg, #74b9ff, #0984e3);
            color: white;
        }

        .house.powered {
            background: linear-gradient(135deg, #00b894, #00a085);
            animation: glow 1.5s infinite alternate;
        }

        .connection {
            position: absolute;
            background: #f39c12;
            height: 4px;
            border-radius: 2px;
            opacity: 0.8;
            transition: all 0.3s ease;
            box-shadow: 0 2px 4px rgba(243, 156, 18, 0.3);
        }

        .connection.training-connection {
            background: #e74c3c;
            opacity: 0.6;
            animation: training-flow 0.5s infinite;
        }

        .connection.training-connection.active {
            background: #e67e22;
            opacity: 0.8;
        }

        .connection.final-connection {
            background: #f39c12;
        }

        .connection.final-connection.active {
            background: #27ae60;
            animation: flow 1s infinite;
            box-shadow: 0 0 10px rgba(39, 174, 96, 0.5);
        }

        @keyframes training-flow {
            0% { opacity: 0.4; }
            50% { opacity: 0.8; }
            100% { opacity: 0.4; }
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.1); }
        }

        @keyframes glow {
            0% { box-shadow: 0 0 10px rgba(0, 184, 148, 0.5); }
            100% { box-shadow: 0 0 20px rgba(0, 184, 148, 0.8); }
        }

        @keyframes flow {
            0% { opacity: 0.5; }
            50% { opacity: 1; }
            100% { opacity: 0.5; }
        }

        .stats {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 15px;
            margin-top: 20px;
        }

        .stat-card {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
        }

        .stat-value {
            font-size: 2em;
            font-weight: bold;
            margin-bottom: 5px;
        }

        .stat-label {
            font-size: 0.9em;
            opacity: 0.9;
        }

        .charts-section {
            grid-column: 1 / -1;
            margin-top: 30px;
        }

        .charts-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            margin-top: 20px;
        }

        .chart-panel {
            background: white;
            border-radius: 15px;
            padding: 25px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            border: 2px solid #e8f5e8;
        }

        .chart-title {
            font-size: 1.4em;
            font-weight: bold;
            margin-bottom: 20px;
            color: #2c3e50;
            text-align: center;
        }

        .info-panel {
            background: linear-gradient(135deg, #f8f9fa, #e9ecef);
            border-radius: 10px;
            padding: 20px;
            margin-bottom: 20px;
            border-left: 5px solid #3498db;
        }

        .status {
            font-size: 1.1em;
            font-weight: bold;
            padding: 10px;
            border-radius: 5px;
            text-align: center;
            margin-bottom: 15px;
        }

        .status.idle {
            background: #e9ecef;
            color: #6c757d;
        }

        .status.training {
            background: #ffe6e6;
            color: #d63031;
            animation: blink 1s infinite;
        }

        .status.running {
            background: #e6f7f7;
            color: #00a085;
        }

        @keyframes blink {
            0%, 50% { opacity: 1; }
            51%, 100% { opacity: 0.5; }
        }

        .comparison-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
            background: white;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        }

        .comparison-table th,
        .comparison-table td {
            padding: 15px;
            text-align: left;
            border-bottom: 1px solid #eee;
        }

        .comparison-table th {
            background: linear-gradient(135deg, #3498db, #2980b9);
            color: white;
            font-weight: bold;
        }

        .comparison-table tr:hover {
            background: #f8f9fa;
        }

        .icon {
            display: inline-block;
            margin-right: 8px;
        }

        /* Enhanced Performance Dashboard Styles */
        .performance-dashboard {
            background: white;
            border-radius: 15px;
            padding: 25px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            border: 2px solid #e8f5e8;
            display: grid;
            grid-template-rows: auto 1fr auto;
            min-height: 500px;
        }

        .dashboard-header {
            text-align: center;
            margin-bottom: 20px;
        }

        .dashboard-title {
            font-size: 1.4em;
            font-weight: bold;
            color: #2c3e50;
            margin-bottom: 10px;
        }

        .dashboard-subtitle {
            color: #7f8c8d;
            font-size: 0.9em;
        }

        .metrics-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 20px;
        }

        .metric-card {
            background: linear-gradient(135deg, #ffffff, #f8f9fa);
            border: 2px solid #e9ecef;
            border-radius: 12px;
            padding: 20px;
            text-align: center;
            transition: all 0.3s ease;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
        }

        .metric-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
            border-color: #3498db;
        }

        .metric-value {
            font-size: 2.2em;
            font-weight: bold;
            margin-bottom: 8px;
            background: linear-gradient(135deg, #3498db, #2980b9);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .metric-label {
            font-size: 0.9em;
            color: #7f8c8d;
            font-weight: 500;
        }

        .efficiency-gauge {
            position: relative;
            width: 200px;
            height: 200px;
            margin: 20px auto;
        }

        .gauge-container {
            position: relative;
            width: 100%;
            height: 100%;
        }

        .gauge-bg {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background: conic-gradient(
                from 0deg,
                #e9ecef 0deg 270deg,
                transparent 270deg 360deg
            );
            mask: radial-gradient(circle at center, transparent 60%, black 61%);
        }

        .gauge-fill {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background: conic-gradient(
                from 0deg,
                #27ae60 0deg,
                #f39c12 90deg,
                #e74c3c 180deg,
                #e74c3c 270deg,
                transparent 270deg 360deg
            );
            mask: radial-gradient(circle at center, transparent 60%, black 61%);
            transform: rotate(0deg);
            transition: transform 1s ease-out;
        }

        .gauge-center {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 80px;
            height: 80px;
            background: white;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        }

        .gauge-value {
            font-size: 1.4em;
            font-weight: bold;
            color: #2c3e50;
        }

        .gauge-label {
            font-size: 0.7em;
            color: #7f8c8d;
            margin-top: 2px;
        }

        .performance-comparison {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-top: 20px;
        }

        .comparison-card {
            background: linear-gradient(135deg, #f8f9fa, #ffffff);
            border-radius: 12px;
            padding: 20px;
            border: 2px solid #e9ecef;
            transition: all 0.3s ease;
        }

        .comparison-card.training {
            border-color: #ff6b6b;
            background: linear-gradient(135deg, #fff5f5, #ffffff);
        }

        .comparison-card.running {
            border-color: #4ecdc4;
            background: linear-gradient(135deg, #f0fffe, #ffffff);
        }

        .comparison-card:hover {
            transform: scale(1.02);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
        }

        .comparison-header {
            display: flex;
            align-items: center;
            margin-bottom: 15px;
        }

        .comparison-icon {
            font-size: 2em;
            margin-right: 10px;
        }

        .comparison-title {
            font-size: 1.1em;
            font-weight: bold;
            color: #2c3e50;
        }

        .comparison-stats {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 15px;
        }

        .comparison-stat {
            text-align: center;
            padding: 10px;
            background: rgba(255, 255, 255, 0.8);
            border-radius: 8px;
        }

        .comparison-stat-value {
            font-size: 1.3em;
            font-weight: bold;
            color: #3498db;
        }

        .comparison-stat-label {
            font-size: 0.8em;
            color: #7f8c8d;
            margin-top: 2px;
        }

        /* AI Insights Panel */
        .ai-insights {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border-radius: 12px;
            padding: 20px;
            margin-top: 20px;
        }

        .insights-header {
            display: flex;
            align-items: center;
            margin-bottom: 15px;
        }

        .insights-icon {
            font-size: 1.5em;
            margin-right: 10px;
        }

        .insights-title {
            font-size: 1.2em;
            font-weight: bold;
        }

        .insights-content {
            font-size: 0.9em;
            line-height: 1.6;
            opacity: 0.9;
        }

        .recommendation {
            background: rgba(255, 255, 255, 0.1);
            border-radius: 8px;
            padding: 15px;
            margin-top: 15px;
            border-left: 4px solid #feca57;
        }

        .recommendation-title {
            font-weight: bold;
            margin-bottom: 8px;
            color: #feca57;
        }

        /* Export Controls */
        .export-controls {
            display: flex;
            gap: 10px;
            margin-top: 20px;
            justify-content: center;
            flex-wrap: wrap;
        }

        .btn-small {
            padding: 8px 16px;
            font-size: 14px;
            min-width: 100px;
        }

        /* Progress Indicators */
        .progress-container {
            width: 100%;
            height: 8px;
            background: #e9ecef;
            border-radius: 4px;
            margin: 10px 0;
            overflow: hidden;
        }

        .progress-bar {
            height: 100%;
            background: linear-gradient(135deg, #4ecdc4, #44a08d);
            border-radius: 4px;
            width: 0%;
            transition: width 0.3s ease;
        }

        /* Real-time Metrics */
        .realtime-metrics {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
            margin: 20px 0;
        }

        .realtime-metric {
            background: rgba(255, 255, 255, 0.9);
            border-radius: 8px;
            padding: 15px;
            text-align: center;
            border: 2px solid #e9ecef;
            transition: all 0.3s ease;
        }

        .realtime-metric:hover {
            border-color: #3498db;
            transform: translateY(-2px);
        }

        .metric-number {
            font-size: 1.5em;
            font-weight: bold;
            color: #2c3e50;
            margin-bottom: 5px;
        }

        .metric-text {
            font-size: 0.8em;
            color: #7f8c8d;
        }

        @media (max-width: 768px) {
            .main-content {
                grid-template-columns: 1fr;
            }
            
            .charts-container {
                grid-template-columns: 1fr;
            }
            
            .controls {
                flex-direction: column;
            }
            
            .btn {
                width: 100%;
            }

            .metrics-grid {
                grid-template-columns: 1fr;
            }

            .performance-comparison {
                grid-template-columns: 1fr;
            }

            .realtime-metrics {
                grid-template-columns: repeat(2, 1fr);
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🎮 Energy Grid Learner</h1>
            <p>🧠 Program Edukasi AI Berbasis Q-Learning untuk Distribusi Energi</p>
            <p>202331163_Evan Atha Abqary</p>
        </div>

        <div class="main-content">
            <div class="game-panel">
                <div class="info-panel">
                    <h3>🎯 Tujuan Program</h3>
                    <p>AI akan belajar mengelola distribusi daya listrik menggunakan Q-Learning. Lihat bagaimana AI berkembang dari tidak tahu apa-apa hingga mengambil keputusan optimal!</p>
                </div>

                <div class="controls">
                    <button class="btn btn-train" onclick="trainAI()">
                        <span class="icon">🤖</span>Train AI
                    </button>
                    <button class="btn btn-run" onclick="runAI()" disabled>
                        <span class="icon">⚡</span>Run AI
                    </button>
                    <button class="btn btn-reset" onclick="resetGame()">
                        <span class="icon">🔄</span>Reset
                    </button>
                    <button class="btn btn-export" onclick="exportData()">
                        <span class="icon">📊</span>Export
                    </button>
                </div>

                <div class="status idle" id="status">
                    <span class="icon">⏳</span>Menunggu pelatihan AI...
                </div>

                <div class="progress-container" id="progress-container" style="display: none;">
                    <div class="progress-bar" id="progress-bar"></div>
                </div>

                <div class="grid-container" id="grid">
                    <!-- Grid akan diisi oleh JavaScript -->
                </div>

                <div class="stats">
                    <div class="stat-card">
                        <div class="stat-value" id="episode">0</div>
                        <div class="stat-label">Episode</div>
                    </div>
                    <div class="stat-card">
                        <div class="stat-value" id="score">0</div>
                        <div class="stat-label">Skor Total</div>
                    </div>
                    <div class="stat-card">
                        <div class="stat-value" id="powered">0</div>
                        <div class="stat-label">Rumah Teraliri</div>
                    </div>
                    <div class="stat-card">
                        <div class="stat-value" id="efficiency">0%</div>
                        <div class="stat-label">Efisiensi</div>
                    </div>
                </div>

                <div class="realtime-metrics">
                    <div class="realtime-metric">
                        <div class="metric-number" id="learning-rate">0.1</div>
                        <div class="metric-text">Learning Rate</div>
                    </div>
                    <div class="realtime-metric">
                        <div class="metric-number" id="exploration-rate">100%</div>
                        <div class="metric-text">Exploration</div>
                    </div>
                    <div class="realtime-metric">
                        <div class="metric-number" id="convergence">0%</div>
                        <div class="metric-text">Convergence</div>
                    </div>
                    <div class="realtime-metric">
                        <div class="metric-number" id="stability">0%</div>
                        <div class="metric-text">Stability</div>
                    </div>
                </div>
            </div>

            <div class="performance-dashboard">
                <div class="dashboard-header">
                    <div class="dashboard-title">🚀 Dashboard Performa AI</div>
                    <div class="dashboard-subtitle">Analisis Real-time & Perbandingan Hasil</div>
                </div>

                <div class="metrics-grid">
                    <div class="metric-card">
                        <div class="metric-value" id="avg-score">0</div>
                        <div class="metric-label">Rata-rata Skor</div>
                    </div>
                    <div class="metric-card">
                        <div class="metric-value" id="max-efficiency">0%</div>
                        <div class="metric-label">Efisiensi Maksimal</div>
                    </div>
                    <div class="metric-card">
                        <div class="metric-value" id="improvement-rate">0%</div>
                        <div class="metric-label">Tingkat Peningkatan</div>
                    </div>
                    <div class="metric-card">
                        <div class="metric-value" id="total-connections">0</div>
                        <div class="metric-label">Total Koneksi</div>
                    </div>
                </div>

                <div class="efficiency-gauge">
                    <div class="gauge-container">
                        <div class="gauge-bg"></div>
                        <div class="gauge-fill" id="efficiency-gauge-fill"></div>
                        <div class="gauge-center">
                            <div class="gauge-value" id="gauge-efficiency">0%</div>
                            <div class="gauge-label">Efisiensi</div>
                        </div>
                    </div>
                </div>

                <div class="performance-comparison">
                    <div class="comparison-card training">
                        <div class="comparison-header">
                            <div class="comparison-icon">🤖</div>
                            <div class="comparison-title">Mode Training</div>
                        </div>
                        <div>Eksplorasi & pembelajaran melalui trial-error dengan Q-Learning algorithm</div>
                        <div class="comparison-stats">
                            <div class="comparison-stat">
                                <div class="comparison-stat-value" id="training-episodes">0</div>
                                <div class="comparison-stat-label">Episodes</div>
                            </div>
                            <div class="comparison-stat">
                                <div class="comparison-stat-value" id="training-avg-score">0</div>
                                <div class="comparison-stat-label">Avg Score</div>
                            </div>
                        </div>
                    </div>
                    <div class="comparison-card running">
                        <div class="comparison-header">
                            <div class="comparison-icon">⚡</div>
                            <div class="comparison-title">Mode Execution</div>
                        </div>
                        <div>Eksekusi strategi optimal berdasarkan hasil pembelajaran</div>
                        <div class="comparison-stats">
                            <div class="comparison-stat">
                                <div class="comparison-stat-value" id="execution-score">-</div>
                                <div class="comparison-stat-label">Final Score</div>
                            </div>
                            <div class="comparison-stat">
                                <div class="comparison-stat-value" id="execution-efficiency">-</div>
                                <div class="comparison-stat-label">Efficiency</div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="ai-insights">
                    <div class="insights-header">
                        <div class="insights-icon">🧠</div>
                        <div class="insights-title">AI Insights & Rekomendasi</div>
                    </div>
                    <div class="insights-content" id="ai-insights-content">
                        Mulai pelatihan AI untuk mendapatkan insights dan rekomendasi optimisasi distribusi energi.
                    </div>
                    <div class="recommendation" id="ai-recommendation" style="display: none;">
                        <div class="recommendation-title">💡 Rekomendasi Strategi</div>
                        <div id="recommendation-text">Menunggu hasil analisis...</div>
                    </div>
                </div>

                <div class="export-controls">
                    <button class="btn btn-small btn-export" onclick="exportTrainingData()">
                        <span class="icon">📈</span>Data Training
                    </button>
                    <button class="btn btn-small btn-export" onclick="exportQTable()">
                        <span class="icon">🧠</span>Q-Table
                    </button>
                    <button class="btn btn-small btn-export" onclick="generateReport()">
                        <span class="icon">📄</span>Laporan
                    </button>
                </div>
            </div>
        </div>

        <div class="charts-section">
            <h2 style="text-align: center; color: #2c3e50; margin-bottom: 20px;">
                📈 Visualisasi Pembelajaran AI
            </h2>
            <div class="charts-container">
                <div class="chart-panel">
                    <div class="chart-title">🤖 Grafik Pelatihan AI (Training Progress)</div>
                    <canvas id="trainingChart"></canvas>
                </div>
                <div class="chart-panel">
                    <div class="chart-title">⚡ Analisis Konvergensi (Learning Convergence)</div>
                    <canvas id="convergenceChart"></canvas>
                </div>
            </div>
            
            <div class="charts-container" style="margin-top: 30px;">
                <div class="chart-panel">
                    <div class="chart-title">📊 Distribusi Reward (Reward Distribution)</div>
                    <canvas id="rewardChart"></canvas>
                </div>
                <div class="chart-panel">
                    <div class="chart-title">🎯 Efisiensi per Episode (Efficiency Trend)</div>
                    <canvas id="efficiencyChart"></canvas>
                </div>
            </div>
        </div>

        <!-- Comparison Table Section -->
        <div style="grid-column: 1 / -1; margin-top: 30px; padding: 0 30px;">
            <div class="game-panel">
                <h3>📊 Perbandingan Train AI vs Run AI</h3>
                <table class="comparison-table">
                    <thead>
                        <tr>
                            <th>Fitur</th>
                            <th><span class="icon">🤖</span>Train AI</th>
                            <th><span class="icon">⚡</span>Run AI</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><span class="icon">🎯</span>Tujuan</td>
                            <td>Melatih AI dengan Q-Learning</td>
                            <td>Menjalankan hasil pelatihan optimal</td>
                        </tr>
                        <tr>
                            <td><span class="icon">🎮</span>Aksi</td>
                            <td>AI eksplorasi (trial & error)</td>
                            <td>AI eksploitasi (strategi terbaik)</td>
                        </tr>
                        <tr>
                            <td><span class="icon">👀</span>Visual</td>
                            <td>Garis merah/orange dinamis</td>
                            <td>Garis hijau stabil optimal</td>
                        </tr>
                        <tr>
                            <td><span class="icon">📈</span>Performa</td>
                            <td>Fluktuatif, pembelajaran bertahap</td>
                            <td>Stabil, performa maksimal</td>
                        </tr>
                        <tr>
                            <td><span class="icon">🔄</span>Proses</td>
                            <td>Multiple episodes (100x)</td>
                            <td>Single execution</td>
                        </tr>
                        <tr>
                            <td><span class="icon">🧠</span>Output</td>
                            <td>Q-table & learning metrics</td>
                            <td>Performance evaluation</td>
                        </tr>
                    </tbody>
                </table>

                <div style="margin-top: 20px; padding: 15px; background: #f8f9fa; border-radius: 10px;">
                    <h4>🔍 Status Pelatihan:</h4>
                    <div id="training-info">
                        <p><span class="icon">📊</span>Episode Terbaik: <span id="best-episode">-</span></p>
                        <p><span class="icon">🏆</span>Skor Terbaik: <span id="best-score">-</span></p>
                        <p><span class="icon">⚡</span>Efisiensi Terbaik: <span id="best-efficiency">-</span></p>
                        <p><span class="icon">🎯</span>Konvergensi: <span id="convergence-status">Belum dimulai</span></p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Enhanced Game State
        let gameState = {
            isTraining: false,
            isRunning: false,
            episode: 0,
            totalEpisodes: 100,
            qTable: {},
            trainingData: [],
            runData: null,
            bestScore: -Infinity,
            bestEpisode: 0,
            bestEfficiency: 0,
            convergenceData: [],
            rewardHistory: [],
            efficiencyHistory: [],
            learningRate: 0.1,
            explorationRate: 1.0,
            convergenceThreshold: 0.95,
            isConverged: false
        };

        // Grid Configuration
        const gridConfig = {
            width: 8,
            height: 6,
            powerPlants: [{x: 0, y: 2}],
            houses: [
                {x: 6, y: 1}, {x: 7, y: 1},
                {x: 6, y: 3}, {x: 7, y: 3},
                {x: 5, y: 4}, {x: 6, y: 4}
            ]
        };

        // Charts
        let trainingChart = null;
        let convergenceChart = null;
        let rewardChart = null;
        let efficiencyChart = null;

        // Initialize Game
        function initGame() {
            createGrid();
            initCharts();
            updateStats();
            updateDashboard();
        }

        function createGrid() {
            const grid = document.getElementById('grid');
            grid.innerHTML = '';

            // Add power plants
            gridConfig.powerPlants.forEach((plant, index) => {
                const element = document.createElement('div');
                element.className = 'grid-item power-plant';
                element.innerHTML = '🏭';
                element.style.left = `${plant.x * 45 + 10}px`;
                element.style.top = `${plant.y * 60 + 10}px`;
                element.id = `plant-${index}`;
                grid.appendChild(element);
            });

            // Add houses
            gridConfig.houses.forEach((house, index) => {
                const element = document.createElement('div');
                element.className = 'grid-item house';
                element.innerHTML = '🏠';
                element.style.left = `${house.x * 45 + 10}px`;
                element.style.top = `${house.y * 60 + 10}px`;
                element.id = `house-${index}`;
                grid.appendChild(element);
            });
        }

        function initCharts() {
            // Training Chart
            const trainingCtx = document.getElementById('trainingChart').getContext('2d');
            trainingChart = new Chart(trainingCtx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Skor per Episode',
                        data: [],
                        borderColor: '#ff6b6b',
                        backgroundColor: 'rgba(255, 107, 107, 0.1)',
                        borderWidth: 3,
                        fill: true,
                        tension: 0.4
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: true,
                            position: 'top'
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: 'Skor'
                            }
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Episode'
                            }
                        }
                    }
                }
            });

            // Convergence Chart
            const convergenceCtx = document.getElementById('convergenceChart').getContext('2d');
            convergenceChart = new Chart(convergenceCtx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Konvergensi',
                        data: [],
                        borderColor: '#4ecdc4',
                        backgroundColor: 'rgba(78, 205, 196, 0.1)',
                        borderWidth: 3,
                        fill: true,
                        tension: 0.4
                    }, {
                        label: 'Target (95%)',
                        data: [],
                        borderColor: '#e74c3c',
                        borderWidth: 2,
                        borderDash: [5, 5],
                        fill: false
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: true,
                            position: 'top'
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            max: 100,
                            title: {
                                display: true,
                                text: 'Konvergensi (%)'
                            }
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Episode'
                            }
                        }
                    }
                }
            });

            // Reward Distribution Chart
            const rewardCtx = document.getElementById('rewardChart').getContext('2d');
            rewardChart = new Chart(rewardCtx, {
                type: 'bar',
                data: {
                    labels: ['0-25', '26-50', '51-75', '76-100', '101-125', '126-150', '151+'],
                    datasets: [{
                        label: 'Frekuensi Reward',
                        data: [0, 0, 0, 0, 0, 0, 0],
                        backgroundColor: [
                            '#ff6b6b', '#feca57', '#48dbfb', '#0abde3',
                            '#00d2d3', '#54a0ff', '#5f27cd'
                        ],
                        borderWidth: 2,
                        borderColor: '#fff'
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: false
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: 'Frekuensi'
                            }
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Range Reward'
                            }
                        }
                    }
                }
            });

            // Efficiency Chart
            const efficiencyCtx = document.getElementById('efficiencyChart').getContext('2d');
            efficiencyChart = new Chart(efficiencyCtx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Efisiensi (%)',
                        data: [],
                        borderColor: '#27ae60',
                        backgroundColor: 'rgba(39, 174, 96, 0.1)',
                        borderWidth: 3,
                        fill: true,
                        tension: 0.4
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: true,
                            position: 'top'
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            max: 100,
                            title: {
                                display: true,
                                text: 'Efisiensi (%)'
                            }
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Episode'
                            }
                        }
                    }
                }
            });
        }

        async function trainAI() {
            if (gameState.isTraining) return;

            gameState.isTraining = true;
            gameState.episode = 0;
            gameState.trainingData = [];
            gameState.convergenceData = [];
            gameState.rewardHistory = [];
            gameState.efficiencyHistory = [];
            gameState.qTable = {};
            gameState.bestScore = -Infinity;
            gameState.bestEpisode = 0;
            gameState.bestEfficiency = 0;
            gameState.isConverged = false;

            updateStatus('training', '🤖 AI sedang berlatih...');
            document.querySelector('.btn-train').disabled = true;
            document.querySelector('.btn-run').disabled = true;
            
            // Show progress bar
            document.getElementById('progress-container').style.display = 'block';

            // Clear previous training data
            resetCharts();

            // Training loop
            for (let episode = 1; episode <= gameState.totalEpisodes; episode++) {
                gameState.episode = episode;
                
                // Update learning parameters
                gameState.explorationRate = Math.max(0.1, 1.0 - (episode / gameState.totalEpisodes));
                
                // Clear previous connections before each episode
                clearConnections();
                
                const result = simulateEpisode();
                gameState.trainingData.push(result);
                gameState.rewardHistory.push(result.score);
                gameState.efficiencyHistory.push(result.efficiency);

                // Calculate convergence
                const convergence = calculateConvergence(episode);
                gameState.convergenceData.push(convergence);

                // Animate connections for this episode
                await animateTrainingConnections(result);

                // Update best scores
                if (result.score > gameState.bestScore) {
                    gameState.bestScore = result.score;
                    gameState.bestEpisode = episode;
                    gameState.bestEfficiency = result.efficiency;
                }

                // Update progress bar
                const progress = (episode / gameState.totalEpisodes) * 100;
                document.getElementById('progress-bar').style.width = `${progress}%`;

                // Update charts every 5 episodes
                if (episode % 5 === 0 || episode === gameState.totalEpisodes) {
                    updateAllCharts();
                }

                updateStats();
                updateDashboard();
                updateTrainingInfo();
                updateAIInsights();

                // Delay for visual effect
                await new Promise(resolve => setTimeout(resolve, 100));
            }

            // Hide progress bar
            document.getElementById('progress-container').style.display = 'none';
            
            // Clear connections after training
            clearConnections();
            
            gameState.isTraining = false;
            updateStatus('idle', '✅ Pelatihan selesai! AI siap dijalankan.');
            document.querySelector('.btn-train').disabled = false;
            document.querySelector('.btn-run').disabled = false;
            
            // Final updates
            updateRewardDistribution();
            updateAIInsights();
        }

        function simulateEpisode() {
            const actions = ['connect_direct', 'connect_via_hub', 'optimize_load'];
            const state = 'distributing_power';
            
            // Epsilon-greedy exploration
            const epsilon = gameState.explorationRate;
            const explore = Math.random() < epsilon;
            
            let action;
            if (explore || !gameState.qTable[state]) {
                action = actions[Math.floor(Math.random() * actions.length)];
            } else {
                action = gameState.qTable[state].bestAction;
            }

            // Simulate reward based on action and learning progress
            let reward = 0;
            let poweredHouses = 0;
            
            const learningBonus = (gameState.episode / gameState.totalEpisodes) * 40;
            const randomFactor = (Math.random() - 0.5) * 20;
            
            switch (action) {
                case 'connect_direct':
                    reward = 30 + learningBonus + randomFactor;
                    poweredHouses = Math.max(1, Math.min(4, Math.floor(2 + learningBonus / 15)));
                    break;
                case 'connect_via_hub':
                    reward = 60 + learningBonus + randomFactor;
                    poweredHouses = Math.max(2, Math.min(5, Math.floor(3 + learningBonus / 12)));
                    break;
                case 'optimize_load':
                    reward = 90 + learningBonus + randomFactor;
                    poweredHouses = Math.max(3, Math.min(6, Math.floor(4 + learningBonus / 10)));
                    break;
            }

            reward = Math.max(0, reward);
            poweredHouses = Math.max(0, Math.min(6, poweredHouses));

            // Update Q-table
            if (!gameState.qTable[state]) {
                gameState.qTable[state] = {};
                actions.forEach(a => gameState.qTable[state][a] = 0);
            }
            
            const alpha = gameState.learningRate;
            const gamma = 0.9;
            gameState.qTable[state][action] = gameState.qTable[state][action] + 
                alpha * (reward - gameState.qTable[state][action]);

            // Find best action
            let bestAction = actions[0];
            let bestValue = gameState.qTable[state][actions[0]];
            actions.forEach(a => {
                if (gameState.qTable[state][a] > bestValue) {
                    bestValue = gameState.qTable[state][a];
                    bestAction = a;
                }
            });
            gameState.qTable[state].bestAction = bestAction;

            const efficiency = Math.round((poweredHouses / gridConfig.houses.length) * 100);

            return {
                episode: gameState.episode,
                action: action,
                reward: reward,
                score: Math.round(reward),
                poweredHouses: poweredHouses,
                efficiency: efficiency
            };
        }

        function calculateConvergence(episode) {
            if (episode < 10) return 0;
            
            // Calculate stability of recent scores
            const recent = gameState.trainingData.slice(-10);
            const avgRecent = recent.reduce((sum, d) => sum + d.score, 0) / recent.length;
            const variance = recent.reduce((sum, d) => sum + Math.pow(d.score - avgRecent, 2), 0) / recent.length;
            const stability = Math.max(0, 100 - variance);
            
            return Math.min(100, stability);
        }

        async function runAI() {
            if (gameState.isRunning || !gameState.qTable['distributing_power']) {
                return;
            }

            gameState.isRunning = true;
            updateStatus('running', '⚡ AI menjalankan strategi terbaik...');
            document.querySelector('.btn-run').disabled = true;

            // Clear previous connections
            clearConnections();

            // Run AI with best learned strategy
            const bestAction = gameState.qTable['distributing_power'].bestAction;
            let poweredHouses = 0;
            let finalScore = 0;

            // Simulate the best strategy with animation
            switch (bestAction) {
                case 'connect_direct':
                    poweredHouses = 4;
                    finalScore = 120;
                    await animateConnections([0, 1, 2, 3]);
                    break;
                case 'connect_via_hub':
                    poweredHouses = 5;
                    finalScore = 150;
                    await animateConnections([0, 1, 2, 3, 4]);
                    break;
                case 'optimize_load':
                    poweredHouses = 6;
                    finalScore = 180;
                    await animateConnections([0, 1, 2, 3, 4, 5]);
                    break;
            }

            const efficiency = Math.round((poweredHouses / gridConfig.houses.length) * 100);

            gameState.runData = {
                action: bestAction,
                poweredHouses: poweredHouses,
                totalHouses: gridConfig.houses.length,
                score: finalScore,
                efficiency: efficiency
            };

            updateStats();
            updateDashboard();
            updateAIInsights();
            
            gameState.isRunning = false;
            updateStatus('idle', '🏆 AI berhasil menjalankan strategi optimal!');
            document.querySelector('.btn-run').disabled = false;
        }

        function updateDashboard() {
            // Update metrics
            if (gameState.trainingData.length > 0) {
                const avgScore = gameState.trainingData.reduce((sum, d) => sum + d.score, 0) / gameState.trainingData.length;
                document.getElementById('avg-score').textContent = Math.round(avgScore);
                
                const maxEff = Math.max(...gameState.trainingData.map(d => d.efficiency));
                document.getElementById('max-efficiency').textContent = `${maxEff}%`;
                
                const improvement = gameState.trainingData.length > 1 ? 
                    ((gameState.bestScore - gameState.trainingData[0].score) / gameState.trainingData[0].score * 100) : 0;
                document.getElementById('improvement-rate').textContent = `${Math.round(improvement)}%`;
            }

            // Update total connections
            const totalConnections = gameState.trainingData.reduce((sum, d) => sum + d.poweredHouses, 0);
            document.getElementById('total-connections').textContent = totalConnections;

            // Update efficiency gauge
            const currentEfficiency = gameState.runData ? gameState.runData.efficiency : 
                (gameState.trainingData.length > 0 ? gameState.trainingData[gameState.trainingData.length - 1].efficiency : 0);
            
            document.getElementById('gauge-efficiency').textContent = `${currentEfficiency}%`;
            const gaugeFill = document.getElementById('efficiency-gauge-fill');
            const rotation = (currentEfficiency / 100) * 270; // 270 degrees max
            gaugeFill.style.transform = `rotate(${rotation}deg)`;

            // Update comparison cards
            if (gameState.trainingData.length > 0) {
                document.getElementById('training-episodes').textContent = gameState.trainingData.length;
                const trainingAvg = gameState.trainingData.reduce((sum, d) => sum + d.score, 0) / gameState.trainingData.length;
                document.getElementById('training-avg-score').textContent = Math.round(trainingAvg);
            }
            
            if (gameState.runData) {
                document.getElementById('execution-score').textContent = gameState.runData.score;
                document.getElementById('execution-efficiency').textContent = `${gameState.runData.efficiency}%`;
            }

            // Update real-time metrics
            document.getElementById('learning-rate').textContent = gameState.learningRate.toFixed(2);
            document.getElementById('exploration-rate').textContent = `${Math.round(gameState.explorationRate * 100)}%`;
            
            const convergence = gameState.convergenceData.length > 0 ? 
                gameState.convergenceData[gameState.convergenceData.length - 1] : 0;
            document.getElementById('convergence').textContent = `${Math.round(convergence)}%`;
            
            const stability = gameState.trainingData.length > 10 ?
                Math.round(100 - (gameState.trainingData.slice(-5).reduce((sum, d, i, arr) => {
                    if (i === 0) return 0;
                    return sum + Math.abs(d.score - arr[i-1].score);
                }, 0) / 4)) : 0;
            document.getElementById('stability').textContent = `${Math.max(0, stability)}%`;
        }

        function updateAIInsights() {
            const insightsContent = document.getElementById('ai-insights-content');
            const recommendation = document.getElementById('ai-recommendation');
            const recommendationText = document.getElementById('recommendation-text');
            
            if (gameState.trainingData.length === 0) {
                insightsContent.textContent = 'Mulai pelatihan AI untuk mendapatkan insights dan rekomendasi optimisasi distribusi energi.';
                recommendation.style.display = 'none';
                return;
            }

            let insights = '';
            let recommendationContent = '';

            if (gameState.isTraining && gameState.trainingData.length > 0) {
                const latestScore = gameState.trainingData[gameState.trainingData.length - 1].score;
                const avgScore = gameState.trainingData.reduce((sum, d) => sum + d.score, 0) / gameState.trainingData.length;
                
                insights = `AI sedang belajar... Episode ${gameState.episode}/${gameState.totalEpisodes}. `;
                insights += `Skor rata-rata: ${Math.round(avgScore)}. `;
                insights += `Tingkat eksplorasi: ${Math.round(gameState.explorationRate * 100)}%. `;
                
                if (latestScore > avgScore * 1.2) {
                    insights += 'AI menemukan strategi yang menjanjikan!';
                } else if (latestScore < avgScore * 0.8) {
                    insights += 'AI masih mengeksplorasi berbagai kemungkinan.';
                }
                
                recommendationContent = 'Biarkan AI menyelesaikan pelatihan untuk mendapatkan strategi optimal.';
            } else if (gameState.trainingData.length > 0) {
                const bestAction = gameState.qTable['distributing_power']?.bestAction || 'unknown';
                const convergence = gameState.convergenceData.length > 0 ? 
                    gameState.convergenceData[gameState.convergenceData.length - 1] : 0;
                
                insights = `Pelatihan selesai! AI telah mempelajari ${gameState.trainingData.length} episode. `;
                insights += `Strategi terbaik: ${getActionDescription(bestAction)}. `;
                insights += `Tingkat konvergensi: ${Math.round(convergence)}%. `;
                
                if (convergence > 80) {
                    insights += 'AI telah mencapai pembelajaran yang stabil.';
                    recommendationContent = 'AI siap untuk dijalankan. Klik "Run AI" untuk melihat performa optimal.';
                } else {
                    insights += 'AI masih bisa ditingkatkan dengan pelatihan tambahan.';
                    recommendationContent = 'Pertimbangkan untuk melatih AI lebih lama untuk hasil yang lebih stabil.';
                }
            }

            if (gameState.runData) {
                insights += ` Hasil eksekusi: ${gameState.runData.poweredHouses}/${gameState.runData.totalHouses} rumah teraliri (${gameState.runData.efficiency}% efisiensi).`;
                
                if (gameState.runData.efficiency >= 90) {
                    recommendationContent = 'Excellent! AI mencapai performa hampir optimal. Sistem siap untuk implementasi.';
                } else if (gameState.runData.efficiency >= 70) {
                    recommendationContent = 'Good! Performa AI cukup baik. Masih ada ruang untuk optimisasi lebih lanjut.';
                } else {
                    recommendationContent = 'AI memerlukan pelatihan tambahan untuk mencapai efisiensi yang lebih tinggi.';
                }
            }

            insightsContent.textContent = insights;
            recommendationText.textContent = recommendationContent;
            recommendation.style.display = 'block';
        }

        function getActionDescription(action) {
            switch (action) {
                case 'connect_direct': return 'Koneksi Langsung';
                case 'connect_via_hub': return 'Koneksi via Hub';
                case 'optimize_load': return 'Optimisasi Beban';
                default: return 'Tidak Diketahui';
            }
        }

        function updateAllCharts() {
            // Update training chart
            trainingChart.data.labels.push(gameState.episode);
            trainingChart.data.datasets[0].data.push(gameState.trainingData[gameState.trainingData.length - 1].score);
            trainingChart.update();

            // Update convergence chart
            convergenceChart.data.labels.push(gameState.episode);
            convergenceChart.data.datasets[0].data.push(gameState.convergenceData[gameState.convergenceData.length - 1]);
            convergenceChart.data.datasets[1].data.push(95); // Target line
            convergenceChart.update();

            // Update efficiency chart
            efficiencyChart.data.labels.push(gameState.episode);
            efficiencyChart.data.datasets[0].data.push(gameState.trainingData[gameState.trainingData.length - 1].efficiency);
            efficiencyChart.update();
        }

        function updateRewardDistribution() {
            const ranges = [0, 0, 0, 0, 0, 0, 0]; // 0-25, 26-50, etc.
            
            gameState.rewardHistory.forEach(reward => {
                if (reward <= 25) ranges[0]++;
                else if (reward <= 50) ranges[1]++;
                else if (reward <= 75) ranges[2]++;
                else if (reward <= 100) ranges[3]++;
                else if (reward <= 125) ranges[4]++;
                else if (reward <= 150) ranges[5]++;
                else ranges[6]++;
            });
            
            rewardChart.data.datasets[0].data = ranges;
            rewardChart.update();
        }

        function resetCharts() {
            // Reset all charts
            trainingChart.data.labels = [];
            trainingChart.data.datasets[0].data = [];
            trainingChart.update();
            
            convergenceChart.data.labels = [];
            convergenceChart.data.datasets[0].data = [];
            convergenceChart.data.datasets[1].data = [];
            convergenceChart.update();
            
            rewardChart.data.datasets[0].data = [0, 0, 0, 0, 0, 0, 0];
            rewardChart.update();
            
            efficiencyChart.data.labels = [];
            efficiencyChart.data.datasets[0].data = [];
            efficiencyChart.update();
        }

        async function animateTrainingConnections(episodeResult) {
            const grid = document.getElementById('grid');
            
            let houseIndices = [];
            
            switch (episodeResult.action) {
                case 'connect_direct':
                    houseIndices = getRandomHouseIndices(episodeResult.poweredHouses);
                    break;
                case 'connect_via_hub':
                    houseIndices = getSequentialHouseIndices(episodeResult.poweredHouses);
                    break;
                case 'optimize_load':
                    houseIndices = getOptimalHouseIndices(episodeResult.poweredHouses);
                    break;
            }
            
            // Animate connections rapidly during training
            for (let i = 0; i < houseIndices.length; i++) {
                const houseIndex = houseIndices[i];
                const house = gridConfig.houses[houseIndex];
                const plant = gridConfig.powerPlants[0];

                // Create connection line
                const connection = document.createElement('div');
                connection.className = 'connection training-connection';
                
                const dx = (house.x - plant.x) * 45;
                const dy = (house.y - plant.y) * 60;
                const length = Math.sqrt(dx*dx + dy*dy);
                const angle = Math.atan2(dy, dx) * 180 / Math.PI;

                connection.style.width = `${length}px`;
                connection.style.left = `${plant.x * 45 + 30}px`;
                connection.style.top = `${plant.y * 60 + 30}px`;
                connection.style.transform = `rotate(${angle}deg)`;
                connection.style.transformOrigin = '0 50%';

                grid.appendChild(connection);

                // Quick activation for training visualization
                setTimeout(() => {
                    connection.classList.add('active');
                    const houseElement = document.getElementById(`house-${houseIndex}`);
                    houseElement.classList.add('powered');
                }, 20);

                await new Promise(resolve => setTimeout(resolve, 30));
            }
        }

        function getRandomHouseIndices(count) {
            const indices = [...Array(gridConfig.houses.length).keys()];
            for (let i = indices.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [indices[i], indices[j]] = [indices[j], indices[i]];
            }
            return indices.slice(0, count);
        }

        function getSequentialHouseIndices(count) {
            const indices = [];
            for (let i = 0; i < Math.min(count, gridConfig.houses.length); i++) {
                indices.push(i);
            }
            return indices;
        }

        function getOptimalHouseIndices(count) {
            const distances = gridConfig.houses.map((house, index) => ({
                index: index,
                distance: Math.sqrt(
                    Math.pow(house.x - gridConfig.powerPlants[0].x, 2) + 
                    Math.pow(house.y - gridConfig.powerPlants[0].y, 2)
                )
            }));
            
            distances.sort((a, b) => a.distance - b.distance);
            return distances.slice(0, count).map(d => d.index);
        }

        async function animateConnections(houseIndices) {
            const grid = document.getElementById('grid');
            
            for (let i = 0; i < houseIndices.length; i++) {
                const houseIndex = houseIndices[i];
                const house = gridConfig.houses[houseIndex];
                const plant = gridConfig.powerPlants[0];

                const connection = document.createElement('div');
                connection.className = 'connection final-connection';
                
                const dx = (house.x - plant.x) * 45;
                const dy = (house.y - plant.y) * 60;
                const length = Math.sqrt(dx*dx + dy*dy);
                const angle = Math.atan2(dy, dx) * 180 / Math.PI;

                connection.style.width = `${length}px`;
                connection.style.left = `${plant.x * 45 + 30}px`;
                connection.style.top = `${plant.y * 60 + 30}px`;
                connection.style.transform = `rotate(${angle}deg)`;
                connection.style.transformOrigin = '0 50%';

                grid.appendChild(connection);

                setTimeout(() => {
                    connection.classList.add('active');
                    const houseElement = document.getElementById(`house-${houseIndex}`);
                    houseElement.classList.add('powered');
                }, 100);

                await new Promise(resolve => setTimeout(resolve, 300));
            }
        }

        function clearConnections() {
            const connections = document.querySelectorAll('.connection');
            connections.forEach(conn => conn.remove());
            
            const houses = document.querySelectorAll('.house');
            houses.forEach(house => house.classList.remove('powered'));
        }

        function resetGame() {
            gameState = {
                isTraining: false,
                isRunning: false,
                episode: 0,
                totalEpisodes: 100,
                qTable: {},
                trainingData: [],
                runData: null,
                bestScore: -Infinity,
                bestEpisode: 0,
                bestEfficiency: 0,
                convergenceData: [],
                rewardHistory: [],
                efficiencyHistory: [],
                learningRate: 0.1,
                explorationRate: 1.0,
                convergenceThreshold: 0.95,
                isConverged: false
            };

            clearConnections();
            updateStatus('idle', '⏳ Menunggu pelatihan AI...');
            
            // Hide progress bar
            document.getElementById('progress-container').style.display = 'none';
            
            // Reset charts
            resetCharts();
            
            updateStats();
            updateDashboard();
            updateTrainingInfo();
            updateAIInsights();
            
            document.querySelector('.btn-train').disabled = false;
            document.querySelector('.btn-run').disabled = true;
        }

        function updateStatus(type, message) {
            const statusElement = document.getElementById('status');
            statusElement.className = `status ${type}`;
            statusElement.innerHTML = message;
        }

        function updateStats() {
            document.getElementById('episode').textContent = gameState.episode;
            
            if (gameState.runData) {
                document.getElementById('score').textContent = gameState.runData.score;
                document.getElementById('powered').textContent = 
                    `${gameState.runData.poweredHouses}/${gameState.runData.totalHouses}`;
                document.getElementById('efficiency').textContent = `${gameState.runData.efficiency}%`;
            } else if (gameState.trainingData.length > 0) {
                const latestData = gameState.trainingData[gameState.trainingData.length - 1];
                document.getElementById('score').textContent = latestData.score;
                document.getElementById('powered').textContent = 
                    `${latestData.poweredHouses}/${gridConfig.houses.length}`;
                document.getElementById('efficiency').textContent = `${latestData.efficiency}%`;
            } else {
                document.getElementById('score').textContent = '0';
                document.getElementById('powered').textContent = `0/${gridConfig.houses.length}`;
                document.getElementById('efficiency').textContent = '0%';
            }
        }

        function updateTrainingInfo() {
            document.getElementById('best-episode').textContent = 
                gameState.bestEpisode > 0 ? gameState.bestEpisode : '-';
            document.getElementById('best-score').textContent = 
                gameState.bestScore > -Infinity ? Math.round(gameState.bestScore) : '-';
            document.getElementById('best-efficiency').textContent = 
                gameState.bestEfficiency > 0 ? `${gameState.bestEfficiency}%` : '-';
            
            const convergence = gameState.convergenceData.length > 0 ? 
                gameState.convergenceData[gameState.convergenceData.length - 1] : 0;
            
            let convergenceStatus = 'Belum dimulai';
            if (gameState.isTraining) {
                convergenceStatus = `${Math.round(convergence)}% - Sedang belajar`;
            } else if (convergence > 0) {
                if (convergence >= 80) {
                    convergenceStatus = `${Math.round(convergence)}% - Konvergen`;
                } else {
                    convergenceStatus = `${Math.round(convergence)}% - Perlu pelatihan`;
                }
            }
            
            document.getElementById('convergence-status').textContent = convergenceStatus;
        }

        // Export Functions
        function exportData() {
            if (gameState.trainingData.length === 0) {
                alert('Tidak ada data untuk diekspor. Silakan latih AI terlebih dahulu.');
                return;
            }
            
            const data = {
                trainingData: gameState.trainingData,
                qTable: gameState.qTable,
                runData: gameState.runData,
                bestScore: gameState.bestScore,
                bestEpisode: gameState.bestEpisode,
                convergenceData: gameState.convergenceData,
                rewardHistory: gameState.rewardHistory,
                efficiencyHistory: gameState.efficiencyHistory
            };
            
            const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `energy_grid_ai_data_${new Date().toISOString().slice(0, 10)}.json`;
            a.click();
            URL.revokeObjectURL(url);
        }

        function exportTrainingData() {
            if (gameState.trainingData.length === 0) {
                alert('Tidak ada data training untuk diekspor.');
                return;
            }
            
            const csvContent = "Episode,Action,Score,PoweredHouses,Efficiency\n" +
                gameState.trainingData.map(d => 
                    `${d.episode},${d.action},${d.score},${d.poweredHouses},${d.efficiency}`
                ).join('\n');
            
            const blob = new Blob([csvContent], { type: 'text/csv' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `training_data_${new Date().toISOString().slice(0, 10)}.csv`;
            a.click();
            URL.revokeObjectURL(url);
        }

        function exportQTable() {
            if (Object.keys(gameState.qTable).length === 0) {
                alert('Q-Table kosong. Silakan latih AI terlebih dahulu.');
                return;
            }
            
            const blob = new Blob([JSON.stringify(gameState.qTable, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `q_table_${new Date().toISOString().slice(0, 10)}.json`;
            a.click();
            URL.revokeObjectURL(url);
        }

        function generateReport() {
            if (gameState.trainingData.length === 0) {
                alert('Tidak ada data untuk laporan. Silakan latih AI terlebih dahulu.');
                return;
            }
            
            const avgScore = gameState.trainingData.reduce((sum, d) => sum + d.score, 0) / gameState.trainingData.length;
            const avgEfficiency = gameState.trainingData.reduce((sum, d) => sum + d.efficiency, 0) / gameState.trainingData.length;
            const convergence = gameState.convergenceData.length > 0 ? 
                gameState.convergenceData[gameState.convergenceData.length - 1] : 0;
            
            const report = `
# Laporan Pembelajaran AI - Energy Grid Learner
## Tanggal: ${new Date().toLocaleDateString('id-ID')}

### Ringkasan Pelatihan
- Total Episode: ${gameState.trainingData.length}
- Skor Rata-rata: ${Math.round(avgScore)}
- Skor Terbaik: ${Math.round(gameState.bestScore)} (Episode ${gameState.bestEpisode})
- Efisiensi Rata-rata: ${Math.round(avgEfficiency)}%
- Efisiensi Terbaik: ${gameState.bestEfficiency}%
- Tingkat Konvergensi: ${Math.round(convergence)}%

### Strategi Terbaik
${gameState.qTable['distributing_power'] ? 
  `Aksi Optimal: ${getActionDescription(gameState.qTable['distributing_power'].bestAction)}` : 
  'Belum ada strategi optimal'}

### Hasil Eksekusi
${gameState.runData ? 
  `- Skor Akhir: ${gameState.runData.score}
- Rumah Teraliri: ${gameState.runData.poweredHouses}/${gameState.runData.totalHouses}
- Efisiensi: ${gameState.runData.efficiency}%` : 
  'Belum dijalankan'}

### Rekomendasi
${convergence >= 80 ? 
  'AI telah mencapai pembelajaran yang optimal dan siap untuk implementasi.' :
  'AI memerlukan pelatihan tambahan untuk mencapai konvergensi yang lebih baik.'}

---
Generated by Energy Grid Learner AI System
            `;
            
            const blob = new Blob([report], { type: 'text/markdown' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `ai_learning_report_${new Date().toISOString().slice(0, 10)}.md`;
            a.click();
            URL.revokeObjectURL(url);
        }

        // Initialize game when page loads
        document.addEventListener('DOMContentLoaded', function() {
            initGame();
            
            // Add interactive effects to grid items
            document.querySelectorAll('.grid-item').forEach(item => {
                item.addEventListener('mouseenter', function() {
                    this.style.transform = 'scale(1.1)';
                    this.style.zIndex = '10';
                });
                
                item.addEventListener('mouseleave', function() {
                    this.style.transform = 'scale(1)';
                    this.style.zIndex = '1';
                });
            });

            // Add click effects to buttons
            document.querySelectorAll('.btn').forEach(btn => {
                btn.addEventListener('click', function() {
                    this.style.transform = 'scale(0.95)';
                    setTimeout(() => {
                        this.style.transform = '';
                    }, 150);
                });
            });
            
            // Add tooltips
            const tooltips = {
                '.power-plant': 'Pembangkit Listrik - Sumber energi utama untuk distribusi',
                '.house': 'Rumah - Konsumen energi yang memerlukan pasokan listrik',
                '.btn-train': 'Melatih AI menggunakan algoritma Q-Learning untuk optimisasi distribusi',
                '.btn-run': 'Menjalankan AI dengan strategi terbaik hasil pembelajaran',
                '.btn-reset': 'Reset semua data dan mulai pembelajaran dari awal',
                '.btn-export': 'Ekspor data pembelajaran untuk analisis lebih lanjut'
            };
            
            Object.keys(tooltips).forEach(selector => {
                document.querySelectorAll(selector).forEach(element => {
                    element.title = tooltips[selector];
                });
            });
        });

        // Keyboard shortcuts
        document.addEventListener('keydown', function(e) {
            if (e.key === '1' && !gameState.isTraining) {
                trainAI();
            } else if (e.key === '2' && !gameState.isRunning && gameState.qTable['distributing_power']) {
                runAI();
            } else if (e.key === '3') {
                resetGame();
            } else if (e.key === '4') {
                exportData();
            }
        });
    </script>
</body>
</html>
