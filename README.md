# <!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pro Supplement Auditor (Crop + Excel + OCR)</title>

<script src='https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js'></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.css"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<style>
    :root {
        --primary: #8e44ad;
        --secondary: #2c3e50;
        --success: #27ae60;
        --danger: #c0392b;
        --bg: #f5f7fa;
    }
    body {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        line-height: 1.6;
        color: #333;
        max-width: 1400px;
        margin: 0 auto;
        padding: 20px;
        background-color: var(--bg);
    }

    /* HEADER & TABS */
    header { text-align: center; margin-bottom: 20px; padding: 20px; background-color: var(--secondary); color: white; border-radius: 8px; }
    h1 { margin: 0; }
    
    .tabs { display: flex; border-bottom: 2px solid #ddd; margin-bottom: 20px; }
    .tab {
        padding: 12px 25px; cursor: pointer; background: #e0e0e0; margin-right: 5px; border-radius: 8px 8px 0 0; font-weight: bold;
    }
    .tab.active { background: var(--primary); color: white; }
    
    .tab-content { display: none; background: white; padding: 20px; border-radius: 8px; }
    .tab-content.active { display: block; }

    /* LAYOUTS */
    .audit-grid { display: grid; grid-template-columns: 45% 55%; gap: 20px; }
    .panel { border: 1px solid #ddd; padding: 15px; border-radius: 8px; background: #fff; }

    /* CROPPER AREA */
    .img-container {
        height: 400px;
        background: #333;
        margin-bottom: 15px;
        border-radius: 4px;
        overflow: hidden;
    }
    #image-to-crop { max-width: 100%; display: block; } /* CropperJS requires block image */

    /* TEXT EDITOR */
    .text-editor {
        width: 100%; height: 150px; padding: 10px; border: 2px dashed var(--primary);
        border-radius: 6px; background: #fdfbff; font-family: monospace; resize: vertical; box-sizing: border-box;
    }

    /* BUTTONS */
    .btn { border: none; padding: 10px 15px; border-radius: 4px; cursor: pointer; font-weight: bold; width: 100%; margin-top: 5px; color: white; transition: 0.2s; }
    .btn-crop { background: var(--primary); }
    .btn-excel { background: #27ae60; }
    .btn-parse { background: #2c3e50; }
    .btn:hover { opacity: 0.9; }

    /* RESULTS */
    .audit-row { display: grid; grid-template-columns: 2fr 1fr 1fr 0.5fr; gap: 5px; padding: 8px; border-bottom: 1px solid #eee; align-items: center; }
    .audit-row select, .audit-row input { padding: 5px; border: 1px solid #ccc; width: 100%; box-sizing: border-box;}
    
    /* BADGES */
    .badge { padding: 4px 8px; border-radius: 4px; color: white; font-size: 0.85em; font-weight: bold; }
    .bg-green { background: #27ae60; }
    .bg-red { background: #c0392b; }
    
    /* PROGRESS */
    .progress-bar { height: 5px; background: #eee; margin-top: 5px; display: none; }
    .progress-fill { height: 100%; background: var(--primary); width: 0%; transition: width 0.2s; }

    .file-input-wrapper { margin-bottom: 10px; }
    input[type="file"] { display: block; margin-bottom: 5px; }

</style>
</head>
<body>

<header>
    <h1>Pro Supplement Auditor</h1>
    <p>Crop Image | Upload Excel | AI Text Scan</p>
</header>

<div class="tabs">
    <div class="tab active" data-tab="audit">AI Label Auditor</div>
    <div class="tab" data-tab="minerals">Minerals Calculator</div>
</div>

<div id="audit" class="tab-content active">
    <div class="audit-grid">
        
        <div class="panel">
            <h3 style="margin-top:0;">1. Input Source</h3>
            
            <div style="background:#e8f5e9; padding:10px; border-radius:4px; margin-bottom:15px; border:1px solid #c8e6c9;">
                <strong>Option A: Upload Excel</strong>
                <input type="file" id="excel-upload" accept=".xlsx, .xls" onchange="handleExcel(event)">
                <small style="color:#666;">Format: Col A (Ingredient), Col B (Amount)</small>
            </div>

            <hr style="border:0; border-top:1px solid #eee; margin:15px 0;">

            <div>
                <strong>Option B: Image Scan</strong>
                <input type="file" id="img-upload" accept="image/*" onchange="setupCropper(event)">
                
                <div class="img-container">
                    <img id="image-to-crop" src="" alt="Upload an image to start">
                </div>

                <button class="btn btn-crop" id="btn-scan-crop" onclick="scanCroppedImage()" style="display:none;">
                    ✂️ Crop & Scan Selection
                </button>
                
                <div class="progress-bar" id="progress-bar"><div class="progress-fill" id="progress-fill"></div></div>
                <div id="ocr-status" style="text-align:center; font-size:0.85em; color:#666; margin-top:5px;"></div>
            </div>
        </div>

        <div class="panel">
            <h3 style="margin-top:0;">2. Edit Text & Verify</h3>
            
            <p style="font-size:0.9em; margin-bottom:5px;">Scanned text will appear below. You can also paste manually.</p>
            <textarea id="text-editor" class="text-editor" placeholder="Paste label text here...
Example:
Zinc Sulphate
42 mg"></textarea>
            
            <button class="btn btn-parse" onclick="processTextToRows()">⬇️ 3. Process Ingredients</button>

            <div style="margin-top:20px; border-top:2px solid #eee; padding-top:10px;">
                <div style="display:flex; justify-content:space-between; align-items:center;">
                    <strong>Detected Ingredients</strong>
                    <button onclick="addEmptyRow()" style="width:auto; padding:5px 10px; background:#ddd; color:#333;">+ Manual Row</button>
                </div>

                <div id="rows-container" style="max-height:300px; overflow-y:auto; border:1px solid #eee; margin-top:10px;">
                    <p style="text-align:center; color:#999; padding:20px;">Rows will appear here...</p>
                </div>
            </div>

            <div style="margin-top:15px;">
                <label>RDA Standard:</label>
                <select id="rda-group" style="padding:8px; width:100%;">
                    <option value="men">Adult Men (ICMR 2020)</option>
                    <option value="women">Adult Women (ICMR 2020)</option>
                </select>
                <button class="btn btn-excel" onclick="generateReport()">Generate Final Report</button>
            </div>

            <div id="final-report" style="display:none; margin-top:15px; border-top:2px solid #8e44ad; padding-top:10px;">
                <h4>Audit Report</h4>
                <table style="width:100%; border-collapse:collapse; font-size:0.9em;">
                    <thead style="background:#eee;"><tr><th style="padding:5px;">Ingredient</th><th>Yield</th><th>RDA Status</th></tr></thead>
                    <tbody id="report-body"></tbody>
                </table>
            </div>
        </div>
    </div>
</div>

<div id="minerals" class="tab-content">
    <h3>Minerals Calculator</h3>
    <p>Please switch to the <strong>AI Label Auditor</strong> tab for the image/excel features.</p>
</div>

<script>
// --- DATABASE & KEYWORDS ---
const db = {
    "zinc_sulphate": { name: "Zinc Sulphate", yield: 0.42, rda: { men: 17, women: 13.2 } },
    "ferrous_bisglycinate": { name: "Ferrous Bisglycinate", yield: 0.20, rda: { men: 19, women: 29 } },
    "nac": { name: "N-Acetyl L-Cysteine", yield: 1.0, rda: null },
    "b3": { name: "Vitamin B3", yield: 1.0, rda: { men: 18, women: 14 } },
    "quercetin": { name: "Quercetin", yield: 1.0, rda: null },
    "grape_seed": { name: "Grape Seed Extract", yield: 1.0, rda: null },
    "green_tea": { name: "Green Tea Extract", yield: 1.0, rda: null },
    "b5": { name: "Vitamin B5", yield: 0.92, rda: { men: 5, women: 5 } },
    "sodium_selenite": { name: "Sodium Selenite", yield: 0.45, unit_pref: "mcg", rda: { men: 40, women: 40 } }
};

const keyMap = [
    { key: "zinc_sulphate", keywords: ["zinc", "sulphate"] },
    { key: "ferrous_bisglycinate", keywords: ["ferrous", "bisglycinate"] },
    { key: "nac", keywords: ["acetyl", "cysteine"] },
    { key: "b3", keywords: ["b3", "nicotinamide"] },
    { key: "quercetin", keywords: ["quercetin"] },
    { key: "grape_seed", keywords: ["grape", "seed"] },
    { key: "green_tea", keywords: ["green", "tea"] },
    { key: "b5", keywords: ["b5", "pantothenate"] },
    { key: "sodium_selenite", keywords: ["selenite", "selenium"] }
];

// --- CROPPER LOGIC ---
let cropper;

function setupCropper(event) {
    const file = event.target.files[0];
    if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
            const image = document.getElementById('image-to-crop');
            image.src = e.target.result;
            
            // Destroy old instance if exists
            if (cropper) { cropper.destroy(); }
            
            // Init Cropper
            cropper = new Cropper(image, {
                viewMode: 1,
                autoCropArea: 0.8,
            });
            document.getElementById('btn-scan-crop').style.display = 'block';
        };
        reader.readAsDataURL(file);
    }
}

function scanCroppedImage() {
    if (!cropper) return;
    
    // Get cropped area as canvas
    const canvas = cropper.getCroppedCanvas();
    const dataUrl = canvas.toDataURL('image/png');
    
    // Start Tesseract
    const status = document.getElementById('ocr-status');
    const bar = document.getElementById('progress-bar');
    const fill = document.getElementById('progress-fill');
    
    bar.style.display = 'block';
    status.textContent = "Scanning cropped area...";
    
    Tesseract.recognize(dataUrl, 'eng', {
        logger: m => {
            if (m.status === 'recognizing text') {
                fill.style.width = (m.progress * 100) + '%';
            }
        }
    }).then(({ data: { text } }) => {
        bar.style.display = 'none';
        status.textContent = "Scan complete!";
        
        // DUMP TEXT INTO EDITOR
        document.getElementById('text-editor').value = text;
    });
}

// --- EXCEL LOGIC ---
function handleExcel(e) {
    const file = e.target.files[0];
    const reader = new FileReader();
    
    reader.onload = (event) => {
        const data = new Uint8Array(event.target.result);
        const workbook = XLSX.read(data, {type: 'array'});
        const firstSheet = workbook.Sheets[workbook.SheetNames[0]];
        const jsonData = XLSX.utils.sheet_to_json(firstSheet, {header: 1}); // Read as array of arrays
        
        let textOutput = "";
        // Convert Excel rows to "Name \n Amount" format for the parser
        jsonData.forEach(row => {
            if(row[0] && row[1]) {
                textOutput += `${row[0]}\n${row[1]}\n`;
            }
        });
        
        document.getElementById('text-editor').value = textOutput;
        alert("Excel loaded into Editor. Click 'Process Ingredients' to finish.");
    };
    reader.readAsArrayBuffer(file);
}

// --- PARSING & UI LOGIC ---
let rowCount = 0;

function processTextToRows() {
    const text = document.getElementById('text-editor').value;
    const lines = text.split('\n');
    const container = document.getElementById('rows-container');
    container.innerHTML = ''; // Clear

    lines.forEach((line, index) => {
        line = line.toLowerCase().trim();
        if(!line) return;

        let matchedKey = null;
        for(let item of keyMap) {
            if(item.keywords.some(k => line.includes(k))) {
                matchedKey = item.key;
                break;
            }
        }

        if(matchedKey) {
            // Found ingredient, look for amount in this line or next
            let amtObj = extractAmount(line);
            
            // If no amount in current line, check next line
            if(!amtObj.amt && lines[index+1]) {
                let nextLine = lines[index+1].toLowerCase();
                let nextAmt = extractAmount(nextLine);
                if(nextAmt.amt) amtObj = nextAmt;
            }
            
            createRow(matchedKey, amtObj.amt, amtObj.unit);
        }
    });
}

function extractAmount(str) {
    const match = str.match(/(\d+(\.\d+)?)\s*(mg|mcg|iu|g)/);
    if(match) return { amt: match[1], unit: match[3] };
    const num = str.match(/^(\d+(\.\d+)?)$/);
    if(num) return { amt: num[1], unit: 'mg' };
    return { amt: null, unit: 'mg' };
}

function createRow(key, amt, unit) {
    rowCount++;
    const div = document.createElement('div');
    div.className = 'audit-row';
    div.id = `row-${rowCount}`;
    
    let opts = '';
    for(let k in db) opts += `<option value="${k}" ${k===key?'selected':''}>${db[k].name}</option>`;
    
    div.innerHTML = `
        <select id="k-${rowCount}">${opts}</select>
        <input type="number" id="a-${rowCount}" value="${amt||''}" placeholder="0">
        <select id="u-${rowCount}">
            <option value="mg" ${unit==='mg'?'selected':''}>mg</option>
            <option value="mcg" ${unit==='mcg'?'selected':''}>mcg</option>
        </select>
        <button onclick="this.parentElement.remove()" style="color:red; background:none; border:none; cursor:pointer;">X</button>
    `;
    document.getElementById('rows-container').appendChild(div);
}

function addEmptyRow() { createRow('zinc_sulphate', '', 'mg'); }

function generateReport() {
    const group = document.getElementById('rda-group').value;
    const tbody = document.getElementById('report-body');
    tbody.innerHTML = '';
    
    const rows = document.querySelectorAll('.audit-row');
    rows.forEach(row => {
        const id = row.id.split('-')[1];
        const key = document.getElementById(`k-${id}`).value;
        const amt = parseFloat(document.getElementById(`a-${id}`).value);
        const unit = document.getElementById(`u-${id}`).value;

        if(key && !isNaN(amt)) {
            const d = db[key];
            let elem = (unit==='mcg' ? amt/1000 : amt) * d.yield;
            
            let rdaHtml = '-';
            if(d.rda) {
                let target = d.rda[group];
                let p = (d.unit_pref === 'mcg') 
                    ? ((elem*1000)/target)*100 
                    : (elem/target)*100;
                
                let cls = p > 100 ? 'bg-red' : 'bg-green';
                let txt = p > 100 ? 'High' : 'Safe';
                rdaHtml = `<span class="badge ${cls}">${p.toFixed(0)}%</span> <small>${txt}</small>`;
            }
            
            let displayElem = d.unit_pref === 'mcg' ? (elem*1000).toFixed(1)+' mcg' : elem.toFixed(2)+' mg';

            tbody.innerHTML += `<tr>
                <td style="padding:5px;">${d.name}</td>
                <td>${displayElem}</td>
                <td>${rdaHtml}</td>
            </tr>`;
        }
    });
    document.getElementById('final-report').style.display = 'block';
}

// Tab Logic
document.querySelectorAll('.tab').forEach(t => {
    t.addEventListener('click', () => {
        document.querySelectorAll('.tab, .tab-content').forEach(c => c.classList.remove('active'));
        t.classList.add('active');
        document.getElementById(t.dataset.tab).classList.add('active');
    });
});
</script>
</body>
</html>
