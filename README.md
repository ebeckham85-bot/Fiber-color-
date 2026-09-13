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
            height: 100dvh;
            background-color: #000;
            color: #fff;
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", sans-serif;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        #camera-container {
            position: relative;
            width: 100%;
            flex: 1;
            min-height: 180px;
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
            width: 32px;
            height: 32px;
            border: 2px solid #00ffcc;
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
            width: 4px;
            height: 4px;
            background: #ff0055;
            border-radius: 50%;
        }

        #screen-flash {
            position: absolute;
            inset: 0;
            background: rgba(255, 255, 255, 0);
            pointer-events: none;
            z-index: 20;
            transition: background 0.3s ease;
        }
        .screen-flash-on {
            background: rgba(255, 255, 255, 0.85) !important;
        }

        #start-overlay {
            position: absolute;
            inset: 0;
            background: rgba(0, 0, 0, 0.95);
            z-index: 100;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 20px;
            text-align: center;
        }
        #start-btn {
            padding: 16px 32px;
            font-size: 1.2rem;
            font-weight: 800;
            background: #007aff;
            color: white;
            border: none;
            border-radius: 14px;
            margin-top: 20px;
            cursor: pointer;
        }

        /* Fixed Bottom Panel with High Z-Index */
        #result-panel {
            width: 100%;
            background: #1c1c1e;
            border-top: 2px solid #38383a;
            padding: 10px 12px calc(12px + env(safe-area-inset-bottom, 12px)) 12px;
            display: flex;
            flex-direction: column;
            gap: 8px;
            z-index: 50;
            flex-shrink: 0;
        }

        .result-card {
            display: flex;
            align-items: center;
            background: #2c2c2e;
            border-radius: 12px;
            padding: 8px 10px;
            border: 1.5px solid #444;
            gap: 10px;
        }
        .color-badge {
            width: 42px;
            height: 42px;
            border-radius: 8px;
            border: 2px solid #fff;
            flex-shrink: 0;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
        }
        .result-info {
            flex: 1;
            overflow: hidden;
        }
        .result-title {
            font-size: 1.5rem;
            font-weight: 900;
            line-height: 1.1;
            color: #ffffff;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
        }
        .result-sub {
            font-size: 0.8rem;
            font-weight: 600;
            color: #34c759;
            margin-top: 2px;
        }

        .brightness-control {
            display: flex;
            align-items: center;
            gap: 8px;
            background: #2c2c2e;
            padding: 6px 10px;
            border-radius: 8px;
            border: 1px solid #38383a;
        }
        .brightness-control label {
            font-size: 0.8rem;
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
            gap: 6px;
            width: 100%;
        }
        button.ctrl-btn {
            flex: 1;
            padding: 12px 2px;
            font-size: 0.8rem;
            font-weight: 700;
            border: none;
            border-radius: 8px;
            background-color: #2c2c2e;
            color: #0a84ff;
            border: 1px solid #3a3a3c;
            text-align: center;
            white-space: nowrap;
        }
        button.ctrl-btn:active {
            background-color: #3a3a3c;
        }
        button.speech-disabled {
            color: #ff453a !important;
            border-color: #ff453a !important;
            background-color: #3a1c1c !important;
        }
        button.torch-active {
            background-color: #ffd60a !important;
            color: #000 !important;
            font-weight: 900 !important;
        }
    </style>
