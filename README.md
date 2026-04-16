<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Calculadora ISR + Prima Vacacional</title>

<style>
body {
    font-family: 'Segoe UI', sans-serif;
    background: linear-gradient(135deg, #1e3c72, #2a5298);
    margin: 0;
    padding: 20px;
    color: #333;
}

.container {
    max-width: 450px;
    margin: auto;
    background: white;
    padding: 20px;
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

h2 {
    text-align: center;
}

.section {
    margin-bottom: 15px;
    padding: 12px;
    background: #f7f9fc;
    border-radius: 10px;
}

.section h3 {
    margin-bottom: 10px;
    color: #2a5298;
}

label {
    font-size: 14px;
    font-weight: bold;
}

input {
    width: 100%;
    padding: 12px;
    margin: 8px 0;
    border-radius: 8px;
    border: 1px solid #ccc;
    font-size: 15px;
    transition: 0.2s;
}

input:focus {
    outline: none;
    border: 1px solid #2a5298;
    box-shadow: 0 0 5px rgba(42,82,152,0.4);
}

button {
    width: 100%;
    padding: 12px;
    border-radius: 8px;
    border: none;
    font-size: 15px;
    font-weight: bold;
    cursor: pointer;
    transition: 0.3s;
}

.main-btn {
    background: #2a5298;
    color: white;
}

.main-btn:hover {
    background: #1e3c72;
}

.toggle {
    display: flex;
    gap: 10px;
}

.toggle button {
    flex: 1;
    background: #ccc;
}

.toggle .activo {
    background: #2a5298;
    color: white;
}

.resultado {
    margin-top: 15px;
    background: linear-gradient(135deg, #e3f2fd, #bbdefb);
    padding: 15px;
    border-radius: 10px;
    text-align: center;
    font-weight: bold;
    font-size: 16px;
}

.historial {
    margin-top: 20px;
}

.historial-item {
    background: #f5f5f5;
    padding: 10px;
    border-radius: 8px;
    margin-bottom: 8px;
    font-size: 13px;
}

.clear-btn {
    background: #c62828;
    color: white;
    margin-top: 10px;
}
</style>
</head>

<body>

<div class="container">
    <h2>Calculadora ISR</h2>

    <div class="section">
        <h3>Tipo de cálculo</h3>
        <div class="toggle">
            <button onclick="setTipo('mensual')" id="btnMensual" class="activo">Mensual</button>
            <button onclick="setTipo('anual')" id="btnAnual">Anual</button>
        </div>
        <input type="hidden" id="tipo" value="mensual">
    </div>

    <div class="section">
        <h3>Datos</h3>

        <label>Sueldo</label>
        <input type="number" id="sueldo" placeholder="Ej: 15000">

        <label>Días de vacaciones</label>
        <input type="number" id="dias" placeholder="Ej: 12">

        <label>Prima vacacional (%)</label>
        <input type="number" step="0.01" id="prima" value="0.25">
    </div>

    <button class="main-btn" onclick="calcular()">Calcular</button>

    <div class="resultado" id="resultado"></div>

    <div class="historial">
        <h3>Historial</h3>
        <div id="listaHistorial"></div>
        <button class="clear-btn" onclick="limpiarHistorial()">Borrar historial</button>
    </div>
</div>

<script>
function setTipo(valor) {
    document.getElementById("tipo").value = valor;

    document.getElementById("btnMensual").classList.remove("activo");
    document.getElementById("btnAnual").classList.remove("activo");

    if (valor === "mensual") {
        document.getElementById("btnMensual").classList.add("activo");
    } else {
        document.getElementById("btnAnual").classList.add("activo");
    }
}

function formato(num) {
    return num.toLocaleString('es-MX', {
        style: 'currency',
        currency: 'MXN'
    });
}

function calcular() {
    let tipo = document.getElementById("tipo").value;
    let sueldo = parseFloat(document.getElementById("sueldo").value);
    let dias = parseFloat(document.getElementById("dias").value);
    let prima = parseFloat(document.getElementById("prima").value);
    let UMA = 117.31;

    if (isNaN(sueldo) || isNaN(dias) || isNaN(prima)) {
        alert("Completa los campos");
        return;
    }

    let sueldoDiario = (tipo === "mensual") ? sueldo / 30 : sueldo / 365;
    let vacaciones = sueldoDiario * dias;
    let primaVacacional = vacaciones * prima;

    let exento = UMA * 15;
    let gravado = Math.max(0, primaVacacional - exento);

    let base = sueldo + gravado;

    let tabla = [
        [0.01, 10135.11, 0.00, 0.0192],
        [10135.12, 86022.11, 194.59, 0.064],
        [86022.12, 151176.19, 5051.37, 0.1088],
        [151176.20, 175735.66, 12140.13, 0.16],
        [175735.67, 210403.69, 16069.64, 0.1792],
        [210403.70, 424353.97, 22282.14, 0.2136],
        [424353.98, 668840.14, 67981.92, 0.2352],
        [668840.15, 1276925.98, 125485.07, 0.30],
        [1276925.99, 1702567.97, 307910.81, 0.32],
        [1702567.98, 5107703.92, 444116.23, 0.34],
        [5107703.93, Infinity, 1601862.46, 0.35]
    ];

    let isr = 0;

    for (let i = 0; i < tabla.length; i++) {
        let [li, ls, cuota, tasa] = tabla[i];
        if (base >= li && base <= ls) {
            isr = (base - li) * tasa + cuota;
            break;
        }
    }

    if (tipo === "mensual") isr /= 12;

    let neto = sueldo + primaVacacional - isr;

    let resultadoTexto = `
        Prima: ${formato(primaVacacional)} <br>
        ISR: ${formato(isr)} <br>
        Neto: ${formato(neto)}
    `;

    document.getElementById("resultado").innerHTML = resultadoTexto;

    guardarHistorial(resultadoTexto);
}

function guardarHistorial(texto) {
    let historial = JSON.parse(localStorage.getItem("historial")) || [];
    historial.unshift(texto);
    localStorage.setItem("historial", JSON.stringify(historial));
    mostrarHistorial();
}

function mostrarHistorial() {
    let historial = JSON.parse(localStorage.getItem("historial")) || [];
    let contenedor = document.getElementById("listaHistorial");
    contenedor.innerHTML = "";

    historial.forEach(item => {
        let div = document.createElement("div");
        div.className = "historial-item";
        div.innerHTML = item;
        contenedor.appendChild(div);
    });
}

function limpiarHistorial() {
    localStorage.removeItem("historial");
    mostrarHistorial();
}

window.onload = mostrarHistorial;
</script>

</body>
</html>
