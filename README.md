<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Fiber Optic Color Identifier</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            -webkit-user-select: none;
        }
        html, body {
            width: 100%;
            height: 100%;
            height: -webkit-fill-available;
            background-color: #000;
            color: #fff;
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", sans-serif;
            overflow: hidden;
            display: flex;
            flex-direction: column;
        }

        /* Top Camera View - Flexible sizing */
        #camera-container {
            position: relative;
            width: 100%;
            flex: 1;
            min-height: 35vh;
            background: #111;
            overflow: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        video {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        canvas#overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }
        .reticle {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 50px;
            height: 50px;
            border: 3px solid #00ffcc;
            border-radius: 50%;
            pointer-events: none;
            z-index: 15;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.45);
        }
        .reticle::after {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 8px;
            height: 8px;
            background: #ff0055;
            border-radius: 50%;
        }

        #start-overlay {
            position: absolute;
            inset: 0;
            background: rgba(0, 0, 0, 0.92);
            z-index: 100;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 24px;
            text-align: center;
        }
        #start-btn {
            padding: 18px 32px;
            font-size: 1.3rem;
            font-weight: 800;
            background: #007aff;
            color: white;
            border: none;
            border-radius: 16px;
            margin-top: 20px;
            cursor: pointer;
        }

        /* Bottom Panel - Automatically fits device screens */
        #result-panel {
            flex: 0 0 auto;
            background: #1c1c1e;
            border-top: 2px solid #38383a;
            padding: 12px 14px calc(12px + env(safe-area-inset-bottom, 12px)) 14px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            box-sizing: border-box;
        }
        .result-card {
            display: flex;
            align-items: center;
            background: #2c2c2e;
            border-radius: 14px;
            padding: 10px 12px;
            border: 2px solid #444;
            gap: 12px;
        }
        .color-badge {
            width: 52px;
            height: 52px;
            border-radius: 10px;
            border: 2px solid #fff;
            flex-shrink: 0;
            box-shadow: 0 4px 12px rgba(0,0,0,0.5);
        }
        .result-info {
            flex: 1;
            overflow: hidden;
        }
        .result-title {
            font-size: 1.8rem;
            font-weight: 900;
            line-height: 1.1;
            color: #ffffff;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
        }
        .result-sub {
            font-size: 0.9rem;
            font-weight: 600;
            color: #34c759;
            margin-top: 2px;
        }

        .brightness-control {
            display: flex;
            align-items: center;
            gap: 10px;
            background: #2c2c2e;
            padding: 8px 12px;
            border-radius: 10px;
            border: 1px solid #38383a;
        }
        .brightness-control label {
            font-size: 0.85rem;
            font-weight: bold;
            color: #aaa;
            white-space: nowrap;
        }
        .brightness-control input[type=range] {
            flex: 1;
            accent-color: #0a84ff;
            height: 6px;
        }

        .controls {
            display: flex;
            gap: 8px;
        }
        button.ctrl-btn {
            flex: 1;
            padding: 12px 6px;
            font-size: 0.85rem;
            font-weight: 700;
            border: none;
            border-radius: 10px;
            background-color: #2c2c2e;
            color: #0a84ff;
            border: 1px solid #3a3a3c;
        }
        button.ctrl-btn:active {
            background-color: #3a3a3c;
        }
        button.speech-disabled {
            color: #ff453a !important;
            border-color: #ff453a !important;
        }
        button.torch-active {
            background-color: #ffd60a !important;
            color: #000 !important;
        }
    </style>
