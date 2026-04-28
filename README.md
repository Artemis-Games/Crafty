<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Plushie Crafter: Touch Edition</title>
    <style>
        :root { --pink: #ff85a2; --light: #fff0f5; }
        body { margin: 0; background: var(--light); font-family: 'Comic Sans MS', cursive; display: flex; touch-action: none; overflow: hidden; }
        
        /* Sidebar Toolbar */
        #toolbar {
            width: 90px; height: 100vh; background: white; border-right: 4px solid var(--pink);
            display: flex; flex-direction: column; align-items: center; padding-top: 20px; gap: 15px;
            overflow-y: auto; z-index: 10;
        }
        .tool-group { display: flex; flex-direction: column; gap: 8px; align-items: center; margin-bottom: 20px; }
        .swatch {
            width: 60px; height: 60px; border-radius: 12px; border: 3px solid #ddd;
            cursor: pointer; background-size: cover; transition: 0.2s;
        }
        .swatch.active { border-color: var(--pink); transform: scale(1.1); box-shadow: 0 0 10px var(--pink); }
        .label { font-size: 10px; color: var(--pink); font-weight: bold; text-transform: uppercase; }

        /* Main Area */
        #workspace { flex-grow: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; }
        canvas { background: white; border-radius: 30px; box-shadow: 0 15px 30px rgba(0,0,0,0.1); max-width: 95%; height: auto; }
        
        #magic-btn {
            position: absolute; bottom: 30px; padding: 15px 30px; background: var(--pink);
            color: white; border: none; border-radius: 50px; font-size: 20px; font-weight: bold;
            box-shadow: 0 5px 0 #d64d6d; cursor: pointer;
        }
        #magic-btn:active { transform: translateY(3px); box-shadow: 0 2px 0 #d64d6d; }
    </style>
</head>
<body>

<div id="toolbar">
    <div class="label">Fabric</div>
    <div class="tool-group" id="fabric-group"></div>
    
    <div class="label">Eyes</div>
    <div class="tool-group" id="eyes-group"></div>

    <div class="label">Decor</div>
    <div class="tool-group" id="acc-group"></div>
</div>

<div id="workspace">
    <canvas id="plushieCanvas"></canvas>
    <button id="magic-btn" onclick="magicFinish()">✨ STUFF IT! ✨</button>
</div>

<script>
    const canvas = document.getElementById('plushieCanvas');
    const ctx = canvas.getContext('2d');
    canvas.width = 450; canvas.height = 500;

    // --- GAME DATA ---
    const fabrics = [
        { color: '#ffb6c1', label: 'Pink' },
        { color: '#b0e0e6', label: 'Blue' },
        { color: '#dda0dd', label: 'Purple' },
        { color: '#ffebcd', label: 'Cream' }
    ];
    const eyesSet = ['👀', '✨', '💎', '🎀'];
    const accs = [
        { icon: '❌', val: 'none' },
        { icon: '💝', val: 'heart' },
        { icon: '🦋', val: 'wings' },
        { icon: '👑', val: 'crown' }
    ];

    let currentFab = 0, currentEyes = 0, currentAcc = 0, frame = 0, sparkles = [];

    // --- INITIALIZE TOOLBAR ---
    function initUI() {
        const fGroup = document.getElementById('fabric-group');
        fabrics.forEach((f, i) => {
            let div = document.createElement('div');
            div.className = `swatch ${i===0?'active':''}`;
            div.style.backgroundColor = f.color;
            div.onclick = () => { selectTool('fabric', i, div); currentFab = i; };
            fGroup.appendChild(div);
        });

        const eGroup = document.getElementById('eyes-group');
        eyesSet.forEach((e, i) => {
            let div = document.createElement('div');
            div.className = `swatch ${i===0?'active':''}`;
            div.innerHTML = `<div style="font-size:30px; text-align:center; line-height:60px">${e}</div>`;
            div.onclick = () => { selectTool('eyes', i, div); currentEyes = i; };
            eGroup.appendChild(div);
        });

        const aGroup = document.getElementById('acc-group');
        accs.forEach((a, i) => {
            let div = document.createElement('div');
            div.className = `swatch ${i===0?'active':''}`;
            div.innerHTML = `<div style="font-size:30px; text-align:center; line-height:60px">${a.icon}</div>`;
            div.onclick = () => { selectTool('acc', i, div); currentAcc = i; };
            aGroup.appendChild(div);
        });
    }

    function selectTool(type, index, el) {
        el.parentNode.querySelectorAll('.swatch').forEach(s => s.classList.remove('active'));
        el.classList.add('active');
    }

    // --- DRAWING ENGINE ---
    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frame++;
        const fab = fabrics[currentFab];
        const bob = Math.sin(frame * 0.05) * 5; // Gentle floating animation

        ctx.save();
        ctx.translate(225, 250 + bob);

        // 1. Draw Ears
        ctx.fillStyle = fab.color;
        ctx.beginPath(); ctx.arc(-80, -100, 50, 0, 7); ctx.fill();
        ctx.beginPath(); ctx.arc(80, -100, 50, 0, 7); ctx.fill();

        // 2. Draw Body (Stuffie Shape)
        ctx.beginPath();
        ctx.moveTo(-120, 100);
        ctx.bezierCurveTo(-150, -150, 150, -150, 120, 100);
        ctx.bezierCurveTo(130, 220, -130, 220, -120, 100);
        ctx.fill();

        // 3. Stitched Edge Effect
        ctx.setLineDash([5, 5]);
        ctx.strokeStyle = 'rgba(0,0,0,0.1)';
        ctx.lineWidth = 4;
        ctx.stroke();

        // 4. Face
        ctx.font = "60px Arial";
        ctx.textAlign = "center";
        ctx.fillText(eyesSet[currentEyes], 0, -20);
        
        // Muzzle
        ctx.fillStyle = "white";
        ctx.beginPath(); ctx.ellipse(0, 30, 40, 30, 0, 0, 7); ctx.fill();
        ctx.fillStyle = "black";
        ctx.beginPath(); ctx.arc(0, 20, 8, 0, 7); ctx.fill(); // Nose

        // 5. Accessories
        const acc = accs[currentAcc].val;
        ctx.font = "80px Arial";
        if (acc === 'heart') ctx.fillText("💝", 0, 110);
        if (acc === 'wings') {
            ctx.globalAlpha = 0.5;
            ctx.fillText("🦋", -110, 0); ctx.fillText("🦋", 110, 0);
            ctx.globalAlpha = 1;
        }
        if (acc === 'crown') ctx.fillText("👑", 0, -120);

        ctx.restore();

        // Sparkles
        sparkles.forEach((s, i) => {
            ctx.fillStyle = s.c;
            ctx.beginPath(); ctx.arc(s.x, s.y, s.s, 0, 7); ctx.fill();
            s.y -= 2; s.s *= 0.98;
            if (s.s < 0.1) sparkles.splice(i, 1);
        });

        requestAnimationFrame(draw);
    }

    function magicFinish() {
        for(let i=0; i<30; i++) {
            sparkles.push({
                x: Math.random() * canvas.width,
                y: canvas.height,
                s: Math.random() * 10 + 5,
                c: `hsl(${Math.random()*360}, 100%, 70%)`
            });
        }
    }

    initUI();
    draw();
</script>
</body>
</html>
