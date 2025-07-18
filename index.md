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
            text-align: center;
        }
        
        .container {
            width: 100%;
            max-width: 1000px;
        }
        
        .canvas-wrapper {
            width: 100%;
            position: relative;
            margin: 20px 0;
        }
        
        canvas {
            width: 100%;
            height: auto;
            max-width: 600px;
            max-height: 600px;
            background-color: white;
            border: 1px solid #ccc;
            display: block;
            margin: 0 auto;
            cursor: grab;
            touch-action: none;
        }
        
        .controls {
            width: 100%;
            max-width: 600px;
            margin: 20px auto;
            background-color: white;
            border-radius: 5px;
            padding: 15px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        
        .control-group {
            display: flex;
            flex-wrap: wrap;
            margin-bottom: 10px;
        }
        
        .control-item {
            flex: 1;
            min-width: 150px;
            margin: 5px;
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
            border-radius: 4px;
            cursor: pointer;
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
            text-align: center;
            font-size: 14px;
            color: #666;
        }
        
        @media (max-width: 600px) {
            .control-item {
                min-width: 100%;
            }
        }
    </style>
</head>
<body>
    <div class="container">
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
                    <input type="range" id="iterations" min="10" max="200" step="10" value="100">
                    <span class="value-display" id="iterationsValue">100</span>
                </div>
            </div>
            
            <div class="button-group">
                <button id="randomize">随机生成</button>
                <button id="reset">重置</button>
                <button id="zoomIn">放大</button>
                <button id="zoomOut">缩小</button>
            </div>
            
            <div class="status" id="status">缩放: 1x | 点击画布可放大该区域 | 拖动可平移视图</div>
        </div>
        
        <div class="canvas-wrapper">
            <canvas id="juliaCanvas"></canvas>
        </div>
    </div>
    
    <script>
        // 获取DOM元素
        const canvas = document.getElementById('juliaCanvas');
        const ctx = canvas.getContext('2d');
        
        // 设置画布大小
        function initCanvasSize() {
            const container = document.querySelector('.canvas-wrapper');
            const size = Math.min(container.clientWidth, 600);
            canvas.width = size;
            canvas.height = size;
        }
        
        // 初始化参数
        let cReal = parseFloat(document.getElementById('real').value);
        let cImag = parseFloat(document.getElementById('imag').value);
        let maxIterations = parseInt(document.getElementById('iterations').value);
        
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
            document.getElementById('realValue').textContent = cReal.toFixed(2);
            document.getElementById('imagValue').textContent = cImag.toFixed(2);
            document.getElementById('iterationsValue').textContent = maxIterations;
            document.getElementById('status').textContent = 
                `缩放: ${view.zoom.toFixed(2)}x | 点击画布可放大该区域 | 拖动可平移视图`;
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
                        // 集合外的点 - 平滑着色
                        const hue = (i / maxIterations) * 360;
                        const saturation = 100;
                        const lightness = 50 + 50 * Math.sin(i / 10);
                        
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
        
        // 平移视图
        function panView(dx, dy) {
            // 将像素位移转换为视图坐标位移
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
        
        // 初始化画布和事件监听
        function init() {
            initCanvasSize();
            window.addEventListener('resize', () => {
                initCanvasSize();
                drawJuliaSet();
            });
            
            // 参数变化事件
            document.getElementById('real').addEventListener('input', function() {
                cReal = parseFloat(this.value);
                updateDisplayValues();
                drawJuliaSet();
            });
            
            document.getElementById('imag').addEventListener('input', function() {
                cImag = parseFloat(this.value);
                updateDisplayValues();
                drawJuliaSet();
            });
            
            document.getElementById('iterations').addEventListener('input', function() {
                maxIterations = parseInt(this.value);
                updateDisplayValues();
                drawJuliaSet();
            });
            
            // 按钮事件
            document.getElementById('randomize').addEventListener('click', function() {
                cReal = (Math.random() * 2) - 1;
                cImag = (Math.random() * 2) - 1;
                document.getElementById('real').value = cReal;
                document.getElementById('imag').value = cImag;
                updateDisplayValues();
                drawJuliaSet();
            });
            
            document.getElementById('reset').addEventListener('click', function() {
                cReal = -0.7;
                cImag = 0.27;
                maxIterations = 100;
                document.getElementById('real').value = cReal;
                document.getElementById('imag').value = cImag;
                document.getElementById('iterations').value = maxIterations;
                resetView();
            });
            
            document.getElementById('zoomIn').addEventListener('click', function() {
                zoomAt(canvas.width / 2, canvas.height / 2, 2);
            });
            
            document.getElementById('zoomOut').addEventListener('click', function() {
                zoomAt(canvas.width / 2, canvas.height / 2, 0.5);
            });
            
            // 鼠标事件 - 分离点击和拖动
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
                
                // 设置点击状态
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
                    
                    // 平移视图
                    panView(dx, dy);
                    
                    // 更新起始位置
                    dragStartX = x;
                    dragStartY = y;
                    dragStartViewX = view.x;
                    dragStartViewY = view.y;
                    
                    // 如果移动距离超过阈值，则不是点击
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
                        // 处理点击事件（放大）
                        zoomAt(x, y, 2);
                    }
                    
                    isDragging = false;
                    canvas.style.cursor = 'grab';
                }
            });
            
            canvas.addEventListener('mouseleave', function() {
                isDragging = false;
                canvas.style.cursor = 'grab';
            });
            
            // 触摸事件处理
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
                    
                    // 设置点击状态
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
                    
                    // 平移视图
                    panView(dx, dy);
                    
                    // 更新起始位置
                    dragStartX = x;
                    dragStartY = y;
                    dragStartViewX = view.x;
                    dragStartViewY = view.y;
                    
                    // 如果移动距离超过阈值，则不是点击
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
                        // 处理点击事件（放大）
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