</head>
<body>

    <div id="start-overlay">
        <h1 style="font-size: 2rem;">Fiber Identifier</h1>
        <p style="margin-top: 10px; font-size: 1.1rem; color: #aaa;">Align fiber strand in center dot</p>
        <button id="start-btn">TAP TO START</button>
    </div>

    <div id="camera-container">
        <video id="webcam" autoplay playsinline muted></video>
        <canvas id="overlay"></canvas>
        <div class="reticle"></div>
    </div>

    <div id="result-panel">
        <div class="result-card">
            <div id="swatch" class="color-badge" style="background-color: #555;"></div>
            <div class="result-info">
                <div id="color-name" class="result-title">READY</div>
                <div id="color-pos" class="result-sub">Pos: -- | ΔE: --</div>
            </div>
        </div>

        <div class="brightness-control">
            <label for="brightness">☀️ Feed Brightness:</label>
            <input type="range" id="brightness" min="0.5" max="2.5" step="0.1" value="1.0">
        </div>

        <div class="controls">
            <button id="torch-btn" class="ctrl-btn">🔦 Torch: OFF</button>
            <button id="toggle-speech" class="ctrl-btn">🗣 Voice: ON</button>
            <button id="freeze-btn" class="ctrl-btn">⏸ Freeze</button>
        </div>
    </div>

    <script>
        const TIA598 = [
            { pos: 1,  name: "BLUE",   abbr: "BL", rgb: [0, 90, 212] },
            { pos: 2,  name: "ORANGE", abbr: "OR", rgb: [242, 107, 15] },
            { pos: 3,  name: "GREEN",  abbr: "GR", rgb: [24, 150, 48] },
            { pos: 4,  name: "BROWN",  abbr: "BR", rgb: [102, 57, 24] },
            { pos: 5,  name: "SLATE",  abbr: "SL", rgb: [110, 120, 128] },
            { pos: 6,  name: "WHITE",  abbr: "WH", rgb: [225, 230, 235] },
            { pos: 7,  name: "RED",    abbr: "RD", rgb: [210, 25, 35] },
            { pos: 8,  name: "BLACK",  abbr: "BK", rgb: [25, 25, 28] },
            { pos: 9,  name: "YELLOW", abbr: "YL", rgb: [245, 195, 20] },
            { pos: 10, name: "VIOLET", abbr: "VI", rgb: [115, 38, 140] },
            { pos: 11, name: "ROSE",   abbr: "RS", rgb: [235, 125, 160] },
            { pos: 12, name: "AQUA",   abbr: "AQ", rgb: [0, 175, 190] }
        ];

        function rgbToLab(r, g, b) {
            r /= 255; g /= 255; b /= 255;
            r = r > 0.04045 ? Math.pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
            g = g > 0.04045 ? Math.pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
            b = b > 0.04045 ? Math.pow((b + 0.055) / 1.055, 2.4) : b / 12.92;

            let x = (r * 0.4124 + g * 0.3576 + b * 0.1805) * 100 / 95.047;
            let y = (r * 0.2126 + g * 0.7152 + b * 0.0722) * 100 / 100.000;
            let z = (r * 0.0193 + g * 0.1192 + b * 0.9505) * 100 / 108.883;

            x = x > 0.008856 ? Math.cbrt(x) : (7.787 * x) + (16 / 116);
            y = y > 0.008856 ? Math.cbrt(y) : (7.787 * y) + (16 / 116);
            z = z > 0.008856 ? Math.cbrt(z) : (7.787 * z) + (16 / 116);

            return [(116 * y) - 16, 500 * (x - y), 200 * (y - z)];
        }

        function deltaE00(lab1, lab2) {
            const [L1, a1, b1] = lab1;
            const [L2, a2, b2] = lab2;
            const C1 = Math.hypot(a1, b1), C2 = Math.hypot(a2, b2);
            const C_bar = (C1 + C2) / 2;
            const G = 0.5 * (1 - Math.sqrt(Math.pow(C_bar, 7) / (Math.pow(C_bar, 7) + Math.pow(25, 7))));
            const a1p = (1 + G) * a1, a2p = (1 + G) * a2;
            const C1p = Math.hypot(a1p, b1), C2p = Math.hypot(a2p, b2);
            const C_barp = (C1p + C2p) / 2;
            const h1p = Math.atan2(b1, a1p) * 180 / Math.PI + (Math.atan2(b1, a1p) < 0 ? 360 : 0);
            const h2p = Math.atan2(b2, a2p) * 180 / Math.PI + (Math.atan2(b2, a2p) < 0 ? 360 : 0);
            const H_barp = Math.abs(h1p - h2p) > 180 ? (h1p + h2p + 360) / 2 : (h1p + h2p) / 2;
            const T = 1 - 0.17 * Math.cos((H_barp - 30) * Math.PI / 180) + 0.24 * Math.cos((2 * H_barp) * Math.PI / 180) + 0.32 * Math.cos((3 * H_barp + 6) * Math.PI / 180) - 0.20 * Math.cos((4 * H_barp - 63) * Math.PI / 180);
            const deltahp = Math.abs(h1p - h2p) <= 180 ? h2p - h1p : (h2p <= h1p ? h2p - h1p + 360 : h2p - h1p - 360);
            const deltaLp = L2 - L1, deltaCp = C2p - C1p, deltaHp = 2 * Math.sqrt(C1p * C2p) * Math.sin((deltahp / 2) * Math.PI / 180);
            const Sl = 1 + (0.015 * Math.pow(L1 - 50, 2)) / Math.sqrt(20 + Math.pow(L1 - 50, 2));
            const Sc = 1 + 0.045 * C_barp, Sh = 1 + 0.015 * C_barp * T;
            const Rt = -2 * Math.sqrt(Math.pow(C_barp, 7) / (Math.pow(C_barp, 7) + Math.pow(25, 7))) * Math.sin((60 * Math.exp(-Math.pow((H_barp - 275) / 25, 2))) * Math.PI / 180);
            return Math.sqrt(Math.pow(deltaLp / Sl, 2) + Math.pow(deltaCp / Sc, 2) + Math.pow(deltaHp / Sh, 2) + Rt * (deltaCp / Sc) * (deltaHp / Sh));
        }

        const TIA598_LAB = TIA598.map(item => ({ ...item, lab: rgbToLab(...item.rgb) }));

        const video = document.getElementById('webcam');
        const overlay = document.getElementById('overlay');
        const ctx = overlay.getContext('2d');
        let currentTrack = null;
        let torchOn = false;
        let speechEnabled = true;
        let isFrozen = false;
        let lastSpoken = "";
        const synth = window.speechSynthesis;

        function speak(text) {
            if (!speechEnabled || synth.speaking || lastSpoken === text) return;
            const utterThis = new SpeechSynthesisUtterance(text);
            utterThis.rate = 1.0;
            lastSpoken = text;
            synth.speak(utterThis);
        }

        async function initCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({
                    video: { facingMode: { exact: "environment" } },
                    audio: false
                });
                video.srcObject = stream;
                currentTrack = stream.getVideoTracks()[0];
            } catch (err) {
                const fallbackStream = await navigator.mediaDevices.getUserMedia({
                    video: { facingMode: "environment" }
                });
                video.srcObject = fallbackStream;
                currentTrack = fallbackStream.getVideoTracks()[0];
            }
        }

        async function toggleTorch() {
            if (!currentTrack) return;
            const capabilities = currentTrack.getCapabilities ? currentTrack.getCapabilities() : {};
            if (!capabilities.torch) {
                alert("Hardware Flashlight is restricted on iOS Safari. Use the Feed Brightness slider above.");
                return;
            }

            try {
                torchOn = !torchOn;
                await currentTrack.applyConstraints({ advanced: [{ torch: torchOn }] });
                const torchBtn = document.getElementById('torch-btn');
                torchBtn.innerText = torchOn ? "🔦 Torch: ON" : "🔦 Torch: OFF";
                torchBtn.classList.toggle('torch-active', torchOn);
            } catch (e) {
                alert("Flashlight error: " + e.message);
            }
        }

        function sampleCenterColor() {
            if (video.readyState !== video.HAVE_ENOUGH_DATA || isFrozen) return;

            overlay.width = video.videoWidth;
            overlay.height = video.videoHeight;
            
            const brightnessVal = document.getElementById('brightness').value;
            ctx.filter = `brightness(${brightnessVal})`;
            ctx.drawImage(video, 0, 0, overlay.width, overlay.height);

            const sampleSize = 20;
            const startX = Math.floor(overlay.width / 2 - sampleSize / 2);
            const startY = Math.floor(overlay.height / 2 - sampleSize / 2);
            const frameData = ctx.getImageData(startX, startY, sampleSize, sampleSize).data;

            let validPixels = [];
            for (let i = 0; i < frameData.length; i += 4) {
                const r = frameData[i], g = frameData[i + 1], b = frameData[i + 2];
                const brightness = (r + g + b) / 3;
                if (brightness < 240) {
                    validPixels.push([r, g, b]);
                }
            }

            if (validPixels.length === 0) return;

            const avgR = Math.round(validPixels.reduce((acc, p) => acc + p[0], 0) / validPixels.length);
            const avgG = Math.round(validPixels.reduce((acc, p) => acc + p[1], 0) / validPixels.length);
            const avgB = Math.round(validPixels.reduce((acc, p) => acc + p[2], 0) / validPixels.length);

            const sampledLab = rgbToLab(avgR, avgG, avgB);

            let closest = null;
            let minDistance = Infinity;

            for (const item of TIA598_LAB) {
                const dist = deltaE00(sampledLab, item.lab);
                if (dist < minDistance) {
                    minDistance = dist;
                    closest = item;
                }
            }

            if (closest) {
                document.getElementById('swatch').style.backgroundColor = `rgb(${avgR}, ${avgG}, ${avgB})`;
                document.getElementById('color-name').innerText = `${closest.pos}. ${closest.name}`;
                document.getElementById('color-pos').innerText = `Abbr: ${closest.abbr}  |  ΔE: ${minDistance.toFixed(1)}`;
                speak(`${closest.name}, Position ${closest.pos}`);
            }
        }

        document.getElementById('start-btn').addEventListener('click', async () => {
            const silentUtterance = new SpeechSynthesisUtterance("");
            synth.speak(silentUtterance);

            document.getElementById('start-overlay').style.display = 'none';
            await initCamera();
            setInterval(sampleCenterColor, 200);
        });

        document.getElementById('torch-btn').addEventListener('click', toggleTorch);

        document.getElementById('toggle-speech').addEventListener('click', (e) => {
            speechEnabled = !speechEnabled;
            if (speechEnabled) {
                e.target.innerText = "🗣 Voice: ON";
                e.target.classList.remove('speech-disabled');
                speak("Voice assist active");
            } else {
                e.target.innerText = "🔇 Voice: OFF";
                e.target.classList.add('speech-disabled');
                synth.cancel();
            }
        });

        document.getElementById('freeze-btn').addEventListener('click', (e) => {
            isFrozen = !isFrozen;
            if (isFrozen) {
                video.pause();
                e.target.innerText = "▶ Resume";
            } else {
                video.play();
                e.target.innerText = "⏸ Freeze";
            }
        });
    </script>
</body>
</html>
