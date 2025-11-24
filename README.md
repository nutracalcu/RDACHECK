<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pro Nutrition Auditor Suite</title>

<script src='https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js'></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.css"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<style>
    /* ... (KEEP THE ORIGINAL CSS STYLES HERE) ... */
    :root {
        --primary: #3498db;
        --audit-color: #8e44ad;
        --secondary: #2c3e50;
        --success: #27ae60;
        --danger: #c0392b;
        --bg: #f4f7f6;
        --panel: #ffffff;
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

    /* HEADER */
    header {
        text-align: center;
        margin-bottom: 25px;
        padding: 20px;
        background-color: var(--secondary);
        color: white;
        border-radius: 8px;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }

    /* TABS */
    /* Only one tab, so simplifying the display */
    .tabs { display: flex; margin-bottom: 20px; border-bottom: 3px solid var(--audit-color); }
    .tab {
        padding: 12px 30px; cursor: default; background: var(--audit-color); margin-right: 5px; 
        border-radius: 8px 8px 0 0; font-weight: bold; transition: 0.3s; color: white;
    }
    .tab-content { background: var(--panel); padding: 25px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); display: block; }

    /* LAYOUT GRIDS */
    .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 30px; }
    .audit-grid { display: grid; grid-template-columns: 40% 60%; gap: 20px; }

    /* INPUT GROUPS */
    .input-box { background: #f8f9fa; padding: 15px; border-radius: 6px; border: 1px solid #eee; margin-bottom: 15px; }
    label { display: block; margin-bottom: 5px; font-weight: 600; color: #555; }
    select, input { padding: 10px; border: 1px solid #ddd; border-radius: 4px; width: 100%; box-sizing: border-box; margin-bottom: 10px; }
    
    /* BUTTONS */
    .btn { width: 100%; padding: 12px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; color: white; margin-top: 5px; font-size: 16px; }
    .btn-calc { background: var(--primary); }
    .btn-audit { background: var(--audit-color); }
    .btn-crop { background: #2c3e50; }
    
    /* CROPPER STYLES */
    .img-container {
        height: 350px; background: #333; margin-bottom: 15px; border-radius: 4px; overflow: hidden; display: flex; align-items: center; justify-content: center;
    }
    #image-to-crop { max-width: 100%; display: block; }
    
    /* TEXT EDITOR */
    .text-editor {
        width: 100%; height: 120px; padding: 10px; border: 2px dashed var(--audit-color);
        border-radius: 6px; background: #fdfbff; font-family: monospace; resize: vertical; box-sizing: border-box;
    }

    /* RESULTS & TABLES */
    .result-box { background: var(--secondary); color: white; padding: 20px; text-align: center; border-radius: 8px; margin-top: 20px; }
    .big-res { font-size: 2em; font-weight: bold; display: block; }
    
    table { width: 100%; border-collapse: collapse; margin-top: 15px; }
    th { background: #eee; padding: 10px; text-align: left; font-size: 0.9em; color: #555; }
    td { padding: 10px; border-bottom: 1px solid #eee; }
    
    .badge { padding: 4px 8px; border-radius: 4px; color: white; font-size: 0.8em; font-weight: bold; }
    .bg-green { background: var(--success); }
    .bg-red { background: var(--danger); }
    
    .progress-bar { height: 5px; background: #eee; margin-top: 5px; display: none; }
    .progress-fill { height: 100%; background: var(--audit-color); width: 0%; transition: width 0.2s; }
    
    /* NEW ICMR ALERT STYLE */
    .icmr-alert-box {
        margin-top: 30px;
        padding: 15px;
        border: 2px solid #f39c12; /* Orange for alert */
        background: #fffbe6;
        border-radius: 6px;
        font-size: 0.9em;
    }
    .icmr-alert-box strong { color: #f39c12; }
</style>
</head>
<body>

<header>
    <h1>Pro Nutrition Auditor</h1>
    <p>AI Label Scanning and ICMR RDA Compliance Check</p>
</header>

<div class="tabs">
    <div class="tab">AI Label Auditor</div>
</div>

<div id="unified-content" class="tab-content active">
    
    <h2 style="margin-top: 0; color: var(--audit-color);">🔬 AI Label Auditor (ICMR RDA Check)</h2>
    
    <div class="audit-grid">
        <div>
            <div class="input-box">
                <h4 style="margin-top:0;">Option A: Upload Excel</h4>
                <input type="file" id="excel-upload" accept=".xlsx, .xls" onchange="handleExcel(event)">
            </div>
            
            <div class="input-box">
                <h4 style="margin-top:0;">Option B: Image Scan</h4>
                <input type="file" id="img-upload" accept="image/*" onchange="setupCropper(event)">
                
                <div class="img-container">
                    <img id="image-to-crop" src="" alt="Upload Image">
                </div>
                
                <button class="btn btn-crop" id="btn-crop-scan" onclick="scanCroppedImage()" style="display:none;">✂️ Crop & Scan Selection</button>
                <div class="progress-bar" id="p-bar"><div class="progress-fill" id="p-fill"></div></div>
                <div id="scan-status" style="text-align:center; font-size:0.8em; margin-top:5px; color:#666;"></div>
            </div>
            
            <div class="icmr-alert-box">
                <h4>⚠️ ICMR Guideline Alert System</h4>
                <p>This application uses **ICMR-NIN 2020** RDA data. Since there is no automated notification API for guideline changes, you must manually check for the most recent updates:</p>
                <ul>
                    <li>**Official Source:** Regularly check the **ICMR-National Institute of Nutrition (NIN)** website. New "Nutrient Requirements for Indians" reports are usually announced there.</li>
                    <li>**Search:** Use search terms like "ICMR Nutrient Requirements [current year]" to find the latest published reports.</li>
                    <li>**Action:** If a new report is released, **you are responsible** for updating the RDA values in the `auditDB` within the application's source code.</li>
                </ul>
            </div>
            </div>

        <div>
            <h3>Text Editor & Report</h3>
            <p style="font-size:0.9em; margin-bottom:5px;">Paste your text list (separated by lines or commas):</p>
            <textarea id="text-editor" class="text-editor" placeholder="Example:
Quatrefolic® 0.57 mg, Vitamin B6 2.3 mg, Zinc Sulphate 42 mg"></textarea>
            
            <button class="btn btn-audit" onclick="processTextToRows()">⬇️ Process Ingredients</button>

            <div style="margin-top:20px; border-top:1px solid #eee; padding-top:10px;">
                <div style="display:flex; justify-content:space-between; align-items:center;">
                    <strong>Detected Rows</strong>
                    <button onclick="addEmptyRow()" style="width:auto; padding:5px; background:#ddd; color:#333; border:none; border-radius:3px;">+ Add</button>
                </div>
                
                <div id="rows-container" style="max-height:250px; overflow-y:auto; border:1px solid #eee; margin-top:10px; padding:5px;">
                    <p style="text-align:center; color:#999; padding:20px;">Rows will appear here...</p>
                </div>
            </div>

            <div style="margin-top:15px;">
                <label>RDA Standard:</label>
                <select id="rda-group">
                    <option value="men">Adult Men (ICMR 2020)</option>
                    <option value="women">Adult Women (ICMR 2020)</option>
                </select>
                <button class="btn btn-audit" onclick="generateReport()">Generate Final Report</button>
            </div>

            <div id="audit-report" style="display:none; margin-top:15px;">
                <table style="font-size:0.9em;">
                    <thead style="background:#f3e5f5;"><tr><th>Ingredient</th><th>Scan</th><th>Yield</th><th>Status (vs RDA)</th></tr></thead>
                    <tbody id="report-body"></tbody>
                </table>
            </div>
        </div>
    </div>
</div>

<script>
// NOTE: MineralData and VitaminData are removed as their associated calculators were removed.

// --- AUDIT DATABASE (BASED ON PARTIAL ICMR 2020 DATA) ---
const auditDB = {
    "zinc_sulphate": { name: "Zinc Sulphate", yield: 0.42, rda: { men: 17, women: 13.2 } },
    "ferrous_bisglycinate": { name: "Ferrous Bisglycinate", yield: 0.20, rda: { men: 19, women: 29 } },
    "nac": { name: "N-Acetyl L-Cysteine", yield: 1.0, rda: null },
    "b3": { name: "Vitamin B3", yield: 1.0, rda: { men: 18, women: 14 } },
    "b5": { name: "Vitamin B5", yield: 0.92, rda: { men: 5, women: 5 } },
    "b6": { name: "Vitamin B6", yield: 0.82, rda: { men: 1.9, women: 1.9 } }, // Pyridoxine HCl Yield
    "b9": { name: "Folic Acid (B9)", yield: 1.0, unit_pref:"mcg", rda: { men: 100, women: 100 } },
    "b12": { name: "Vitamin B12", yield: 1.0, unit_pref:"mcg", rda: { men: 2.2, women: 2.2 } },
    "sodium_selenite": { name: "Sodium Selenite", yield: 0.45, unit_pref: "mcg", rda: { men: 40, women: 40 } },
    "quercetin": { name: "Quercetin", yield: 1.0, rda: null },
    "grape_seed": { name: "Grape Seed Ext", yield: 1.0, rda: null },
    "green_tea": { name: "Green Tea Ext", yield: 1.0, rda: null }
};

const keyMap = [
    { key: "zinc_sulphate", keywords: ["zinc", "sulphate"] },
    { key: "ferrous_bisglycinate", keywords: ["ferrous", "bisglycinate"] },
    { key: "nac", keywords: ["acetyl", "cysteine"] },
    { key: "b3", keywords: ["b3", "nicotinamide"] },
    { key: "b5", keywords: ["b5", "pantothenate"] },
    { key: "b6", keywords: ["b6", "pyridoxine"] },
    { key: "b9", keywords: ["folic", "folate", "quatrefolic", "b9"] },
    { key: "b12", keywords: ["b12", "cyanocobalamin", "cobalamin"] },
    { key: "quercetin", keywords: ["quercetin"] },
    { key: "grape_seed", keywords: ["grape", "seed"] },
    { key: "green_tea", keywords: ["green", "tea"] },
    { key: "sodium_selenite", keywords: ["selenite", "selenium"] }
];

// --- AUDIT LOGIC FUNCTIONS ---
let cropper;
let rowCount = 0;

function setupCropper(e) {
    const file = e.target.files[0];
    if(file) {
        const reader = new FileReader();
        reader.onload = (evt) => {
            const img = document.getElementById('image-to-crop');
            img.src = evt.target.result;
            if(cropper) cropper.destroy();
            cropper = new Cropper(img, { viewMode:1, autoCropArea:0.8 });
            document.getElementById('btn-crop-scan').style.display = 'block';
        };
        reader.readAsDataURL(file);
    }
}

function scanCroppedImage() {
    if(!cropper) return;
    const canvas = cropper.getCroppedCanvas();
    const url = canvas.toDataURL('image/png');
    
    document.getElementById('p-bar').style.display = 'block';
    document.getElementById('scan-status').innerText = "AI Scanning...";
    
    Tesseract.recognize(url, 'eng', {
        logger: m => { if(m.status === 'recognizing text') document.getElementById('p-fill').style.width = (m.progress*100)+'%'; }
    }).then(({ data: { text } }) => {
        document.getElementById('text-editor').value = text;
        document.getElementById('p-bar').style.display = 'none';
        document.getElementById('scan-status').innerText = "Scan Complete! Check editor.";
    });
}

function handleExcel(e) {
    const file = e.target.files[0];
    const reader = new FileReader();
    reader.onload = (evt) => {
        const data = new Uint8Array(evt.target.result);
        const wb = XLSX.read(data, {type:'array'});
        const ws = wb.Sheets[wb.SheetNames[0]];
        const json = XLSX.utils.sheet_to_json(ws, {header:1});
        let txt = "";
        json.forEach(row => { 
            if(row[0] && row[1]) {
                txt += `${row[0]} ${row[1]}\n`; 
            } else if (row[0]) {
                txt += `${row[0]}\n`;
            }
        });
        document.getElementById('text-editor').value = txt;
    };
    reader.readAsArrayBuffer(file);
}

function processTextToRows() {
    let rawText = document.getElementById('text-editor').value;
    rawText = rawText.replace(/,\s*/g, '\n');
    
    const lines = rawText.split('\n');
    const box = document.getElementById('rows-container');
    box.innerHTML = '';
    
    lines.forEach((line, i) => {
        line = line.toLowerCase().trim();
        if(!line) return;
        
        let key = null;
        for(let km of keyMap) {
            if(km.keywords.some(k => line.includes(k))) { key = km.key; break; }
        }
        
        if(key) {
            let a = extractAmount(line);
            if(!a.val && lines[i+1]) a = extractAmount(lines[i+1].toLowerCase());
            
            createRow(key, a.val, a.unit);
        }
    });
    if (box.children.length === 0) {
        box.innerHTML = '<p style="text-align:center; color:#999; padding:20px;">Rows will appear here...</p>';
    }
}

function extractAmount(s) {
    // Regex matches decimal numbers followed by unit (mg, mcg, iu)
    const m = s.match(/(\d+(\.\d+)?)\s*(mg|mcg|iu)/);
    if(m) return {val: m[1], unit: m[3]};
    // Fallback: just a number, default to mg
    const n = s.match(/(\d+(\.\d+)?)/); 
    if(n) return {val: n[1], unit: 'mg'}; 
    return {val: '', unit: 'mg'};
}

function createRow(key, val, unit) {
    rowCount++;
    const div = document.createElement('div');
    div.style = "display:grid; grid-template-columns:2fr 1fr 1fr 0.5fr; gap:5px; margin-bottom:5px; border-bottom:1px solid #eee; padding:5px;";
    div.id = `r-${rowCount}`;
    
    let opts = '';
    for(let k in auditDB) opts += `<option value="${k}" ${k===key?'selected':''}>${auditDB[k].name}</option>`;
    
    let unitOpts = '';
    unitOpts += `<option value="mg" ${unit==='mg'?'selected':''}>mg</option>`;
    unitOpts += `<option value="mcg" ${unit==='mcg'?'selected':''}>mcg</option>`;
    // Note: IU is removed from Audit unit selector to simplify calculation logic to final mg/mcg for RDA comparison.

    div.innerHTML = `
        <select id="k-${rowCount}">${opts}</select>
        <input type="number" id="v-${rowCount}" value="${val}" placeholder="0">
        <select id="u-${rowCount}">${unitOpts}</select>
        <button onclick="this.parentElement.remove()" style="color:red; background:none; border:none; cursor:pointer;">X</button>
    `;
    document.getElementById('rows-container').appendChild(div);
}

function addEmptyRow() { createRow('zinc_sulphate', '', 'mg'); }

function generateReport() {
    const group = document.getElementById('rda-group').value;
    const tbody = document.getElementById('report-body');
    tbody.innerHTML = '';
    
    const rows = document.getElementById('rows-container').children;
    for(let r of rows) {
        if(r.tagName === 'P') continue;
        const id = r.id.split('-')[1];
        const key = document.getElementById(`k-${id}`).value;
        const val = parseFloat(document.getElementById(`v-${id}`).value);
        const unit = document.getElementById(`u-${id}`).value;
        
        if(!key || isNaN(val) || val <= 0) continue; 
        
        const d = auditDB[key];
        // Calculate the elemental/active amount in mg
        let total_mg = (unit==='mcg' ? val/1000 : val) * d.yield;
        
        let status = '-';
        if(d.rda) {
            let target = d.rda[group];
            let actual_value = (d.unit_pref === 'mcg') ? (total_mg*1000) : total_mg;
            
            // Percentage of RDA
            let p = (actual_value/target)*100;
            
            let cls = p > 100 ? 'bg-red' : 'bg-green';
            let txt = p > 100 ? 'High' : 'Safe/Suf.';
            status = `<span class="badge ${cls}">${p.toFixed(0)}%</span> <small>${txt}</small>`;
        }
        
        // Display yield based on preferred unit
        let yield_display = d.unit_pref === 'mcg' ? (total_mg*1000).toFixed(1)+' mcg' : total_mg.toFixed(2)+' mg';

        tbody.innerHTML += `<tr>
            <td>${d.name}</td>
            <td>${val} ${unit}</td>
            <td>${yield_display}</td>
            <td>${status}</td>
        </tr>`;
    }
    document.getElementById('audit-report').style.display = 'block';
}

// TAB LOGIC (Removed tab switching, kept only the combined content)
</script>
</body>
</html>
