<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>船舶碳排放合规计算器</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.8/dist/chart.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">

    <!-- Tailwind 配置 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#165DFF',
                        secondary: '#36CFC9',
                        accent: '#FF7D00',
                        neutral: '#F5F7FA',
                        dark: '#1D2129',
                    },
                    fontFamily: {
                        inter: ['Inter', 'sans-serif'],
                    },
                }
            }
        }
    </script>

    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .card-shadow {
                box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05), 0 8px 10px -6px rgba(0, 0, 0, 0.02);
            }
            .hover-scale {
                transition: transform 0.2s ease-in-out;
            }
            .hover-scale:hover {
                transform: scale(1.02);
            }
            .gradient-bg {
                background: linear-gradient(135deg, #165DFF 0%, #36CFC9 100%);
            }
            .gradient-text {
                background: linear-gradient(90deg, #165DFF 0%, #36CFC9 100%);
                -webkit-background-clip: text;
                -webkit-text-fill-color: transparent;
            }
        }
    </style>
</head>
<body class="font-inter bg-neutral text-dark min-h-screen flex flex-col">
    <!-- 导航栏 -->
    <nav class="bg-white shadow-md sticky top-0 z-50 transition-all duration-300">
        <div class="container mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fa-solid fa-ship text-primary text-2xl"></i>
                <h1 class="text-xl font-bold gradient-text">船舶碳排放合规计算器</h1>
            </div>
            <div class="flex items-center space-x-4">
                <button id="theme-toggle" class="p-2 rounded-full hover:bg-neutral transition-colors">
                    <i class="fa-solid fa-moon"></i>
                </button>
                <button id="help-button" class="p-2 rounded-full hover:bg-neutral transition-colors">
                    <i class="fa-solid fa-question-circle"></i>
                </button>
            </div>
        </div>
    </nav>

    <!-- 主内容区 -->
    <main class="flex-grow container mx-auto px-4 py-6">
        <!-- 欢迎卡片 -->
        <div class="bg-white rounded-xl p-6 mb-8 card-shadow hover-scale">
            <h2 class="text-[clamp(1.5rem,3vw,2.5rem)] font-bold mb-4">船舶碳排放合规计算工具</h2>
            <p class="text-gray-600 mb-6">本工具基于IMO GFI标准，帮助船舶运营商计算碳排放合规成本，优化燃料配比策略，实现节能减排目标。</p>
            <div class="flex flex-wrap gap-4">
                <button id="start-calculation" class="bg-primary hover:bg-primary/90 text-white px-6 py-3 rounded-lg font-medium transition-all shadow-lg hover:shadow-primary/30 flex items-center">
                    <i class="fa-solid fa-calculator mr-2"></i> 开始计算
                </button>
                <button id="reset-all" class="bg-white border border-gray-300 hover:border-primary text-dark px-6 py-3 rounded-lg font-medium transition-all shadow-md hover:shadow-sm flex items-center">
                    <i class="fa-solid fa-refresh mr-2"></i> 重置参数
                </button>
                <button id="export-excel" class="bg-accent hover:bg-accent/90 text-white px-6 py-3 rounded-lg font-medium transition-all shadow-lg hover:shadow-accent/30 flex items-center">
                    <i class="fa-solid fa-download mr-2"></i> 导出Excel
                </button>
            </div>
        </div>

        <!-- 输入参数区域 -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6 mb-8">
            <!-- 政策参数卡片 -->
            <div class="bg-white rounded-xl p-6 card-shadow hover-scale">
                <h3 class="text-xl font-semibold mb-4 flex items-center">
                    <i class="fa-solid fa-gavel text-primary mr-2"></i> 政策参数
                </h3>
                <div class="space-y-4">
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">减排目标 (%)</label>
                        <div class="flex items-center">
                            <input type="range" id="reduction-target" min="0" max="100" value="20" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-primary">
                            <span id="reduction-target-value" class="ml-2 text-sm font-medium">20%</span>
                        </div>
                    </div>
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">奖励倍数</label>
                        <input type="number" id="reward-multiplier" value="2" min="0" step="0.1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                    </div>
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">合规池百分比 (%)</label>
                        <input type="number" id="compliance-pool" value="2" min="0" max="10" step="0.1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                    </div>
                </div>
            </div>

            <!-- 燃料参数卡片 -->
            <div class="bg-white rounded-xl p-6 card-shadow hover-scale">
                <h3 class="text-xl font-semibold mb-4 flex items-center">
                    <i class="fa-solid fa-gas-pump text-primary mr-2"></i> 燃料参数
                </h3>
                <div class="space-y-4">
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">燃料类型</label>
                        <select id="fuel-type" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                            <option value="HFO">HFO (重质燃料油)</option>
                            <option value="VLSFO">VLSFO (超低硫燃料油)</option>
                            <option value="MDO">MDO (船用柴油)</option>
                            <option value="LNG">LNG (液化天然气)</option>
                            <option value="Bio-diesel">Bio-diesel (生物柴油)</option>
                            <option value="Bio-methane">Bio-methane (生物甲烷)</option>
                            <option value="ZNZ">ZNZ (零碳燃料)</option>
                        </select>
                    </div>
                    <div class="grid grid-cols-2 gap-4">
                        <div class="form-group">
                            <label class="block text-sm font-medium text-gray-700 mb-1">份额 (%)</label>
                            <input type="number" id="fuel-share" value="40" min="0" max="100" step="1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                        </div>
                        <div class="form-group">
                            <label class="block text-sm font-medium text-gray-700 mb-1">低位热值 (MJ/kg)</label>
                            <input type="number" id="fuel-lhv" value="40.4" min="0" step="0.1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                        </div>
                        <div class="form-group">
                            <label class="block text-sm font-medium text-gray-700 mb-1">排放因子 (gCO₂eq/MJ)</label>
                            <input type="number" id="fuel-ef" value="94.0" min="0" step="0.1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                        </div>
                    </div>
                    <button id="add-fuel" class="w-full bg-primary hover:bg-primary/90 text-white py-2 rounded-lg font-medium transition-all">
                        <i class="fa-solid fa-plus mr-1"></i> 添加燃料
                    </button>
                </div>
            </div>

            <!-- 其他参数卡片 -->
            <div class="bg-white rounded-xl p-6 card-shadow hover-scale">
                <h3 class="text-xl font-semibold mb-4 flex items-center">
                    <i class="fa-solid fa-sliders text-primary mr-2"></i> 其他参数
                </h3>
                <div class="space-y-4">
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">碳价 (USD/tCO₂)</label>
                        <input type="number" id="carbon-price" value="100" min="0" step="1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                    </div>
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">IMO GFI参考强度 (gCO₂eq/MJ)</label>
                        <input type="number" id="imo-reference" value="91.5" min="0" step="0.1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                    </div>
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">ZNZ燃料份额 (%)</label>
                        <input type="number" id="znz-share" value="0" min="0" max="100" step="1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                    </div>
                    <div class="form-group">
                        <label class="block text-sm font-medium text-gray-700 mb-1">ZNZ燃料碳强度 (gCO₂eq/MJ)</label>
                        <input type="number" id="znz-intensity" value="10" min="0" step="0.1" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary">
                    </div>
                </div>
            </div>
        </div>

        <!-- 燃料配比表格 -->
        <div class="bg-white rounded-xl p-6 mb-8 card-shadow">
            <h3 class="text-xl font-semibold mb-4 flex items-center">
                <i class="fa-solid fa-table text-primary mr-2"></i> 燃料配比
            </h3>
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">燃料类型</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">份额 (%)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">低位热值 (MJ/kg)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">排放因子 (gCO₂eq/MJ)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">操作</th>
                        </tr>
                    </thead>
                    <tbody id="fuel-mix-table" class="bg-white divide-y divide-gray-200">
                        <!-- JavaScript 将动态填充此表格 -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- 计算结果区域 -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
            <!-- 结果摘要卡片 -->
            <div class="bg-white rounded-xl p-6 card-shadow">
                <h3 class="text-xl font-semibold mb-4 flex items-center">
                    <i class="fa-solid fa-chart-pie text-primary mr-2"></i> 计算结果摘要
                </h3>
                <div class="space-y-6">
                    <div class="grid grid-cols-2 gap-4">
                        <div class="bg-blue-50 rounded-lg p-4">
                            <p class="text-sm text-gray-600 mb-1">基准强度</p>
                            <p id="baseline-intensity" class="text-2xl font-bold">91.5 gCO₂eq/MJ</p>
                        </div>
                        <div class="bg-green-50 rounded-lg p-4">
                            <p class="text-sm text-gray-600 mb-1">目标强度</p>
                            <p id="target-intensity" class="text-2xl font-bold">73.2 gCO₂eq/MJ</p>
                        </div>
                        <div class="bg-purple-50 rounded-lg p-4">
                            <p class="text-sm text-gray-600 mb-1">实际强度</p>
                            <p id="actual-intensity" class="text-2xl font-bold">85.3 gCO₂eq/MJ</p>
                        </div>
                        <div class="bg-red-50 rounded-lg p-4">
                            <p class="text-sm text-gray-600 mb-1">合规缺口</p>
                            <p id="compliance-gap" class="text-2xl font-bold">12.1 gCO₂eq/MJ</p>
                        </div>
                    </div>
                    <div class="bg-yellow-50 rounded-lg p-4">
                        <p class="text-sm text-gray-600 mb-1">合规成本</p>
                        <p id="compliance-cost" class="text-3xl font-bold">USD 1,250,000</p>
                    </div>
                </div>
            </div>

            <!-- 图表区域 -->
            <div class="bg-white rounded-xl p-6 card-shadow">
                <h3 class="text-xl font-semibold mb-4 flex items-center">
                    <i class="fa-solid fa-chart-line text-primary mr-2"></i> 排放趋势
                </h3>
                <div class="h-80">
                    <canvas id="emission-chart"></canvas>
                </div>
            </div>
        </div>

        <!-- 详细计算步骤 -->
        <div class="bg-white rounded-xl p-6 mb-8 card-shadow">
            <h3 class="text-xl font-semibold mb-4 flex items-center">
                <i class="fa-solid fa-list-ol text-primary mr-2"></i> 详细计算步骤
            </h3>
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">步骤</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">描述</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">计算过程</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">结果</th>
                        </tr>
                    </thead>
                    <tbody id="calculation-steps" class="bg-white divide-y divide-gray-200">
                        <!-- JavaScript 将动态填充此表格 -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- 逐年结果表格 -->
        <div class="bg-white rounded-xl p-6 card-shadow">
            <h3 class="text-xl font-semibold mb-4 flex items-center">
                <i class="fa-solid fa-calendar-alt text-primary mr-2"></i> 逐年合规计算
            </h3>
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">年份</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">基准强度 (gCO₂eq/MJ)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">目标强度 (gCO₂eq/MJ)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">调整后目标 (gCO₂eq/MJ)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">实际强度 (gCO₂eq/MJ)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">合规缺口 (gCO₂eq/MJ)</th>
                            <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">合规成本 (USD)</th>
                        </tr>
                    </thead>
                    <tbody id="yearly-results" class="bg-white divide-y divide-gray-200">
                        <!-- JavaScript 将动态填充此表格 -->
                    </tbody>
                </table>
            </div>
        </div>
    </main>

    <!-- 页脚 -->
    <footer class="bg-dark text-white py-6">
        <div class="container mx-auto px-4">
            <div class="flex flex-col md:flex-row justify-between items-center">
                <div class="mb-4 md:mb-0">
                    <div class="flex items-center space-x-2">
                        <i class="fa-solid fa-ship text-primary text-xl"></i>
                        <span class="font-bold text-lg">船舶碳排放合规计算器</span>
                    </div>
                    <p class="text-gray-400 text-sm mt-2">基于IMO GFI标准的碳排放合规计算工具</p>
                </div>
                <div class="flex space-x-4">
                    <a href="#" class="text-gray-400 hover:text-white transition-colors">
                        <i class="fa-brands fa-github"></i>
                    </a>
                    <a href="#" class="text-gray-400 hover:text-white transition-colors">
                        <i class="fa-brands fa-linkedin"></i>
                    </a>
                    <a href="#" class="text-gray-400 hover:text-white transition-colors">
                        <i class="fa-brands fa-twitter"></i>
                    </a>
                </div>
            </div>
            <div class="border-t border-gray-800 mt-6 pt-6 text-center text-gray-400 text-sm">
                © 2025 船舶碳排放合规计算器 | 保留所有权利
            </div>
        </div>
    </footer>

    <!-- 帮助模态框 -->
    <div id="help-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
        <div class="bg-white rounded-xl p-6 max-w-lg w-full max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold">使用帮助</h3>
                <button id="close-help" class="text-gray-500 hover:text-gray-700">
                    <i class="fa-solid fa-times"></i>
                </button>
            </div>
            <div class="space-y-4">
                <div>
                    <h4 class="font-semibold text-primary">1. 输入参数</h4>
                    <p class="text-gray-600">在左侧面板设置政策参数、燃料参数和其他计算所需的参数。</p>
                </div>
                <div>
                    <h4 class="font-semibold text-primary">2. 配置燃料配比</h4>
                    <p class="text-gray-600">在燃料参数面板选择燃料类型，设置份额、低位热值和排放因子，然后点击"添加燃料"按钮。</p>
                </div>
                <div>
                    <h4 class="font-semibold text-primary">3. 执行计算</h4>
                    <p class="text-gray-600">点击"开始计算"按钮，系统将根据您的参数进行碳排放合规计算。</p>
                </div>
                <div>
                    <h4 class="font-semibold text-primary">4. 查看结果</h4>
                    <p class="text-gray-600">计算完成后，您可以查看结果摘要、排放趋势图表、详细计算步骤和逐年合规数据。</p>
                </div>
                <div>
                    <h4 class="font-semibold text-primary">5. 导出数据</h4>
                    <p class="text-gray-600">点击"导出Excel"按钮，将所有计算结果保存为Excel文件。</p>
                </div>
            </div>
        </div>
    </div>

    <script>
        // === 数据模型 ===
        const appData = {
            reductionTarget: 20, // %
            rewardMultiplier: 2,
            compliancePool: 2, // %
            carbonPrice: 100, // USD/tCO2
            imoReferenceIntensity: 91.5, // gCO2e/MJ
            znzShare: 0, // %
            znzIntensity: 10, // gCO2e/MJ
            fuelMix: [
                { type: 'HFO', share: 40, lhv: 40.4, ef: 94.0 },
                { type: 'VLSFO', share: 40, lhv: 41.2, ef: 92.6 },
                { type: 'MDO', share: 20, lhv: 42.7, ef: 90.0 }
            ],
            years: Array.from({ length: 26 }, (_, i) => 2025 + i),
            yearlyResults: []
        };

        // 预定义燃料属性参考数据
        const fuelReferenceData = {
            "HFO": { lhv: 40.4, ef: 94.0, description: "重质燃料油，传统高硫燃料" },
            "VLSFO": { lhv: 41.2, ef: 92.6, description: "超低硫燃料油，符合IMO 2020硫限制" },
            "MDO": { lhv: 42.7, ef: 90.0, description: "船用柴油，低硫燃料" },
            "LNG": { lhv: 50.0, ef: 70.0, description: "液化天然气，低碳替代燃料" },
            "Bio-diesel": { lhv: 37.0, ef: 22.3, description: "生物柴油，可再生燃料" },
            "Bio-methane": { lhv: 55.0, ef: 20.6, description: "生物甲烷，低碳替代燃料" },
            "ZNZ": { lhv: 40.0, ef: 10.0, description: "零碳燃料，几乎无碳排放" }
        };

        // === 图表实例 ===
        let emissionChart = null;

        // === DOM 元素 ===
        const reductionTargetSlider = document.getElementById('reduction-target');
        const reductionTargetValue = document.getElementById('reduction-target-value');
        const addFuelBtn = document.getElementById('add-fuel');
        const fuelMixTable = document.getElementById('fuel-mix-table');
        const startCalculationBtn = document.getElementById('start-calculation');
        const resetAllBtn = document.getElementById('reset-all');
        const exportExcelBtn = document.getElementById('export-excel');
        const helpButton = document.getElementById('help-button');
        const closeHelpBtn = document.getElementById('close-help');
        const helpModal = document.getElementById('help-modal');

        // === 初始化页面 ===
        function init() {
            // 初始化燃料表格
            renderFuelMixTable();

            // 初始化滑块值显示
            reductionTargetValue.textContent = `${reductionTargetSlider.value}%`;

            // 初始化图表
            initChart();

            // 绑定事件监听
            bindEvents();
        }

        // 新增：初始化燃料数据
        function initFuelData() {
            // 默认选择HFO并填充数据
            const fuelSelect = document.getElementById('fuel-type');
            fuelSelect.value = 'HFO';
            const fuelData = fuelReferenceData['HFO'];

            if (fuelData) {
                document.getElementById('fuel-lhv').value = fuelData.lhv;
                document.getElementById('fuel-ef').value = fuelData.ef;
            }
        }

        // === 事件绑定 ===
        function bindEvents() {
            // 减排目标滑块事件
            reductionTargetSlider.addEventListener('input', (e) => {
                reductionTargetValue.textContent = `${e.target.value}%`;
                appData.reductionTarget = parseInt(e.target.value);
            });

            // 添加燃料按钮事件
            addFuelBtn.addEventListener('click', addFuel);

            // 开始计算按钮事件
            startCalculationBtn.addEventListener('click', calculate);

            // 重置按钮事件
            resetAllBtn.addEventListener('click', resetAll);

            // 导出Excel按钮事件
            exportExcelBtn.addEventListener('click', exportToExcel);

            // 帮助按钮事件
            helpButton.addEventListener('click', () => {
                helpModal.classList.remove('hidden');
            });

            // 关闭帮助模态框事件
            closeHelpBtn.addEventListener('click', () => {
                helpModal.classList.add('hidden');
            });

            // 点击模态框外部关闭
            helpModal.addEventListener('click', (e) => {
                if (e.target === helpModal) {
                    helpModal.classList.add('hidden');
                }
            });

            // 燃料类型选择事件监听
            document.getElementById('fuel-type').addEventListener('change', function() {
                const fuelType = this.value;
                const fuelData = fuelReferenceData[fuelType];

                if (fuelData) {
                    document.getElementById('fuel-lhv').value = fuelData.lhv;
                    document.getElementById('fuel-ef').value = fuelData.ef;
                }
            });
        }

        // === 燃料管理 ===
        function addFuel() {
            const fuelType = document.getElementById('fuel-type').value;
            const fuelShare = parseFloat(document.getElementById('fuel-share').value);
            const fuelLhv = parseFloat(document.getElementById('fuel-lhv').value);
            const fuelEf = parseFloat(document.getElementById('fuel-ef').value);

            // 检查份额总和是否超过100%
            const currentTotalShare = appData.fuelMix.reduce((sum, fuel) => sum + fuel.share, 0);
            if (currentTotalShare + fuelShare > 100) {
                alert(`燃料份额总和不能超过100%，当前已使用 ${currentTotalShare}%`);
                return;
            }

            // 添加燃料
            appData.fuelMix.push({
                type: fuelType,
                share: fuelShare,
                lhv: fuelLhv,
                ef: fuelEf
            });

            // 重新渲染表格
            renderFuelMixTable();
        }

        function removeFuel(index) {
            appData.fuelMix.splice(index, 1);
            renderFuelMixTable();
        }

        function renderFuelMixTable() {
            fuelMixTable.innerHTML = '';

            appData.fuelMix.forEach((fuel, index) => {
                const row = document.createElement('tr');
                row.className = index % 2 === 0 ? 'bg-white' : 'bg-gray-50';
                row.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="flex items-center">
                            <div class="h-8 w-8 rounded-full bg-primary/10 flex items-center justify-center mr-3">
                                <i class="fa-solid fa-oil-can text-primary"></i>
                            </div>
                            <div>
                                <div class="text-sm font-medium text-gray-900">${fuel.type}</div>
                            </div>
                        </div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">${fuel.share}%</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">${fuel.lhv} MJ/kg</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">${fuel.ef} gCO₂eq/MJ</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                        <button onclick="removeFuel(${index})" class="text-red-600 hover:text-red-900">
                            <i class="fa-solid fa-trash"></i>
                        </button>
                    </td>
                `;
                fuelMixTable.appendChild(row);
            });
        }

        // === 图表初始化 ===
        function initChart() {
            const ctx = document.getElementById('emission-chart').getContext('2d');

            // 销毁现有图表（如果存在）
            if (emissionChart) {
                emissionChart.destroy();
            }

            // 创建新图表
            emissionChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: appData.years,
                    datasets: [
                        {
                            label: '基准强度',
                            data: appData.years.map(() => appData.imoReferenceIntensity),
                            borderColor: '#165DFF',
                            backgroundColor: 'rgba(22, 93, 255, 0.1)',
                            borderWidth: 2,
                            tension: 0.1,
                            fill: false
                        },
                        {
                            label: '目标强度',
                            data: [],
                            borderColor: '#36CFC9',
                            backgroundColor: 'rgba(54, 207, 201, 0.1)',
                            borderWidth: 2,
                            tension: 0.1,
                            fill: false
                        },
                        {
                            label: '实际强度',
                            data: [],
                            borderColor: '#FF7D00',
                            backgroundColor: 'rgba(255, 125, 0, 0.1)',
                            borderWidth: 2,
                            tension: 0.1,
                            fill: false
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'top',
                        },
                        tooltip: {
                            mode: 'index',
                            intersect: false,
                        }
                    },
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: '年份'
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: '强度 (gCO₂eq/MJ)'
                            }
                        }
                    }
                }
            });
        }

        // === 计算函数 ===
        function calculate() {
            // 清空之前的结果
            appData.yearlyResults = [];

            // 计算逐年结果
            for (let i = 0; i < appData.years.length; i++) {
                const year = appData. years[i];
                // 假设减排目标逐年递增，从20%到90%
                const reductionTarget = 20 + (i * (70 / (appData.years.length - 1)));

                // 计算目标强度
                const targetIntensity = appData.imoReferenceIntensity * (1 - reductionTarget / 100);

                // 计算实际强度
                let totalEnergy = 0;
                let totalEmissions = 0;

                appData.fuelMix.forEach(fuel => {
                    const share = fuel. share / 100; // 转为小数
                    totalEnergy += share * fuel.lhv;
                    totalEmissions += share * fuel.lhv * fuel.ef;
                });

                const normalIntensity = totalEmissions / totalEnergy;

                // 考虑ZNZ燃料的影响
                const znzShareDecimal = appData.znzShare / 100;
                const actualIntensity = (1 - znzShareDecimal) * normalIntensity + znzShareDecimal * appData.znzIntensity;

                // 调整目标强度（考虑合规池和奖励倍数）
                const adjustedTarget = (targetIntensity * (1 - appData.compliancePool / 100)) / (1 + znzShareDecimal * (appData.rewardMultiplier - 1));

                // 计算合规缺口
                        // 计算合规缺口
                const complianceGap = actualIntensity - adjustedTarget;

                // 计算合规成本（假设总能耗为1,000,000 MJ）
                const totalEnergyMJ = 1000000; // 假设值，实际应用中可以根据船舶类型和运营情况调整
                const complianceCost = complianceGap > 0 ?
                    complianceGap * totalEnergyMJ * 1e-6 * appData.carbonPrice : 0;

                // 保存结果
                appData.yearlyResults.push({
                    year,
                    baseIntensity: appData.imoReferenceIntensity,
                    targetIntensity,
                    adjustedTarget,
                    actualIntensity,
                    complianceGap,
                    complianceCost
                });
            }

            // 更新图表数据
            updateChart();

            // 更新摘要结果
            updateSummaryResults();

            // 生成详细计算步骤
            generateCalculationSteps();

            // 渲染逐年结果表格
            renderYearlyResultsTable();

            // 显示成功消息
            showSuccessMessage();
        }

        // 更新图表数据
        function updateChart() {
            if (!emissionChart) return;

            emissionChart.data.datasets[1].data = appData.yearlyResults.map(result => result.targetIntensity);
            emissionChart.data.datasets[2].data = appData.yearlyResults.map(result => result.actualIntensity);
            emissionChart.update();
        }

        // 更新摘要结果
        function updateSummaryResults() {
            // 使用最近一年的结果作为摘要
            const latestResult = appData.yearlyResults[appData.yearlyResults.length - 1] || {
                baseIntensity: appData.imoReferenceIntensity,
                targetIntensity: appData.imoReferenceIntensity * (1 - appData.reductionTarget / 100),
                actualIntensity: 0,
                complianceGap: 0,
                complianceCost: 0
            };

            document.getElementById('baseline-intensity').textContent = `${latestResult.baseIntensity.toFixed(1)} gCO₂eq/MJ`;
            document.getElementById('target-intensity').textContent = `${latestResult.targetIntensity.toFixed(1)} gCO₂eq/MJ`;
            document.getElementById('actual-intensity').textContent = `${latestResult.actualIntensity.toFixed(1)} gCO₂eq/MJ`;
            document.getElementById('compliance-gap').textContent = `${latestResult.complianceGap.toFixed(1)} gCO₂eq/MJ`;
            document.getElementById('compliance-cost').textContent = `USD ${formatNumber(latestResult.complianceCost.toFixed(0))}`;
        }

        // 生成详细计算步骤
        function generateCalculationSteps() {
            const stepsTable = document.getElementById('calculation-steps');
            stepsTable.innerHTML = '';

            // 假设使用最近一年的计算步骤
            const result = appData.yearlyResults[appData.yearlyResults.length - 1] || {};

            // 计算燃料混合的总能量和排放
            let totalEnergy = 0;
            let totalEmissions = 0;

            appData.fuelMix.forEach(fuel => {
                const share = fuel. share / 100; // 转为小数
                totalEnergy += share * fuel.lhv;
                totalEmissions += share * fuel.lhv * fuel.ef;
            });

            const normalIntensity = totalEnergy > 0 ? totalEmissions / totalEnergy : 0;
            const znzShareDecimal = appData.znzShare / 100;

            // 创建计算步骤
            const steps = [
                {
                    step: 1,
                    description: '计算目标强度',
                    calculation: `基准强度 × (1 - 减排目标百分比) = ${appData.imoReferenceIntensity} × (1 - ${appData.reductionTarget}%)`,
                    result: `${result.targetIntensity ? result.targetIntensity.toFixed(1) : '-'} gCO₂eq/MJ`
                },
                {
                    step: 2,
                    description: '计算燃料混合的加权平均排放强度',
                    calculation: `∑(每种燃料份额 × 低位热值 × 排放因子) / ∑(每种燃料份额 × 低位热值)`,
                    result: `${normalIntensity.toFixed(1)} gCO₂eq/MJ`
                },
                {
                    step: 3,
                    description: '考虑ZNZ燃料的影响',
                    calculation: `(1 - ZNZ份额) × 加权平均排放强度 + ZNZ份额 × ZNZ排放因子 = (1 - ${znzShareDecimal}) × ${normalIntensity.toFixed(1)} + ${znzShareDecimal} × ${appData.znzIntensity}`,
                    result: `${result.actualIntensity ? result.actualIntensity.toFixed(1) : '-'} gCO₂eq/MJ`
                },
                {
                    step: 4,
                    description: '调整目标强度（考虑合规池和奖励倍数）',
                    calculation: `目标强度 × (1 - 合规池百分比) / (1 + ZNZ份额 × (奖励倍数 - 1)) = ${result.targetIntensity ? result.targetIntensity.toFixed(1) : '-'} × (1 - ${appData.compliancePool}%) / (1 + ${znzShareDecimal} × (${appData.rewardMultiplier} - 1))`,
                    result: `${result.adjustedTarget ? result.adjustedTarget.toFixed(1) : '-'} gCO₂eq/MJ`
                },
                {
                    step: 5,
                    description: '计算合规缺口',
                    calculation: `实际强度 - 调整后目标强度 = ${result.actualIntensity ? result.actualIntensity.toFixed(1) : '-'} - ${result.adjustedTarget ? result.adjustedTarget.toFixed(1) : '-'}`,
                    result: `${result.complianceGap ? result.complianceGap.toFixed(1) : '-'} gCO₂eq/MJ`
                },
                {
                    step: 6,
                    description: '计算合规成本',
                    calculation: `合规缺口 × 总能耗 × 碳价 = ${result.complianceGap ? result.complianceGap.toFixed(1) : '-'} × ${formatNumber(1000000)} × ${appData.carbonPrice} ÷ 1,000,000`,
                    result: `USD ${result.complianceCost ? formatNumber(result.complianceCost.toFixed(0)) : '-'}`
                }
            ];

            // 渲染计算步骤
            steps.forEach(step => {
                const row = document.createElement('tr');
                row.className = step.Step % 2 === 0 ? 'bg-white' : 'bg-gray-50';
                row.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm font-medium text-gray-900">步骤 ${step.step}</div>
                    </td>
                    <td class="px-6 py-4">
                        <div class="text-sm text-gray-900">${step.description}</div>
                    </td>
                    <td class="px-6 py-4">
                        <div class="text-sm text-gray-900">${step.calculation}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm font-medium ${step.step === 5 && result.complianceGap < 0 ? 'text-green-600' : 'text-gray-900'}">${step.result}</div>
                    </td>
                `;
                stepsTable.appendChild(row);
            });
        }

        // 渲染逐年结果表格
        function renderYearlyResultsTable() {
            const resultsTable = document.getElementById('yearly-results');
            resultsTable.innerHTML = '';

            appData.yearlyResults.forEach((result, index) => {
                const row = document.createElement('tr');
                row.className = index % 2 === 0 ? 'bg-white' : 'bg-gray-50';
                row.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm font-medium text-gray-900">${result.year}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">${result.baseIntensity.toFixed(1)}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">${result.targetIntensity.toFixed(1)}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">${result.adjustedTarget.toFixed(1)}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">${result.actualIntensity.toFixed(1)}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm font-medium ${result.complianceGap < 0 ? 'text-green-600' : 'text-red-600'}">${result.complianceGap.toFixed(1)}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap">
                        <div class="text-sm text-gray-900">USD ${formatNumber(result.complianceCost.toFixed(0))}</div>
                    </td>
                `;
                resultsTable.appendChild(row);
            });
        }

        // 格式化数字（添加千位分隔符）
        function formatNumber(num) {
            return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
        }

        // 重置所有参数
        function resetAll() {
            // 重置数据模型
            appData.reductionTarget = 20;
            appData.rewardMultiplier = 2;
            appData.compliancePool = 2;
            appData.carbonPrice = 100;
            appData.imoReferenceIntensity = 91.5;
            appData.znzShare = 0;
            appData.znzIntensity = 10;
            appData.fuelMix = [
                { type: 'HFO', share: 40, lhv: 40.4, ef: 94.0 },
                { type: 'VLSFO', share: 40, lhv: 41.2, ef: 92.6 },
                { type: 'MDO', share: 20, lhv: 42.7, ef: 90.0 }
            ];
            appData.yearlyResults = [];

            // 重置UI
            reductionTargetSlider. value = 20;
            reductionTargetValue.textContent = '20%';
            document.getElementById('reward-multiplier').value = 2;
            document.getElementById('compliance-pool').value = 2;
            document.getElementById('carbon-price').value = 100;
            document.getElementById('imo-reference').value = 91.5;
            document.getElementById('znz-share').value = 0;
            document.getElementById('znz-intensity').value = 10;
            document.getElementById('fuel-type').value = 'HFO';
            document.getElementById('fuel-share').value = 40;
            document.getElementById('fuel-lhv').value = 40.4;
            document.getElementById('fuel-ef').value = 94.0;

            // 重新渲染表格
            renderFuelMixTable();

            // 重置图表
            initChart();

            // 重置结果区域
            document.getElementById('baseline-intensity').textContent = '91.5 gCO₂eq/MJ';
            document.getElementById('target-intensity').textContent = '73.2 gCO₂eq/MJ';
            document.getElementById('actual-intensity').textContent = '85.3 gCO₂eq/MJ';
            document.getElementById('compliance-gap').textContent = '12.1 gCO₂eq/MJ';
            document.getElementById('compliance-cost').textContent = 'USD 1,250,000';

            // 清空计算步骤和逐年结果
            document.getElementById('calculation-steps').innerHTML = '';
            document.getElementById('yearly-results').innerHTML = '';
        }

        // 导出Excel
        function exportToExcel() {
            // 创建工作簿对象
            const wb = XLSX.utils.book_new();

            // 创建摘要工作表
            const summaryData = [
                ['基准强度', `${appData.imoReferenceIntensity} gCO₂eq/MJ`],
                ['减排目标', `${appData.reductionTarget}%`],
                ['目标强度', `${appData.yearlyResults.length > 0 ? appData.yearlyResults[appData.yearlyResults.length - 1].targetIntensity.toFixed(1) : '-'} gCO₂eq/MJ`],
                ['实际强度', `${appData.yearlyResults.length > 0 ? appData.yearlyResults[appData.yearlyResults.length - 1].actualIntensity.toFixed(1) : '-'} gCO₂eq/MJ`],
                ['合规缺口', `${appData.yearlyResults.length > 0 ? appData.yearlyResults[appData.yearlyResults.length - 1].complianceGap.toFixed(1) : '-'} gCO₂eq/MJ`],
                ['合规成本', `USD ${appData.yearlyResults.length > 0 ? formatNumber(appData.yearlyResults[appData.yearlyResults.length - 1].complianceCost.toFixed(0)) : '-'}`]
            ];

            const summaryWS = XLSX.utils.aoa_to_sheet(summaryData);
            XLSX.utils.book_append_sheet(wb, summaryWS, '摘要');

            // 创建燃料配比工作表
            const fuelMixData = [['燃料类型', '份额 (%)', '低位热值 (MJ/kg)', '排放因子 (gCO₂eq/MJ)']];
            appData.fuelMix.forEach(fuel => {
                fuelMixData.push([fuel.type, fuel.share, fuel.lhv, fuel.ef]);
            });

            const fuelMixWS = XLSX.utils.aoa_to_sheet(fuelMixData);
            XLSX.utils.book_append_sheet(wb, fuelMixWS, '燃料配比');

            // 创建逐年结果工作表
            if (appData.yearlyResults.length > 0) {
                const yearlyData = [['年份', '基准强度 (gCO₂eq/MJ)', '目标强度 (gCO₂eq/MJ)', '调整后目标 (gCO₂eq/MJ)', '实际强度 (gCO₂eq/MJ)', '合规缺口 (gCO₂eq/MJ)', '合规成本 (USD)']];

                appData.yearlyResults.forEach(result => {
                    yearlyData.push([
                        result. year,
                        result.baseIntensity.toFixed(1),
                        result.targetIntensity.toFixed(1),
                        result.adjustedTarget.toFixed(1),
                        result.actualIntensity.toFixed(1),
                        result.complianceGap.toFixed(1),
                        result.complianceCost.toFixed(0)
                    ]);
                });

                const yearlyWS = XLSX.utils.aoa_to_sheet(yearlyData);
                XLSX.utils.book_append_sheet(wb, yearlyWS, '逐年结果');
            }

            // 创建详细计算步骤工作表
            const stepsData = [['步骤', '描述', '计算过程', '结果']];
            const stepsTable = document.getElementById('calculation-steps');
            const rows = stepsTable.querySelectorAll('tr');

            rows.forEach(row => {
                const cells = row.querySelectorAll('td');
                if (cells.length >= 4) {
                    stepsData.push([
                        cells[0].textContent,
                        cells[1].textContent,
                        cells[2].textContent,
                        cells[3].textContent
                    ]);
                }
            });

            const stepsWS = XLSX.utils.aoa_to_sheet(stepsData);
            XLSX.utils.book_append_sheet(wb, stepsWS, '计算步骤');

            // 导出Excel文件
            XLSX.writeFile(wb, '船舶碳排放合规计算结果.xlsx');

            // 显示导出成功消息
            showSuccessMessage('Excel文件已成功导出！');
        }

        // 显示成功消息
        function showSuccessMessage(message = '计算完成！') {
            // 创建消息元素
            const messageElement = document.createElement('div');
            messageElement.className = 'fixed bottom-4 right-4 bg-green-500 text-white px-6 py-3 rounded-lg shadow-lg z-50 transform transition-all duration-500 translate-y-20 opacity-0';
            messageElement.innerHTML = `
                <div class="flex items-center">
                    <i class="fa-solid fa-check-circle mr-2"></i>
                    <span>${message}</span>
                </div>
            `;

            // 添加到页面
            document.body.appendChild(messageElement);

            // 显示消息
            setTimeout(() => {
                messageElement.classList.remove('translate-y-20', 'opacity-0');
            }, 100);

            // 3秒后隐藏消息
            setTimeout(() => {
                messageElement.classList.add('translate-y-20', 'opacity-0');
                setTimeout(() => {
                    document.body.removeChild(messageElement);
                }, 500);
            }, 3000);
        }

        // 页面加载完成后初始化
        document.addEventListener('DOMContentLoaded', init);
    </script>
</body>
</html>
