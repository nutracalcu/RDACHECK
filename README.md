<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pro Nutrition Formulator & Auditor</title>

<script src='https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js'></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.css"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<style>
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
    .tabs { display: flex; margin-bottom: 20px; border-bottom: 3px solid #ddd; }
    .tab {
        padding: 12px 30px; cursor: pointer; background: #e0e0e0; margin-right: 5px; 
        border-radius: 8px 8px 0 0; font-weight: bold; transition: 0.3s;
    }
    .tab:hover { background: #d0d0d0; }
    .tab.active { background: var(--primary); color: white; border-color: var(--primary); }
    .tab[data-tab="audit"].active { background: var(--audit-color); }

    .tab-content { display: none; background: var(--panel); padding: 25px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }
    .tab-content.active { display: block; }

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
</style>
</head>
<body>

<header>
    <h1>Pro Nutrition Suite</h1>
    <p>Formulator (Calculators) & Auditor (AI Scanner)</p>
</header>

<div class="tabs">
    <div class="tab active" data-tab="minerals">Minerals</div>
    <div class="tab" data-tab="vitamins">Vitamins</div>
    <div class="tab" data-tab="audit">AI Label Auditor</div>
</div>

<div id="minerals" class="tab-content active">
    <div class="grid-2">
        <div>
            <h3>Input Formulation</h3>
            <div class="input-box">
                <label>1. Select Mineral:</label>
                <select id="min-select">
                    <option value="">-- Select --</option>
                    <option value="calcium">Calcium</option>
                    <option value="iron">Iron</option>
                    <option value="magnesium">Magnesium</option>
                    <option value="zinc">Zinc</option>
                    <option value="selenium">Selenium</option>
                </select>
                
                <label>2. Select Salt Form:</label>
                <select id="min-form"><option>-- Select Mineral First --</option></select>
                <div id="min-yield" style="font-size:0.9em; color:#666; margin-bottom:10px;"></div>
                
                <label>3. Target Elemental Amount:</label>
                <div style="display:flex; gap:10px;">
                    <input type="number" id="min-target" placeholder="e.g. 50">
                    <select id="min-unit" style="width:80px;"><option value="mg">mg</option><option value="mcg">mcg</option></select>
                </div>

                <label>4. Purity & Batch:</label>
                <div style="display:flex; gap:10px;">
                    <input type="number" id="min-purity" value="100" placeholder="Purity %">
                    <input type="number" id="min-batch" value="1" placeholder="Batch Size">
                </div>
                
                <button class="btn btn-calc" id="btn-calc-min">Calculate Salt Dose</button>
            </div>
        </div>
        <div>
            <h3>Results</h3>
            <div class="result-box">
                <span>Required Raw Material (Per Dose):</span>
                <span id="min-result-single" class="big-res">0 mg</span>
            </div>
            <div class="input-box" style="margin-top:20px; background:#e8f4fc; border-color:#b3d7ff;">
                <strong>Total Batch Requirement:</strong>
                <div id="min-result-batch" style="font-size:1.5em; color:var(--primary); font-weight:bold;">0 g</div>
            </div>
        </div>
    </div>
</div>

<div id="vitamins" class="tab-content">
    <div class="grid-2">
        <div>
            <h3>Input Formulation</h3>
            <div class="input-box">
                <label>1. Select Vitamin:</label>
                <select id="vit-select">
                    <option value="">-- Select --</option>
                    <option value="c">Vitamin C</option>
                    <option value="d">Vitamin D3</option>
                    <option value="b12">Vitamin B12</option>
                    <option value="e">Vitamin E</option>
                </select>
                
                <label>2. Select Form:</label>
                <select id="vit-form"><option>-- Select Vitamin First --</option></select>
                
                <label>3. Target Amount:</label>
                <div style="display:flex; gap:10px;">
                    <input type="number" id="vit-target" placeholder="Amount">
                    <select id="vit-unit" style="width:100px;">
                        <option value="mg">mg</option>
                        <option value="mcg">mcg</option>
                        <option value="IU">IU</option>
                    </select>
                </div>

                <label>4. Purity (%):</label>
                <input type="number" id="vit-purity" value="100">
                
                <button class="btn btn-calc" id="btn-calc-vit">Calculate Vitamin Dose</button>
            </div>
        </div>
        <div>
            <h3>Results</h3>
            <div class="result-box">
                <span>Required Raw Material:</span>
                <span id="vit-result-single" class="big-res">0 mg</span>
            </div>
        </div>
    </div>
</div>

<div id="audit" class="tab-content">
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
                    <thead style="background:#f3e5f5;"><tr><th>Ingredient</th><th>Scan</th><th>Yield</th><th>Status</th></tr></thead>
                    <tbody id="report-body"></tbody>
                </table>
            </div>
        </div>
    </div>
</div>

<script>
// --- GLOBAL DATA ---
const mineralData = {
    calcium: [{id:'carbonate', name:'Carbonate', p:0.4}, {id:'citrate', name:'Citrate', p:0.241}],
    iron: [{id:'bisglycinate', name:'Bisglycinate', p:0.2}, {id:'sulphate', name:'Sulphate', p:0.367}],
    zinc: [{id:'sulphate', name:'Sulphate', p:0.42}, {id:'gluconate', name:'Gluconate', p:0.143}],
    magnesium: [{id:'oxide', name:'Oxide', p:0.603}, {id:'citrate', name:'Citrate', p:0.113}],
    selenium: [{id:'selenite', name:'Sodium Selenite', p:0.45}]
};

const vitaminData = {
    c: { forms: [{id:'ascorbic', name:'Ascorbic Acid', p:1.0}] },
    d: { forms: [{id:'cholecalciferol', name:'Cholecalciferol', iu_to_mcg:0.025}] },
    b12: { forms: [{id:'cyano', name:'Cyanocobalamin', p:1.0}] },
    e: { forms: [{id:'acetate', name:'Tocopheryl Acetate', mg_to_iu:1.36}] }
};

// --- AUDIT DATABASE ---
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

// --- 1. MINERAL LOGIC ---
const mSel = document.getElementById('min-select');
const mForm = document.getElementById('min-form');
mSel.addEventListener('change', () => {
    mForm.innerHTML = '<option>-- Select Form --</option>';
    if(mineralData[mSel.value]) mineralData[mSel.value].forEach(x => mForm.add(new Option(x.name, x.id)));
});
mForm.addEventListener('change', () => {
    const s = mineralData[mSel.value].find(x => x.id === mForm.value);
    if(s) document.getElementById('min-yield').innerText = `Yield: ${(s.p*100).toFixed(1)}%`;
});
document.getElementById('btn-calc-min').addEventListener('click', () => {
    const s = mineralData[mSel.value].find(x => x.id === mForm.value);
    const t = parseFloat(document.getElementById('min-target').value);
    const u = document.getElementById('min-unit').value;
    const p = parseFloat(document.getElementById('min-purity').value) || 100;
    const b = parseFloat(document.getElementById('min-batch').value) || 1;
    
    if(!s || isNaN(t)) return;
    let tMg = u==='mcg' ? t/1000 : t;
    let dose = (tMg / s.p) / (p/100);
    
    document.getElementById('min-result-single').innerText = dose.toFixed(3) + " mg";
    let batch = dose * b;
    document.getElementById('min-result-batch').innerText = batch > 1000 ? (batch/1000).toFixed(3)+" g" : batch.toFixed(1)+" mg";
});

// --- 2. VITAMIN LOGIC ---
const vSel = document.getElementById('vit-select');
const vForm = document.getElementById('vit-form');
vSel.addEventListener('change', () => {
    vForm.innerHTML = '<option>-- Select Form --</option>';
    if(vitaminData[vSel.value]) vitaminData[vSel.value].forms.forEach(x => vForm.add(new Option(x.name, x.id)));
});
document.getElementById('btn-calc-vit').addEventListener('click', () => {
    const f = vitaminData[vSel.value].forms.find(x => x.id === vForm.value);
    const t = parseFloat(document.getElementById('vit-target').value);
    const u = document.getElementById('vit-unit').value;
    const p = parseFloat(document.getElementById('vit-purity').value) || 100;
    
    if(!f || isNaN(t)) return;
    let tMg = 0;
    if(u==='mg') tMg = t;
    else if(u==='mcg') tMg = t/1000;
    else if(u==='IU') {
        if(f.iu_to_mcg) tMg = (t*f.iu_to_mcg)/1000;
        else if(f.mg_to_iu) tMg = t/f.mg_to_iu;
    }
    
    let dose = f.p ? (tMg/f.p)/(p/100) : tMg/(p/100);
    document.getElementById('vit-result-single').innerText = dose.toFixed(3) + " mg";
});

// --- 3. AUDIT LOGIC ---
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
        json.forEach(row => { if(row[0] && row[1]) txt += `${row[0]}\n${row[1]}\n`; });
        document.getElementById('text-editor').value = txt;
    };
    reader.readAsArrayBuffer(file);
}

