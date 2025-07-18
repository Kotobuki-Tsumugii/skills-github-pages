<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>可交互的朱利亚集合分形演示</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f0f0f0;
            margin: 0;
            padding: 20px;
        }
        h1 {
            color: #333;
        }
        canvas {
            background-color: white;
            border: 1px solid #ccc;
            margin: 20px auto;
            display: block;
            cursor: crosshair;
        }
        .controls {
            margin: 20px auto;
            padding: 15px;
            background-color: white;
            border-radius: 5px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            max-width: 600px;
        }
        label {
            display: inline-block;
            margin: 5px;
        }
        input {
            vertical-align: middle;
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
            margin: 4px 2px;
            cursor: pointer;
            border-radius: 4px;
        }
        .status {
            margin-top: 10px;
            font-size: 14px;
            color: #666;
        }
    </style>
</head>
<body>
    <h1>可交互的朱利亚集合分形演示</h1>
    
    <div class="controls">
        <label for="real">实部 (c<sub>real</sub>):</label>
        <input type="range" id="real" min="-1" max="1" step="0.01" value="-0.7">
        <span id="realValue">-0.7</span>
        
        <label for="imag">虚部 (c<sub>imag</sub>):</label>
        <input type="range" id="imag" min="-1" max="1" step="0.01" value="0.27">
        <span id="imagValue">0.27</span>
        
        <label for="iterations">迭代次数:</label>
        <input type="range" id="iterations" min="10" max="200" step="10" value="100">
        <span id="iterationsValue">100</span>
        
        <button id="randomize">随机生成</button>
        <button id="reset">重置</button>
        <button id="zoomIn">放大</button>
        <button id="zoomOut">缩小</button>
        <div class="status" id="status">缩放: 1x | 点击画布可放大该区域</div>
    </div>
    
    <canvas id="juliaCanvas" width="600" height="600"></canvas>
    
    <script>
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
        const statusDiv = document.getElementById('status');
        
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
        
        // 更新显示的值
        function updateDisplayValues() {
            realValue.textContent = cReal.toFixed(2);
            imagValue.textContent = cImag.toFixed(2);
            iterationsValue.textContent = maxIterations;
            statusDiv.textContent = `缩放: ${view.zoom.toFixed(2)}x | 点击画布可放大该区域`;
        }
        
        // 计算朱利亚集合的逃逸时间
        function computeJulia(zx, zy) {
            let i = 0;
            while (i < maxIterations && zx * zx + zy * zy < 4) {
                const tmp = zx * zx - zy * zy + cReal;
                zy = 2 * zx * zy + cImag;
                zx = tmp;
                i++;
            }
            return i;
        }
        
        // 绘制朱利亚集合
        function drawJuliaSet() {
            const width = canvas.width;
            const height = canvas.height;
            const imageData = ctx.createImageData(width, height);
            const data = imageData.data;
            
            for (let x = 0; x < width; x++) {
                for (let y = 0; y < height; y++) {
                    // 将像素坐标映射到当前视图范围
                    const zx = view.x + (x / width) * view.width;
                    const zy = view.y + (y / height) * view.height;
                    
                    const i = computeJulia(zx, zy);
                    
                    // 计算像素索引
                    const idx = (y * width + x) * 4;
                    
                    // 根据迭代次数设置颜色
                    if (i === maxIterations) {
                        // 集合内的点 - 黑色
                        data[idx] = 0;
                        data[idx + 1] = 0;
                        data[idx + 2] = 0;
                    } else {
                        // 集合外的点 - 根据迭代次数设置颜色
                        const hue = (i / maxIterations) * 360;
                        const saturation = 100;
                        const lightness = i < maxIterations / 2 ? 50 : 100 - (i / maxIterations) * 100;
                        
                        // 将HSL转换为RGB
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
                    
                    data[idx + 3] = 255; // Alpha通道
                }
            }
            
            ctx.putImageData(imageData, 0, 0);
        }
        
        // 放大指定区域
        function zoomAt(x, y, factor) {
            const newWidth = view.width / factor;
            const newHeight = view.height / factor;
            
            // 计算新的视图中心
            const centerX = view.x + (x / canvas.width) * view.width;
            const centerY = view.y + (y / canvas.height) * view.height;
            
            // 更新视图参数
            view.x = centerX - newWidth / 2;
            view.y = centerY - newHeight / 2;
            view.width = newWidth;
            view.height = newHeight;
            view.zoom *= factor;
            
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
        
        // 事件监听器
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
        
        randomizeBtn.addEventListener('click', function() {
            cReal = (Math.random() * 2) - 1; // -1 到 1
            cImag = (Math.random() * 2) - 1; // -1 到 1
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
        
        canvas.addEventListener('click', function(e) {
            const rect = canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            zoomAt(x, y, 2);
        });
        
        // 初始化
        updateDisplayValues();
        drawJuliaSet();
    </script>
</body>
</html>
