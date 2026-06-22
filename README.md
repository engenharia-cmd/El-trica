<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Elétrica Globe ERP</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    margin:0;
    font-family:Arial;
    background:#f4f6f9;
}

header{
    background:#111827;
    color:#fff;
    padding:15px;
    font-size:20px;
}

.menu{
    display:flex;
    background:#1f2937;
}

.menu button{
    flex:1;
    padding:12px;
    border:none;
    background:#1f2937;
    color:#fff;
    cursor:pointer;
}

.menu button:hover{
    background:#374151;
}

.container{ padding:20px; }

.card{
    background:#fff;
    padding:15px;
    margin-bottom:15px;
    border-radius:10px;
    box-shadow:0 2px 6px rgba(0,0,0,0.1);
}

input, select, button{
    width:100%;
    padding:10px;
    margin:5px 0;
}

.vehicle{
    padding:10px;
    border-left:5px solid #2563eb;
    background:#f9fafb;
    margin:8px 0;
    cursor:pointer;
}

.vehicle:hover{ background:#eef2ff; }

.barbg{height:10px;background:#e5e7eb;border-radius:5px;}
.bar{height:10px;background:#10b981;border-radius:5px;}

.status{
    font-size:12px;
    padding:3px 8px;
    border-radius:5px;
    color:#fff;
}

.red{background:#ef4444;}
.yellow{background:#f59e0b;}
.green{background:#10b981;}
.gray{background:#6b7280;}

.hidden{display:none;}
</style>
</head>

<body>

<header>⚡ Elétrica Globe</header>

<div class="menu">
<button onclick="show('carros')">🚐 Carros</button>
<button onclick="show('horas')">⏱ Horas</button>
</div>

<div class="container">

<!-- CARROS -->
<div id="carros">

<div class="card">
<h3>Cadastro de Veículo</h3>

<input id="veiculo" placeholder="Nome do veículo">
<input id="responsavel" placeholder="Responsável">

<label>Data de entrada</label>
<input id="entrada" type="date">

<label>Data de saída (prazo)</label>
<input id="saida" type="date">

<button onclick="addCar()">Salvar</button>
</div>

<div class="card">
<h3>Lista de Carros</h3>
<div id="lista"></div>
</div>

</div>

<!-- HORAS -->
<div id="horas" class="hidden">

<div class="card">
<h3>Lançar Horas</h3>

<select id="carroSelect"></select>

<select id="eletricista">
<option>Eletricista 1</option>
<option>Eletricista 2</option>
<option>Eletricista 3</option>
</select>

<input id="horasInput" type="number" placeholder="Horas">

<button onclick="addHoras()">Salvar</button>
</div>

<div class="card">
<h3>Resumo Horas</h3>
<div id="resumoHoras"></div>
</div>

</div>

</div>

<script>

let carros = JSON.parse(localStorage.getItem("globe_carros")) || [];
let horas = JSON.parse(localStorage.getItem("globe_horas")) || [];

/* ================= MENU ================= */
function show(t){
document.getElementById("carros").classList.add("hidden");
document.getElementById("horas").classList.add("hidden");
document.getElementById(t).classList.remove("hidden");

if(t==="horas") atualizarSelect();
renderCarros();
renderHoras();
}

/* ================= CARROS ================= */
function addCar(){

let v=document.getElementById("veiculo").value.trim();
let r=document.getElementById("responsavel").value.trim();
let e=document.getElementById("entrada").value;
let s=document.getElementById("saida").value;

if(!v||!r||!e||!s) return alert("Preencha tudo");

if(carros.find(c=>c.veiculo.toLowerCase()===v.toLowerCase()))
return alert("Já existe");

carros.push({
veiculo:v,
responsavel:r,
entrada:e,
saida:s,
checklist:getChecklist()
});

localStorage.setItem("globe_carros",JSON.stringify(carros));

renderCarros();

}

/* ================= ALERTA PRAZO ================= */
function alertaPrazo(c){

let hoje = new Date();
let saida = new Date(c.saida);

let diff = Math.ceil((saida-hoje)/(1000*60*60*24));

if(diff < 0) return {t:"ATRASADO",c:"red"};
if(diff <= 2) return {t:"URGENTE",c:"yellow"};
return {t:"NO PRAZO",c:"green"};

}

/* ================= PROGRESSO ================= */
function progresso(c){

let t=0,d=0;

for(let f in c.checklist){
c.checklist[f].forEach(i=>{
t++;
if(i.done)d++;
});
}

return t?Math.round((d/t)*100):0;
}

/* ================= LISTA ================= */
function renderCarros(){

let div=document.getElementById("lista");
div.innerHTML="";

carros.forEach((c,i)=>{

let p=progresso(c);
let a=alertaPrazo(c);

div.innerHTML+=`
<div class="vehicle">
<b>🚐 ${c.veiculo}</b><br>
<small>${c.responsavel}</small><br>

<span class="status ${a.c}">${a.t}</span>
<span class="status gray">${p}%</span>

<div class="barbg">
<div class="bar" style="width:${p}%"></div>
</div>

<small>Entrada: ${c.entrada} | Saída: ${c.saida}</small>

</div>`;
});

}

/* ================= HORAS ================= */
function atualizarSelect(){

let sel=document.getElementById("carroSelect");
sel.innerHTML="";

carros.forEach(c=>{
sel.innerHTML+=`<option>${c.veiculo}</option>`;
});

}

function addHoras(){

let carro=document.getElementById("carroSelect").value;
let ele=document.getElementById("eletricista").value;
let h=parseFloat(document.getElementById("horasInput").value);

if(!carro||!h) return;

horas.push({carro,ele,h});

localStorage.setItem("globe_horas",JSON.stringify(horas));

renderHoras();

}

function renderHoras(){

let div=document.getElementById("resumoHoras");
div.innerHTML="";

let mapa={};

horas.forEach(h=>{
mapa[h.carro]=(mapa[h.carro]||0)+h.h;
});

for(let c in mapa){
div.innerHTML+=`
<div class="card">
<b>🚐 ${c}</b><br>
<div class="barbg">
<div class="bar" style="width:${mapa[c]*5}%"></div>
</div>
<small>${mapa[c]} horas</small>
</div>`;
}

}

/* ================= CHECKLIST BASE ================= */
function getChecklist(){

return {
"FASE 1":[
{item:"Iluminação CA",done:false},
{item:"Tomadas CA",done:false}
],
"FASE 2":[
{item:"12V geral",done:false},
{item:"Baterias",done:false},
{item:"Solar",done:false}
],
"FASE 3":[
{item:"Central elétrica",done:false},
{item:"Inversor",done:false}
]
};

}

show('carros');

</script>

</body>
</html>
