<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plan Comercial Carrefour</title>
  <!-- Importamos jsPDF para generar el PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <!-- Importamos html2canvas para convertir el HTML en lienzo -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>

  <style>
    /* Estilos base del body */
    body {
      font-family: Arial, sans-serif;
      padding: 10px;
      margin: 0;
      background: #f5f5f5;
    }

    /* Título principal con fondo azul y sombra */
    h1 {
      font-size: 26px;
      text-align: center;
      background: #005aa7;
      color: #fff;
      padding: 16px 12px;
      border-radius: 10px;
      margin: 0 0 20px;
      letter-spacing: 1px;
      box-shadow: 0 2px 6px rgba(0, 0, 0, .2);
      text-shadow: 1px 1px 2px rgba(0,0,0,.3);
      position: relative;
    }
    /* Logo de Carrefour posicionado dentro del h1 */
    h1::after {
      content: '';
      background-image: url('https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Logo_Carrefour.svg/320px-Logo_Carrefour.svg.png');
      background-size: contain;
      background-repeat: no-repeat;
      position: absolute;
      right: 16px;
      top: 10px;
      width: 40px;
      height: 40px;
    }

    /* Grid contenedor para los items dinámicos */
    #inputs {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 10px;
    }

    /* Estilo de cada tarjeta de oferta */
    .item {
      border: 1px solid #ccc;
      padding: 8px;
      background: #fff;
      transition: opacity .2s;
      border-radius: 6px;
      animation: fadeIn 0.4s ease;
    }
    .item h3 {
      margin: 0 0 6px;
      font-size: 15px;
      color: #e4002b;
    }
    .item label {
      display: block;
      font-size: 12px;
      margin: 4px 0;
    }
    .item select, .item input {
      width: 100%;
      padding: 4px;
      font-size: 12px;
      box-sizing: border-box;
      border-radius: 4px;
    }
    /* Contenedor de previsualización de imagen */
    .preview-container {
      margin-top: 6px;
    }
    .preview-container img {
      width: 100%;
      border: 1px solid #ddd;
      border-radius: 4px;
    }

    /* Filas para SMS, descripción y oferta en el formulario */
    .sms-row {
      display: grid;
      grid-template-columns: 1fr 1.5fr 1fr;
      gap: 4px;
      margin-bottom: 4px;
    }

    /* Formulario oculto inicialmente para añadir una oferta */
    #addForm {
      display: none;
      border: 1px solid #007bff;
      padding: 10px;
      margin-top: 15px;
      background: #eaf6ff;
      border-radius: 8px;
      animation: fadeIn 0.3s ease;
    }
    #addForm h2 {
      font-size: 14px;
      margin: 0 0 6px;
      text-align: center;
      color: #005aa7;
    }
    /* Botones inferiores */
    #bottomButtons {
      text-align: center;
      margin: 20px 0;
    }
    button {
      padding: 6px 14px;
      font-size: 13px;
      cursor: pointer;
      border: none;
      border-radius: 6px;
      background: #e4002b;
      color: #fff;
      transition: background 0.3s ease;
    }
    button:hover {
      background: #b3001f;
    }

    /* Campo para el nombre de la tienda */
    #tiendaField {
      text-align: center;
      margin: 10px 0;
    }
    #tiendaField input {
      width: 220px;
      padding: 8px;
      font-size: 15px;
      text-align: center;
      border: 1px solid #ccc;
      border-radius: 6px;
    }

    /* Adaptación para pantallas pequeñas */
    @media(max-width:768px) {
      #inputs { grid-template-columns: 1fr; }
    }

    /* Animación de aparición */
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }
  </style>
