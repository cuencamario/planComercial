<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plan Comercial – Cross/Cabecera + PDF</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body   { font-family: Arial, sans-serif; padding:10px; margin:0 }
    h1     { text-align:center; font-size:18px; margin-bottom:10px }
    #inputs{ display:grid; grid-template-columns:repeat(auto-fit,minmax(200px,1fr)); gap:10px }
    .item  { border:1px solid #ccc; padding:8px; position:relative; transition:opacity .2s; background-image:url('https://www.carrefour.es/_includes/multimedia/es/logo-express-franquicias-inicio_20180820_tcm5-49264.png'); background-size:60px auto; background-repeat:no-repeat; background-position:bottom right }
    .item h3{ margin:0 0 6px; font-size:15px }
    .item label{ display:block; font-size:12px; margin:4px 0 }
    .item select,.item input{ width:100%; padding:4px; font-size:12px; box-sizing:border-box }
    .preview-container{ margin-top:6px; position:relative }
    .preview-container img.preview{ width:100%; border:1px solid #ddd }
    .preview-container img.logo{ position:absolute; width:30px; top:5px; left:5px; pointer-events:none }
    .sms-row{ display:grid; grid-template-columns: 1fr 1.5fr 1fr; gap:4px; margin-bottom:4px }
    #controls{ text-align:center; margin:15px 0 }
    #addForm{ display:none; border:1px solid #007bff; padding:8px; margin-bottom:15px }
    #addForm h2{ font-size:14px; margin:0 0 6px; text-align:center }
    button{ padding:6px 14px; font-size:13px; cursor:pointer }
    @media(max-width:768px){ #inputs{ grid-template-columns:1fr } }
  </style>
</head>
<body>
  <h1>Configura tus 10 Cross y Genera PDF</h1>

  <div id="controls">
    <button id="btnToggleForm">➕ Añadir elemento</button>
  </div>

  <!-- Formulario emergente para añadir nuevos elementos -->
  <div id="addForm">
    <h2>Nuevo elemento</h2>
    <label>Tipo:
      <select id="formTipo">
        <option>Cross</option>
        <option>Cabecera</option>
        <option>Esfera o Exposicion Especial</option>
      </select>
    </label>
    <div id="formRows"></div>
    <button id="btnAddElem">Agregar al listado</button>
  </div>

  <!-- Contenedor de los 10 bloques iniciales + añadidos -->
  <div id="inputs"></div>
  <button id="btnGenerar">Crear PDF</button>

<script>
// ---------- parámetros base ----------
const tipos      = ['Cross','Cabecera','Esfera o Exposicion Especial'];
const imageBase  = './';        // where nombre1.png ... reside
const bgUrl      = 'https://www.carrefour.es/_includes/multimedia/es/logo-express-franquicias-inicio_20180820_tcm5-49264.png';
const container  = document.getElementById('inputs');

// ---------- crear 10 bloques por defecto (con preview) ----------
for(let i=1;i<=10;i++){
  const nombre=`nombre${i}`;
  container.appendChild(makePreviewBlock(i,nombre));
}

// ---------- funciones ----------
function makePreviewBlock(n, nombre){
  const div=document.createElement('div');
  div.className='item';
  div.innerHTML=`<h3>Elemento ${n}</h3>
     <label><input type="checkbox" class="includeItem" checked> Incluir</label>
     <label>Tipo:
       <select class="plantilla">${tipos.map(t=>`<option>${t}</option>`).join('')}</select></label>
     <label>Nombre:
       <input type="text" class="nombre" value="${nombre}"></label>
     <div class="preview-container">
        <img class="preview" src="${imageBase}${nombre}.png" alt="prev">
        <img class="logo" src="${bgUrl}" alt="logo">
     </div>`;
  attachPreviewListeners(div);
  return div;
}

function attachPreviewListeners(item){
  const chk=item.querySelector('.includeItem');
  const inp=item.querySelector('.nombre');
  const img=item.querySelector('.preview');
  inp.addEventListener('input',()=>{ img.src=`${imageBase}${inp.value.trim()}.png`;});
  chk.addEventListener('change',()=>{item.style.opacity=chk.checked?'1':'0.3'});
}

// ---------- formulario añadir extra ----------
const form=document.getElementById('addForm');
const formRows=document.getElementById('formRows');
// generar 6 filas (SMS / Artículo / Oferta)
for(let i=1;i<=6;i++){
  const row=document.createElement('div'); row.className='sms-row';
  row.innerHTML=`<input placeholder="SMS ${i}"><input placeholder="Descripción artículo ${i}"><input placeholder="Oferta ${i}">`;
  formRows.appendChild(row);
}

document.getElementById('btnToggleForm').onclick=()=>{
  form.style.display=form.style.display==='block'?'none':'block';
};

document.getElementById('btnAddElem').onclick=()=>{
  // recolectar datos
  const tipo=document.getElementById('formTipo').value;
  const rows=[...formRows.querySelectorAll('.sms-row')].map(r=>{
      const [sms,desc,offer]=[...r.children].map(i=>i.value.trim());
      return {sms,desc,offer};
  });
  // crear bloque visual
  const idx=container.children.length+1;
  const div=document.createElement('div');
  div.className='item';
  div.innerHTML=`<h3>Elemento ${idx}</h3><label>Tipo:<input readonly value="${tipo}"></label>`+
    rows.map(r=>`<div class="sms-row"><input readonly value="${r.sms}"><input readonly value="${r.desc}"><input readonly value="${r.offer}"></div>`).join('');
  container.appendChild(div);
  form.style.display='none';
};

// ---------- PDF ----------
function getImageData(url){return new Promise(res=>{const i=new Image();i.crossOrigin='anonymous';i.src=url;i.onload=()=>{const c=document.createElement('canvas');c.width=i.width;c.height=i.height;c.getContext('2d').drawImage(i,0,0);res(c.toDataURL('image/png'));};i.onerror=()=>res(null);});}

document.getElementById('btnGenerar').addEventListener('click',async()=>{
  const {jsPDF}=window.jspdf;const doc=new jsPDF({unit:'pt',format:'letter'});
  const bgData=await getImageData(bgUrl);
  const items=[...container.children];
  const perPage=6,cols=2,pw=doc.internal.pageSize.getWidth(),ph=doc.internal.pageSize.getHeight(),bw=(pw-80)/cols,bh=(ph-100)/3;
  items.forEach((item,idx)=>{
    if(idx>0&&idx%perPage===0)doc.addPage();
    const ip=idx%perPage,col=ip%cols,row=Math.floor(ip/cols);
    const x=40+col*(bw+20);let y=60+row*(bh+20);
    if(idx%perPage===0&&bgData)doc.addImage(bgData,'PNG',pw-120,40,90,90);
    doc.setFontSize(14);doc.text(item.querySelector('h3').textContent,x,y);y+=18;
    const tipoInput=item.querySelector('select')?item.querySelector('select').value:item.querySelector('input[readonly]').value;
    doc.setFontSize(12);doc.text(`Tipo: ${tipoInput}`,x,y);y+=16;
    // filas datos
    item.querySelectorAll('.sms-row').forEach(r=>{
      const values=[...r.children].map(i=>i.value||i.textContent);
      doc.text(values.join(' - '),x,y);y+=14;
    });
  });
  doc.save('plan_comercial.pdf');
});
</script>
</body>
</html>
