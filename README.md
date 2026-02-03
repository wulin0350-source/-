<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>我的解压盒子</title>
    <style>
        :root { --p: #007AFF; --bg: #f8f9fa; }
        body { margin: 0; font-family: sans-serif; background: var(--bg); }
        .nav { display: flex; position: fixed; bottom: 0; width: 100%; background: #fff; box-shadow: 0 -2px 10px rgba(0,0,0,0.1); z-index: 100; }
        .nav-btn { flex: 1; padding: 15px; text-align: center; cursor: pointer; color: #666; }
        .nav-btn.active { color: var(--p); font-weight: bold; border-top: 3px solid var(--p); }
        .page { display: none; height: 100vh; padding: 20px; box-sizing: border-box; overflow-y: auto; text-align: center; }
        .page.active { display: block; }
        .card { background: white; padding: 20px; border-radius: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); margin-top: 20px; }
        textarea { width: 100%; height: 80px; margin: 10px 0; padding: 10px; box-sizing: border-box; border-radius: 8px; border: 1px solid #ddd; }
        button { background: var(--p); color: white; border: none; padding: 12px 20px; border-radius: 10px; cursor: pointer; margin: 5px; }
        #wheel { width: 260px; height: 260px; border-radius: 50%; border: 5px solid #fff; margin: 20px auto; transition: 4s cubic-bezier(0.1, 0, 0, 1); box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
    </style>
</head>
<body>

<div id="p1" class="page active">
    <div class="card">
        <h3>💬 废话文学</h3>
        <p id="txt" style="min-height:60px;">点击开启废话模式</p>
        <button onclick="say()">再听一席话</button>
    </div>
</div>

<div id="p2" class="page">
    <h3>🫧 捏泡泡</h3>
    <div id="box" style="display: grid; grid-template-columns: repeat(5,1fr); gap: 10px; max-width: 300px; margin: auto;"></div>
    <button onclick="resetPop()" style="margin-top:20px;">换一张新的</button>
</div>

<div id="p3" class="page">
    <h3>🎡 吃饭大转盘</h3>
    <p id="food" style="font-weight:bold; color:red; height:1.2em;">今天吃什么？</p>
    <div style="position:relative; width:260px; margin:auto;">
        <div style="position:absolute; top:-10px; left:50%; transform:translateX(-50%); width:0; height:0; border-left:10px solid transparent; border-right:10px solid transparent; border-top:20px solid red; z-index:10;"></div>
        <canvas id="wheel" width="600" height="600"></canvas>
    </div>
    <div class="card">
        <textarea id="menuInput"></textarea>
        <button onclick="saveMenu()" style="background:#2ed573">💾 保存菜单</button>
        <button onclick="spin()">开始抽取</button>
    </div>
</div>

<div class="nav">
    <div class="nav-btn active" onclick="tab(1)">废话</div>
    <div class="nav-btn" onclick="tab(2)">泡泡</div>
    <div class="nav-btn" onclick="tab(3)">转盘</div>
</div>

<script>
    function tab(n) {
        document.querySelectorAll('.page, .nav-btn').forEach(el => el.classList.remove('active'));
        document.getElementById('p'+n).classList.add('active');
        document.querySelectorAll('.nav-btn')[n-1].classList.add('active');
        if(n===2 && !document.getElementById('box').innerHTML) resetPop();
        if(n===3) drawWheel();
    }

    const quotes = ["如果你愿意多花点时间了解我，你就会发现你多花了很多时间", "现在还没睡的人一定还没睡吧", "人只要活着，就一定还活着"];
    function say() { document.getElementById('txt').innerText = quotes[Math.floor(Math.random()*quotes.length)]; }

    function resetPop() {
        const b = document.getElementById('box'); b.innerHTML = '';
        for(let i=0; i<25; i++) {
            let s = document.createElement('div');
            s.style = "width:50px;height:50px;background:#ddd;border-radius:50%;box-shadow:inset -3px -3px 5px #bbb;cursor:pointer;";
            s.onclick = function() { this.style.boxShadow = "inset 3px 3px 5px #bbb"; };
            b.appendChild(s);
        }
    }

    const canvas = document.getElementById('wheel');
    const ctx = canvas.getContext('2d');
    const input = document.getElementById('menuInput');
    const defaultMenu = "黄焖鸡, 螺蛳粉, 汉堡, 兰州拉面, 披萨, 减脂餐, 麻辣烫, 烤肉";
    let deg = 0;

    function saveMenu() { localStorage.setItem('myMenu', input.value); drawWheel(); alert("保存成功！"); }
    
    function drawWheel() {
        const saved = localStorage.getItem('myMenu');
        input.value = saved ? saved : defaultMenu;
        const items = input.value.split(/[，,]/).filter(i => i.trim() !== "");
        const step = (Math.PI * 2) / items.length;
        const colors = ['#ff9f43', '#ee5253', '#0abde3', '#10ac84', '#5f27cd', '#ff9ff3', '#54a0ff', '#00d2d3'];
        ctx.clearRect(0,0,600,600);
        items.forEach((t, i) => {
            ctx.save(); ctx.beginPath(); ctx.fillStyle = colors[i%8];
            ctx.moveTo(300,300); ctx.arc(300,300,290,i*step,(i+1)*step); ctx.fill();
            ctx.translate(300,300); ctx.rotate(i*step+step/2);
            ctx.fillStyle="white"; ctx.font="bold 30px sans-serif"; ctx.textAlign="right";
            ctx.fillText(t.trim().substring(0,6),260,10); ctx.restore();
        });
    }

    function spin() {
        deg += Math.floor(Math.random()*360) + 3600;
        canvas.style.transform = `rotate(${deg}deg)`;
    }
</script>
</body>
</html>
