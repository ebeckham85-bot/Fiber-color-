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
            height: 52vh;
            max-height: 52dvh;
            background: #000;
            overflow: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;
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

        #result-panel {
            flex: 1;
            width: 100%;
            background: #1c1c1e;
            border-top: 2px solid #38383a;
            padding: 8px 12px calc(8px + env(safe-area-inset-bottom, 8px)) 12px;
            display: flex;
            flex-direction: column;
            justify-content: space-evenly;
            gap: 6px;
            z-index: 50;
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
            width: 38px;
            height: 38px;
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
            font-size: 1.4rem;
            font-weight: 900;
            line-height: 1.1;
            color: #ffffff;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
        }
        .result-sub {
            font-size: 0.75rem;
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
            font-size: 0.75rem;
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
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 6px;
            width: 100%;
        }
        button.ctrl-btn {
            padding: 10px 2px;
            font-size: 0.75rem;
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
        button.calib-active {
            background-color: #30d158 !important;
            color: #000 !important;
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
            <button id="calib-btn" class="ctrl-btn">🎯 Calibrate White</button>
            <button id="torch-btn" class="ctrl-btn">🔦 Torch</button>
            <button id="toggle-speech" class="ctrl-btn">🗣 Voice: ON</button>
            <button id="freeze-btn" class="ctrl-btn">⏸ Freeze</button>
        </div>
    </div>

    <script>
        // TIA-598 Color Standard calibrated in LAB Space
        const TIA598 = [
            { pos: 1,  name: "BLUE",   abbr: "BL", rgb: [0, 102, 204] },
            { pos: 2,  name: "ORANGE", abbr: "OR", rgb: [255, 102, 0] },
            { pos: 3,  name: "GREEN",  abbr: "GR", rgb: [0, 153, 51] },
            { pos: 4,  name: "BROWN",  abbr: "BR", rgb: [102, 51, 0] },
            { pos: 5,  name: "SLATE",  abbr: "SL", rgb: [128, 128, 128] },
            { pos: 6,  name: "WHITE",  abbr: "WH", rgb: [240, 240, 240] },
            { pos: 7,  name: "RED",    abbr: "RD", rgb: [204, 0, 0] },
            { pos: 8,  name: "BLACK",  abbr: "BK", rgb: [20, 20, 20] },
            { pos: 9,  name: "YELLOW", abbr: "YL", rgb: [255, 204, 0] },
            { pos: 10, name: "VIOLET", abbr: "VI", rgb: [127, 0, 255] },
            { pos: 11, name: "ROSE",   abbr: "RS", rgb: [255, 102, 178] },
            { pos: 12, name: "AQUA",   abbr: "AQ", rgb: [0, 204, 204] }
        ];

        // RGB to LAB perceptual color conversion algorithm
        function rgbToLab(r, g, b) {
            let rN = r / 255, gN = g / 255, bN = b / 255;
            rN = (rN > 0.04045) ? Math.pow((rN + 0.055) / 1.055, 2.4) : rN / 12.92;
            gN = (gN > 0.04045) ? Math.pow((gN + 0.055) / 1.055, 2.4) : gN / 12.92;
            bN = (bN > 0.04045) ? Math.pow((bN + 0.055) / 1.055, 2.4) : bN / 12.92;

            let x = (rN * 0.4124 + gN * 0.3576 + bN * 0.1805) / 0.95047;
            let y = (rN * 0.2126 + gN * 0.7152 + bN * 0.0722) / 1.00000;
            let z = (rN * 0.0193 + gN * 0.1192 + bN * 0.9505) / 1.08883;

            x = (x > 0.008856) ? Math.cbrt(x) : (7.787 * x) + (16 / 116);
            y = (y > 0.008856) ? Math.cbrt(y) : (7.787 * y) + (16 / 116);
            z = (z > 0.008856) ? Math.cbrt(z) : (7.787 * z) + (16 / 116);

            return [(116 * y) - 16, 500 * (x - y), 200 * (y - z)];
        }

        // Cache pre-calculated Lab values for TIA598 targets
        const TIA598_LAB = TIA598.map(item => ({
            ...item,
            lab: rgbToLab(...item.rgb)
        }));

        let wbGain = [1.0, 1.0, 1.0]; // White balance gains

        function calculateColorMatch(r, g, b) {
            // Apply Manual White Balance multiplier
            let adjR = Math.min(255, Math.max(0, r * wbGain[0]));
            let adjG = Math.min(255, Math.max(0, g * wbGain[1]));
            let adjB = Math.min(255, Math.max(0, b * wbGain[2]));

            const max = Math.max(adjR, adjG, adjB);
            const min = Math.min(adjR, adjG, adjB);
            const chroma = max - min;
            const brightness = (adjR + adjG + adjB) / 3;

            // Absolute threshold overrides
            if (max < 38) return { match: TIA598[7], confidence: 98 }; // Black
            if (brightness > 215 && chroma < 18) return { match: TIA598[5], confidence: 95 }; // White
            if (chroma < 14 && brightness >= 38 && brightness <= 215) return { match: TIA598[4], confidence: 90 }; // Slate

            const sampleLab = rgbToLab(adjR, adjG, adjB);
            let bestMatch = null;
            let minDistance = Infinity;

            for (const item of TIA598_LAB) {
                // Delta E Euclidean Distance in CIE-LAB space
                const dL = sampleLab[0] - item.lab[0];
                const da = sampleLab[1] - item.lab[1];
                const db = sampleLab[2] - item.lab[2];
                const distance = Math.sqrt(dL * dL + da * da + db * db);

                if (distance < minDistance) {
                    minDistance = distance;
                    bestMatch = item;
                }
            }

            const confidence = Math.max(10, Math.round(100 - (minDistance * 0.65)));
            return { match: bestMatch, confidence, rgb: [adjR, adjG, adjB] };
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
        let currentSampleRGB = [128, 128, 128];
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
                    video: { facingMode: { exact: "environment" }, width: { ideal: 1280 }, height: { ideal: 720 } },
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

        function sampleCenterColor() {
            if (video.readyState !== video.HAVE_ENOUGH_DATA || isFrozen) return;

            overlay.width = video.videoWidth;
            overlay.height = video.videoHeight;
            
            const brightnessVal = document.getElementById('brightness').value;
            ctx.filter = `brightness(${brightnessVal})`;
            ctx.drawImage(video, 0, 0, overlay.width, overlay.height);

            const sampleSize = 10;
            const startX = Math.floor(overlay.width / 2 - sampleSize / 2);
            const startY = Math.floor(overlay.height / 2 - sampleSize / 2);
            const frameData = ctx.getImageData(startX, startY, sampleSize, sampleSize).data;

            let totalR = 0, totalG = 0, totalB = 0, count = 0;

            for (let i = 0; i < frameData.length; i += 4) {
                totalR += frameData[i];
                totalG += frameData[i + 1];
                totalB += frameData[i + 2];
                count++;
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

            currentSampleRGB = [smoothR, smoothG, smoothB];

            const result = calculateColorMatch(smoothR, smoothG, smoothB);
            const color = result.match;

            if (color) {
                document.getElementById('swatch').style.backgroundColor = `rgb(${result.rgb[0]}, ${result.rgb[1]}, ${result.rgb[2]})`;
                document.getElementById('color-name').innerText = `${color.pos}. ${color.name}`;
                document.getElementById('color-pos').innerText = `Abbr: ${color.abbr}  |  Match: ${result.confidence}%`;
                speak(`${color.name}, Position ${color.pos}`);
            }
        }

        // White Balance Calibration
        document.getElementById('calib-btn').addEventListener('click', (e) => {
            const avg = (currentSampleRGB[0] + currentSampleRGB[1] + currentSampleRGB[2]) / 3 || 1;
            wbGain = [
                avg / (currentSampleRGB[0] || 1),
                avg / (currentSampleRGB[1] || 1),
                avg / (currentSampleRGB[2] || 1)
            ];
            e.target.classList.add('calib-active');
            e.target.innerText = "✓ Calibrated";
            setTimeout(() => {
                e.target.classList.remove('calib-active');
                e.target.innerText = "🎯 Calibrate White";
            }, 1800);
        });

        document.getElementById('start-btn').addEventListener('click', async () => {
            const silentUtterance = new SpeechSynthesisUtterance("");
            synth.speak(silentUtterance);

            document.getElementById('start-overlay').style.display = 'none';
            await initCamera();
            setInterval(sampleCenterColor, 120);
        });

        document.getElementById('torch-btn').addEventListener('click', async () => {
            torchOn = !torchOn;
            const torchBtn = document.getElementById('torch-btn');
            const screenFlash = document.getElementById('screen-flash');

            let hardwareSuccess = false;
            if (currentTrack && currentTrack.applyConstraints) {
                try {
                    await currentTrack.applyConstraints({ advanced: [{ torch: torchOn }] });
                    hardwareSuccess = true;
                } catch (e) { hardwareSuccess = false; }
            }

            if (!hardwareSuccess) {
                screenFlash.classList.toggle('screen-flash-on', torchOn);
            }

            torchBtn.innerText = torchOn ? "🔦 Torch: ON" : "🔦 Torch";
            torchBtn.classList.toggle('torch-active', torchOn);
        });

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
