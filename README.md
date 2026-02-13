<!DOCTYPE html>
<html lang="ro">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculator Lucru Flexibil</title>
    <style>
        body { font-family: sans-serif; padding: 10px; background: #0f172a; color: white; }
        .card { background: #1e293b; padding: 15px; border-radius: 12px; max-width: 500px; margin: auto; box-shadow: 0 4px 20px rgba(0,0,0,0.5); }
        h2 { text-align: center; color: #22d3ee; margin-bottom: 15px; }
        .section { background: #334155; padding: 12px; border-radius: 8px; margin-bottom: 12px; }
        .row { display: flex; gap: 5px; margin-bottom: 8px; flex-wrap: wrap; border-bottom: 1px solid #475569; padding-bottom: 8px; }
        label { display: block; font-size: 12px; color: #94a3b8; margin-bottom: 4px; }
        input, select { background: #1e293b; color: white; border: 1px solid #475569; padding: 8px; border-radius: 4px; font-size: 14px; width: 100%; box-sizing: border-box; }
        .btn-add { background: #10b981; color: white; border: none; padding: 10px; border-radius: 6px; width: 100%; cursor: pointer; font-weight: bold; margin-bottom: 10px; }
        .btn-calc { background: #22d3ee; color: #0f172a; border: none; padding: 15px; border-radius: 8px; width: 100%; font-weight: bold; font-size: 18px; cursor: pointer; }
        .res-area { margin-top: 15px; padding: 15px; background: #000; border: 2px solid #22d3ee; border-radius: 8px; text-align: center; display: none; }
        .del-btn { background: #ef4444; border: none; color: white; padding: 5px 10px; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>

<div class="card">
    <h2>Calculator Producție Personalizat</h2>

    <div class="section">
        <div style="display: flex; gap: 10px;">
            <div style="flex: 1;">
                <label>Ore Lucrate (ex: 4 sau 7.5):</label>
                <input type="number" id="ore_lucrate" step="0.1" value="7.5">
            </div>
            <div style="flex: 1;">
                <label>Nr. Persoane:</label>
                <input type="number" id="global_pers" value="3">
            </div>
        </div>
    </div>

    <div class="section">
        <span style="font-weight: bold; color: #38bdf8; font-size: 14px;">Activități (Etichete, Schimburi, Curățenie)</span>
        <div id="lista-activitati">
            <div class="row">
                <div style="flex: 2; min-width: 100px;">
                    <label>Nume Activitate</label>
                    <input type="text" class="nume" value="Etichete">
                </div>
                <div style="flex: 1;">
                    <label>Cantitate</label>
                    <input type="number" class="val" value="0">
                </div>
                <div style="flex: 1;">
                    <label>Coef.</label>
                    <input type="number" class="coef" step="0.01" value="0">
                </div>
                <div style="flex: 2;">
                    <label>Mod Calcul</label>
                    <select class="tip">
                        <option value="punctaj">Înmulțește (x1.35)</option>
                        <option value="timp">Scade din Timp</option>
                    </select>
                </div>
            </div>
        </div>
        <button class="btn-add" onclick="adaugRand()">+ ADAUGĂ ALTĂ ACTIVITATE</button>
    </div>

    <div class="section">
        <span style="font-weight: bold; color: #6366f1; font-size: 14px;">Lăzi (Calcul Direct Buc x Coef)</span>
        <div id="lista-lazi"></div>
        <button class="btn-add" style="background:#6366f1" onclick="adaugLazi()">+ ADAUGĂ LĂZI</button>
    </div>

    <button class="btn-calc" onclick="calculeaza()">SOCATE REZULTATUL</button>

    <div id="rezultat" class="res-area">
        <div style="color: #94a3b8; font-size: 14px;">TIMP EFECTIV RĂMAS: <span id="timp-r" style="color: white; font-weight: bold;">0</span> min</div>
        <div style="margin-top: 10px; color: #94a3b8;">PUNCTAJ TOTAL:</div>
        <div id="total-punctaj" style="font-size: 32px; color: #22d3ee; font-weight: bold;">0</div>
    </div>
</div>

<script>
function adaugRand() {
    const div = document.createElement('div');
    div.className = 'row';
    div.innerHTML = `
        <div style="flex: 2; min-width: 100px;"><input type="text" class="nume" placeholder="Ex: Umbaun"></div>
        <div style="flex: 1;"><input type="number" class="val" placeholder="Buc/Min"></div>
        <div style="flex: 1;"><input type="number" class="coef" step="0.01" placeholder="Coef"></div>
        <div style="flex: 2;">
            <select class="tip">
                <option value="punctaj">Înmulțește (x1.35)</option>
                <option value="timp">Scade din Timp</option>
            </select>
        </div>
        <button class="del-btn" onclick="this.parentElement.remove()">X</button>`;
    document.getElementById('lista-activitati').appendChild(div);
}

function adaugLazi() {
    const div = document.createElement('div');
    div.className = 'row';
    div.innerHTML = `
        <div style="flex: 2;"><input type="text" placeholder="Sort Lăzi"></div>
        <div style="flex: 1;"><input type="number" class="l-buc" placeholder="Buc"></div>
        <div style="flex: 1;"><input type="number" class="l-coef" placeholder="Coef"></div>
        <button class="del-btn" onclick="this.parentElement.remove()">X</button>`;
    document.getElementById('lista-lazi').appendChild(div);
}

function calculeaza() {
    let ore = parseFloat(document.getElementById('ore_lucrate').value || 0);
    let nrPers = parseFloat(document.getElementById('global_pers').value || 1);
    let timpInitialTotal = ore * 60 * nrPers;
    
    let timpDeScazut = 0;
    let punctajTotal = 0;
    const premieProcent = 1.35;

    // Calculăm activitățile (Etichete, Schimburi etc)
    const randuri = document.querySelectorAll('#lista-activitati .row');
    randuri.forEach(r => {
        let val = parseFloat(r.querySelector('.val').value || 0);
        let coef = parseFloat(r.querySelector('.coef').value || 0);
        let tip = r.querySelector('.tip').value;

        if (tip === "timp") {
            // Dacă scade din timp, înmulțim minutele cu numărul de oameni
            timpDeScazut += (val * nrPers);
        } else {
            // Dacă e punctaj, aplicăm formula ta cu 1.35
            punctajTotal += (val * coef * premieProcent * nrPers);
        }
    });

    // Calculăm lăzile (Direct)
    const randuriLazi = document.querySelectorAll('#lista-lazi .row');
    randuriLazi.forEach(r => {
        let buc = parseFloat(r.querySelector('.l-buc').value || 0);
        let coef = parseFloat(r.querySelector('.l-coef').value || 0);
        punctajTotal += (buc * coef);
    });

    let timpRamas = timpInitialTotal - timpDeScazut;

    document.getElementById('timp-r').innerText = timpRamas.toFixed(0);
    document.getElementById('total-punctaj').innerText = punctajTotal.toFixed(2);
    document.getElementById('rezultat').style.display = 'block';
}
</script>
</body>
</html>
