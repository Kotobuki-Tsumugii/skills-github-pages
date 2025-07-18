<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>朱利亚集合分形</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f0f0f0;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        h1 {
            color: #333;
            margin-bottom: 20px;
        }
        
        .canvas-container {
            width: 100%;
            max-width: 800px;
            margin: 20px 0;
            position: relative;
        }
        
        canvas {
            width: 100%;
            height: auto;
            background-color: white;
            border: 1px solid #ccc;
            display: block;
            cursor: crosshair;
            touch-action: none;
        }
        
        .controls {
            width: 100%;
            max-width: 800px;
            background-color: white;
            border-radius: 5px;
            padding: 15px;
            margin-bottom: 20px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        
        .control-group {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-bottom: 10px;
        }
        
        .control-item {
            flex: 1;
            min-width: 200px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input[type="range"] {
            width: 100%;
        }
        
        .value-display {
            display: inline-block;
            min-width: 40px;
            text-align: right;
        }
        
        .button-group {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
            margin-top: 10px;
        }
        
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 8px 16px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 14px;
            cursor: pointer;
            border-radius: 4px;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: #45a049;
        }
        
        .status {
            margin-top: 10px;
            padding: 10px;
            background-color: #f8f8f8;
            border-radius: 4px;
            font-size: 14px;
            color: #666;
            text-align: center;
        }
        
        .performance-info {
            margin-top: 10px;
            font-size: 12px;
            color: #888;
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>朱利亚集合分形</h1>
    
    <div class="controls">
        <div class="control-group">
            <div class="control-item">
                <label for="real">实部 (c<sub>real</sub>):</label>
                <input type="range" id="real" min="-1" max="1" step="0.01" value="-0.7">
                <span class="value-display" id="realValue">-0.7</span>
            </div>
            
            <div class="control-item">
                <label for="imag">虚部 (c<sub>imag</sub>):</label>
                <input type="range" id="imag" min="-1" max="1" step="0.01" value="0.27">
                <span class="value-display" id="imagValue">0.27</span>
            </div>
            
            <div class="control-item">
                <label for="iterations">迭代次数:</label>
                <input type="range" id="iterations" min="10" max="500" step="10" value="100">
                <span class="value-display" id="iterationsValue">100</span>
            </div>
        </div>
        
        <div class="button-group">
            <button id="randomize">随机生成</button>
            <button id="reset">重置</button>
            <button id="zoomIn">放大</button>
            <button id="zoomOut">缩小</button>
            <button id="highPrecision">高精度模式</button>
        </div>
        
        <div class="status" id="status">缩放: 1x | 点击画布可放大该区域 | 拖动可平移视图</div>
        <div class="performance-info" id="performanceInfo">计算时间: 0ms | 分辨率: 600×600</div>
    </div>
    
    <div class="canvas-container">
        <canvas id="juliaCanvas"></canvas>
    </div>
    
    <script>
        // 获取DOM元素
        const canvas = document.getElementById('juliaCanvas');
        const ctx = canvas.getContext('2d');
        const realInput = document.getElementById('real');
        const imagInput = document.getElementById('imag');
        const iterationsInput = document.getElementById('iterations');
        const realValue = document.getElementById('realValue');
        const imagValue = document.getElementById('imagValue');
        const iterationsValue = document.getElementById('iterationsValue');
        const randomizeBtn = document.getElementById('randomize');
        const resetBtn = document.getElementById('reset');
        const zoomInBtn = document.getElementById('zoomIn');
        const zoomOutBtn = document.getElementById('zoomOut');
        const highPrecisionBtn = document.getElementById('highPrecision');
        const statusDiv = document.getElementById('status');
        const performanceInfo = document.getElementById('performanceInfo');
        
        // 性能优化参数
        let useHighPrecision = false;
        let lastRenderTime = 0;
        
        // 初始化画布大小
        function initCanvasSize() {
            const container = document.querySelector('.canvas-container');
            const size = Math.min(container.clientWidth, 800);
            canvas.width = size;
            canvas.height = size;
            updatePerformanceInfo();
        }
        
        // 默认参数
        let cReal = parseFloat(realInput.value);
        let cImag = parseFloat(imagInput.value);
        let maxIterations = parseInt(iterationsInput.value);
        
        // 视图参数
        let view = {
            x: -2,
            y: -2,
            width: 4,
            height: 4,
            zoom: 1
        };
        
        // 交互状态
        let isDragging = false;
        let dragStartX = 0;
        let dragStartY = 0;
        let dragStartViewX = 0;
        let dragStartViewY = 0;
        let isClick = false;
        let clickTimeout = null;
        
        // 更新显示的值
        function updateDisplayValues() {
            realValue.textContent = cReal.toFixed(2);
            imagValue.textContent = cImag.toFixed(2);
            iterationsValue.textContent = maxIterations;
            statusDiv.textContent = `缩放: ${view.zoom.toFixed(2)}x | 点击画布可放大该区域 | 拖动可平移视图`;
            highPrecisionBtn.textContent = useHighPrecision ? "高精度模式(开启)" : "高精度模式(关闭)";
        }
        
        // 更新性能信息
        function updatePerformanceInfo() {
            performanceInfo.textContent = `计算时间: ${lastRenderTime}ms | 分辨率: ${canvas.width}×${canvas.height}`;
        }
        
        // 高性能计算朱利亚集合的逃逸时间
        function computeJulia(zx, zy) {
            let i = 0;
            if (useHighPrecision) {
                // 高精度计算模式
                let zxHigh = zx;
                let zyHigh = zy;
                while (i < maxIterations) {
                    const zx2 = zxHigh * zxHigh;
                    const zy2 = zyHigh * zyHigh;
                    if (zx2 + zy2 > 4) break;
                    
                    const tmp = zx2 - zy2 + cReal;
                    zyHigh = 2 * zxHigh * zyHigh + cImag;
                    zxHigh = tmp;
                    i++;
                }
            } else {
                // 标准计算模式
                let zx2 = zx * zx;
                let zy2 = zy * zy;
                while (i < maxIterations && zx2 + zy2 < 4) {
                    const tmp = zx2 - zy2 + cReal;
                    zy = 2 * zx * zy + cImag;
                    zx = tmp;
                    zx2 = zx * zx;
                    zy2 = zy * zy;
                    i++;
                }
            }
            return i;
        }
        
        // 使用Web Workers进行并行计算
        function createWorker() {
            const workerCode = `
                self.onmessage = function(e) {
                    const { startY, endY, width, height, cReal, cImag, maxIterations, view, useHighPrecision } = e.data;
                    const imageData = new Uint8ClampedArray(width * (endY - startY) * 4);
                    
                    for (let y = startY; y < endY; y++) {
                        for (let x = 0; x < width; x++) {
                            const zx = view.x + (x / width) * view.width;
                            const zy = view.y + (y / height) * view.height;
                            
                            let i = 0;
                            if (useHighPrecision) {
                                let zxHigh = zx;
                                let zyHigh = zy;
                                while (i < maxIterations) {
                                    const zx2 = zxHigh * zxHigh;
                                    const zy2 = zyHigh * zyHigh;
                                    if (zx2 + zy2 > 4) break;
                                    
                                    const tmp = zx2 - zy2 + cReal;
                                    zyHigh = 2 * zxHigh * zyHigh + cImag;
                                    zxHigh = tmp;
                                    i++;
                                }
                            } else {
                                let zx2 = zx * zx;
                                let zy2 = zy * zy;
                                while (i < maxIterations && zx2 + zy2 < 4) {
                                    const tmp = zx2 - zy2 + cReal;
                                    zy = 2 * zx * zy + cImag;
                                    zx = tmp;
                                    zx2 = zx * zx;
                                    zy2 = zy * zy;
                                    i++;
                                }
                            }
                            
                            const idx = ((y - startY) * width + x) * 4;
                            
                            if (i === maxIterations) {
                                imageData[idx] = 0;
                                imageData[idx + 1] = 0;
                                imageData[idx + 2] = 0;
                            } else {
                                const hue = (i / maxIterations) * 360;
                                const saturation = 100;
                                const lightness = 50 + 50 * Math.sin(i / 10);
                                
                                const c = (1 - Math.abs(2 * lightness / 100 - 1)) * saturation / 100;
                                const x = c * (1 - Math.abs((hue / 60) % 2 - 1));
                                const m = lightness / 100 - c / 2;
                                
                                let r, g, b;
                                if (hue < 60) { r = c; g = x; b = 0; }
                                else if (hue < 120) { r = x; g = c; b = 0; }
                                else if (hue < 180) { r = 0; g = c; b = x; }
                                else if (hue < 240) { r = 0; g = x; b = c; }
                                else if (hue < 300) { r = x; g = 0; b = c; }
                                else { r = c; g = 0; b = x; }
                                
                                imageData[idx] = Math.round((r + m) * 255);
                                imageData[idx + 1] = Math.round((g + m) * 255);
                                imageData[idx + 2] = Math.round((b + m) * 255);
                            }
                            
                            imageData[idx + 3] = 255;
                        }
                    }
                    
                    self.postMessage({ startY, endY, imageData });
                };
            `;
            
            const blob = new Blob([workerCode], { type: 'application/javascript' });
            return new Worker(URL.createObjectURL(blob));
        }
        
        // 使用多线程绘制朱利亚集合
        function drawJuliaSetParallel() {
            const startTime = performance.now();
            const width = canvas.width;
            const height = canvas.height;
            const imageData = ctx.createImageData(width, height);
            const numWorkers = navigator.hardwareConcurrency || 4;
            const workers = [];
            let completedWorkers = 0;
            const rowsPerWorker = Math.ceil(height / numWorkers);
            
            for (let i = 0; i < numWorkers; i++) {
                const startY = i * rowsPerWorker;
                const endY = Math.min(startY + rowsPerWorker, height);
                
                const worker = createWorker();
                workers.push(worker);
                
                worker.onmessage = function(e) {
                    const { startY, endY, imageData: partData } = e.data;
                    
                    for (let y = startY; y < endY; y++) {
                        for (let x = 0; x < width; x++) {
                            const srcIdx = ((y - startY) * width + x) * 4;
                            const dstIdx = (y * width + x) * 4;
                            
                            imageData.data[dstIdx] = partData[srcIdx];
                            imageData.data[dstIdx + 1] = partData[srcIdx + 1];
                            imageData.data[dstIdx + 2] = partData[srcIdx + 2];
                            imageData.data[dstIdx + 3] = partData[srcIdx + 3];
                        }
                    }
                    
                    completedWorkers++;
                    if (completedWorkers === numWorkers) {
                        ctx.putImageData(imageData, 0, 0);
                        lastRenderTime = Math.round(performance.now() - startTime);
                        updatePerformanceInfo();
                        
                        // 终止所有worker
                        workers.forEach(w => w.terminate());
                    }
                };
                
                worker.postMessage({
                    startY,
                    endY,
                    width,
                    height,
                    cReal,
                    cImag,
                    maxIterations,
                    view,
                    useHighPrecision
                });
            }
        }
        
        // 单线程绘制朱利亚集合
        function drawJuliaSetSingle() {
            const startTime = performance.now();
            const width = canvas.width;
            const height = canvas.height;
            const imageData = ctx.createImageData(width, height);
            const data = imageData.data;
            
            for (let x = 0; x < width; x++) {
                for (let y = 0; y < height; y++) {
                    const zx = view.x + (x / width) * view.width;
                    const zy = view.y + (y / height) * view.height;
                    
                    const i = computeJulia(zx, zy);
                    const idx = (y * width + x) * 4;
                    
                    if (i === maxIterations) {
                        data[idx] = 0;
                        data[idx + 1] = 0;
                        data[idx + 2] = 0;
                    } else {
                        const hue = (i / maxIterations) * 360;
                        const saturation = 100;
                        const lightness = 50 + 50 * Math.sin(i / 10);
                        
                        const c = (1 - Math.abs(2 * lightness / 100 - 1)) * saturation / 100;
                        const x = c * (1 - Math.abs((hue / 60) % 2 - 1));
                        const m = lightness / 100 - c / 2;
                        
                        let r, g, b;
                        if (hue < 60) { r = c; g = x; b = 0; }
                        else if (hue < 120) { r = x; g = c; b = 0; }
                        else if (hue < 180) { r = 0; g = c; b = x; }
                        else if (hue < 240) { r = 0; g = x; b = c; }
                        else if (hue < 300) { r = x; g = 0; b = c; }
                        else { r = c; g = 0; b = x; }
                        
                        data[idx] = Math.round((r + m) * 255);
                        data[idx + 1] = Math.round((g + m) * 255);
                        data[idx + 2] = Math.round((b + m) * 255);
                    }
                    
                    data[idx + 3] = 255;
                }
            }
            
            ctx.putImageData(imageData, 0, 0);
            lastRenderTime = Math.round(performance.now() - startTime);
            updatePerformanceInfo();
        }
        
        // 绘制朱利亚集合（自动选择模式）
        function drawJuliaSet() {
            if (canvas.width > 500 && typeof Worker !== 'undefined') {
                drawJuliaSetParallel();
            } else {
                drawJuliaSetSingle();
            }
        }
        
        // 放大指定区域
        function zoomAt(x, y, factor) {
            const newWidth = view.width / factor;
            const newHeight = view.height / factor;
            
            const centerX = view.x + (x / canvas.width) * view.width;
            const centerY = view.y + (y / canvas.height) * view.height;
            
            view.x = centerX - newWidth / 2;
            view.y = centerY - newHeight / 2;
            view.width = newWidth;
            view.height = newHeight;
            view.zoom *= factor;
            
            drawJuliaSet();
            updateDisplayValues();
        }
        
        // 平移视图
        function panView(dx, dy) {
            const dxView = dx * view.width / canvas.width;
            const dyView = dy * view.height / canvas.height;
            
            view.x -= dxView;
            view.y -= dyView;
            
            drawJuliaSet();
            updateDisplayValues();
        }
        
        // 重置视图
        function resetView() {
            view = {
                x: -2,
                y: -2,
                width: 4,
                height: 4,
                zoom: 1
            };
            drawJuliaSet();
            updateDisplayValues();
        }
        
        // 初始化
        function init() {
            initCanvasSize();
            window.addEventListener('resize', () => {
                initCanvasSize();
                drawJuliaSet();
            });
            
            // 参数变化事件
            realInput.addEventListener('input', function() {
                cReal = parseFloat(this.value);
                updateDisplayValues();
                drawJuliaSet();
            });
            
            imagInput.addEventListener('input', function() {
                cImag = parseFloat(this.value);
                updateDisplayValues();
                drawJuliaSet();
            });
            
            iterationsInput.addEventListener('input', function() {
                maxIterations = parseInt(this.value);
                updateDisplayValues();
                drawJuliaSet();
            });
            
            // 按钮事件
            randomizeBtn.addEventListener('click', function() {
                cReal = (Math.random() * 2) - 1;
                cImag = (Math.random() * 2) - 1;
                realInput.value = cReal;
                imagInput.value = cImag;
                updateDisplayValues();
                drawJuliaSet();
            });
            
            resetBtn.addEventListener('click', function() {
                cReal = -0.7;
                cImag = 0.27;
                maxIterations = 100;
                realInput.value = cReal;
                imagInput.value = cImag;
                iterationsInput.value = maxIterations;
                resetView();
            });
            
            zoomInBtn.addEventListener('click', function() {
                zoomAt(canvas.width / 2, canvas.height / 2, 2);
            });
            
            zoomOutBtn.addEventListener('click', function() {
                zoomAt(canvas.width / 2, canvas.height / 2, 0.5);
            });
            
            highPrecisionBtn.addEventListener('click', function() {
                useHighPrecision = !useHighPrecision;
                updateDisplayValues();
                drawJuliaSet();
            });
            
            // 鼠标事件
            canvas.addEventListener('mousedown', function(e) {
                const rect = canvas.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                
                isDragging = true;
                dragStartX = x;
                dragStartY = y;
                dragStartViewX = view.x;
                dragStartViewY = view.y;
                canvas.style.cursor = 'grabbing';
                
                isClick = true;
                if (clickTimeout) clearTimeout(clickTimeout);
                clickTimeout = setTimeout(() => {
                    isClick = false;
                }, 200);
            });
            
            canvas.addEventListener('mousemove', function(e) {
                if (isDragging) {
                    const rect = canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    
                    const dx = x - dragStartX;
                    const dy = y - dragStartY;
                    
                    panView(dx, dy);
                    
                    dragStartX = x;
                    dragStartY = y;
                    dragStartViewX = view.x;
                    dragStartViewY = view.y;
                    
                    if (Math.abs(dx) > 5 || Math.abs(dy) > 5) {
                        isClick = false;
                    }
                }
            });
            
            canvas.addEventListener('mouseup', function(e) {
                if (isDragging) {
                    const rect = canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    
                    if (isClick) {
                        zoomAt(x, y, 2);
                    }
                    
                    isDragging = false;
                    canvas.style.cursor = 'crosshair';
                }
            });
            
            canvas.addEventListener('mouseleave', function() {
                isDragging = false;
                canvas.style.cursor = 'crosshair';
            });
            
            // 触摸事件
            canvas.addEventListener('touchstart', function(e) {
                e.preventDefault();
                if (e.touches.length === 1) {
                    const rect = canvas.getBoundingClientRect();
                    const x = e.touches[0].clientX - rect.left;
                    const y = e.touches[0].clientY - rect.top;
                    
                    isDragging = true;
                    dragStartX = x;
                    dragStartY = y;
                    dragStartViewX = view.x;
                    dragStartViewY = view.y;
                    
                    isClick = true;
                    if (clickTimeout) clearTimeout(clickTimeout);
                    clickTimeout = setTimeout(() => {
                        isClick = false;
                    }, 200);
                }
            });
            
            canvas.addEventListener('touchmove', function(e) {
                e.preventDefault();
                if (isDragging && e.touches.length === 1) {
                    const rect = canvas.getBoundingClientRect();
                    const x = e.touches[0].clientX - rect.left;
                    const y = e.touches[0].clientY - rect.top;
                    
                    const dx = x - dragStartX;
                    const dy = y - dragStartY;
                    
                    panView(dx, dy);
                    
                    dragStartX = x;
                    dragStartY = y;
                    dragStartViewX = view.x;
                    dragStartViewY = view.y;
                    
                    if (Math.abs(dx) > 10 || Math.abs(dy) > 10) {
                        isClick = false;
                    }
                }
            });
            
            canvas.addEventListener('touchend', function(e) {
                e.preventDefault();
                if (isDragging && e.touches.length === 0 && e.changedTouches.length === 1) {
                    const rect = canvas.getBoundingClientRect();
                    const x = e.changedTouches[0].clientX - rect.left;
                    const y = e.changedTouches[0].clientY - rect.top;
                    
                    if (isClick) {
                        zoomAt(x, y, 2);
                    }
                    
                    isDragging = false;
                }
            });
            
            // 初始绘制
            updateDisplayValues();
            drawJuliaSet();
        }
        
        // 启动应用
        init();
    </script>
</body>
</html>
