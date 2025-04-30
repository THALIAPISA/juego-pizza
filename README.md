<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Juego: Arma tu Pizza</title>
  <style>
    body {
      font-family: sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
      background: #ffe5b4;
    }

    #nivel-info {
      margin-bottom: 15px;
      font-size: 20px;
      font-weight: bold;
    }

    #temporizador {
      font-size: 18px;
      color: #444;
      margin-bottom: 10px;
    }

    #juego-container {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: flex-start;
      gap: 20px;
    }

    #ingredientes {
      width: 150px;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    .ingrediente {
      width: 100px;
      padding: 10px;
      background: #fff;
      border: 2px solid #ccc;
      border-radius: 8px;
      text-align: center;
      cursor: grab;
    }

    #pizza {
      width: 400px;
      height: 400px;
      background: url('https://i.imgur.com/oHgDZT3.png') no-repeat center/cover;
      border-radius: 50%;
      position: relative;
      border: 4px solid #a0522d;
      transition: transform 1s ease-in-out, background-image 0.5s ease-in;
    }

    .animar-horno {
      transform: translateX(300px) scale(0.5);
      opacity: 0.5;
    }

    .pizza-cocinada {
      background-image: url('https://i.imgur.com/LQ7IxGW.png') !important;
      opacity: 1 !important;
      transform: none !important;
    }

    .ingrediente-colocado {
      position: absolute;
      width: 60px;
      user-select: none;
      pointer-events: none;
    }

    #mensaje-final {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: #fff3cd;
      border: 2px solid #ffeeba;
      padding: 20px;
      border-radius: 12px;
      font-size: 24px;
      display: none;
      z-index: 10;
      text-align: center;
    }

    #mensaje-final button {
      margin-top: 15px;
      padding: 8px 16px;
      font-size: 18px;
      background-color: #f5a623;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      color: white;
    }

    #horno {
      position: fixed;
      bottom: 20px;
      right: 20px;
      background: url('https://i.imgur.com/fGpY5bd.png') no-repeat center/contain;
      width: 120px;
      height: 120px;
      background-size: contain;
    }
  </style>
</head>
<body>
  <div id="nivel-info">Nivel 1: Pizza Clásica (5 ingredientes)</div>
  <div id="temporizador">Tiempo: 0s</div>
  <div id="juego-container">
    <div id="ingredientes"></div>
    <div id="pizza"></div>
  </div>
  <div id="mensaje-final">
    ¡Felicidades! ¡Tu pizza está lista! 🍕🔥<br>
    <div id="resultado-tiempo"></div>
    <button onclick="reiniciarJuego()">Siguiente nivel</button>
  </div>
  <div id="horno"></div>

  <audio id="sonido-drop" src="https://www.fesliyanstudios.com/play-mp3/387" preload="auto"></audio>
  <audio id="sonido-final" src="https://www.fesliyanstudios.com/play-mp3/6661" preload="auto"></audio>

  <script>
    const niveles = [
      {
        nombre: "Pizza Clásica",
        total: 5,
        ingredientes: [
          { nombre: "Queso", img: "https://i.imgur.com/h0JwA0G.png" },
          { nombre: "Tomates", img: "https://i.imgur.com/gcE7Y9S.png" },
          { nombre: "Peperoni", img: "https://i.imgur.com/8PH3wvA.png" },
          { nombre: "Aceite", img: "https://i.imgur.com/VzbUePg.png" },
          { nombre: "Harina", img: "https://i.imgur.com/Z0gK2ZL.png" },
        ]
      },
      {
        nombre: "Pizza Verde",
        total: 4,
        ingredientes: [
          { nombre: "Espinaca", img: "https://i.imgur.com/FdpjWzQ.png" },
          { nombre: "Albahaca", img: "https://i.imgur.com/OzUjLoB.png" },
          { nombre: "Aguacate", img: "https://i.imgur.com/EJ2CyMh.png" },
          { nombre: "Queso", img: "https://i.imgur.com/h0JwA0G.png" },
        ]
      }
    ];

    let nivelActual = 0;
    let ingredientesAgregados = 0;
    let tiempo = 0;
    let temporizadorIntervalo;

    const pizza = document.getElementById('pizza');
    const ingredientesDiv = document.getElementById('ingredientes');
    const mensajeFinal = document.getElementById('mensaje-final');
    const sonidoDrop = document.getElementById('sonido-drop');
    const sonidoFinal = document.getElementById('sonido-final');
    const nivelInfo = document.getElementById('nivel-info');
    const temporizador = document.getElementById('temporizador');
    const resultadoTiempo = document.getElementById('resultado-tiempo');

    function iniciarTemporizador() {
      tiempo = 0;
      temporizador.textContent = `Tiempo: 0s`;
      temporizadorIntervalo = setInterval(() => {
        tiempo++;
        temporizador.textContent = `Tiempo: ${tiempo}s`;
      }, 1000);
    }

    function detenerTemporizador() {
      clearInterval(temporizadorIntervalo);
    }

    function cargarNivel() {
      const nivel = niveles[nivelActual];
      nivelInfo.textContent = `Nivel ${nivelActual + 1}: ${nivel.nombre} (${nivel.total} ingredientes)`;
      ingredientesDiv.innerHTML = '';
      pizza.innerHTML = '';
      pizza.classList.remove('pizza-cocinada');

      nivel.ingredientes.forEach(i => {
        const div = document.createElement('div');
        div.className = 'ingrediente';
        div.textContent = i.nombre;
        div.setAttribute('draggable', true);
        div.dataset.img = i.img;
        div.addEventListener('dragstart', e => {
          e.dataTransfer.setData('text/plain', i.img);
        });
        ingredientesDiv.appendChild(div);
      });

      ingredientesAgregados = 0;
      mensajeFinal.style.display = 'none';
      iniciarTemporizador();
    }

    pizza.addEventListener('dragover', e => e.preventDefault());

    pizza.addEventListener('drop', e => {
      e.preventDefault();
      const imgURL = e.dataTransfer.getData('text/plain');
      const img = document.createElement('img');
      img.src = imgURL;
      img.className = 'ingrediente-colocado';
      const rect = pizza.getBoundingClientRect();
      img.style.left = (e.clientX - rect.left - 30) + 'px';
      img.style.top = (e.clientY - rect.top - 30) + 'px';
      pizza.appendChild(img);

      sonidoDrop.currentTime = 0;
      sonidoDrop.play();

      ingredientesAgregados++;
      if (ingredientesAgregados >= niveles[nivelActual].total) {
        detenerTemporizador();
        setTimeout(() => {
          pizza.classList.add('animar-horno');
          setTimeout(() => {
            pizza.classList.remove('animar-horno');
            pizza.classList.add('pizza-cocinada');
            resultadoTiempo.textContent = `¡Tiempo usado: ${tiempo} segundos!`;
            mensajeFinal.style.display = 'block';
            sonidoFinal.play();
          }, 1500);
        }, 500);
      }
    });

    function reiniciarJuego() {
      nivelActual++;
      if (nivelActual >= niveles.length) nivelActual = 0;
      cargarNivel();
    }

    cargarNivel();
  </script>
</body>
</html>
