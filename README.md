<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simulador de Crédito e Inversión</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
  <style>
  body {
    background-color: #93bcea;
    padding: 40px;
  }
  .tab-content {
    margin-top: 20px;
  }
  canvas {
    margin-top: 30px;
  }
  .btn-center {
    display: flex;
    justify-content: center;
  }
  .form-label {
    font-weight: bold;
  }
  input[type=number]::-webkit-outer-spin-button,
  input[type=number]::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  input[type=number] {
    -moz-appearance: textfield;
  }
</style>

</head>
<body>
  <div class="container">
    <h1 class="text-center mb-4"> SOLVEXLIM</h1>
    <h2 class="text-center mb-4">¡SIMULA TU CRÉDITO BANCARIO!</h2>
    <ul class="nav nav-tabs" id="simuladorTabs" role="tablist">
      <li class="nav-item" role="presentation">
        <button class="nav-link active" id="credito-tab" data-bs-toggle="tab" data-bs-target="#credito" type="button" role="tab">Crédito</button>
      </li>
      <li class="nav-item" role="presentation">
        <button class="nav-link" id="inversion-tab" data-bs-toggle="tab" data-bs-target="#inversion" type="button" role="tab">Inversión</button>
      </li>
    </ul>
    <div class="tab-content" id="simuladorTabsContent">
      <!-- Crédito -->
      <div class="tab-pane fade show active" id="credito" role="tabpanel">
        <form id="formCredito" class="row g-3">
          <div class="col-md-3">
            <label for="capital" class="form-label">Capital ($)</label>
            <input type="number" class="form-control" id="capital" required>
          </div>
          <div class="col-md-3">
            <label for="plazo" class="form-label">Plazo (meses)</label>
            <input type="number" class="form-control" id="plazo" required>
          </div>
          <div class="col-md-3">
            <label for="tasa" class="form-label">Tasa Anual (%)</label>
            <input type="number" class="form-control" id="tasa" required>
          </div>
          <div class="col-md-3">
            <label for="iva" class="form-label">IVA (%)</label>
            <input type="number" class="form-control" id="iva" value="16" required>
          </div>
          <div class="col-md-12 d-flex justify-content-end">
            <button type="submit" class="btn btn-primary">Calcular</button>
          </div>
        </form>
        <div id="resultCredito" class="mt-4"></div>
        <div id="tablaCredito" class="table-responsive mt-3"></div>
        <div class="btn-center">
          <button class="btn btn-success mt-3" onclick="exportarTabla('tablaCredito', 'Amortizacion_Credito')">Descargar Excel</button>
        </div>
        <canvas id="graficaCredito"></canvas>
        <div id="resultCredito" class="mt-4"></div>
<div class="btn-center">
  <button class="btn btn-secondary mt-2" onclick="location.reload()">Nueva Simulación</button>
</div>
      </div>

      <!-- Inversión -->
      <div class="tab-pane fade" id="inversion" role="tabpanel">
        <form id="formInversion" class="row g-3">
          <div class="col-md-3">
            <label for="inversionCapital" class="form-label">Capital Inicial ($)</label>
            <input type="number" class="form-control" id="inversionCapital" required>
          </div>
          <div class="col-md-3">
            <label for="inversionPlazo" class="form-label">Plazo (meses)</label>
            <input type="number" class="form-control" id="inversionPlazo" required>
          </div>
          <div class="col-md-3">
            <label for="inversionTasa" class="form-label">Tasa Anual (%)</label>
            <input type="number" class="form-control" id="inversionTasa" required>
          </div>
          <div class="col-md-3 d-flex align-items-end">
            <button type="submit" class="btn btn-primary w-100">Calcular</button>
          </div>
        </form>
        <div id="resultInversion" class="mt-4"></div>
        <div id="tablaInversion" class="table-responsive mt-3"></div>
        <div class="btn-center">
          <button class="btn btn-success mt-3" onclick="exportarTabla('tablaInversion', 'Rendimiento_Inversion')">Descargar Excel</button>
        </div>
        
        <canvas id="graficaInversion"></canvas>

        <div id="resultInversion" class="mt-4"></div>
<div class="btn-center">
  <button class="btn btn-secondary mt-2" onclick="location.reload()">Nueva Simulación</button>
</div>
      </div>
    </div>
  </div>

  <script>
    let graficaCreditoActual, graficaInversionActual;
    document.getElementById('formCredito').addEventListener('submit', function(e) {
  e.preventDefault();
  const capital = parseFloat(document.getElementById('capital').value);
  const plazo = parseInt(document.getElementById('plazo').value);
  const tasa = parseFloat(document.getElementById('tasa').value);
  const ivaPorcentaje = parseFloat(document.getElementById('iva').value);
  const ivaTasa = ivaPorcentaje / 100;

  const mensualidad = (capital / plazo) * (tasa / 100) * (1 + ivaTasa);

  let saldo = capital;
  let tabla = '<table class="table table-bordered"><thead><tr><th>Mes</th><th>Pago Mensual</th><th>Saldo</th></tr></thead><tbody>';
  let totalPagado = 0;
  let saldos = [];

  for (let i = 1; i <= plazo; i++) {
    saldo -= capital / plazo;
    totalPagado += mensualidad;
    tabla += `<tr><td>${i}</td><td>${mensualidad.toFixed(2)}</td><td>${saldo.toFixed(2)}</td></tr>`;
    saldos.push(saldo.toFixed(2));
  }

  tabla += '</tbody></table>';
  document.getElementById('tablaCredito').innerHTML = tabla;
  document.getElementById('resultCredito').innerHTML = `
    <p><strong>Pago Mensual:</strong> $${mensualidad.toFixed(2)}</p>
    <p><strong>Total a Pagar:</strong> $${totalPagado.toFixed(2)}</p>
  `;

  if (graficaCreditoActual) graficaCreditoActual.destroy();
  graficaCreditoActual = new Chart(document.getElementById('graficaCredito'), {
    type: 'line',
    data: {
      labels: [...Array(plazo).keys()].map(i => i + 1),
      datasets: [{
        label: 'Saldo Restante',
        data: saldos,
        borderColor: 'rgba(75, 192, 192, 1)',
        tension: 0.1
      }]
    }
  });
});
    function exportarTabla(idTabla, nombreArchivo) {
      let tabla = document.getElementById(idTabla).getElementsByTagName('table')[0];
      let wb = XLSX.utils.table_to_book(tabla, {sheet: "Sheet1"});
      XLSX.writeFile(wb, `${nombreArchivo}.xlsx`);
    }
  </script>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
