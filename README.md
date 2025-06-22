<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plan Comercial – Cross/Cabecera + PDF</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body   { font-family: Arial, sans-serif; padding:10px; margin:0; background:#d8f5d0; }
    h1     { text-align:center; font-size:18px; margin-bottom:10px; background: linear-gradient(135deg, #00c9a7, #005b94); color: white; padding: 12px; border-radius: 10px; box-shadow: 0 2px 6px rgba(0,0,0,0.2); letter-spacing: 1px; text-shadow: 1px 1px 2px rgba(0,0,0,0.3); font-size: 26px; }
    #inputs{ display:grid; grid-template-columns:repeat(auto-fit,minmax(200px,1fr)); gap:10px }
    .item  { border:1px solid #ccc; padding:8px; position:relative; transition:opacity .2s; }
    .item h3{ margin:0 0 6px; font-size:15px }
    .item label{ display:block; font-size:12px; margin:4px 0 }
    .item select,.item input{ width:100%; padding:4px; font-size:12px; box-sizing:border-box }
    .preview-container{ margin-top:6px; position:relative }
    .preview-container img.preview{ width:100%; border:1px solid #ddd }
    .sms-row{ display:grid; grid-template-columns: 1fr 1.5fr 1fr; gap:4px; margin-bottom:4px }
    #controls{ text-align:center; margin:15px 0 }
    #addForm{ display:none; border:1px solid #007bff; padding:8px; margin-bottom:15px; background:#f2fef2 }
    #addForm h2{ font-size:14px; margin:0 0 6px; text-align:center }
    #bottomButtons{ text-align:center; margin:20px 0 }
    button{ padding:6px 14px; font-size:13px; cursor:pointer }
    @media(max-width:768px){ #inputs{ grid-template-columns:1fr } }
  </style>
</head>
<body>
  <h1>
    PLANO PLAN COMERCIAL
  </h1>

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

  <div id="bottomButtons">
    <button id="btnToggleForm">➕ Añadir elemento</button>
    <button id="btnGenerar">Crear PDF</button>
  </div>

<script>
// ---------- parámetros base ----------
const tipos      = ['Cross','Cabecera','Esfera o Exposicion Especial'];
const imageBase  = './';        // where nombre1.png ... reside
const container  = document.getElementById('inputs');

// ---------- crear 10 bloques por defecto (vacíos) ----------
for(let i=1;i<=10;i++){
  container.appendChild(makePreviewBlock(i));
}

// ---------- funciones ----------
function makePreviewBlock(n){
  const div=document.createElement('div');
  div.className='item';
  div.innerHTML=`<h3 style="color:#27ae60; font-weight:bold; font-size:16px;">🎯 OFERTA TOP ${n}</h3>
     <label><input type="checkbox" class="includeItem" checked> Incluir</label>
     <label>Tipo:
       <select class="plantilla">${tipos.map(t=>`<option>${t}</option>`).join('')}</select></label>
     <label>Nombre:
       <input type="text" class="nombre" placeholder="Escribe nombre libremente"></label>
     <div class="preview-container">
        <img class="preview" src="" alt="prev">
     </div>`;
  attachPreviewListeners(div);
  return div;
}

function attachPreviewListeners(item){
  const chk=item.querySelector('.includeItem');
  const inp=item.querySelector('.nombre');
  const img=item.querySelector('.preview');
  inp.addEventListener('input',()=>{ const v=inp.value.trim(); img.src=v?`${imageBase}${v}.png`:'';});
  chk.addEventListener('change',()=>{item.style.opacity=chk.checked?'1':'0.3'});
}

// ---------- formulario añadir extra ----------
const form=document.getElementById('addForm');
const formRows=document.getElementById('formRows');
for(let i=1;i<=6;i++){
  const row=document.createElement('div'); row.className='sms-row';
  row.innerHTML=`<input placeholder="SMS ${i}"><input placeholder="Descripción artículo ${i}"><input placeholder="Oferta ${i}">`;
  formRows.appendChild(row);
}

document.getElementById('btnToggleForm').onclick=()=>{
  form.style.display=form.style.display==='block'?'none':'block';
};

document.getElementById('btnAddElem').onclick=()=>{
  const tipo=document.getElementById('formTipo').value;
  const rows=[...formRows.querySelectorAll('.sms-row')].map(r=>{
    const [sms,desc,offer]=[...r.children].map(i=>i.value.trim());
    return {sms,desc,offer};
  });
  const idx=container.children.length+1;
  const div=document.createElement('div');
  div.className='item';
  div.innerHTML=`<h3 style="color:#27ae60; font-weight:bold; font-size:16px;">🎯 OFERTA EXTRA ${idx}</h3><label>Tipo:<input readonly value="${tipo}"></label>`+
    rows.map(r=>`<div class="sms-row"><input readonly value="${r.sms}"><input readonly value="${r.desc}"><input readonly value="${r.offer}"></div>`).join('');
  container.appendChild(div);
  form.style.display='none';
};

// ---------- PDF ----------
function getImageData(url){return new Promise(res=>{const i=new Image();i.crossOrigin='anonymous';i.src=url;i.onload=()=>{const c=document.createElement('canvas');c.width=i.width;c.height=i.height;c.getContext('2d').drawImage(i,0,0);res(c.toDataURL('image/png'));};i.onerror=()=>res(null);});}

document.getElementById('btnGenerar').addEventListener('click',async()=>{
  const {jsPDF}=window.jspdf;const doc=new jsPDF({unit:'pt',format:'letter'});
  const items=[...container.children];
  const perPage=6,cols=2,pw=doc.internal.pageSize.getWidth(),ph=doc.internal.pageSize.getHeight(),bw=(pw-80)/cols,bh=(ph-100)/3;
  for(let idx=0;idx<items.length;idx++){
    const item=items[idx];
    if(idx>0&&idx%perPage===0)doc.addPage();
    const ip=idx%perPage,col=ip%cols,row=Math.floor(ip/cols);
    const x=40+col*(bw+20);let y=60+row*(bh+20);
    doc.setFontSize(14);doc.text(item.querySelector('h3').textContent,x,y);y+=18;
    const tipoInput=item.querySelector('select')?item.querySelector('select').value:item.querySelector('input[readonly]').value;
    doc.setFontSize(12);doc.text(`Tipo: ${tipoInput}`,x,y);y+=16;
    const nameField=item.querySelector('.nombre');
    if(nameField){const v=nameField.value.trim();if(v){const imgData=await getImageData(`${imageBase}${v}.png`);if(imgData){const sz=Math.min(bw-20,bh-60);doc.addImage(imgData,'PNG',x,y,sz,sz);y+=sz+10;}}}
    item.querySelectorAll('.sms-row').forEach(r=>{const vals=[...r.children].map(i=>i.value||i.textContent);if(vals.some(v=>v)) {doc.text(vals.join(' - '),x,y);y+=14;}});
  }
  doc.save('plan_comercial.pdf');
});
</script>
</body>
</html>
