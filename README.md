<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Globe ERP Oficina Elétrica</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{font-family:Arial;margin:0;background:#f4f6f9}
header{background:#111827;color:#fff;padding:15px;font-size:18px}
.container{padding:15px}

.tabs button{
padding:10px;margin:5px;border:0;cursor:pointer;
background:#e5e7eb;border-radius:6px
}

.card{
background:#fff;padding:15px;margin:10px 0;
border-radius:10px;box-shadow:0 2px 6px rgba(0,0,0,.1)
}

input,select{
padding:8px;margin:5px 0;width:100%;
border-radius:6px;border:1px solid #ccc
}

button{
padding:8px;background:#2563eb;color:#fff;
border:0;border-radius:6px;cursor:pointer
}

.progress{height:10px;background:#ddd;border-radius:10px;overflow:hidden}
.bar{height:100%;background:green}

.alert-green{color:green;font-weight:bold}
.alert-orange{color:orange;font-weight:bold}
.alert-red{color:red;font-weight:bold}

.small{font-size:12px;color:#666}
</style>
</head>

<body>

<header>⚡ Globe ERP Oficina Elétrica</header>

<div class="container">

<!-- ABA BOTOES -->
<div class="tabs">
<button onclick="tab('dash')">Dashboard</button>
<button onclick="tab('cars')">Veículos</button>
<button onclick="tab('hours')">Horas</button>
</div>

<!-- DASHBOARD -->
<div id="dash" class="card">

<h3>📊 Produtividade Geral</h3>
<div id="chart"></div>

<hr>

<h3>🚨 Alertas Inteligentes</h3>
<div id="alerts"></div>

</div>

<!-- VEICULOS -->
<div id="cars" class="card" style="display:none">

<h3>🚐 Cadastro de Veículos</h3>

<input id="cliente" placeholder="Cliente">
<input id="modelo" placeholder="Modelo">
<input id="os" placeholder="OS">
<input id="entrada" type="date">
<input id="prev" type="date">

<button onclick="addCar()">Salvar Veículo</button>

<hr>

<input id="search" placeholder="Buscar..." oninput="renderCars()">

<div id="carsList"></div>

</div>

<!-- HORAS -->
<div id="hours" class="card" style="display:none">

<h3>👨‍🔧 Controle de Horas</h3>

<select id="worker">
<option>João</option>
<option>Pedro</option>
<option>Carlos</option>
</select>

<input id="hoursValue" type="number" placeholder="Horas trabalhadas">
<input id="task" placeholder="OS ou Veículo">

<button onclick="addHours()">Salvar Horas</button>

<hr>

<div id="hoursList"></div>

</div>

</div>

<script>

// ================= STORAGE =================
let cars = JSON.parse(localStorage.getItem("cars")||"[]");
let logs = JSON.parse(localStorage.getItem("logs")||"[]");

function save(){
localStorage.setItem("cars",JSON.stringify(cars));
localStorage.setItem("logs",JSON.stringify(logs));
}

// ================= NAV =================
function tab(t){
dash.style.display="none";
cars.style.display="none";
hours.style.display="none";

document.getElementById(t).style.display="block";

if(t==="cars")renderCars();
if(t==="dash")updateDashboard();
if(t==="hours")renderHours();
}

// ================= CHECKLIST BASE =================
const checklist = {
CA:["Iluminação","Tomadas"],
CC:["Iluminação CC","Aquecedor","Placa solar","Ar"],
BAT:["Inversor","Cabos","DCDC"],
LIG:["Lâmpadas","Teste"],
CENT:["Fusíveis","Quadro"],
PAINEL:["Botões","Teste final"]
};

// ================= VEICULOS =================
function addCar(){

if(!cliente.value || !modelo.value || !os.value){
alert("Preencha todos os campos");
return;
}

// anti duplicação
let dup = cars.find(c => c.os === os.value);
if(dup){
alert("Já existe este veículo!");
return;
}

cars.push({
id:Date.now(),
cliente:cliente.value,
modelo:modelo.value,
os:os.value,
entrada:entrada.value,
prev:prev.value,
check:{}
});

save();
renderCars();
}

// ================= PROGRESS =================
function progress(c){

let total=0,done=0;

Object.values(checklist).forEach(f=>{
total+=f.length;
f.forEach(i=>{
if(c.check[i])done++;
});
});

return total?Math.round((done/total)*100):0;
}

// ================= ALERTAS =================
function alertStatus(c){

let p = progress(c);
let today = new Date();
let prev = new Date(c.prev);

if(p<30 && today>prev) return "🔴 CRÍTICO";
if(p<70 && today>prev) return "🟠 ATRASO";
if(p<70) return "🟡 EM ANDAMENTO";
return "🟢 NO PRAZO";
}

// ================= RENDER CARROS =================
function renderCars(){

let s = search.value?.toLowerCase()||"";
carsList.innerHTML="";

cars
.filter(c=>c.cliente.toLowerCase().includes(s)||c.modelo.toLowerCase().includes(s))
.forEach(c=>{

carsList.innerHTML+=`
<div class="card">
<b>${c.cliente} - ${c.modelo}</b><br>
OS: ${c.os}<br>

<span class="small">${alertStatus(c)}</span>

<div class="progress">
<div class="bar" style="width:${progress(c)}%"></div>
</div>

${progress(c)}%

</div>
`;
});
}

// ================= HORAS =================
function addHours(){

logs.push({
worker:worker.value,
hours:Number(hoursValue.value),
task:task.value,
date:new Date()
});

save();
renderHours();
updateDashboard();
}

// ================= RENDER HORAS =================
function renderHours(){

hoursList.innerHTML="";

logs.slice(-10).forEach(l=>{
hoursList.innerHTML+=`
<div class="small">
${l.worker} - ${l.hours}h - ${l.task}
</div>
`;
});
}

// ================= PRODUTIVIDADE =================
function productivity(){

let map={};

logs.forEach(l=>{
map[l.worker]=(map[l.worker]||0)+l.hours;
});

return map;
}

// ================= ALERTAS =================
function alerts(){

let p = productivity();
let out=[];

for(let w in p){
if(p[w]<20) out.push(`🔴 ${w} baixa produtividade`);
else if(p[w]<40) out.push(`🟠 ${w} média produtividade`);
else out.push(`🟢 ${w} alta produtividade`);
}

return out;
}

// ================= DASHBOARD =================
function updateDashboard(){

// gráficos simples
let data = productivity();

chart.innerHTML="";

for(let w in data){

let bar = Math.min(data[w]*2,100);

chart.innerHTML+=`
<div>
<b>${w}</b> - ${data[w]}h
<div class="progress">
<div class="bar" style="width:${bar}%"></div>
</div>
</div>
`;
}

// alerts
alertsBox.innerHTML = alerts().map(a=>{
let color =
a.includes("🟢")?"alert-green":
a.includes("🟠")?"alert-orange":"alert-red";

return `<div class="${color}">${a}</div>`;
}).join("");
}

// ================= INIT =================
updateDashboard();

</script>

</body>
</html>