function processTextToRows() {
    // UPDATED: Pre-process commas to newlines to handle "Name 5mg, Name 10mg" format
    let rawText = document.getElementById('text-editor').value;
    // Replace commas that are likely separators (followed by space or new word)
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
            // Find amount in this line
            let a = extractAmount(line);
            // If not found, check next line (vertical list style)
            if(!a.val && lines[i+1]) a = extractAmount(lines[i+1].toLowerCase());
            
            createRow(key, a.val, a.unit);
        }
    });
}

function extractAmount(s) {
    // Regex matches decimal numbers (e.g. 0.0025, 42.5, 100) followed by unit
    const m = s.match(/(\d+(\.\d+)?)\s*(mg|mcg|iu)/);
    if(m) return {val: m[1], unit: m[3]};
    // Fallback: just a number
    const n = s.match(/(\d+(\.\d+)?)/);
    if(n) return {val: n[1], unit: 'mg'}; // Default mg if no unit found
    return {val: '', unit: 'mg'};
}

function createRow(key, val, unit) {
    rowCount++;
    const div = document.createElement('div');
    div.style = "display:grid; grid-template-columns:2fr 1fr 1fr 0.5fr; gap:5px; margin-bottom:5px; border-bottom:1px solid #eee; padding:5px;";
    div.id = `r-${rowCount}`;
    
    let opts = '';
    for(let k in auditDB) opts += `<option value="${k}" ${k===key?'selected':''}>${auditDB[k].name}</option>`;
    
    div.innerHTML = `
        <select id="k-${rowCount}">${opts}</select>
        <input type="number" id="v-${rowCount}" value="${val}" placeholder="0">
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
    
    const rows = document.getElementById('rows-container').children;
    for(let r of rows) {
        if(r.tagName === 'P') continue;
        const id = r.id.split('-')[1];
        const key = document.getElementById(`k-${id}`).value;
        const val = parseFloat(document.getElementById(`v-${id}`).value);
        const unit = document.getElementById(`u-${id}`).value;
        
        if(!key || isNaN(val)) continue;
        
        const d = auditDB[key];
        let elem = (unit==='mcg' ? val/1000 : val) * d.yield;
        
        let status = '-';
        if(d.rda) {
            let target = d.rda[group];
            let p = (d.unit_pref === 'mcg') ? ((elem*1000)/target)*100 : (elem/target)*100;
            // STRICT LOGIC: > 100% is RED (High), <= 100% is GREEN (Safe)
            let cls = p > 100 ? 'bg-red' : 'bg-green';
            let txt = p > 100 ? 'High' : 'Safe';
            status = `<span class="badge ${cls}">${p.toFixed(0)}%</span> <small>${txt}</small>`;
        }
        
        tbody.innerHTML += `<tr>
            <td>${d.name}</td>
            <td>${val} ${unit}</td>
            <td>${d.unit_pref === 'mcg' ? (elem*1000).toFixed(1)+' mcg' : elem.toFixed(2)+' mg'}</td>
            <td>${status}</td>
        </tr>`;
    }
    document.getElementById('audit-report').style.display = 'block';
}

// TAB LOGIC
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
