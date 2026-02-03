<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>今天吃什么</title>
    <style>
        body { font-family: sans-serif; background: #f8f9fa; display: flex; flex-direction: column; align-items: center; padding: 20px; margin: 0; }
        .wheel-box { position: relative; width: 300px; height: 300px; margin: 20px 0; }
        #wheel { width: 100%; height: 100%; border-radius: 50%; transition: transform 4s cubic-bezier(0.15, 0, 0.15, 1); border: 5px solid #fff; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .arrow { position: absolute; top: -10px; left: 50%; transform: translateX(-50%); width: 0; height: 0; border-left: 10px solid transparent; border-right: 10px solid transparent; border-top: 20px solid #ff4757; z-index: 10; }
        .io-zone { width: 95%; max-width: 400px; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); box-sizing: border-box; }
        textarea { width: 100%; height: 80px; margin: 10px 0; padding: 10px; box-sizing: border-box; border-radius: 8px; border: 1px solid #ddd; font-size: 14px; }
        .btn-group { display: flex; gap: 10px; margin-top: 10px; }
        button { flex: 1; padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; transition: 0.2s; }
        button:active { transform: scale(0.95); }
        .btn-spin { background: #ff4757; color: white; font-size: 1.1rem; }
        .btn-save { background: #2ed573; color: white; }
        #result { font-size: 1.5rem; color: #333; margin: 10px 0; height: 1.8rem; font-weight: bold; }
        #status { color: #2ed573; font-size: 0.8rem; height: 1rem; margin-bottom: 5px; }
    </style>
</head>
<body>

    <div id="result">今天吃什么？</div>

    <div class="wheel-box">
        <div class="arrow"></div>
        <canvas id="wheel" width="600" height="600"></canvas>
    </div>

    <div class="io-zone">
        <div id="status"></div>
        <textarea id="menuInput" placeholder="输入菜单，用逗号隔开..."></textarea>
        <div class="btn-group">
            <button class="btn-save" onclick="saveMenu()">💾 保存我的菜单</button>
        </div>
        <button class="btn-spin" id="spinBtn" onclick="spin()" style="width:100%; margin-top:15px;">开始抽取</button>
    </div>

<script>
    const canvas = document.getElementById('wheel');
    const ctx = canvas.getContext('2d');
    const input = document.getElementById('menuInput');
    const status = document.getElementById('status');
    const defaultMenu = "黄焖鸡, 螺蛳粉, 汉堡, 撒浪嘿萨马拉, 兰州拉面, 减脂餐, 麻辣烫, 疯狂星期四";
    
    let items = [];
    let currentDeg = 0;

    function init() {
        const saved = localStorage.getItem('myCustomMenu');
        input.value = saved ? saved : defaultMenu;
        draw();
    }

    function saveMenu() {
        localStorage.setItem('myCustomMenu', input.value);
        status.innerText = "✅ 菜单已保存，下次打开还在！";
        draw();
        setTimeout(() => status.innerText = "", 2000);
    }

    function draw() {
        items = input.value.split(/[，,]/).filter(i => i.trim() !== "");
        if(items.length < 2) return;
        const step = (Math.PI * 2) / items.length;
        const colors = ['#ff9f43', '#ee5253', '#0abde3', '#10ac84', '#5f27cd', '#ff9ff3', '#54a0ff', '#00d2d3'];
        
        ctx.clearRect(0, 0, 600, 600);
        items.forEach((text, i) => {
            ctx.save();
            ctx.beginPath();
            ctx.fillStyle = colors[i % colors.length];
            ctx.moveTo(300, 300);
            ctx.arc(300, 300, 290, i * step, (i + 1) * step);
            ctx.fill();
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 2;
            ctx.stroke();
            
            ctx.translate(300, 300);
            ctx.rotate(i * step + step / 2);
            ctx.fillStyle = "white";
            ctx.font = "bold 28px sans-serif";
            ctx.textAlign = "right";
            ctx.fillText(text.trim().substring(0, 6), 250, 10);
            ctx.restore();
        });
    }

    function spin() {
        const btn = document.getElementById('spinBtn');
        btn.disabled = true;
        const extra = Math.floor(Math.random() * 360) + 3600;
        currentDeg += extra;
        canvas.style.transform = `rotate(${currentDeg}deg)`;
        
        setTimeout(() => {
            btn.disabled = false;
            const actual = currentDeg % 360;
            const step = 360 / items.length;
            const index = Math.floor((360 - (actual % 360) + 270) % 360 / step);
            document.getElementById('result').innerText = "🎉 决定了： " + items[index];
        }, 4000);
    }

    init();
</script>
</body>
</html>
