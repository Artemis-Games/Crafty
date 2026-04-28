<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Magic Plushie Studio</title>
    <style>
        body { margin: 0; background: #fff0f5; font-family: 'Comic Sans MS', cursive; overflow: hidden; display: flex; flex-direction: column; align-items: center; }
        #canvas-container { position: relative; margin-top: 20px; border: 10px solid #ffb6c1; border-radius: 50px; background: white; box-shadow: 0 10px 20px rgba(0,0,0,0.1); }
        canvas { display: block; border-radius: 40px; }
        .shelf { width: 90%; background: #ffe4e1; padding: 15px; border-radius: 20px; display: flex; justify-content: center; gap: 15px; margin-top: 20px; box-shadow: inset 0 4px 6px rgba(0,0,0,0.1); flex-wrap: wrap; }
        .btn { padding: 10px 20px; background: white; border: 3px solid #ff69b4; border-radius: 15px; cursor: pointer; font-size: 16px; transition: 0.2s; color: #ff1493; font-weight: bold; }
        .btn:active { transform: scale(0.9); background: #ffc0cb; }
        #ui { position: absolute; top: 10px; width: 100%; text-align: center; pointer-events: none; }
    </style>
</head>
<body>

<div id="canvas-container">
    <div id="ui"><h1 style="color:#ff69b4; margin:0;">Magic Plushie Studio ✨</h1></div>
    <canvas id="studio"></canvas>
</div>

<div class="shelf">
    <button class="btn" onclick="nextFabric()">🎨 Change Fabric</button>
    <button class="btn" onclick="nextEyes()">👀 Swap Eyes</button>
    <button class="btn" onclick="nextAccessory()">🎀 Add Accessory</button>
    <button class="btn" onclick="magicFinish()">💖 Magic Finish!</button>
</div>

<script>
    const canvas = document.getElementById('studio');
    const ctx = canvas.getContext('2d');
    canvas.width = 400; canvas.height = 450;

    // Game State
    const fabrics = [
        { name: 'Pink Felt', color: '#ffb6c1', texture: 'felt' },
        { name: 'Sky Sparkle', color: '#87ceeb', texture: 'sparkle' },
        { name: 'Lavender Fluff', color: '#e6e6fa', texture: 'fluff' },
        { name: 'Mint Velvet', color: '#98fb98', texture: 'velvet' }
    ];
    const eyesSet = ['classic', 'sparkle', 'sleepy', 'happy'];
    const accessories = ['none', 'bow', 'heart', 'wings', 'star'];

    let currentFab = 0, currentEyes = 0, currentAcc = 0, isMagic = false, particles = [];

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        const fab = fabrics[currentFab];

        // Draw Shadows
        ctx.fillStyle = 'rgba(0,0,0,0.05)';
        ctx.beginPath(); ctx.ellipse(200, 380, 100, 30, 0, 0, Math.PI*2); ctx.fill();

        // Draw Body (The "Stuffie" Base)
        ctx.fillStyle = fab.color;
        
        // Ears
        ctx.beginPath(); ctx.arc(120, 150, 40, 0, Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(280, 150, 40, 0, Math.PI*2); ctx.fill();

        // Main Body
        ctx.beginPath(); 
        ctx.moveTo(100, 250);
        ctx.quadraticCurveTo(200, 100, 300, 250);
        ctx.quadraticCurveTo(320, 380, 200, 380);
        ctx.quadraticCurveTo(80, 380, 100, 250);
        ctx.fill();

        // Fabric Texture Effect
        if (fab.texture === 'sparkle') {
            for(let i=0; i<30; i++) {
                ctx.fillStyle = 'white';
                ctx.fillRect(120+Math.random()*160, 180+Math.random()*150, 2, 2);
            }
        }

        // Eyes
        ctx.fillStyle = 'black';
        const eyeType = eyesSet[currentEyes];
        if (eyeType === 'classic') {
            ctx.beginPath(); ctx.arc(160, 220, 8, 0, 7); ctx.fill();
            ctx.beginPath(); ctx.arc(240, 220, 8, 0, 7); ctx.fill();
        } else if (eyeType === 'sparkle') {
            ctx.beginPath(); ctx.arc(160, 220, 10, 0, 7); ctx.fill();
            ctx.beginPath(); ctx.arc(240, 220, 10, 0, 7); ctx.fill();
            ctx.fillStyle = 'white';
            ctx.beginPath(); ctx.arc(157, 217, 3, 0, 7); ctx.fill();
            ctx.beginPath(); ctx.arc(237, 217, 3, 0, 7); ctx.fill();
        } else if (eyeType === 'sleepy') {
            ctx.lineWidth = 3; ctx.strokeStyle = 'black';
            ctx.beginPath(); ctx.arc(160, 220, 10, 0, Math.PI, false); ctx.stroke();
            ctx.beginPath(); ctx.arc(240, 220, 10, 0, Math.PI, false); ctx.stroke();
        }

        // Accessory
        const acc = accessories[currentAcc];
        if (acc === 'bow') {
            ctx.fillStyle = '#ff1493';
            ctx.beginPath(); ctx.moveTo(200, 140); ctx.lineTo(170, 120); ctx.lineTo(170, 160); ctx.fill();
            ctx.beginPath(); ctx.moveTo(200, 140); ctx.lineTo(230, 120); ctx.lineTo(230, 160); ctx.fill();
            ctx.beginPath(); ctx.arc(200, 140, 8, 0, 7); ctx.fill();
        } else if (acc === 'heart') {
            ctx.fillStyle = '#ff69b4';
            ctx.font = '40px Arial'; ctx.fillText('❤️', 180, 320);
        } else if (acc === 'wings') {
            ctx.fillStyle = 'rgba(255,255,255,0.6)';
            ctx.beginPath(); ctx.ellipse(80, 230, 40, 60, 0.5, 0, 7); ctx.fill();
            ctx.beginPath(); ctx.ellipse(320, 230, 40, 60, -0.5, 0, 7); ctx.fill();
        }

        if (isMagic) drawParticles();
        requestAnimationFrame(draw);
    }

    function nextFabric() { currentFab = (currentFab + 1) % fabrics.length; }
    function nextEyes() { currentEyes = (currentEyes + 1) % eyesSet.length; }
    function nextAccessory() { currentAcc = (currentAcc + 1) % accessories.length; }

    function magicFinish() {
        isMagic = true;
        for(let i=0; i<50; i++) {
            particles.push({
                x: 200, y: 250, 
                vx: (Math.random()-0.5)*10, vy: (Math.random()-0.5)*10,
                color: fabrics[currentFab].color, size: Math.random()*8
            });
        }
        setTimeout(() => { isMagic = false; particles = []; }, 2000);
    }

    function drawParticles() {
        particles.forEach(p => {
            ctx.fillStyle = p.color;
            ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, 7); ctx.fill();
            p.x += p.vx; p.y += p.vy; p.size *= 0.95;
        });
    }

    draw();
</script>
</body>
</html>
