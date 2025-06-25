<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plano del Plan comercial</title>
  <!-- Librerías para PDF y lienzo -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <!-- Celebración con confeti -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.1/dist/confetti.browser.min.js"></script>
  <!-- SMTPJS para envío de email -->
  <script src="https://smtpjs.com/v3/smtp.js"></script>
  <style>
    :root {
      --bg-light: linear-gradient(135deg, #eaf9ee, #009640);
      --bg-dark: #013d25;
      --text-light: #009640;
      --text-dark: #eaf9ee;
      --card-light: #fff;
      --card-dark: #0e3a27;
    }
    body {
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      padding: 10px;
      background: var(--bg-light);
      color: var(--text-light);
      min-height: 100vh;
      transition: background .5s, color .5s;
    }
    body.dark {
      background: var(--bg-dark);
      color: var(--text-dark);
    }
    /* Aseguramos que textos y inputs cambien de color en modo oscuro */
    body.dark, body.dark h1, body.dark p, body.dark label {
      color: var(--text-dark);
    }
    body.dark input, body.dark select, body.dark .sms-row input {
      background: #333;
      color: var(--text-dark);
      border-color: #555;
    }
    h1 {
      font-size: 28px;
      text-align: center;
      background: rgba(255,255,255,.9);
      color: var(--text-light);
      padding: 16px;
      border-radius: 12px;
      margin: 0 0 20px;
      position: relative;
      animation: bounceIn .8s ease;
    }
    body.dark h1 {
      background: rgba(0,0,0,.5);
      color: var(--text-dark);
    }
    h1::after {
      content: '';
      background-image: url('https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Logo_Carrefour.svg/320px-Logo_Carrefour.svg.png');
      background-size: contain;
      background-repeat: no-repeat;
      position: absolute;
      right: 20px;
      top: 12px;
      width: 50px;
      height: 50px;
      animation: rotateLogo 2s infinite linear;
    }
    @keyframes fadeIn { from{opacity:0;transform:translateY(20px);}to{opacity:1;transform:translateY(0);} }
    @keyframes bounceIn { 0%{transform:scale(0);}60%{transform:scale(1.1);}100%{transform:scale(1);} }
    @keyframes rotateLogo { from{transform:rotate(0deg);}to{transform:rotate(360deg);} }
    #inputs {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(220px,1fr));
      gap: 14px;
    }
    .item {
      background: var(--card-light);
      border-radius: 10px;
      padding: 12px;
      box-shadow: 0 2px 6px rgba(0,0,0,.2);
      transition: transform .3s, box-shadow .3s, background .5s, opacity .3s;
      animation: fadeIn .5s ease;
    }
    body.dark .item {
      background: var(--card-dark);
    }
    .item:hover {
      transform: translateY(-6px);
      box-shadow: 0 6px 18px rgba(0,0,0,.3);
    }
    .item h3 {
      margin: 0 0 8px;
      color: var(--text-light);
    }
    body.dark .item h3 {
      color: var(--text-dark);
    }
    .item.excluded {
      opacity: .3;
    }
    .controls button {
      position: relative;
      overflow: hidden;
      padding: 8px 16px;
      font-size: 14px;
      border: none;
      border-radius: 6px;
      background: #fff;
      color: var(--text-light);
      margin: 4px;
      cursor: pointer;
      transition: background .3s, color .3s;
    }
    body.dark .controls button {
      color: var(--text-dark);
    }
    .controls button:hover {
      background: var(--text-light);
      color: #fff;
    }
    .controls button.dark-toggle:hover {
      background: var(--text-dark);
    }
    button.ripple-effect:after {
      content: '';
      position: absolute;
      background: rgba(255,255,255,.5);
      border-radius: 50%;
      transform: scale(0);
      opacity: 0;
      transition: transform .5s, opacity 1s;
    }
    button.ripple-effect:active:after {
      width: 200px;
      height: 200px;
      top: calc(50% - 100px);
      left: calc(50% - 100px);
      transform: scale(1);
      opacity: 1;
      transition: 0s;
    }
    @media(max-width:768px) {
      #inputs { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <h1>Plano del Plan comercial</h1>
  <div id="tiendaField" style="text-align:center;margin-bottom:20px;">
    <input type="text" id="nombreTienda" placeholder="Introduce nombre de la tienda" style="width:260px;padding:10px;border-radius:6px;border:1px solid #ccc;text-align:center;">
  </div>
  <div id="inputs"></div>
  <div id="addForm" style="display:none;background:rgba(255,255,255,.9);padding:12px;border-radius:8px;box-shadow:0 2px 8px rgba(0,0,0,.2);margin:12px 0;">
    <h2 style="text-align:center;color:var(--text-light);">➕ Nueva Oferta</h2>
    <label>Tipo:<select id="formTipo"><option>Cross</option><option>Cabecera</option><option>Esfera</option><option>Exposición Especial</option></select></label>
    <div id="formRows"></div>
    <label>Descripción:<input type="text" id="formDescripcion" placeholder="Descripción"></label>
    <label>Imagen:<input type="file" id="formImagen" accept="image/*"></label>
    <button id="btnAddElem" class="ripple-effect">Agregar</button>
  </div>
  <div class="controls" style="text-align:center;margin:20px 0;">
    <button id="btnToggleForm" class="ripple-effect">➕ Añadir Oferta</button>
    <button id="btnGenerar" class="ripple-effect">🎉 Crear PDF</button>
    <button id="btnClear" class="ripple-effect">🗑️ Limpiar Ofertas</button>
    <button id="btnTheme" class="ripple-effect dark-toggle">🌙 Modo Oscuro</button>
  </div>
  <!-- Sección de envío por email -->
  <div id="emailSection" style="text-align:center;margin:20px 0;">
    <input type="email" id="emailTo" value="mario_cuenca_gomez@carrefour.com" readonly style="padding:8px;border-radius:4px;border:1px solid #ccc;width:250px;">
    <button id="btnEmail" class="ripple-effect">✉️ Enviar por Email</button>
  </div>
<script>
  document.addEventListener('DOMContentLoaded', () => {
    const container = document.getElementById('inputs');
    const addForm = document.getElementById('addForm');
    const formRows = document.getElementById('formRows');
    const toggleButton = document.getElementById('btnToggleForm');
    const btnAddElem = document.getElementById('btnAddElem');
    const btnClear = document.getElementById('btnClear');
    const btnTheme = document.getElementById('btnTheme');
    const btnGenerar = document.getElementById('btnGenerar');
    // Setup form rows
    for (let i = 1; i <= 6; i++) {
      const row = document.createElement('div');
      row.className = 'sms-row';
      row.innerHTML = `<input placeholder="SMS ${i}"><input placeholder="Artículo ${i}"><input placeholder="Oferta ${i}">`;
      formRows.appendChild(row);
    }
    // Function to attach include toggle
    function setupInclude(div) {
      const checkbox = div.querySelector('.includeItem');
      checkbox.addEventListener('change', () => {
        if (!checkbox.checked) div.classList.add('excluded');
        else div.classList.remove('excluded');
      });
    }
    // Add top offers
    function addTopOffers() {
      container.innerHTML = ''; 
      for (let i = 1; i <= 4; i++) {
        const box = document.createElement('div');
        box.className = 'item';
        box.innerHTML = `
          <h3>🎯 OFERTA TOP ${i}</h3>
          <label><input type="checkbox" class="includeItem" checked> Incluir</label>
          <label>Tipo:<select class="plantilla"><option>Cross</option><option>Cabecera</option><option>Esfera</option><option>Exposición Especial</option></select></label>
          <label>Descripción:<input type="text" class="descripcion" placeholder="Añade texto libre"></label>
          <div class="preview-container"><img src="./nombre${i}.png" alt="prev"></div>
        `;
        container.appendChild(box);
        setupInclude(box);
      }
    }
    addTopOffers();
    // Toggle form
    toggleButton.addEventListener('click', () => addForm.style.display = addForm.style.display==='block'?'none':'block');
    // Add extra offer
    btnAddElem.addEventListener('click', () => {
      const tipo = document.getElementById('formTipo').value;
      const desc = document.getElementById('formDescripcion').value;
      const imgFile = document.getElementById('formImagen').files[0];
      const div = document.createElement('div'); div.className = 'item';
      div.innerHTML = `
        <h3>🎯 OFERTA EXTRA</h3>
        <label><input type="checkbox" class="includeItem" checked> Incluir</label>
        <p><strong>Tipo:</strong> ${tipo}</p>
        <p>${desc}</p>
        <div class='preview-container'></div>
      `;
      if (imgFile) {
        const reader = new FileReader(); reader.onload = () => { const img = document.createElement('img'); img.src = reader.result; div.querySelector('.preview-container').appendChild(img); };
        reader.readAsDataURL(imgFile);
      }
      container.appendChild(div);
      setupInclude(div);
      addForm.style.display='none';
    });
    // Clear offers
    btnClear.addEventListener('click', addTopOffers);
    // Toggle theme
    btnTheme.addEventListener('click', () => document.body.classList.toggle('dark'));
    // Generate PDF
    btnGenerar.addEventListener('click', () => {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF({unit:'pt',format:'a4'});
      const tienda = document.getElementById('nombreTienda').value.trim()||'Plan';
      doc.html(container,{callback:()=>{doc.save(`${tienda}.pdf`);confetti({particleCount:150,spread:60});},x:20,y:40,html2canvas:{scale:0.7}});
    });
    // Send by Email
    const btnEmail = document.getElementById('btnEmail');
    btnEmail.addEventListener('click', () => {
      const toEmail = document.getElementById('emailTo').value.trim();
      if (!toEmail) { alert('Por favor, introduce un email válido.'); return; }
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF({unit:'pt',format:'a4'});
      doc.html(container,{callback:() => {
          const tienda = document.getElementById('nombreTienda').value.trim()||'Plan';
          const pdfData = doc.output('datauristring');
          Email.send({
            SecureToken: "TU_SECURE_TOKEN_AQUI",
            To: "mario_cuenca_gomez@carrefour.com",
            From: "tu@empresa.com",}`,
            Body: "Adjunto encontrarás el plan comercial en PDF.",
            Attachments: [{ name: `${tienda}.pdf`, data: pdfData }]
          }).then(msg => { console.log('SMTPJS response:', msg); alert('Email enviado correctamente.'); })
            .catch(err => { console.error('Error SMTPJS:', err); alert('Error al enviar email. Revisa la consola.'); });
        },x:20,y:40,html2canvas:{scale:0.7}});
    });
  });
</script>
</body>
</html>