</head>
<body>

    <div id="start-overlay">
        <h1 style="font-size: 1.8rem;">Fiber Identifier</h1>
        <p style="margin-top: 10px; font-size: 1rem; color: #aaa;">Align fiber strand in center dot</p>
        <button id="start-btn">TAP TO START</button>
    </div>

    <div id="camera-container">
        <video id="webcam" autoplay playsinline muted></video>
        <canvas id="overlay"></canvas>
        <div class="reticle"></div>
        <div id="screen-flash"></div>
    </div>

    <div id="result-panel">
        <div class="result-card">
            <div id="swatch" class="color-badge" style="background-color: #555;"></div>
            <div class="result-info">
                <div id="color-name" class="result-title">READY</div>
                <div id="color-pos" class="result-sub">Pos: -- | Match: --</div>
            </div>
        </div>

        <div class="brightness-control">
            <label for="brightness">☀️ Exposure:</label>
            <input type="range" id="brightness" min="0.5" max="2.5" step="0.1" value="1.0">
        </div>

        <div class="controls">
            <button id="torch-btn" class="ctrl-btn">🔦 Torch</button>
            <button id="toggle-speech" class="ctrl-btn">🗣 Voice: ON</button>
            <button id="freeze-btn" class="ctrl-btn">⏸ Freeze</button>
        </div>
    </div>

    <script>
        const TIA598 = [
            { pos: 1,  name: "BLUE",   abbr: "BL", rgb: [10, 110, 230] },
            { pos: 2,  name: "ORANGE", abbr: "OR", rgb: [240, 110, 20] },
            { pos: 3,  name: "GREEN",  abbr: "GR", rgb: [20, 160, 50] },
            { pos: 4,  name: "BROWN",  abbr: "BR", rgb: [110, 65, 35] },
            { pos: 5,  name: "SLATE",  abbr: "SL", rgb: [130, 140, 150] },
            { pos: 6,  name: "WHITE",  abbr: "WH", rgb: [230, 230, 230] },
            { pos: 7,  name: "RED",    abbr: "RD", rgb: [220, 30, 40] },
            { pos: 8,  name: "BLACK",  abbr: "BK", rgb: [30, 30, 35] },
            { pos: 9,  name: "YELLOW", abbr: "YL", rgb: [240, 205, 30] },
            { pos: 10, name: "VIOLET", abbr: "VI", rgb: [130, 50, 160] },
            { pos: 11, name: "ROSE",   abbr: "RS", rgb: [235, 120, 165] },
            { pos: 12, name: "AQUA",   abbr: "AQ", rgb: [0, 185, 205] }
        ];

        function normalizeRGB(r, g, b) {
            const sum = r + g + b || 1;
            return [r / sum, g / sum, b / sum];
        }

        function calculateColorMatch(r, g, b) {
            const sum = r + g + b;
            const norm = normalizeRGB(r, g, b);
            
            if (sum < 100) return { match: TIA598[7], confidence: 95 };
            if (sum > 620 && Math.max(r,g,b) - Math.min(r,g,b) < 30) return { match: TIA598[5], confidence: 95 };

            let bestMatch = null;
            let minScore = Infinity;

            for (const item of TIA598) {
                const itemNorm = normalizeRGB(...item.rgb);
                const dr = norm[0] - itemNorm[0];
                const dg = norm[1] - itemNorm[1];
                const db = norm[2] - itemNorm[2];
                const score = Math.sqrt(dr * dr + dg * dg + db * db);

                if (score < minScore) {
                    minScore = score;
                    bestMatch = item;
                }
            }

            return { match: bestMatch, confidence: Math.max(0, Math.round((1 - minScore * 2.5) * 100)) };
        }

        const video = document.getElementById('webcam');
        const overlay = document.getElementById('overlay');
        const ctx = overlay.getContext('2d');
        let currentTrack = null;
        let torchOn = false;
        let speechEnabled = true;
        let isFrozen = false;
        let lastSpoken = "";
        let colorHistory = [];
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
            torchOn = !torchOn;
            const torchBtn = document.getElementById('torch-btn');
            const screenFlash = document.getElementById('screen-flash');
            const brightnessSlider = document.getElementById('brightness');

            let hardwareSuccess = false;

            if (currentTrack && currentTrack.applyConstraints) {
                try {
                    await currentTrack.applyConstraints({
                        advanced: [{ torch: torchOn }]
                    });
                    hardwareSuccess = true;
                } catch (e) {
                    hardwareSuccess = false;
                }
            }

            if (!hardwareSuccess) {
                if (torchOn) {
                    screenFlash.classList.add('screen-flash-on');
                    brightnessSlider.value = "1.8";
                } else {
                    screenFlash.classList.remove('screen-flash-on');
                    brightnessSlider.value = "1.0";
                }
            }

            torchBtn.innerText = torchOn ? "🔦 Torch: ON" : "🔦 Torch";
            torchBtn.classList.toggle('torch-active', torchOn);
        }

        function sampleCenterColor() {
            if (video.readyState !== video.HAVE_ENOUGH_DATA || isFrozen) return;

            overlay.width = video.videoWidth;
            overlay.height = video.videoHeight;
            
            const brightnessVal = document.getElementById('brightness').value;
            ctx.filter = `brightness(${brightnessVal})`;
            ctx.drawImage(video, 0, 0, overlay.width, overlay.height);

            const sampleSize = 12;
            const startX = Math.floor(overlay.width / 2 - sampleSize / 2);
            const startY = Math.floor(overlay.height / 2 - sampleSize / 2);
            const frameData = ctx.getImageData(startX, startY, sampleSize, sampleSize).data;

            let totalR = 0, totalG = 0, totalB = 0, count = 0;

            for (let i = 0; i < frameData.length; i += 4) {
                const r = frameData[i], g = frameData[i + 1], b = frameData[i + 2];
                if ((r + g + b) < 730) {
                    totalR += r;
                    totalG += g;
                    totalB += b;
                    count++;
                }
            }

            if (count === 0) return;

            const avgR = Math.round(totalR / count);
            const avgG = Math.round(totalG / count);
            const avgB = Math.round(totalB / count);

            colorHistory.push([avgR, avgG, avgB]);
            if (colorHistory.length > 5) colorHistory.shift();

            const smoothR = Math.round(colorHistory.reduce((s, c) => s + c[0], 0) / colorHistory.length);
            const smoothG = Math.round(colorHistory.reduce((s, c) => s + c[1], 0) / colorHistory.length);
            const smoothB = Math.round(colorHistory.reduce((s, c) => s + c[2], 0) / colorHistory.length);

            const result = calculateColorMatch(smoothR, smoothG, smoothB);
            const color = result.match;

            if (color) {
                document.getElementById('swatch').style.backgroundColor = `rgb(${smoothR}, ${smoothG}, ${smoothB})`;
                document.getElementById('color-name').innerText = `${color.pos}. ${color.name}`;
                document.getElementById('color-pos').innerText = `Abbr: ${color.abbr}  |  Match: ${result.confidence}%`;
                speak(`${color.name}, Position ${color.pos}`);
            }
        }

        document.getElementById('start-btn').addEventListener('click', async () => {
            const silentUtterance = new SpeechSynthesisUtterance("");
            synth.speak(silentUtterance);

            document.getElementById('start-overlay').style.display = 'none';
            await initCamera();
            setInterval(sampleCenterColor, 150);
        });

        document.getElementById('torch-btn').addEventListener('click', toggleTorch);

        document.getElementById('toggle-speech').addEventListener('click', (e) => {
            speechEnabled = !speechEnabled;
            if (speechEnabled) {
                e.target.innerText = "🗣 Voice: ON";
                e.target.classList.remove('speech-disabled');
                speak("Voice active");
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
