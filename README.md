<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>私房菜决策编辑器</title>
    <link rel="apple-touch-icon" href="logo.png">
<link rel="icon" type="image/png" href="logo.png">

<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="美食大转盘">
    <style>
        :root { --p: #007AFF; --s: #2ed573; --bg: #f8fafd; --text: #2d3436; }
        body { 
            margin: 0; font-family: -apple-system, sans-serif; 
            background: var(--bg); color: var(--text);
            display: flex; flex-direction: column; align-items: center; 
            min-height: 100vh; padding: 20px; box-sizing: border-box;
            overflow-x: hidden;
        }

        #food-res { 
            font-size: 2.2rem; font-weight: 800; color: #ff4757; 
            margin: 10px 0; height: 3.5rem; text-align: center;
        }

        .app-container { width: 100%; max-width: 1000px; display: flex; flex-direction: column; gap: 20px; }

        /* --- 动态剧场布局核心 --- */
        .display-stage { 
            display: flex; 
            width: 100%;
            align-items: center; 
            min-height: 420px;
            position: relative;
        }

        /* 转盘包裹层：初始宽度 100% 且内部居中 */
        .wheel-section { 
            width: 100%; 
            display: flex; 
            flex-direction: column; 
            align-items: center; 
            transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
            flex-shrink: 0;
        }

        /* 抽取结束后的状态：转盘缩减宽度并靠左 */
        .display-stage.has-result .wheel-section {
            width: 45%; 
        }

        .wheel-box { position: relative; width: 340px; height: 340px; }
        #wheel { 
            width: 100%; height: 100%; border-radius: 50%; border: 10px solid #fff; 
            box-shadow: 0 15px 45px rgba(0,0,0,0.1); 
            transition: transform 4s cubic-bezier(0.15, 0, 0.15, 1); 
        }
        .arrow { position: absolute; top: -18px; left: 50%; transform: translateX(-50%); width: 0; height: 0; border-left: 18px solid transparent; border-right: 18px solid transparent; border-top: 35px solid #ff4757; z-index: 10; }

        /* 右侧展示区：初始宽度 0，隐藏 */
        .recipe-display-area { 
            width: 0;
            opacity: 0;
            overflow: hidden;
            transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
            white-space: nowrap; /* 防止动画时文字换行闪烁 */
        }

        /* 抽取结束后：右侧扩展宽度 */
        .display-stage.has-result .recipe-display-area { 
            width: 55%;
            opacity: 1;
            padding-left: 20px;
        }

        .card { background: white; padding: 25px; border-radius: 25px; box-shadow: 0 10px 30px rgba(0,0,0,0.05); white-space: normal; }
        .recipe-card { border-left: 8px solid var(--p); }

        /* --- 编辑区样式 --- */
        .editor-panel { margin-top: 30px; border-top: 1px solid #ddd; padding-top: 20px; width: 100%;}
        .tag-container { display: flex; flex-wrap: wrap; gap: 10px; margin: 15px 0; }
        .menu-tag { padding: 8px 16px; background: #eee; border-radius: 20px; cursor: pointer; transition: 0.2s; font-size: 0.9rem; }
        .menu-tag.active { background: var(--p); color: white; }
        .edit-box { background: #fff; padding: 20px; border-radius: 20px; display: none; border: 1px solid #eee; margin-top: 10px; }
        .edit-box.active { display: block; animation: fadeIn 0.3s; }
        textarea { width: 100%; border: 1px solid #ddd; border-radius: 12px; padding: 12px; font-size: 1rem; box-sizing: border-box; outline: none; }
        .btn-go { background: var(--p); color: white; border: none; padding: 15px 40px; border-radius: 30px; font-weight: bold; font-size: 1.2rem; cursor: pointer; margin-top: 20px; box-shadow: 0 5px 15px rgba(0,122,255,0.3); }
        .btn-save { background: var(--s); color: white; border: none; padding: 12px; border-radius: 10px; cursor: pointer; margin-top: 20px; width: 100%; font-weight: bold;}

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        @media (max-width: 800px) {
            .display-stage.has-result { flex-direction: column; }
            .display-stage.has-result .wheel-section, .display-stage.has-result .recipe-display-area { width: 100%; padding: 0; }
        }
    </style>
</head>
<body>

    <div id="food-res">今天吃什么？</div>
    
    <div class="app-container">
        <div id="stage" class="display-stage">
            <div class="wheel-section">
                <div class="wheel-box">
                    <div class="arrow"></div>
                    <canvas id="wheel" width="600" height="600"></canvas>
                </div>
                <button class="btn-go" id="spinBtn" onclick="spinWheel()">🚀 开始抽取</button>
            </div>

            <div class="recipe-display-area">
                <div class="card recipe-card">
                    <h3 id="res-name" style="margin:0 0 10px 0; color:var(--p)">做法参考</h3>
                    <p id="res-step" style="line-height:1.8; color:#555; white-space:pre-line; margin:0;"></p>
                </div>
            </div>
        </div>

        <div class="editor-panel">
            <div style="font-weight:bold; color:#666;">✍️ 管理菜单（用逗号隔开菜名）</div>
            <textarea id="menuInput" style="height: 50px; margin-top:10px;" oninput="syncTags()" placeholder="输入菜名..."></textarea>
            
            <div style="margin-top:20px; font-weight:bold; color:#666;">📖 点击标签编辑详细做法：</div>
            <div id="tagContainer" class="tag-container"></div>
            
            <div id="editArea"></div>
            <button class="btn-save" onclick="saveData()">💾 保存全部修改</button>
        </div>
    </div>

<script>
    let deg = 0;
    let recipes = JSON.parse(localStorage.getItem('recipeMapV6') || '{}');

    function init() {
        const menu = localStorage.getItem('menuListV6') || "黄焖鸡, 螺蛳粉, 汉堡, 麻辣烫";
        document.getElementById('menuInput').value = menu;
        syncTags();
        drawWheel();
    }

    function syncTags() {
        const names = document.getElementById('menuInput').value.split(/[，,]/).filter(i => i.trim() !== "");
        const container = document.getElementById('tagContainer');
        const editArea = document.getElementById('editArea');
        container.innerHTML = ''; editArea.innerHTML = '';

        names.forEach(name => {
            name = name.trim();
            const tag = document.createElement('div');
            tag.className = 'menu-tag';
            tag.innerText = name;
            tag.onclick = () => {
                document.querySelectorAll('.menu-tag').forEach(t => t.classList.remove('active'));
                document.querySelectorAll('.edit-box').forEach(b => b.classList.remove('active'));
                tag.classList.add('active');
                document.getElementById(`edit-${name}`).classList.add('active');
            };
            container.appendChild(tag);

            const box = document.createElement('div');
            box.className = 'edit-box';
            box.id = `edit-${name}`;
            box.innerHTML = `<textarea oninput="recipes['${name}'] = this.value">${recipes[name] || ''}</textarea>`;
            editArea.appendChild(box);
        });
        drawWheel();
    }

    function saveData() {
        localStorage.setItem('menuListV6', document.getElementById('menuInput').value);
        localStorage.setItem('recipeMapV6', JSON.stringify(recipes));
        alert("💾 保存成功！");
    }

    function drawWheel() {
        const canvas = document.getElementById('wheel');
        const ctx = canvas.getContext('2d');
        const items = document.getElementById('menuInput').value.split(/[，,]/).filter(i => i.trim() !== "");
        if(!items.length) return;
        const step = (Math.PI * 2) / items.length;
        const colors = ['#FF7675', '#00CEC9', '#FAB1A0', '#0984E3', '#6C5CE7', '#FDCB6E'];
        ctx.clearRect(0,0,600,600);
        items.forEach((t, i) => {
            ctx.save(); ctx.beginPath(); ctx.fillStyle = colors[i % colors.length];
            ctx.moveTo(300,300); ctx.arc(300,300,290,i*step,(i+1)*step); ctx.fill();
            ctx.translate(300,300); ctx.rotate(i*step + step/2);
            ctx.fillStyle="white"; ctx.font="bold 30px sans-serif"; ctx.textAlign="right";
            ctx.fillText(t.trim().substring(0,6), 260, 10); ctx.restore();
        });
    }

    function spinWheel() {
        const items = document.getElementById('menuInput').value.split(/[，,]/).filter(i => i.trim() !== "");
        if(!items.length) return;
        const btn = document.getElementById('spinBtn');
        const stage = document.getElementById('stage');
        btn.disabled = true;
        
        // 关键逻辑：开始转动时重置居中
        stage.classList.remove('has-result');
        document.getElementById('food-res').innerText = "🍽️ 正在为您挑选...";

        const rotation = Math.floor(Math.random()*360) + 3600;
        deg += rotation;
        document.getElementById('wheel').style.transform = `rotate(${deg}deg)`;
        
        setTimeout(() => {
            btn.disabled = false;
            const index = Math.floor((360 - (deg % 360) + 270) % 360 / (360 / items.length));
            const result = items[index].trim();
            
            // 结果出来，整体向左挤压，右侧展开
            stage.classList.add('has-result');
            document.getElementById('food-res').innerText = "🎉 决定了！";
            document.getElementById('res-name').innerText = "🍳 " + result;
            document.getElementById('res-step').innerText = recipes[result] || "暂无做法，请在下方点击标签编辑。";
        }, 4000);
    }

    init();
</script>
</body>
</html>
