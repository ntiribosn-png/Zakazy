<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Мои заказы</title>

  <style>
    body {
      font-family: Arial;
      padding: 20px;
      background: #f5f5f5;
    }

    input, textarea, button {
      width: 100%;
      margin-top: 10px;
      padding: 10px;
      font-size: 16px;
      box-sizing: border-box;
    }

    canvas {
      border: 2px solid black;
      margin-top: 10px;
      background: white;
      touch-action: none;
    }

    .order {
      background: white;
      padding: 12px;
      margin-top: 15px;
      border-radius: 10px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
    }

    #app { display: none; }
  </style>
</head>
<body>

<!-- Экран входа -->
<div id="login">
  <h2>Введите пароль</h2>
  <input type="password" id="password" placeholder="Пароль">
  <button onclick="checkPassword()">Войти</button>
</div>

<!-- Основное приложение -->
<div id="app">
  <h2>Добавить заказ</h2>

  <input id="name" placeholder="Имя клиента">
  <input id="address" placeholder="Адрес">
  <textarea id="desc" placeholder="Описание заказа"></textarea>

  <canvas id="canvas" width="300" height="200"></canvas>

  <button onclick="saveOrder()">Сохранить заказ</button>

  <h2>Список заказов</h2>
  <div id="orders"></div>
</div>

<script>
/* ====== ПАРОЛЬ ====== */
const SITE_PASSWORD = "1234"; // ← можешь поменять

function checkPassword() {
  const pass = document.getElementById("password").value;

  if (pass === SITE_PASSWORD) {
    document.getElementById("login").style.display = "none";
    document.getElementById("app").style.display = "block";
    showOrders();
  } else {
    alert("Неверный пароль");
  }
}

/* ====== РИСОВАНИЕ ====== */
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
let drawing = false;

canvas.addEventListener("mousedown", () => drawing = true);
canvas.addEventListener("mouseup", () => { drawing = false; ctx.beginPath(); });
canvas.addEventListener("mousemove", draw);

canvas.addEventListener("touchstart", () => drawing = true);
canvas.addEventListener("touchend", () => { drawing = false; ctx.beginPath(); });
canvas.addEventListener("touchmove", drawTouch);

function draw(e) {
  if (!drawing) return;

  ctx.lineWidth = 2;
  ctx.lineCap = "round";
  ctx.lineTo(e.offsetX, e.offsetY);
  ctx.stroke();
  ctx.beginPath();
  ctx.moveTo(e.offsetX, e.offsetY);
}

function drawTouch(e) {
  if (!drawing) return;

  e.preventDefault();
  const rect = canvas.getBoundingClientRect();
  const x = e.touches[0].clientX - rect.left;
  const y = e.touches[0].clientY - rect.top;

  ctx.lineWidth = 2;
  ctx.lineCap = "round";
  ctx.lineTo(x, y);
  ctx.stroke();
  ctx.beginPath();
  ctx.moveTo(x, y);
}

/* ====== СОХРАНЕНИЕ ЗАКАЗА ====== */
function saveOrder() {
  const name = document.getElementById("name").value;
  const address = document.getElementById("address").value;
  const desc = document.getElementById("desc").value;
  const image = canvas.toDataURL();
  const date = new Date().toLocaleString(); // дата и время

  if (!name) {
    alert("Введите имя клиента");
    return;
  }

  const order = { name, address, desc, image, date };

  const orders = JSON.parse(localStorage.getItem("orders") || "[]");
  orders.push(order);
  localStorage.setItem("orders", JSON.stringify(orders));

  showOrders();

  // очистка формы
  document.getElementById("name").value = "";
  document.getElementById("address").value = "";
  document.getElementById("desc").value = "";
  ctx.clearRect(0, 0, canvas.width, canvas.height);
}

/* ====== ПОКАЗ ЗАКАЗОВ ====== */
function showOrders() {
  const orders = JSON.parse(localStorage.getItem("orders") || "[]");
  const div = document.getElementById("orders");

  div.innerHTML = "";

  orders.reverse().forEach(o => {
    div.innerHTML += `
      <div class="order">
        <b>${o.name}</b> — ${o.date}<br><br>
        <b>Адрес:</b> ${o.address || "-"}<br>
        <b>Описание:</b> ${o.desc || "-"}<br><br>
        <img src="${o.image}" width="200">
      </div>
    `;
  });
}
</script>

</body>
</html>
