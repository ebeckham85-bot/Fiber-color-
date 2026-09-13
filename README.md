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
            height: 50vh;
            max-height: 50dvh;
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
            width: 28px;
            height: 28px;
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

        /* 4 Button Layout Matrix */
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
            <button id="calib-btn" class="ctrl-btn">🎯 Calibrate White</button>
            <button id="torch-btn" class="ctrl-btn">🔦 Torch</button>
            <button id="toggle-speech" class="ctrl-btn">🗣 Voice: ON</button>
            <button id="freeze-btn" class="ctrl-btn">⏸ Freeze</button>
        </div>
    </div>

    <script>
        const TIA598 = [
            { pos: 1,  name: "BLUE",   abbr: "BL" },
            { pos: 2,  name: "ORANGE", abbr: "OR" },
            { pos: 3,  name: "GREEN",  abbr: "GR" },
            { pos: 4,  name: "BROWN",  abbr: "BR" },
            { pos: 5,  name: "SLATE",  abbr: "SL" },
            { pos: 6,  name: "WHITE",  abbr: "WH" },
            { pos: 7,  name: "RED",    abbr: "RD" },
            { pos: 8,  name: "BLACK",  abbr: "BK" },
            { pos: 9,  name: "YELLOW", abbr: "YL" },
            { pos: 10, name: "VIOLET", abbr: "VI" },
            { pos: 11, name: "ROSE",   abbr: "RS" },
            { pos: 12, name: "AQUA",   abbr: "AQ" }
        ];

        let wbGains = [1.0, 1.0, 1.0];

        // Converts RGB to HSV (Hue 0-360, Saturation 0-1, Value 0-1)
        function rgbToHsv(r, g, b) {
            r /= 255; g /= 255; b /= 255;
            let max = Math.max(r, g, b), min = Math.min(r, g, b);
            let h, s, v = max;
            let d = max - min;
            s = max === 0 ? 0 : d / max;

            if (max === min) {
                h = 0;
            } else {
                switch (max) {
                    case r: h = (g - b) / d + (g < b ? 6 : 0); break;
                    case g: h = (b - r) / d + 2; break;
                    case b: h = (r - g) / d + 4; break;
                }
                h /= 6;
            }
            return [h * 360, s, v];
        }

        // Dedicated Fiber TIA-598 Classifier
        function classifyFiberColor(r, g, b) {
            // Apply Manual White Balance Adjustment
            let adjR = Math.min(255, Math.max(0, r * wbGains[0]));
            let adjG = Math.min(255, Math.max(0, g * wbGains[1]));
            let adjB = Math.min(255, Math.max(0, b * wbGains[2]));

            let [h, s, v] = rgbToHsv(adjR, adjG, adjB);

            // 1. Black & White & Slate (Achromatic bounds)
            if (v < 0.18) return { match: TIA598[7], conf: 98, rgb: [adjR, adjG, adjB] }; // Black
            if (s < 0.16) {
                if (v > 0.72) return { match: TIA598[5], conf: 95, rgb: [adjR, adjG, adjB] }; // White
                return { match: TIA598[4], conf: 90, rgb: [adjR, adjG, adjB] }; // Slate
            }

            // 2. Chromatic Fiber Classification by calibrated Hue and Brightness/Saturation
            let match = null;

            if (h >= 345 || h < 11) {
                match = TIA598[6]; // Red
            } else if (h >= 11 && h < 38) {
                // Separation between Orange, Brown, and Yellow
                if (v < 0.45 && s > 0.35) {
                    match = TIA598[3]; // Brown (low brightness orange/yellow)
                } else {
                    match = TIA598[1]; // Orange
                }
            } else if (h >= 38 && h < 68) {
                // Separation between Yellow and Brown
                if (v < 0.40 && s > 0.40) {
                    match = TIA598[3]; // Brown
                } else {
                    match = TIA598[8]; // Yellow
                }
            } else if (h >= 68 && h < 165) {
                match = TIA598[2]; // Green
            } else if (h >= 165 && h < 200) {
                match = TIA598[11]; // Aqua
            } else if (h >= 200 && h < 255) {
                match = TIA598[0]; // Blue
            } else if (h >= 255 && h < 310) {
                match = TIA598[9]; // Violet
            } else if (h >= 310 && h < 345) {
                match = TIA598[10]; // Rose
            }

            return { match, conf: 92, rgb: [adjR, adjG, adjB] };
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
        let currentRawRGB = [128, 128, 128];
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

            const sampleSize = 8;
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

            currentRawRGB = [smoothR, smoothG, smoothB];

            const result = classifyFiberColor(smoothR, smoothG, smoothB);
            const color = result.match;

            if (color) {
                document.getElementById('swatch').style.backgroundColor = `rgb(${result.rgb[0]}, ${result.rgb[1]}, ${result.rgb[2]})`;
                document.getElementById('color-name').innerText = `${color.pos}. ${color.name}`;
                document.getElementById('color-pos').innerText = `Abbr: ${color.abbr}  |  Match: ${result.conf}%`;
                speak(`${color.name}, Position ${color.pos}`);
            }
        }

        // White Balance Calibration Action
        document.getElementById('calib-btn').addEventListener('click', (e) => {
            const avg = (currentRawRGB[0] + currentRawRGB[1] + currentRawRGB[2]) / 3 || 1;
            wbGains = [
                avg / (currentRawRGB[0] || 1),
                avg / (currentRawRGB[1] || 1),
                avg / (currentRawRGB[2] || 1)
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
