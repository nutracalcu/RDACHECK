<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pro Nutrition Auditor Suite (ICMR 2020 Data)</title>

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
    
    /* ICMR ALERT STYLE */
    .icmr-alert-box {
        margin-top: 30px;
        padding: 15px;
        border: 2px solid #f39c12; 
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
                <p>This application uses **ICMR-NIN 2020** RDA data[cite: 50, 56]. Since there is no automated notification API for guideline changes, you must manually check for the most recent updates:</p>
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
Quatrefolic® 0.57 mg, Zinc Sulphate 42 mg"></textarea>
            
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
                    <option value="men">Adult Men (Sedentary ICMR 2020)</option>
                    <option value="women">Adult Women (Sedentary ICMR 2020)</option>
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
// --- FULLY INTEGRATED AUDIT DATABASE (ICMR 2020 & Salt Yields) ---
const auditDB = {
    // --- MINERALS (RDA values from Annexure IA & IB, Yields from WalPar) ---
    // Calcium
    "calcium_carbonate": { name: "Calcium Carbonate", yield: 0.40, unit_pref: "mg", rda: { men: 1000, women: 1000 } }, // 40% [cite: 71], 1000 mg [cite: 46]
    "calcium_citrate": { name: "Calcium Citrate", yield: 0.2407, unit_pref: "mg", rda: { men: 1000, women: 1000 } }, // 24.07% [cite: 71]
    "dicalcium_phosphate": { name: "Dicalcium Phosphate", yield: 0.2945, unit_pref: "mg", rda: { men: 1000, women: 1000 } }, // 29.45% [cite: 71]
    
    // Iron
    "ferrous_bisglycinate": { name: "Ferrous Bisglycinate", yield: 0.2737, unit_pref: "mg", rda: { men: 19, women: 29 } }, // 27.37% [cite: 81], Fe: 19 mg (M), 29 mg (W) [cite: 46]
    "ferrous_sulphate": { name: "Ferrous Sulphate", yield: 0.3675, unit_pref: "mg", rda: { men: 19, women: 29 } }, // 36.75% [cite: 83]
    
    // Zinc
    "zinc_sulphate": { name: "Zinc Sulphate", yield: 0.4049, unit_pref: "mg", rda: { men: 17, women: 13.2 } }, // 40.49% [cite: 95], Zn: 17 mg (M), 13.2 mg (W) [cite: 46]
    "zinc_gluconate": { name: "Zinc Gluconate", yield: 0.1434, unit_pref: "mg", rda: { men: 17, women: 13.2 } }, // 14.34% [cite: 95]
    "zinc_bisglycinate": { name: "Zinc Bisglycinate", yield: 0.3062, unit_pref: "mg", rda: { men: 17, women: 13.2 } }, // 30.62% [cite: 95]

    // Magnesium
    "magnesium_oxide": { name: "Magnesium Oxide", yield: 0.6030, unit_pref: "mg", rda: { men: 440, women: 370 } }, // 60.30% [cite: 120], Mg: 440 mg (M), 370 mg (W) [cite: 46]
    "magnesium_citrate": { name: "Magnesium Citrate", yield: 0.1133, unit_pref: "mg", rda: { men: 440, women: 370 } }, // 11.33% [cite: 120]
    
    // Selenium
    "sodium_selenite": { name: "Sodium Selenite", yield: 0.4565, unit_pref: "mcg", rda: { men: 40, women: 40 } }, // 45.65% [cite: 71], Se: 40 µg [cite: 54]
    "selenium_dioxide": { name: "Selenium Dioxide", yield: 0.7116, unit_pref: "mcg", rda: { men: 40, women: 40 } }, // 71.16% [cite: 125]
    
    // Potassium
    "potassium_chloride": { name: "Potassium Chloride", yield: 0.5243, unit_pref: "mg", rda: { men: 3500, women: 3500 } }, // 52.43% [cite: 111], K: 3500 mg [cite: 54]
    
    // Copper
    "copper_sulphate": { name: "Copper Sulphate", yield: 0.3981, unit_pref: "mg", rda: { men: 1.7, women: 1.7 } }, // 39.81% [cite: 93], Cu: 1.7 mg [cite: 54]
    
    // Chromium
    "chromium_picolinate": { name: "Chromium Picolinate", yield: 0.1242, unit_pref: "mcg", rda: { men: 50, women: 50 } }, // 12.42% [cite: 113], Cr: 50 µg [cite: 54]

    // Phosphorous
    "phosphorous_salt": { name: "Phosphorous (Salt Placeholder)", yield: 1.0, unit_pref: "mg", rda: { men: 1000, women: 1000 } }, // P: 1000 mg [cite: 54]
    
    // Sodium
    "sodium_chloride": { name: "Sodium Chloride (as Salt)", yield: 0.3932, unit_pref: "mg", rda: { men: 2000, women: 2000 } }, // 39.32% [cite: 71], Na: 2000 mg [cite: 54]

    // Iodine
    "potassium_iodide": { name: "Potassium Iodide", yield: 0.7644, unit_pref: "mcg", rda: { men: 140, women: 140 } }, // 76.44% (for Iodine) [cite: 111], I: 140 µg [cite: 46]

    // --- VITAMINS (Yields from WalPar for specific forms) ---
    // Vitamin B1 (Thiamine)
    "thiamine_mononitrate": { name: "Thiamine Mononitrate", yield: 0.8105, unit_pref: "mg", rda: { men: 1.4, women: 1.4 } }, // 81.05% [cite: 113], B1: 1.4 mg [cite: 46]
    
    // Vitamin B2 (Riboflavin)
    "riboflavin": { name: "Riboflavin", yield: 1.0, unit_pref: "mg", rda: { men: 2.0, women: 1.7 } }, // 2.0 mg (M), 1.7 mg (W) [cite: 46]
    "riboflavin_sod_phosphate": { name: "Riboflavin Sodium Phosphate", yield: 0.7668, unit_pref: "mg", rda: { men: 2.0, women: 1.7 } }, // 76.68% [cite: 107]
    
    // Vitamin B3 (Niacin)
    "niacin": { name: "Niacin", yield: 1.0, unit_pref: "mg", rda: { men: 14, women: 11 } }, // 14 mg (M), 11 mg (W) [cite: 46]
    
    // Vitamin B5 (Pantothenic Acid)
    "calcium_pantothenate": { name: "Calcium Pantothenate", yield: 0.9160, unit_pref: "mg", rda: { men: 5, women: 5 } }, // 91.60% (as Pantothenic Acid) [cite: 71], B5: 5 mg (AI) [cite: 53]

    // Vitamin B6 (Pyridoxine)
    "pyridoxine_hcl": { name: "Pyridoxine HCL", yield: 0.8226, unit_pref: "mg", rda: { men: 1.9, women: 1.9 } }, // 82.26% [cite: 125], B6: 1.9 mg [cite: 46]
    "pyridoxal_5_phosphate": { name: "Pyridoxal-5-Phosphate", yield: 0.6845, unit_pref: "mg", rda: { men: 1.9, women: 1.9 } }, // 68.45% [cite: 125]
    
    // Vitamin B9 (Folate)
    "l_methyl_folate_calcium": { name: "L-Methyl Folate Calcium", yield: 0.8872, unit_pref: "mcg", rda: { men: 300, women: 220 } }, // 88.72% (as Folic Acid) [cite: 125], Folate: 300 µg (M), 220 µg (W) [cite: 46]
    "quatrefolic": { name: "Quatrefolic® (L-5-MTHF)", yield: 0.96, unit_pref: "mcg", rda: { men: 300, women: 220 } }, // 96% (as Folic Acid) [cite: 127]
    
    // Vitamin B12
    "vitamin_b12": { name: "Vitamin B12", yield: 1.0, unit_pref: "mcg", rda: { men: 2.2, women: 2.2 } }, // B12: 2.2 µg [cite: 46]
    
    // Vitamin C
    "ascorbic_acid": { name: "Ascorbic Acid", yield: 1.0, unit_pref: "mg", rda: { men: 80, women: 65 } }, // 80 mg (M), 65 mg (W) [cite: 46]
    "calcium_ascorbate": { name: "Calcium L-Ascorbate", yield: 0.826, unit_pref: "mg", rda: { men: 80, women: 65 } }, // 82.6% (as Ascorbic Acid) [cite: 71]
    
    // Vitamin A
    "vitamin_a": { name: "Vitamin A", yield: 1.0, unit_pref: "mcg", rda: { men: 1000, women: 840 } }, // Vit A: 1000 µg (M), 840 µg (W) [cite: 46]

    // Vitamin D
    "vitamin_d": { name: "Vitamin D", yield: 1.0, unit_pref: "IU", rda: { men: 600, women: 600 } }, // Vit D: 600 IU [cite: 46]

    // Vitamin E
    "vitamin_e_acetate": { name: "Vitamin E Acetate", yield: 0.9110, unit_pref: "mg", rda: { men: 7.5, women: 7.5 } }, // 91.10% (as Vit E) [cite: 125], Vit E: 7.5-10 mg (using 7.5mg/d) [cite: 53]

    // --- OTHER NUTRIENTS ---
    "biotin": { name: "Biotin", yield: 1.0, unit_pref: "mcg", rda: { men: 40, women: 40 } }, // Biotin: 40 µg (AI) [cite: 53]
    "vitamin_k": { name: "Vitamin K", yield: 1.0, unit_pref: "mcg", rda: { men: 55, women: 55 } }, // Vit K: 55 µg (AI) [cite: 53]
    "manganese_sulphate": { name: "Manganese Sulphate", yield: 0.3637, unit_pref: "mg", rda: { men: 4, women: 4 } }, // 36.37% [cite: 122], Mn: 4 mg [cite: 54]
    "molybdenum_pentoxide": { name: "Molybdenum Pentoxide", yield: 0.6665, unit_pref: "mcg", rda: { men: 45, women: 45 } }, // 66.65% [cite: 118], Mo: 45 µg [cite: 54]
    "molybdenum_salt": { name: "Molybdenum (Salt Placeholder)", yield: 1.0, unit_pref: "mcg", rda: { men: 45, women: 45 } },

    // Amino Acids/Non-Essential (RDA is N/A)
    "n_acetyl_cysteine": { name: "N-Acetyl Cysteine", yield: 0.7424, rda: null }, // 74.24% (as Cysteine) [cite: 127]
    "l_cysteine_hcl": { name: "L-Cysteine Hydrochloride", yield: 0.7686, rda: null }, // 76.86% (as Cysteine) [cite: 127]
    "l_lysine_hcl": { name: "L-Lysine HCL", yield: 0.80, rda: null } // 80% (as Lysine) [cite: 98]
};

const keyMap = [
    // Minerals
    { key: "zinc_sulphate", keywords: ["zinc", "sulphate", "sulfate"] },
    { key: "ferrous_bisglycinate", keywords: ["ferrous", "bisglycinate", "iron"] },
    { key: "calcium_carbonate", keywords: ["calcium", "carbonate"] },
    { key: "magnesium_oxide", keywords: ["magnesium", "oxide"] },
    { key: "sodium_selenite", keywords: ["sodium", "selenite", "selenium"] },
    { key: "potassium_chloride", keywords: ["potassium", "chloride"] },
    { key: "copper_sulphate", keywords: ["copper", "sulphate", "cupric"] },
    { key: "chromium_picolinate", keywords: ["chromium", "picolinate"] },
    { key: "zinc_bisglycinate", keywords: ["zinc", "bisglycinate"] },
    { key: "potassium_iodide", keywords: ["iodide", "iodine"] },

    // Vitamins (Specific Forms)
    { key: "pyridoxine_hcl", keywords: ["b6", "pyridoxine", "hcl"] },
    { key: "riboflavin", keywords: ["b2", "riboflavin"] },
    { key: "thiamine_mononitrate", keywords: ["b1", "thiamine", "mononitrate"] },
    { key: "calcium_pantothenate", keywords: ["b5", "pantothenate"] },
    { key: "ascorbic_acid", keywords: ["c", "ascorbic"] },
    { key: "calcium_ascorbate", keywords: ["calcium", "ascorbate"] },
    { key: "quatrefolic", keywords: ["folic", "folate", "quatrefolic", "b9"] },
    { key: "vitamin_b12", keywords: ["b12", "cyanocobalamin", "cobalamin"] },
    { key: "vitamin_a", keywords: ["vitamin", "a"] },
    { key: "vitamin_d", keywords: ["vitamin", "d"] },
    { key: "vitamin_e_acetate", keywords: ["vitamin", "e", "acetate", "tocopheryl"] },
    { key: "niacin", keywords: ["b3", "niacin", "nicotinamide"] },

    // Other Compounds
    { key: "n_acetyl_cysteine", keywords: ["acetyl", "cysteine", "nac"] },
    { key: "l_lysine_hcl", keywords: ["l-lysine", "lysine", "hcl"] }
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
    // Use mg, mcg, and IU as units for input, even though RDA comparison uses mg/mcg
    unitOpts += `<option value="mg" ${unit==='mg'?'selected':''}>mg</option>`;
    unitOpts += `<option value="mcg" ${unit==='mcg'?'selected':''}>mcg</option>`;
    unitOpts += `<option value="IU" ${unit==='IU'?'selected':''}>IU</option>`;

    div.innerHTML = `
        <select id="k-${rowCount}">${opts}</select>
        <input type="number" id="v-${rowCount}" value="${val}" placeholder="0">
        <select id="u-${rowCount}">${unitOpts}</select>
        <button onclick="this.parentElement.remove()" style="color:red; background:none; border:none; cursor:pointer;">X</button>
    `;
    document.getElementById('rows-container').appendChild(div);
}

function addEmptyRow() { createRow('zinc_sulphate', '', 'mg'); }

function convertToStandardUnit(val, unit, key) {
    let mg = 0;
    let mcg = 0;
    
    if (unit === 'IU') {
        const name = auditDB[key].name;
        // Apply ICMR 2020 conversion factors [cite: 63, 64, 65]
        if (name.includes('Vitamin D')) {
            mcg = val * 0.025; // 1 IU = 0.025 µg [cite: 64]
        } else if (name.includes('Vitamin E')) {
            // Assuming common dl-alpha-tocopherol (1.1 IU/mg)
            mg = val / 1.1; // 1 IU dl-alpha-tocopherol is approx. 0.909 mg dl-alpha-tocopherol [cite: 65]
        } else if (name.includes('Vitamin A')) {
            mcg = val / 3.33; // 1 µg = 3.33 IU [cite: 63]
        }
    } else if (unit === 'mg') {
        mg = val;
    } else if (unit === 'mcg') {
        mcg = val;
    }

    // Convert everything to the preferred unit of the final report (mg/mcg)
    const prefUnit = auditDB[key].unit_pref;
    
    if (prefUnit === 'mcg') {
        return (mg * 1000) + mcg; // Output in mcg
    } else {
        return (mcg / 1000) + mg; // Output in mg
    }
}

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
        
        // Step 1: Convert scanned value to base (mg or mcg) using IU conversion if necessary
        const base_value = convertToStandardUnit(val, unit, key);
        
        // Step 2: Apply elemental yield
        // total_elemental_value is in the database's unit_pref (mg or mcg)
        let total_elemental_value = base_value * d.yield;
        
        let status = '-';
        if(d.rda) {
            let target = d.rda[group];
            
            // Percentage of RDA
            let p = (total_elemental_value/target)*100;
            
            let cls = p > 100 ? 'bg-red' : 'bg-green';
            let txt = p > 100 ? 'High' : 'Safe/Suf.';
            status = `<span class="badge ${cls}">${p.toFixed(0)}%</span> <small>${txt}</small>`;
        }
        
        // Display yield based on preferred unit
        let yield_display = (d.unit_pref === 'mcg' ? total_elemental_value.toFixed(1) + ' µg' : total_elemental_value.toFixed(2) + ' mg') || total_elemental_value.toFixed(2) + ' units';

        tbody.innerHTML += `<tr>
            <td>${d.name}</td>
            <td>${val} ${unit}</td>
            <td>${yield_display}</td>
            <td>${status}</td>
        </tr>`;
    }
    document.getElementById('audit-report').style.display = 'block';
}
</script>
</body>
</html>
