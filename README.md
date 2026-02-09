<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Glow Studio Pro - Ultimate HD</title>
    <style>
        :root { --accent: #00d4ff; --bg: #050505; }
        body {
            font-family: 'Segoe UI', sans-serif;
            background-color: var(--bg);
            color: #fff;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
        }
        .header { text-align: center; margin-bottom: 15px; }
        .header h1 { color: var(--accent); margin: 0; font-size: 22px; text-transform: uppercase; text-shadow: 0 0 10px var(--accent); }
        
        .preview-box {
            width: 100%;
            max-width: 380px;
            height: 380px;
            background: #000;
            border: 2px solid #1a1a1a;
            border-radius: 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
            margin-bottom: 20px;
            position: relative;
            box-shadow: inset 0 0 20px rgba(0,0,0,1);
        }
        #image-preview {
            max-width: 85%;
            max-height: 85%;
            /* HD Rendering Settings from pichli files */
            image-rendering: -webkit-optimize-contrast;
            image-rendering: crisp-edges;
        }

        /* Pulse Animation Logic */
        @keyframes pulseGlow {
            0%, 100% { filter: var(--base-filter) brightness(var(--curr-br)); }
            50% { filter: var(--base-filter) brightness(calc(var(--curr-br) + 0.6)); }
        }

        .controls-card {
            width: 100%;
            max-width: 380px;
            background: #111;
            padding: 20px;
            border-radius: 20px;
            box-sizing: border-box;
            max-height: 550px;
            overflow-y: auto;
            border: 1px solid #222;
        }
        .controls-card::-webkit-scrollbar { width: 5px; }
        .controls-card::-webkit-scrollbar-thumb { background: #333; border-radius: 10px; }

        .section-title {
            color: var(--accent);
            font-size: 11px;
            font-weight: bold;
            margin: 15px 0 10px 0;
            text-transform: uppercase;
            border-bottom: 1px solid #222;
            padding-bottom: 5px;
        }
        .control-group { margin-bottom: 12px; }
        label { display: flex; justify-content: space-between; font-size: 12px; color: #aaa; margin-bottom: 6px; }
        label span { color: var(--accent); }
        input[type="range"] { width: 100%; accent-color: var(--accent); cursor: pointer; }

        .upload-btn {
            background: var(--accent);
            color: #000;
            width: 100%;
            max-width: 380px;
            padding: 15px;
            border: none;
            border-radius: 12px;
            font-weight: bold;
            margin-bottom: 15px;
            cursor: pointer;
            text-transform: uppercase;
            box-shadow: 0 4px 15px rgba(0, 212, 255, 0.3);
        }
    </style>
</head>
<body>

    <div class="header">
        <h1>Glow Studio Ultimate HD</h1>
    </div>

    <input type="file" id="upload" accept="image/*" style="display: none;">
    <button class="upload-btn" onclick="document.getElementById('upload').click()">Upload Fingerprint</button>

    <div class="preview-box">
        <svg width="0" height="0" style="position: absolute;">
            <filter id="colorFilter">
                <feColorMatrix type="matrix" values="1 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 1 0" id="matrixValues" />
            </filter>
        </svg>
        <img id="image-preview" src="https://via.placeholder.com/250?text=Select+Image" alt="Preview">
    </div>

    <div class="controls-card">
        <div class="section-title">HD Transformation</div>
        <div class="control-group">
            <label>Contrast (HD Depth) <span id="v-con">2.0</span></label>
            <input type="range" id="contrast" min="1" max="5" step="0.1" value="2">
        </div>
        <div class="control-group">
            <label>Sharpness (Edge) <span id="v-sh">1.5</span></label>
            <input type="range" id="sharpness" min="1" max="10" step="0.5" value="1.5">
        </div>
        <div class="control-group">
            <label>Brightness <span id="v-br">1.0</span></label>
            <input type="range" id="brightness" min="0.1" max="3" step="0.1" value="1">
        </div>
        <div class="control-group">
            <label>Zoom Level <span id="v-zoom">1.0x</span></label>
            <input type="range" id="zoom" min="0.5" max="3" step="0.1" value="1">
        </div>

        <div class="section-title">Glow Animation</div>
        <div class="control-group">
            <label>Glow Power <span id="v-glow">20px</span></label>
            <input type="range" id="glow" min="0" max="100" value="20">
        </div>
        <div class="control-group">
            <label>Pulse Speed (Tim-Tim) <span id="v-speed">Off</span></label>
            <input type="range" id="speed" min="0" max="4" step="0.1" value="0">
        </div>

        <div class="section-title">Color & Style</div>
        <div class="control-group">
            <label>Invert (Negative) <span id="v-inv">0%</span></label>
            <input type="range" id="invert" min="0" max="100" value="0">
        </div>
        <div class="control-group">
            <label>Theme Color</label>
            <input type="color" id="colorPicker" value="#00d4ff" style="width:100%; height:40px; border:none; background:none; cursor:pointer;">
        </div>
    </div>

    <script>
        const img = document.getElementById('image-preview');
        const upload = document.getElementById('upload');
        const matrixValues = document.getElementById('matrixValues');

        const controls = {
            con: document.getElementById('contrast'),
            sh: document.getElementById('sharpness'),
            br: document.getElementById('brightness'),
            zoom: document.getElementById('zoom'),
            glow: document.getElementById('glow'),
            speed: document.getElementById('speed'),
            invert: document.getElementById('invert'),
            color: document.getElementById('colorPicker')
        };

        const labels = {
            con: document.getElementById('v-con'),
            sh: document.getElementById('v-sh'),
            br: document.getElementById('v-br'),
            zoom: document.getElementById('v-zoom'),
            glow: document.getElementById('v-glow'),
            speed: document.getElementById('v-speed'),
            invert: document.getElementById('v-inv')
        };

        upload.onchange = (e) => {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (ev) => {
                    img.src = ev.target.result;
                    // Auto-apply HD settings on upload
                    setTimeout(updateEffects, 50);
                };
                reader.readAsDataURL(file);
            }
        };

        function hexToMatrix(hex) {
            const r = parseInt(hex.slice(1, 3), 16) / 255;
            const g = parseInt(hex.slice(3, 5), 16) / 255;
            const b = parseInt(hex.slice(5, 7), 16) / 255;
            return `${r} 0 0 0 0 0 ${g} 0 0 0 0 0 ${b} 0 0 0 0 0 1 0`;
        }

        function updateEffects() {
            const con = controls.con.value;
            const sh = controls.sh.value;
            const br = controls.br.value;
            const z = controls.zoom.value;
            const g = controls.glow.value;
            const s = controls.speed.value;
            const inv = controls.invert.value;
            const col = controls.color.value;

            // Update Labels
            labels.con.innerText = con;
            labels.sh.innerText = sh;
            labels.br.innerText = br;
            labels.zoom.innerText = z + 'x';
            labels.glow.innerText = g + 'px';
            labels.speed.innerText = s > 0 ? s + 's' : 'Off';
            labels.invert.innerText = inv + '%';

            // Global CSS Variables
            document.documentElement.style.setProperty('--accent', col);
            document.documentElement.style.setProperty('--curr-br', br);
            matrixValues.setAttribute("values", hexToMatrix(col));

            // Combined Master Filter (HD + Glow + Negative)
            const filterString = `grayscale(1) invert(${inv}%) brightness(${br}) contrast(${con}) saturate(${sh}) url(#colorFilter) drop-shadow(0 0 ${g}px ${col})`;
            
            document.documentElement.style.setProperty('--base-filter', filterString);

            if(s > 0) {
                img.style.animation = `pulseGlow ${s}s infinite ease-in-out`;
            } else {
                img.style.animation = 'none';
                img.style.filter = filterString;
            }

            img.style.transform = `scale(${z})`;
        }

        Object.values(controls).forEach(el => el.oninput = updateEffects);
    </script>
</body>
</html>
