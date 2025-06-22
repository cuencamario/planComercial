# planComercial
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Elige tu cross y genera PDF</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; padding: 10px; margin: 0; }
    h1 { text-align: center; font-size: 18px; margin-bottom: 10px; }
    #inputs { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 10px; }
    .item { border: 1px solid #ccc; padding: 8px; transition: opacity 0.2s; background-image: url('https://www.carrefour.es/_includes/multimedia/es/logo-express-franquicias-inicio_20180820_tcm5-49264.png'); background-size: 60px auto; background-repeat: no-repeat; background-position: bottom right; }
    .item label { display: block; font-size: 12px; margin: 4px 0; }
    .item select, .item input[type="text"], .item input[type="checkbox"] { width: 100%; font-size: 12px; padding: 4px; box-sizing: border-box; }
    .preview-container { margin-top: 6px; position: relative; }
    .preview-container img.preview { width: 100%; height: auto; border: 1px solid #ddd; }
    .preview-container img.logo { position: absolute; width: 30px; top: 5px; left: 5px; pointer-events: none; }
    button { display: block; margin: 15px auto; padding: 8px 16px; font-size: 14px; }
  </style>
</head>
<body>
  <h1>Configura tus 10 cross y genera PDF</h1>
  <div id="inputs"></div>
  <button id="btnGenerar">Crear PDF</button>
  <script>
    const tipos = ['Cross','Cabecera','Esfera o Exposicion Especial'];
    const container = document.getElementById('inputs');
    const imageBase = './'; // carpeta donde están nombre1.png ... nombre10.png
    const bgUrl = 'https://www.carrefour.es/_includes/multimedia/es/logo-express-franquicias-inicio_20180820_tcm5-49264.png';
    // Crear bloques
    for (let i = 1; i <= 10; i++) {
      const nombreDef = `nombre${i}`;
      const div = document.createElement('div'); div.className = 'item';
      div.innerHTML = `
        <h3>Elemento ${i}</h3>
        <label><input type="checkbox" class="includeItem" checked> Incluir</label>
        <label>Tipo:<select class="plantilla">${tipos.map(t=>`<option>${t}</option>`).join('')}</select></label>
        <label>Nombre:<input type="text" class="nombre" value="${nombreDef}"></label>
        <div class="preview-container">
          <img class="preview" src="${imageBase}${nombreDef}.png" alt="Vista previa">
          <img class="logo" src="${bgUrl}" alt="Logo">
        </div>`;
      container.appendChild(div);
    }
    // Toggle y update preview if nombre cambia
    container.querySelectorAll('.item').forEach(item=>{
      const chk = item.querySelector('.includeItem');
      const nombreInput = item.querySelector('.nombre');
      const img = item.querySelector('.preview');
      nombreInput.addEventListener('input', ()=>{
        img.src = `${imageBase}${nombreInput.value.trim()}.png`;
      });
      chk.addEventListener('change', ()=>{
        item.style.opacity = chk.checked ? '1' : '0.3';
      });
    });
    // dataURL converter
    function getImageData(url){return new Promise(res=>{const img=new Image();img.crossOrigin='anonymous';img.src=url;img.onload=()=>{const c=document.createElement('canvas');c.width=img.width;c.height=img.height;c.getContext('2d').drawImage(img,0,0);res(c.toDataURL(url.endsWith('.png')?'image/png':'image/jpeg'));};img.onerror=()=>res(null);});}
    document.getElementById('btnGenerar').addEventListener('click',async()=>{
      const { jsPDF } = window.jspdf; const doc=new jsPDF({unit:'pt',format:'letter'});
      const items=Array.from(document.querySelectorAll('.item')).filter(it=>it.querySelector('.includeItem').checked);
      const [bgData,...images]=await Promise.all([getImageData(bgUrl),...items.map(it=>getImageData(`${imageBase}${it.querySelector('.nombre').value.trim()}.png`))]);
      const perPage=6,cols=3,rows=2; const pw=doc.internal.pageSize.getWidth(),ph=doc.internal.pageSize.getHeight();
      const bw=(pw-80)/cols,bh=(ph-100)/rows;
      items.forEach((item,idx)=>{if(idx>0&&idx%perPage===0)doc.addPage(); if(idx%perPage===0&&bgData)doc.addImage(bgData,'PNG',pw-100-40,40,100,100);
        const ip=idx%perPage,col=ip%cols,row=Math.floor(ip/cols);const x=40+col*(bw+10),y0=60+row*(bh+10);let y=y0;
        const tipo=item.querySelector('.plantilla').value,nombre=item.querySelector('.nombre').value,imgData=images[idx];
        doc.setFontSize(12);doc.text(`#${idx+1} ${tipo}`,x,y);y+=14;doc.setFontSize(10);doc.text(`Nombre: ${nombre}`,x,y);y+=16;
        if(imgData){const sz=Math.min(bw-20,bh-40),fmt=imgData.startsWith('data:image/png')?'PNG':'JPEG';doc.addImage(imgData,fmt,x,y,sz,sz);const lsz=sz*0.2;doc.addImage(bgData,'PNG',x+5,y+5,lsz,lsz);} });
      doc.save('mis_10_cross.pdf');
    });
  </script>
</body>
</html>
