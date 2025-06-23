<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plan Comercial – Cross/Cabecera + PDF</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body{font-family:Arial, sans-serif;padding:10px;margin:0;background:#d8f5d0}
    h1{font-size:26px;text-align:center;background:linear-gradient(135deg,#00c9a7 0%,#005b94 100%);color:#fff;padding:12px 8px;border-radius:10px;box-shadow:0 2px 6px rgba(0,0,0,.2);margin:0 0 10px;letter-spacing:1px;text-shadow:1px 1px 2px rgba(0,0,0,.3)}
    #inputs{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:10px}
    .item{border:1px solid #ccc;padding:8px;background:#fff;transition:opacity .2s}
    .item h3{margin:0 0 6px;font-size:15px}
    .item label{display:block;font-size:12px;margin:4px 0}
    .item select,.item input{width:100%;padding:4px;font-size:12px;box-sizing:border-box}
    .preview-container{margin-top:6px}
    .preview-container img{width:100%;border:1px solid #ddd}
    .sms-row{display:grid;grid-template-columns:1fr 1.5fr 1fr;gap:4px;margin-bottom:4px}
    #addForm{display:none;border:1px solid #007bff;padding:8px;margin-bottom:15px;background:#f2fef2}
    #addForm h2{font-size:14px;margin:0 0 6px;text-align:center}
    #bottomButtons{text-align:center;margin:20px 0}
    button{padding:6px 14px;font-size:13px;cursor:pointer}
    #tiendaField{text-align:center;margin:10px 0;}
    #tiendaField input{width:200px;padding:6px;font-size:14px;text-align:center;border:1px solid #ccc;border-radius:5px;}
    @media(max-width:768px){#inputs{grid-template-columns:1fr}}
  </style>
</head>
<body>
  <h1>PLANO PLAN COMERCIAL</h1>

  <div id="tiendaField">
    <label for="nombreTienda"><strong>Nombre de tienda:</strong></label><br>
    <input type="text" id="nombreTienda" placeholder="Introduce nombre de tienda">
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
    <label>Descripción:<input type="text" id="formDescripcion" placeholder="Descripción del elemento"></label>
    <label>Nombre de imagen:<input type="text" id="formNombreImagen" placeholder="nombre1, nombre2..."></label>
    <button id="btnAddElem">Agregar al listado</button>
  </div>

  <div id="inputs"></div>

  <div id="bottomButtons">
    <button id="btnToggleForm">➕ Añadir elemento</button>
    <button id="btnGenerar">Crear PDF</button>
  </div>

<script>
const tipos=['Cross','Cabecera','Esfera o Exposicion Especial'];
const imageBase='./';
const container=document.getElementById('inputs');

for(let i=1;i<=10;i++){
  container.appendChild(makePreviewBlock(i,`nombre${i}`));
}

function makePreviewBlock(num,fileName){
  const div=document.createElement('div');
  div.className='item';
  div.innerHTML=`<h3 style="color:#27ae60;font-weight:bold;font-size:16px;">🎯 OFERTA TOP ${num}</h3>
    <label><input type="checkbox" class="includeItem" checked> Incluir</label>
    <label>Tipo:<select class="plantilla">${tipos.map(t=>`<option>${t}</option>`).join('')}</select></label>
    <input type="hidden" class="nombre" value="${fileName}">
    <label>Descripción:<input type="text" class="descripcion" placeholder="Añade texto libre"></label>
    <div class="preview-container"><img class="preview" src="${imageBase}${fileName}.png" alt="prev"></div>`;
  const chk=div.querySelector('.includeItem');
  chk.addEventListener('change',()=>{div.style.opacity=chk.checked?'1':'0.3'});
  return div;
}

const form=document.getElementById('addForm');
const formRows=document.getElementById('formRows');
for(let i=1;i<=6;i++){
  const row=document.createElement('div');row.className='sms-row';
  row.innerHTML=`<input placeholder="SMS ${i}"><input placeholder="Descripción artículo ${i}"><input placeholder="Oferta ${i}">`;
  formRows.appendChild(row);
}

document.getElementById('btnToggleForm').onclick=()=>{form.style.display=form.style.display==='block'?'none':'block'};

document.getElementById('btnAddElem').onclick=()=>{
  const tipo=document.getElementById('formTipo').value;
  const descripcion=document.getElementById('formDescripcion').value.trim();
  const nombreImagen=document.getElementById('formNombreImagen').value.trim();
  const rows=[...formRows.querySelectorAll('.sms-row')].map(r=>{const [sms,desc,off]=[...r.children].map(i=>i.value.trim());return{sms,desc,off}});
  const idx=container.children.length+1;
  const div=document.createElement('div');div.className='item';
  div.innerHTML=`<h3 style="color:#27ae60;font-weight:bold;font-size:16px;">🎯 OFERTA EXTRA ${idx}</h3>
    <label><input type="checkbox" class="includeItem" checked> Incluir</label>
    <label>Tipo:<input readonly value="${tipo}"></label>
    <input type="hidden" class="nombre" value="${nombreImagen}">
    <label>Descripción:<input type="text" class="descripcion" value="${descripcion}" placeholder="Añade texto libre"></label>
    <div class="preview-container"><img class="preview" src="${nombreImagen?`${imageBase}${nombreImagen}.png`:''}" alt="prev"></div>`+
    rows.map(r=>`<div class="sms-row"><input readonly value="${r.sms}"><input readonly value="${r.desc}"><input readonly value="${r.off}"></div>`).join('');
  const chk=div.querySelector('.includeItem');
  chk.addEventListener('change',()=>{div.style.opacity=chk.checked?'1':'0.3'});
  container.appendChild(div);
  form.style.display='none';
};

function getImageData(url){return new Promise(res=>{const img=new Image();img.crossOrigin='anonymous';img.src=url;img.onload=()=>{const c=document.createElement('canvas');c.width=img.width;c.height=img.height;c.getContext('2d').drawImage(img,0,0);res(c.toDataURL('image/png'));};img.onerror=()=>res(null);});}

document.getElementById('btnGenerar').addEventListener('click',async()=>{
  const {jsPDF}=window.jspdf;const doc=new jsPDF({unit:'pt',format:'letter'});
  const nombreTienda=document.getElementById('nombreTienda').value.trim();
  if(nombreTienda){doc.setFontSize(16);doc.text(`Tienda: ${nombreTienda}`,40,40);}
  const items=[...container.children];
  const perPage=6,cols=2,pw=doc.internal.pageSize.getWidth(),ph=doc.internal.pageSize.getHeight(),bw=(pw-80)/cols,bh=(ph-100)/3;
  let count=0;
  for(let idx=0;idx<items.length;idx++){
    const item=items[idx];
    if(!item.querySelector('.includeItem')?.checked) continue;
    if(count>0&&count%perPage===0)doc.addPage();
    const ip=count%perPage,col=ip%cols,row=Math.floor(ip/cols);
    const x=40+col*(bw+20);let y=60+row*(bh+20);
    doc.setFontSize(14);doc.text(item.querySelector('h3').textContent,x,y);y+=18;
    const tipo=item.querySelector('select')?item.querySelector('select').value:item.querySelector('input[readonly]').value;
    doc.setFontSize(12);doc.text(`Tipo: ${tipo}`,x,y);y+=16;
    const imgName=item.querySelector('.nombre')?.value;
    if(imgName){const imgDat=await getImageData(`${imageBase}${imgName}.png`);if(imgDat){const sz=Math.min(bw-20,bh-60);doc.addImage(imgDat,'PNG',x,y,sz,sz);y+=sz+10;}}
    const desc=item.querySelector('.descripcion');if(desc){const d=desc.value.trim();if(d){doc.text(d,x,y);y+=14;}}
    item.querySelectorAll('.sms-row').forEach(r=>{const vals=[...r.children].map(i=>i.value||i.textContent);if(vals.some(v=>v)){doc.text(vals.join(' - '),x,y);y+=14;}});
    count++;
  }
  doc.save('plan_comercial.pdf');
});
</script>
</body>
</html>
