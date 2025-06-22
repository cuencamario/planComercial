<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plan Comercial y Generador de PDF</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; padding: 10px; margin: 0; }
    header { text-align: center; margin-bottom: 10px; }
    #controls { text-align: center; margin-bottom: 10px; }
    #addForm { display: none; border: 1px solid #007bff; padding: 10px; margin-bottom: 15px; }
    #addForm label { display: block; margin: 5px 0; font-size: 12px; }
    #smsContainer .sms-group { display: flex; gap: 5px; margin-bottom: 5px; }
    #smsContainer input { flex: 1; padding: 4px; font-size: 12px; box-sizing: border-box; }
    #inputs { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 15px; }
    .item {
      border: 1px solid #ccc;
      padding: 10px;
      position: relative;
      /* Fondo eliminado */
    }
    .item h3 { font-size: 16px; margin: 0 0 8px; }
    .item label { display: block; font-size: 12px; margin: 4px 0; }
    .item select, .item input { width: 100%; padding: 4px; font-size: 12px; box-sizing: border-box; }
    .item .sms-group { display: flex; gap: 5px; margin-bottom: 4px; }
    .item .sms-group input { flex: 1; }
    button { padding: 8px 16px; font-size: 14px; cursor: pointer; }
    @media (max-width: 768px) { #inputs { grid-template-columns: 1fr; } }
  </style>
</head>
<body>
  <header><h1>Plan Comercial: Añade y Genera PDF</h1></header>
  <div id="controls">
    <button id="btnToggleAdd">Añadir elemento</button>
  </div>
  <div id="addForm">
    <label>Tipo:
      <select id="newTipo">
        <option>Cross</option>
        <option>Cabecera</option>
      </select>
    </label>
    <div id="smsContainer"></div>
    <button id="btnAddItem">Agregar</button>
  </div>
  <div id="inputs"></div>
  <button id="btnGenerar">Crear PDF</button>

  <script>
    const container = document.getElementById('inputs');
    const smsContainer = document.getElementById('smsContainer');
    const newTipo = document.getElementById('newTipo');
    let count = 0;
    const bgUrl = 'https://www.carrefour.es/_includes/multimedia/es/logo-express-franquicias-inicio_20180820_tcm5-49264.png';

    // Crear 6 inputs SMS/nombre en addForm
    for (let i = 1; i <= 6; i++) {
      const div = document.createElement('div'); div.className = 'sms-group';
      div.innerHTML = `
        <input type="text" class="smsCode" placeholder="Código SMS ${i}">
        <input type="text" class="smsName" placeholder="Nombre artículo ${i}">`;
      smsContainer.appendChild(div);
    }

    // Pre-popular con 10 elementos vacíos inicialmente
    for (let i = 1; i <= 10; i++) {
      createItem('Cross', Array.from({length:6}).map(_ => ({code:'', name:''})));  
    }
        img.onerror = () => res(null);
      });
    }

    // Generar PDF
    document.getElementById('btnGenerar').addEventListener('click', async () => {
      const { jsPDF } = window.jspdf; const doc = new jsPDF({ unit:'pt', format:'letter' });
      const items = Array.from(container.children);
      const bgData = await getImageData(bgUrl);
      const perPage = 6, cols = 2;
      const pw = doc.internal.pageSize.getWidth(), ph = doc.internal.pageSize.getHeight();
      const bw = (pw - 80)/cols, bh = (ph - 100)/3;

      items.forEach((item, idx) => {
        if(idx>0 && idx%perPage===0) doc.addPage();
        if(idx%perPage===0 && bgData) doc.addImage(bgData,'PNG',pw-140,40,100,100);
        const ip = idx%perPage, col = ip%cols, row = Math.floor(ip/cols);
        const x = 40 + col*(bw+20); let y = 60 + row*(bh+20);
        doc.setFontSize(14); doc.text(item.querySelector('h3').textContent, x, y); y+=18;
        doc.setFontSize(12); doc.text(`Tipo: ${item.querySelector('input[readonly]').value}`, x, y); y+=16;
        item.querySelectorAll('.sms-group').forEach(g => { const [c,n] = g.querySelectorAll('input'); doc.text(`${c.value} - ${n.value}`, x, y); y+=14; });
      });
      doc.save('plan_comercial.pdf');
    });
  </script>
</body>
</html>

