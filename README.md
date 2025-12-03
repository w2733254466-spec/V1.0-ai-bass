
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>鲈鱼战术大师 DeepSeek版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        body { background-color: #f8fafc; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; -webkit-tap-highlight-color: transparent; }
        .card { background: white; border-radius: 16px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); overflow: hidden; transition: transform 0.2s; }
        .select-btn { border: 1.5px solid #e2e8f0; transition: all 0.2s; background: white; color: #64748b; }
        .select-btn.active { background-color: #0f172a; color: white; border-color: #0f172a; transform: scale(1.02); box-shadow: 0 4px 12px rgba(15, 23, 42, 0.2); }
        .tab-btn { border-bottom: 3px solid transparent; color: #94a3b8; transition: all 0.3s; }
        .tab-btn.active { border-color: #3b82f6; color: #3b82f6; font-weight: 700; }
        .fade-in { animation: fadeIn 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards; opacity: 0; transform: translateY(10px); }
        @keyframes fadeIn { to { opacity: 1; transform: translateY(0); } }
        /* 拟饵颜色圆点 */
        .color-dot { width: 12px; height: 12px; border-radius: 50%; display: inline-block; border: 1px solid rgba(0,0,0,0.1); }
    </style>
</head>
<body class="pb-32 text-slate-800">

    <div id="setup-screen" class="p-6 max-w-lg mx-auto min-h-screen flex flex-col justify-center bg-white">
        <div class="mb-10">
            <h1 class="text-4xl font-black text-slate-900 tracking-tight">Bass<span class="text-blue-600">AI</span></h1>
            <p class="text-slate-400 font-medium text-sm mt-1">Powered by DeepSeek V3</p>
        </div>

        <div class="space-y-8">
            <div>
                <label class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 block">1. 选择时段</label>
                <div class="grid grid-cols-4 gap-2">
                    <button onclick="setOption('time', '早上', this)" class="select-btn p-3 rounded-xl text-sm font-bold">🌅 早上</button>
                    <button onclick="setOption('time', '中午', this)" class="select-btn p-3 rounded-xl text-sm font-bold">☀️ 中午</button>
                    <button onclick="setOption('time', '下午', this)" class="select-btn p-3 rounded-xl text-sm font-bold">🌇 下午</button>
                    <button onclick="setOption('time', '晚上', this)" class="select-btn p-3 rounded-xl text-sm font-bold">🌙 晚上</button>
                </div>
            </div>

            <div>
                <label class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 block">2. 预计时长</label>
                <div class="grid grid-cols-4 gap-2">
                    <button onclick="setOption('duration', '1小时', this)" class="select-btn p-3 rounded-xl text-sm font-bold">1h</button>
                    <button onclick="setOption('duration', '2小时', this)" class="select-btn p-3 rounded-xl text-sm font-bold">2h</button>
                    <button onclick="setOption('duration', '3小时', this)" class="select-btn p-3 rounded-xl text-sm font-bold">3h</button>
                    <button onclick="setOption('duration', '全天', this)" class="select-btn p-3 rounded-xl text-sm font-bold">全天</button>
                </div>
            </div>

            <div class="grid grid-cols-2 gap-4">
                <div>
                    <label class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 block">3. 水域</label>
                    <div class="grid grid-cols-1 gap-2">
                        <button onclick="setOption('venue', '野钓自然水域', this)" class="select-btn p-3 rounded-xl text-sm font-bold">🏞️ 野钓</button>
                        <button onclick="setOption('venue', '黑坑管理场', this)" class="select-btn p-3 rounded-xl text-sm font-bold">🎣 管理场</button>
                    </div>
                </div>
                <div>
                    <label class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 block">4. 模式</label>
                    <div class="grid grid-cols-1 gap-2">
                        <button onclick="setOption('mode', '岸钓', this)" class="select-btn p-3 rounded-xl text-sm font-bold">👟 岸钓</button>
                        <button onclick="setOption('mode', '船钓', this)" class="select-btn p-3 rounded-xl text-sm font-bold">🚤 船钓</button>
                    </div>
                </div>
            </div>
        </div>

        <button onclick="startAnalysis()" class="mt-12 w-full bg-slate-900 text-white py-4 rounded-2xl font-bold text-lg shadow-xl shadow-slate-300 hover:bg-black transition-all active:scale-95 flex items-center justify-center gap-2">
            生成战术方案 <i class="fas fa-arrow-right"></i>
        </button>
    </div>

    <div id="loading-screen" class="hidden fixed inset-0 bg-white/95 backdrop-blur z-50 flex flex-col items-center justify-center p-6 text-center">
        <div class="w-16 h-16 border-4 border-blue-100 border-t-blue-600 rounded-full animate-spin mb-6"></div>
        <h2 class="text-2xl font-bold text-slate-800" id="loading-text">连接卫星数据...</h2>
        <p class="text-slate-500 text-sm mt-3 max-w-xs mx-auto leading-relaxed" id="loading-sub">正在获取当前坐标的气温与气压</p>
    </div>

    <div id="dashboard-screen" class="hidden max-w-lg mx-auto min-h-screen relative">
        
        <div class="bg-white p-6 rounded-b-3xl shadow-sm z-20 sticky top-0 border-b border-gray-100">
            <div class="flex justify-between items-start mb-2">
                <div>
                    <div class="flex items-center gap-2 mb-1">
                        <span class="inline-block w-2 h-2 rounded-full bg-green-500 animate-pulse"></span>
                        <span class="text-xs text-slate-400 uppercase tracking-wider font-bold">实时环境</span>
                    </div>
                    <div class="flex items-baseline gap-2">
                        <span class="text-4xl font-black text-slate-800" id="disp-temp">--°</span>
                        <span class="text-sm font-medium text-slate-500" id="disp-desc">--</span>
                    </div>
                </div>
                <div class="text-right">
                    <div class="bg-blue-50 px-3 py-1 rounded-lg inline-block border border-blue-100">
                        <span class="text-blue-700 font-bold text-xl" id="disp-pressure">--</span>
                        <span class="text-blue-600 text-xs">hPa</span>
                    </div>
                    <p class="text-[10px] text-slate-400 mt-1 font-mono">LIVE DATA</p>
                </div>
            </div>
            
            <div class="flex gap-4 mt-4 pt-4 border-t border-gray-50">
                <div class="flex items-center gap-2 text-xs text-slate-600 font-bold bg-slate-100 px-2 py-1 rounded">
                    <i class="fas fa-wind text-slate-400"></i> <span id="disp-wind">--</span>
                </div>
                <div class="ml-auto text-xs text-slate-400 py-1" id="context-tag">--</div>
            </div>
        </div>

        <div class="bg-slate-50 sticky top-[150px] z-10 flex px-2 pt-2 gap-2">
            <button onclick="renderTab('A')" id="tab-A" class="tab-btn active flex-1 py-3 text-sm text-center bg-white rounded-t-lg shadow-sm">方案 A<br><span class="text-[10px] font-normal opacity-70">首选推荐</span></button>
            <button onclick="renderTab('B')" id="tab-B" class="tab-btn flex-1 py-3 text-sm text-center bg-white/50 rounded-t-lg">方案 B<br><span class="text-[10px] font-normal opacity-70">备用策略</span></button>
            <button onclick="renderTab('C')" id="tab-C" class="tab-btn flex-1 py-3 text-sm text-center bg-white/50 rounded-t-lg">方案 C<br><span class="text-[10px] font-normal opacity-70">高压精细</span></button>
        </div>

        <div id="content-area" class="p-4 space-y-4 pb-32 bg-slate-50 min-h-[500px]">
            </div>

        <div class="fixed bottom-6 left-4 right-4 max-w-[480px] mx-auto flex gap-3 z-30">
            <button onclick="retryStrategy()" class="flex-1 bg-slate-800 text-white py-3.5 rounded-2xl font-bold shadow-xl shadow-slate-900/20 active:scale-95 transition-transform flex items-center justify-center gap-2 text-sm">
                <i class="fas fa-skull-crossbones text-gray-400"></i> 没钓到鱼，AI复盘
            </button>
            <button onclick="logCatch()" class="w-14 h-14 bg-blue-600 rounded-2xl shadow-xl shadow-blue-500/30 flex items-center justify-center text-white text-xl active:scale-90 transition-transform">
                <i class="fas fa-camera"></i>
            </button>
        </div>
    </div>

    <script>
        // ================= 配置区 =================
        const DEEPSEEK_API_KEY = 'sk-f0025c927f5a46048528c453defa12a6'; 
        const WEATHER_API_KEY = '1e2e1b277a2de43bedf7c3c3e6a20028'; 
        // =========================================

        let userState = { time: '', duration: '', venue: '', mode: '' };
        let weatherData = null;
        let aiStrategies = null;
        let isRetrying = false;

        // 拟饵图片库 (关键词匹配)
        const lureLib = {
            'vib': 'https://images.unsplash.com/photo-1599496032734-75eb9949826d?w=300&h=300&fit=crop',
            'minnow': 'https://images.unsplash.com/photo-1582213782179-e0d53f98f2ca?w=300&h=300&fit=crop',
            'worm': 'https://images.unsplash.com/photo-1601633596590-7561f0d366a6?w=300&h=300&fit=crop', // 软虫
            'jig': 'https://images.unsplash.com/photo-1528607929212-2636ec44253e?w=300&h=300&fit=crop', // 铅头/亮片
            'topwater': 'https://images.unsplash.com/photo-1498611688622-c3681464455f?w=300&h=300&fit=crop', // 波爬
            'crank': 'https://plus.unsplash.com/premium_photo-1661962360408-012053073740?w=300&h=300&fit=crop', // 胖子
            'default': 'https://images.unsplash.com/photo-1535591273665-5f5954ddfd85?w=300&h=300&fit=crop'
        };

        function setOption(key, value, btn) {
            userState[key] = value;
            btn.parentElement.querySelectorAll('button').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
        }

        async function startAnalysis() {
            if (!userState.time || !userState.duration || !userState.venue || !userState.mode) {
                alert('请完成所有选项选择！');
                return;
            }

            document.getElementById('setup-screen').classList.add('hidden');
            document.getElementById('loading-screen').classList.remove('hidden');

            // 优先尝试获取真实定位
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    (pos) => fetchWeather(pos.coords.latitude, pos.coords.longitude),
                    () => {
                        // 如果用户拒绝，使用默认坐标（上海淀山湖附近）
                        console.warn("无法获取定位，使用默认坐标");
                        fetchWeather(31.11, 120.94); 
                    }
                );
            } else {
                fetchWeather(31.11, 120.94);
            }
        }

        async function fetchWeather(lat, lon) {
            try {
                const res = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${WEATHER_API_KEY}&units=metric`);
                const data = await res.json();
                
                if (data.cod !== 200) throw new Error("天气API错误");

                weatherData = {
                    temp: Math.round(data.main.temp),
                    pressure: data.main.pressure,
                    desc: data.weather[0].description,
                    wind: data.wind.speed,
                    hum: data.main.humidity
                };
            } catch (e) {
                // 容错：如果天气API挂了，使用模拟数据
                weatherData = { temp: 24, pressure: 1012, desc: "多云(数据离线)", wind: 3, hum: 60 };
            }

            updateWeatherUI();
            
            // 开始调用 DeepSeek
            document.getElementById('loading-text').innerText = "DeepSeek 思考中...";
            document.getElementById('loading-sub').innerText = "正在根据气压与光照计算鱼层";
            await callDeepSeekAPI();
        }

        function updateWeatherUI() {
            document.getElementById('disp-temp').innerText = weatherData.temp + "°";
            document.getElementById('disp-desc').innerText = weatherData.desc;
            document.getElementById('disp-pressure').innerText = weatherData.pressure;
            document.getElementById('disp-wind').innerText = weatherData.wind + "m/s";
            document.getElementById('context-tag').innerText = `${userState.venue} · ${userState.mode}`;
        }

        // ================= 核心：调用 DeepSeek =================
        async function callDeepSeekAPI(retry = false) {
            const systemPrompt = `你是一位世界级的鲈钓锦标赛（Bassmaster Elite）职业选手。
你的任务是根据提供的环境数据，生成JSON格式的做钓策略。
请务必根据实际的【气压】和【温度】来动态调整策略，绝对不要使用千篇一律的预设回复。
如果气压低（<1005hPa），推荐慢速/底层；如果气压高（>1015hPa），推荐反应饵/中上层。
必须包含具体的【饵的颜色】推荐（例如：水清用自然色，水浑用黑色/查特色）。`;

            const userPrompt = `
当前环境：
- 气温：${weatherData.temp}°C
- 气压：${weatherData.pressure} hPa (请在分析中引用此数值)
- 场景：${userState.venue}
- 模式：${userState.mode}
- 时间：${userState.time}
- 时长：${userState.duration}

${retry ? '!!! 紧急修正：用户反馈刚才的策略没钓到鱼。请分析原因（是否鱼层判断错误？是否颜色不对？）。请给出完全不同的Plan C作为补救方案。' : ''}

请返回一个标准的 JSON 对象，不要包含 Markdown 格式，格式如下：
{
  "strategies": {
    "A": {
      "name": "方案名称",
      "reasoning": "一句话分析，必须明确提到当前气压 ${weatherData.pressure}hPa 如何影响了鱼的活性。",
      "timeline": [
        {"time": "前30分钟", "action": "具体操作", "lure_hint": "饵名简写"}
      ],
      "baits": [
        {
          "brand": "品牌(如Jackall)",
          "model": "型号(如TN60)",
          "type": "类型(英文，如 VIB, Minnow, Worm, Jig)",
          "color": "具体颜色(如: 银色, 绿南瓜)",
          "technique": "操作手法"
        }
      ]
    },
    "B": { ... },
    "C": { ... }
  }
}`;

            try {
                const response = await fetch('https://api.deepseek.com/chat/completions', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${DEEPSEEK_API_KEY}`
                    },
                    body: JSON.stringify({
                        model: "deepseek-chat",
                        messages: [
                            { role: "system", content: systemPrompt },
                            { role: "user", content: userPrompt }
                        ],
                        response_format: { type: "json_object" }, // 强制 JSON 模式，解决解析错误
                        temperature: 0.7 
                    })
                });

                if (!response.ok) throw new Error(`DeepSeek API Error: ${response.status}`);

                const data = await response.json();
                const jsonContent = JSON.parse(data.choices[0].message.content);
                
                aiStrategies = jsonContent.strategies;
                isRetrying = retry;

                document.getElementById('loading-screen').classList.add('hidden');
                document.getElementById('dashboard-screen').classList.remove('hidden');
                
                // 默认显示 A，如果是重试则显示 C
                renderTab(retry ? 'C' : 'A');
                if(retry) alert("DeepSeek 已重新复盘：切换到修正方案 C");

            } catch (error) {
                console.error(error);
                alert("AI 连接超时，请检查网络或 Key 额度。");
                document.getElementById('loading-screen').classList.add('hidden');
            }
        }

        // ================= 渲染逻辑 =================
        function renderTab(plan) {
            ['A', 'B', 'C'].forEach(p => {
                const btn = document.getElementById(`tab-${p}`);
                if (p === plan) {
                    btn.classList.add('active');
                    btn.classList.remove('bg-white/50');
                    btn.classList.add('bg-white');
                } else {
                    btn.classList.remove('active');
                    btn.classList.add('bg-white/50');
                    btn.classList.remove('bg-white');
                }
            });

            const data = aiStrategies[plan];
            const container = document.getElementById('content-area');

            // 1. 生成时间轴
            let timelineHtml = data.timeline.map((item, idx) => `
                <div class="flex gap-4 relative">
                    ${idx !== data.timeline.length - 1 ? '<div class="absolute left-[11px] top-8 bottom-[-16px] w-[2px] bg-slate-200"></div>' : ''}
                    <div class="w-6 h-6 rounded-full bg-blue-100 text-blue-600 flex items-center justify-center text-xs font-bold shrink-0 mt-1 border-2 border-white shadow-sm">${idx + 1}</div>
                    <div class="pb-6">
                        <div class="text-xs font-bold text-slate-400 uppercase mb-1">${item.time}</div>
                        <div class="text-slate-800 text-sm font-medium leading-relaxed">${item.action}</div>
                        <div class="text-blue-500 text-xs mt-1 font-bold flex items-center gap-1"><i class="fas fa-link"></i> ${item.lure_hint}</div>
                    </div>
                </div>
            `).join('');

            // 2. 生成饵料卡片 (新增颜色推荐)
            let baitsHtml = data.baits.map(bait => {
                // 图片匹配逻辑
                let imgKey = 'default';
                const t = (bait.type + ' ' + bait.model).toLowerCase();
                if (t.includes('vib') || t.includes('lv')) imgKey = 'vib';
                else if (t.includes('minnow') || t.includes('jerk')) imgKey = 'minnow';
                else if (t.includes('worm') || t.includes('senko') || t.includes('soft')) imgKey = 'worm';
                else if (t.includes('jig') || t.includes('rubber')) imgKey = 'jig';
                else if (t.includes('top') || t.includes('pop')) imgKey = 'topwater';
                else if (t.includes('crank')) imgKey = 'crank';

                return `
                <div class="card p-3 flex gap-3 border border-slate-100">
                    <div class="w-24 h-24 bg-gray-100 rounded-xl bg-cover bg-center shrink-0 shadow-inner" style="background-image: url('${lureLib[imgKey]}')"></div>
                    <div class="flex-1 min-w-0 flex flex-col justify-center">
                        <h4 class="font-bold text-slate-800 text-base leading-tight">${bait.brand} <br> <span class="text-blue-600 text-sm">${bait.model}</span></h4>
                        
                        <div class="mt-2 flex items-center gap-2">
                             <span class="text-xs bg-orange-50 text-orange-700 px-2 py-1 rounded border border-orange-100 font-bold">
                                🎨 ${bait.color}
                             </span>
                        </div>
                        
                        <div class="mt-2 text-xs text-slate-500 bg-slate-100 p-1.5 rounded">
                            <i class="fas fa-hand-sparkles"></i> 手法：${bait.technique}
                        </div>
                    </div>
                </div>
                `;
            }).join('');

            // 注入 HTML
            container.innerHTML = `
                <div class="card p-5 bg-gradient-to-br from-white to-blue-50 border border-blue-100 fade-in">
                    <div class="flex justify-between items-start mb-2">
                        <h3 class="font-bold text-lg text-slate-800">${data.name}</h3>
                        <i class="fas fa-brain text-blue-200 text-xl"></i>
                    </div>
                    <p class="text-sm text-slate-600 leading-relaxed font-medium">
                        <span class="bg-blue-600 text-white text-[10px] px-1.5 py-0.5 rounded mr-1">AI分析</span>
                        ${data.reasoning}
                    </p>
                </div>

                <div class="fade-in" style="animation-delay: 0.1s">
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm">
                        <h4 class="font-bold text-slate-800 mb-4 text-sm flex items-center gap-2">
                            <i class="far fa-clock text-blue-500"></i> ${userState.duration} 做钓规划
                        </h4>
                        ${timelineHtml}
                    </div>
                </div>

                <div class="fade-in" style="animation-delay: 0.2s">
                    <h4 class="font-bold text-slate-800 mb-3 text-sm flex items-center gap-2 ml-1">
                        <i class="fas fa-box-open text-blue-500"></i> DeepSeek 推荐装备
                    </h4>
                    <div class="space-y-3">${baitsHtml}</div>
                </div>
            `;
        }

        function retryStrategy() {
            if(!confirm("DeepSeek 将基于【当前环境 + 失败结果】重新推理。确定吗？")) return;
            
            document.getElementById('loading-screen').classList.remove('hidden');
            document.getElementById('dashboard-screen').classList.add('hidden');
            document.getElementById('loading-text').innerText = "正在进行战术复盘...";
            document.getElementById('loading-sub').innerText = "分析鱼情变化与操作失误可能性";
            
            callDeepSeekAPI(true); // 开启重试模式
        }

        function logCatch() {
            // 这里可以扩展为保存到LocalStorage
            const btn = event.currentTarget;
            btn.innerHTML = '<i class="fas fa-check"></i>';
            btn.classList.add('bg-green-500');
            alert("🎉 恭喜中鱼！位置已记录。");
        }
    </script>
</body>
</html>