</head>
<body>
  <!-- Título de la página -->
  <h1>Plan Comercial Carrefour</h1>

  <!-- Campo para introducir el nombre de la tienda -->
  <div id="tiendaField">
    <label for="nombreTienda"><strong>Nombre de la tienda:</strong></label><br>
    <input type="text" id="nombreTienda" placeholder="Introduce nombre de la tienda">
  </div>

  <!-- Contenedor donde se insertan dinámicamente las ofertas -->
  <div id="inputs"></div>

  <!-- Formulario para añadir una nueva oferta -->
  <div id="addForm">
    <h2>Nuevo elemento</h2>
    <label>Tipo:
      <select id="formTipo">
        <option>Cross</option>
        <option>Cabecera</option>
        <option>Esfera </option>
        <option>Exposición Especial</option>
      </select>
    </label>
    <!-- Aquí se generan las filas de SMS/descripción/oferta -->
    <div id="formRows"></div>
    <label>Descripción:
      <input type="text" id="formDescripcion" placeholder="Descripción del elemento">
    </label>
    <label>Imagen (opcional):
      <input type="file" id="formImagen" accept="image/*">
    </label>
    <button id="btnAddElem">Agregar al listado</button>
  </div>

  <!-- Botones para mostrar el formulario y generar el PDF -->
  <div id="bottomButtons">
    <button id="btnToggleForm">➕ Añadir Oferta</button>
    <button id="btnGenerar">Crear PDF</button>
  </div>

  <script>
    // Referencias a elementos clave del DOM
    const container = document.getElementById('inputs');
    const addForm = document.getElementById('addForm');
    const formRows = document.getElementById('formRows');
    const toggleButton = document.getElementById('btnToggleForm');
    const btnAddElem = document.getElementById('btnAddElem');

    // Generar automáticamente 6 filas para el formulario de SMS/descripción/oferta
    for (let i = 1; i <= 6; i++) {
      const row = document.createElement('div');
      row.className = 'sms-row';
      row.innerHTML = `
        <input placeholder="SMS ${i}">
        <input placeholder="Descripción artículo ${i}">
        <input placeholder="Oferta ${i}">
      `;
      formRows.appendChild(row);
    }

    // Mostrar u ocultar el formulario al clicar ➕ Añadir Oferta
    toggleButton.onclick = () => {
      addForm.style.display = addForm.style.display === 'block' ? 'none' : 'block';
      if (addForm.style.display === 'block') {
        addForm.scrollIntoView({ behavior: 'smooth', block: 'end' });
      }
    };

    // Al hacer clic en "Agregar al listado", creamos una nueva tarjeta .item
    btnAddElem.onclick = () => {
      const tipo = document.getElementById('formTipo').value;
      const descripcion = document.getElementById('formDescripcion').value;
      const imgFile = document.getElementById('formImagen').files[0];

      // Recogemos los valores de cada fila de SMS/descripción/oferta
      const rows = [...formRows.querySelectorAll('.sms-row')].map(r => {
        const [sms, desc, off] = [...r.children].map(i => i.value.trim());
        return { sms, desc, off };
      });

      // Creamos el contenedor .item
      const div = document.createElement('div');
      div.className = 'item';
      div.innerHTML = `
        <h3>🎯 OFERTA EXTRA</h3>
        <label><input type="checkbox" class="includeItem" checked> Incluir</label>
        <label>Tipo: <input readonly value="${tipo}"></label>
        <label>Descripción: <input type="text" value="${descripcion}" class="descripcion"></label>
        <div class="preview-container"></div>
        ${rows.map(r => `
          <div class="sms-row">
            <input readonly value="${r.sms}">
            <input readonly value="${r.desc}">
            <input readonly value="${r.off}">
          </div>
        `).join('')}
      `;

      // Si el usuario subió una imagen, la mostramos en la previsualización
      if (imgFile) {
        const reader = new FileReader();
        reader.onload = () => {
          const imgTag = document.createElement('img');
          imgTag.src = reader.result;
          div.querySelector('.preview-container').appendChild(imgTag);
        };
        reader.readAsDataURL(imgFile);
      }

      container.appendChild(div);
      // Ocultamos de nuevo el formulario para ahorrar espacio
      addForm.style.display = 'none';
    };

    // Creamos 4 ofertas "TOP" preexistentes al cargar la página
    for (let i = 1; i <= 4; i++) {
      const div = document.createElement('div');
      div.className = 'item';
      div.innerHTML = `
        <h3>🎯 OFERTA TOP ${i}</h3>
        <label><input type="checkbox" class="includeItem" checked> Incluir</label>
        <label>Tipo:
          <select class="plantilla">
            <option>Cross</option>
            <option>Cabecera</option>
            <option>Esfera o Exposición Especial</option>
          </select>
        </label>
        <label>Descripción:
          <input type="text" class="descripcion" placeholder="Añade texto libre">
        </label>
        <div class="preview-container">
          <img src="./nombre${i}.png" alt="prev">
        </div>
      `;
      container.appendChild(div);
    }

    // Al hacer clic en "Crear PDF", generamos el documento y lo descargamos
    document.getElementById('btnGenerar').onclick = () => {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF({ orientation: 'portrait', unit: 'pt', format: 'a4' });
      // Obtenemos el nombre de la tienda (o usamos "plan" si está vacío)
      const tienda = document.getElementById('nombreTienda').value.trim() || 'plan';

      // Convertimos el contenedor #inputs a canvas y lo añadimos al PDF
      doc.html(container, {
        callback: function (doc) {
          doc.save(`${tienda}.pdf`); // Descarga automática del PDF
        },
        x: 20,
        y: 40,
        html2canvas: { scale: 0.6 }
      });
    };
  </script>
</body>
</html>
