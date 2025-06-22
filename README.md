# planComercial
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Elige tu cross y genera PDF en 2 hojas</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    #inputs {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 15px;
    }
    .item {
      border: 1px solid #ccc;
      padding: 10px;
      background-image: url('https://www.carrefour.es/_includes/multimedia/es/logo-express-franquicias-inicio_20180820_tcm5-49264.png');
      background-size: 80px auto;
      background-repeat: no-repeat;
      background-position: bottom right;
    }
    h3 { margin: 0 0 8px; font-size: 16px; }
    label { display: block; margin: 5px 0; font-size: 12px; }
    select, input[type="text"] { width: 100%; padding: 4px; font-size: 12px; box-sizing: border-box; }
    .preview-container { position: relative; margin-top: 8px; }
    .preview-container img.preview { display: block; max-width: 100%; height: auto; border: 1px solid #ddd; }
    .preview-container img.logo { position: absolute; width: 30px; top: 5px; left: 5px; pointer-events: none; }
    .controls { margin-top: 5px; }
    button { padding: 8px 16px; font-size: 14px; margin-top: 20px; }
  </style>
</head>
<body>
  <h1 style="font-size:18px; text-align:center;">Configura tus 10 cross y genera PDF en 2 páginas</h1>
  <div id="inputs"></div>
  <button id="btnGenerar" style="display:block; margin:20px auto;">Crear PDF</button>

  <script>
    const tipos = ['Cross','Cabecera','Esfera o Exposicion Especial'];
    const container = document.getElementById('inputs');
    const bgUrl = 'https://www.carrefour.es/_includes/multimedia/es/logo-express-franquicias-inicio_20180820_tcm5-49264.png';
    const defaultImageUrl = 'https://res.cloudinary.com/dylcexk0m/image/upload/v1750593766/Nombre1_wjs9n3.png';
    const logoUrl = bgUrl;

    // Crear bloques con checkbox para incluir el elemento
    for (let i = 1; i <= 10; i++) {
      const div = document.createElement('div');
      div.className = 'item';
      div.innerHTML = `
        <h3>Elemento ${i}</h3>
        <label>Incluir:
          <input type="checkbox" class="includeItem" checked>
        </label>
        <label>Tipo:
          <select class="plantilla">
            ${tipos.map(t => `<option>${t}</option>`).join('')}
          </select>
        </label>
        <label>Nombre:
          <input type="text" class="nombre" placeholder="Nombre">
        </label>
        <label>URL imagen (opcional):
          <input type="text" class="imgName" placeholder="${defaultImageUrl}">
        </label>
        <div class="preview-container">
          <img class="preview" src="${defaultImageUrl}" alt="Vista previa">
          <img class="logo" src="${logoUrl}" alt="Logo">
        </div>
      `;
      container.appendChild(div);
    }

    // Actualizar preview y checkbox
    container.querySelectorAll('.item').forEach(item => {
      const input = item.querySelector('.imgName');
      const img   = item.querySelector('.preview');
      const chk   = item.querySelector('.includeItem');
      input.addEventListener('input', () => {
        img.src = input.value.trim() || defaultImageUrl;
      });
      chk.addEventListener('change', () => {
        item.style.opacity = chk.checked ? '1' : '0.5';
      });
    });

    // Convierte URL a DataURL
    function getImageData(url) {
      return new Promise((resolve) => {
        const img = new Image();
        img.crossOrigin = 'anonymous';
        img.src = url;
        img.onload = () => {
          const canvas = document.createElement('canvas');
          canvas.width = img.width;
          canvas.height = img.height;
          canvas.getContext('2d').drawImage(img, 0, 0);
          const format = url.toLowerCase().endsWith('.png') ? 'image/png' : 'image/jpeg';
          resolve(canvas.toDataURL(format));
        };
        img.onerror = () => resolve(null);
      });
    }

    document.getElementById('btnGenerar').addEventListener('click', async () => {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF({ unit: 'pt', format: 'letter' });
      const items = Array.from(document.querySelectorAll('.item'));

      // Cargar fondo y las imágenes de cada elemento
      const [bgData, logoData, ...images] = await Promise.all([
        getImageData(bgUrl),
        getImageData(logoUrl),
        ...items.map(item => getImageData(item.querySelector('.imgName').value.trim() || defaultImageUrl))
      ]);

      const perPage = 6; // ajustar si 2 filas x3
      const pageWidth = doc.internal.pageSize.getWidth();
      const pageHeight = doc.internal.pageSize.getHeight();
      const cols = 3;
      const rows = 2;
      const blockWidth = (pageWidth - 80) / cols;
      const blockHeight = (pageHeight - 100) / rows;

      items.forEach((item, idx) => {
        const pageIndex = Math.floor(idx / perPage);
        const idxInPage = idx % perPage;
        if (idx > 0 && idxInPage === 0) doc.addPage();
        // pintar fondo una vez por página
        if (idxInPage === 0 && bgData) {
          const bw = 100, bh = 100;
          doc.addImage(bgData,'PNG', pageWidth - bw - 40, 40, bw, bh);
        }
        const col = idxInPage % cols;
        const row = Math.floor(idxInPage / cols);
        const x0 = 40 + col * (blockWidth + 10);
        const y0 = 60 + row * (blockHeight + 10);
        let y = y0;

        const tipo = item.querySelector('.plantilla').value;
        const nombre = item.querySelector('.nombre').value || '(sin nombre)';
        const imgData = images[idx];

        // texto
        doc.setFontSize(12);
        doc.text(`#${idx+1} ${tipo}`, x0, y);
        y += 14;
        doc.setFontSize(10);
        doc.text(`Nombre: ${nombre}`, x0, y);
        y += 16;

        // imagen principal
        if (imgData) {
          const imgSize = Math.min(blockWidth - 20, blockHeight - 40);
          const fmt = imgData.startsWith('data:image/png')?'PNG':'JPEG';
          doc.addImage(imgData, fmt, x0, y, imgSize, imgSize);
          // overlay logo en la esquina superior izquierda de la imagen
          if (logoData) {
            const logoSize = imgSize * 0.2; // 20% tamaño de la imagen
            doc.addImage(logoData,'PNG', x0 + 5, y + 5, logoSize, logoSize);
          }
        }
      });

      doc.save('mis_10_cross_con_logo_overlay.pdf');
    });
  </script>
</body>
</html>
