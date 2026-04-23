
justlove20180-a11y
Mahbub
Repository navigation
Code
Issues
Pull requests
Actions
Projects
Wiki
Security and quality
Insights
Settings
Commit 89c669a
justlove20180-a11y
justlove20180-a11y
authored
1 minute ago
Verified
Add files via upload
main
1 parent 
63e170b
 commit 
89c669a
1 file changed

+158
Lines changed: 158 additions & 0 deletions
File tree
Filter files…
gemini-code-1776924025632.html
Search within code
 
‎gemini-code-1776924025632.html‎
+158
Lines changed: 158 additions & 0 deletions
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,158 @@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Seamless Collage with Adjustable Gap</title>
    <style>
        :root { --bg: #0f172a; --card: #1e293b; }
        body { font-family: sans-serif; background: var(--bg); color: white; text-align: center; padding: 20px; margin: 0; }
        
        .toolbar { background: var(--card); padding: 15px; border-radius: 12px; margin-bottom: 20px; display: inline-flex; gap: 15px; align-items: center; flex-wrap: wrap; justify-content: center; position: sticky; top: 10px; z-index: 100; }
        
        /* Gap Control Styling */
        .gap-control { display: flex; align-items: center; gap: 8px; font-weight: bold; }
        #collage-preview { 
            display: grid; 
            width: 90vw; 
            max-width: 600px;
            margin: 0 auto; 
            background: #fff; /* Gap er color ekhane change kora jabe */
            border: 2px solid #fff;
            overflow: hidden;
            gap: 5px; /* Default Gap */
        }
        .style-grid { grid-template-columns: 1fr 1fr; grid-auto-rows: 300px; }
        .style-mosaic { grid-template-columns: 1fr 1fr; grid-template-rows: 300px 300px; }
        .style-mosaic .img-box:first-child { grid-row: span 2; }
        .style-wide { grid-template-columns: 1fr; grid-auto-rows: 400px; }
        .img-box { position: relative; width: 100%; height: 100%; overflow: hidden; cursor: move; background: #000; }
        .img-box img { position: absolute; top: 0; left: 0; user-select: none; -webkit-user-drag: none; min-width: 100%; min-height: 100%; }
        
        button { background: #6366f1; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
        button:hover { background: #4f46e5; }
        input[type="file"] { background: #334155; padding: 8px; border-radius: 6px; color: white; }
        input[type="range"] { cursor: pointer; }
    </style>
</head>
<body>
    <div class="toolbar">
        <input type="file" id="imageUpload" multiple accept="image/*">
        
        <div class="gap-control">
            <span>Gap:</span>
            <input type="range" id="gapRange" min="0" max="20" value="5" oninput="updateGap(this.value)">
        </div>
        <button onclick="setStyle('style-grid')">Grid</button>
        <button onclick="setStyle('style-mosaic')">Mosaic</button>
        <button onclick="setStyle('style-wide')">Stack</button>
        <button style="background: #10b981;" onclick="downloadCollage()">Save Collage</button>
    </div>
    <div id="collage-preview" class="style-grid"></div>
    <canvas id="exportCanvas" style="display:none;"></canvas>
    <script>
        const upload = document.getElementById('imageUpload');
        const preview = document.getElementById('collage-preview');
        
        function updateGap(val) {
            preview.style.gap = val + "px";
        }
        upload.addEventListener('change', (e) => {
            const files = Array.from(e.target.files);
            preview.innerHTML = '';
            files.forEach(file => {
                const reader = new FileReader();
                reader.onload = (event) => {
                    const box = document.createElement('div');
                    box.className = 'img-box';
                    const img = new Image();
                    img.src = event.target.result;
                    img.onload = () => setupDraggable(img, box);
                    box.appendChild(img);
                    preview.appendChild(box);
                };
                reader.readAsDataURL(file);
            });
        });
        function setStyle(className) {
            preview.className = className;
        }
        function setupDraggable(img, box) {
            let isDragging = false, startX, startY, initialLeft, initialTop;
            
            const boxRect = box.getBoundingClientRect();
            const scale = Math.max(boxRect.width / img.naturalWidth, boxRect.height / img.naturalHeight);
            img.style.width = (img.naturalWidth * scale) + "px";
            img.style.height = (img.naturalHeight * scale) + "px";
            img.onmousedown = (e) => {
                isDragging = true;
                startX = e.clientX; startY = e.clientY;
                initialLeft = img.offsetLeft; initialTop = img.offsetTop;
            };
            window.onmousemove = (e) => {
                if (!isDragging) return;
                img.style.left = (initialLeft + (e.clientX - startX)) + "px";
                img.style.top = (initialTop + (e.clientY - startY)) + "px";
            };
            window.onmouseup = () => isDragging = false;
        }
        function downloadCollage() {
            const boxes = preview.querySelectorAll('.img-box');
            if (!boxes.length) return alert("Upload images!");
            const canvas = document.getElementById('exportCanvas');
            const ctx = canvas.getContext('2d');
            
            const containerRect = preview.getBoundingClientRect();
            const scaleFactor = 2000 / containerRect.width; 
            
            canvas.width = containerRect.width * scaleFactor;
            canvas.height = containerRect.height * scaleFactor;
            // Background color for the gap (White)
            ctx.fillStyle = "#ffffff";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            boxes.forEach(box => {
                const img = box.querySelector('img');
                const boxRect = box.getBoundingClientRect();
                
                const x = (boxRect.left - containerRect.left) * scaleFactor;
                const y = (boxRect.top - containerRect.top) * scaleFactor;
                const w = boxRect.width * scaleFactor;
                const h = boxRect.height * scaleFactor;
                const imgLeft = parseFloat(img.style.left || 0) * scaleFactor;
                const imgTop = parseFloat(img.style.top || 0) * scaleFactor;
                const imgW = parseFloat(img.style.width) * scaleFactor;
                const imgH = parseFloat(img.style.height) * scaleFactor;
                ctx.save();
                ctx.beginPath();
                ctx.rect(x, y, w, h);
                ctx.clip();
                ctx.drawImage(img, x + imgLeft, y + imgTop, imgW, imgH);
                ctx.restore();
            });
            const link = document.createElement('a');
            link.download = 'collage-with-gap.png';
            link.href = canvas.toDataURL('image/png', 1.0);
            link.click();
        }
    </script>
</body>
</html>
