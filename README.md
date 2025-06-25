<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plano del Plan comercial</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.1/dist/confetti.browser.min.js"></script>
  <script src="https://smtpjs.com/v3/smtp.js"></script>
  <style>
    :root { --bg-light: linear-gradient(135deg,#eaf9ee,#009640); --bg-dark:#013d25; --text-light:#009640; --text-dark:#eaf9ee; --card-light:#fff; --card-dark:#0e3a27; }
    body { font-family:'Segoe UI',sans-serif; margin:0; padding:10px; background:var(--bg-light); color:var(--text-light); transition:background .5s,color .5s; }
    body.dark { background:var(--bg-dark); color:var(--text-dark); }
    h1 { position:relative; text-align:center; font-size:28px; background:rgba(255,255,255,.9); color:var(--text-light); padding:16px; margin:0 0 20px; border-radius:12px; }
    body.dark h1 { background:rgba(0,0,0,.5); color:var(--text-dark); }
    h1 img { position:absolute; left:16px; top:50%; transform:translateY(-50%); height:40px; }
    #tiendaField {
      text-align: center;
      margin-bottom: 20px;
    }
    /* Input más grande para nombre de tienda */
    #nombreTienda {
      width: 80%;
      max-width: 500px;
      padding: 12px 16px;
      font-size: 16px;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
    #inputs { display:grid; grid-template-columns:repeat(auto-fit,minmax(220px,1fr)); gap:14px; margin-bottom:20px; }
    .item { background:var(--card-light); border-radius:10px; padding:12px; box-shadow:0 2px 6px rgba(0,0,0,.2); transition:opacity .3s; }
    .item.excluded { opacity:.3; }
    .item h3 { margin:0 0 8px; color:var(--text-light); }
    body.dark .item { background:var(--card-dark); }
    body.dark .item h3 { color:var(--text-dark); }
    .sms-row { display:grid; grid-template-columns:1fr 1.5fr 1fr; gap:4px; margin-bottom:4px; }
    .controls, #emailSection { text-align:center; margin:10px 0; }
    button { padding:8px 16px; margin:4px; border:none; border-radius:6px; background:#fff; color:var(--text-light); cursor:pointer; }
    body.dark button { color:var(--text-dark); }
    /* Hover effects para botones */
    button:hover {
      background: var(--text-light);
      color: #fff;
    }
    body.dark button:hover {
      background: var(--text-dark);
      color: #000;
    }
    @media(max-width:768px){ #inputs{grid-template-columns:1fr;} }
  </style>
</head>
<body>
  <h1><img src="carrefourespress.png">Plano del Plan comercial</h1>
  <div id="tiendaField"><input type="text" id="nombreTienda" placeholder="Introduce nombre de la tienda"></div>
  <div id="inputs"></div>
  <div id="addForm" style="display:none; background:rgba(255,255,255,.9); padding:12px; border-radius:8px;">
    <h2>Nueva Oferta</h2>
    <select id="formTipo"><option>Cross</option><option>Cabecera</option><option>Esfera</option><option>Exposición Especial</option></select>
    <div id="formRows"></div>
    <input type="text" id="formDescripcion" placeholder="Descripción">
    <input type="file" id="formImagen">
    <button id="btnAddElem">Agregar</button>
  </div>
  <div class="controls">
    <button id="btnToggleForm">Añadir Oferta</button>
    <button id="btnClear">Limpiar Ofertas</button>
    <button id="btnTheme">Modo Oscuro</button>
    <button id="btnGenerar">Crear PDF</button>
  </div>
  <div id="emailSection"><input type="email" id="emailTo" value="mario_cuenca_gomez@carrefour.com" readonly><button id="btnEmail">Enviar por Email</button></div>
<script>
document.addEventListener('DOMContentLoaded',()=>{
  const container=document.getElementById('inputs'),addForm=document.getElementById('addForm'),formRows=document.getElementById('formRows');
  const btnToggleForm=document.getElementById('btnToggleForm'),btnAddElem=document.getElementById('btnAddElem');
  const btnClear=document.getElementById('btnClear'),btnTheme=document.getElementById('btnTheme');
  const btnGenerar=document.getElementById('btnGenerar'),btnEmail=document.getElementById('btnEmail');
  // Formular filas
  for(let i=1;i<=6;i++){const row=document.createElement('div');row.className='sms-row';row.innerHTML=`<input placeholder="SMS ${i}"><input placeholder="Artículo ${i}"><input placeholder="Oferta ${i}">`;formRows.appendChild(row);}  
  // Toggle incluir
  const setupInclude=div=>{const cb=div.querySelector('.includeItem');cb.addEventListener('change',()=>div.classList.toggle('excluded',!cb.checked));};
  // Añadir TOP
  function addTopOffers(){container.innerHTML='';for(let i=1;i<=4;i++){const box=document.createElement('div');box.className='item';box.innerHTML=`<h3>🎯 OFERTA TOP ${i}</h3><label><input type="checkbox" class="includeItem" checked> Incluir</label><label>Tipo:<select class="plantilla"><option>Cross</option><option>Cabecera</option><option>Esfera</option><option>Exposición Especial</option></select></label><label>Descripción:<input type="text" class="descripcion" placeholder="Añade texto libre"></label><div class="preview-container"><img src="./nombre${i}.png" alt="preview oferta"></div>`;container.appendChild(box);setupInclude(box);} }
  addTopOffers();
  btnToggleForm.addEventListener('click',()=>addForm.style.display=addForm.style.display==='block'?'none':'block');
  btnAddElem.addEventListener('click',()=>{const tipo=document.getElementById('formTipo').value,desc=document.getElementById('formDescripcion').value,imgFile=document.getElementById('formImagen').files[0];const div=document.createElement('div');div.className='item';div.innerHTML=`<h3>🎯 OFERTA EXTRA</h3><label><input type="checkbox" class="includeItem" checked> Incluir</label><p><strong>Tipo:</strong> ${tipo}</p><p>${desc}</p><div class="preview-container"></div>`;if(imgFile){const r=new FileReader();r.onload=()=>{const img=document.createElement('img');img.src=r.result;div.querySelector('.preview-container').appendChild(img);};r.readAsDataURL(imgFile);}container.appendChild(div);setupInclude(div);addForm.style.display='none';});
  btnClear.addEventListener('click',addTopOffers);
  btnTheme.addEventListener('click',()=>document.body.classList.toggle('dark'));
  btnGenerar.addEventListener('click',()=>{const{jsPDF}=window.jspdf;const doc=new jsPDF({unit:'pt',format:'a4'});const tienda=document.getElementById('nombreTienda').value.trim()||'plan';doc.html(container,{callback:()=>{doc.save(`${tienda}.pdf`);confetti({particleCount:150,spread:60});},x:20,y:40,html2canvas:{scale:0.7}});});
  btnEmail.addEventListener('click',()=>{const to='mario_cuenca_gomez@carrefour.com';const{jsPDF}=window.jspdf;const doc=new jsPDF({unit:'pt',format:'a4'});doc.html(container,{callback:()=>{const data=doc.output('datauristring');Email.send({SecureToken:'TU_SECURE_TOKEN_AQUI',To:to,From:'tu@empresa.com',Subject:'Plano del Plan comercial',Body:'Adjunto PDF',Attachments:[{name:'plan.pdf',data:data}]}).then(()=>alert('Email enviado')).catch(e=>alert('Error '+e));},x:20,y:40,html2canvas:{scale:0.7}});});
});
</script>
</body>
</html>
