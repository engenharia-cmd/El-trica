<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Elétrica Globe ERP</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{margin:0;font-family:Arial;background:#f4f6f9;}

header{
background:#111827;color:#fff;padding:15px;font-size:20px;
}

.menu{display:flex;background:#1f2937;}
.menu button{flex:1;padding:12px;border:none;color:#fff;background:#1f2937;cursor:pointer;}
.menu button:hover{background:#374151;}

.container{padding:20px;}

.card{
background:#fff;padding:15px;margin-bottom:15px;
border-radius:10px;box-shadow:0 2px 6px rgba(0,0,0,0.1);
}

input,select,button{width:100%;padding:10px;margin:5px 0;}

.vehicle{
background:#f9fafb;border-left:5px solid #2563eb;
padding:10px;margin:8px 0;cursor:pointer;
}

.vehicle:hover{background:#eef2ff;}

.hidden{display:none;}

.barbg{height:10px;background:#e5e7eb;border-radius:5px;}
.bar{height:10px;background:#10b981;height:10px;}

.status{
font-size:12px;padding:3px 8px;border-radius:5px;color:#fff;
}
.red{background:#ef4444;}
.yellow{background:#f59e0b;}
.green{background:#10b981;}
.gray{background:#6b7280;}

.item{display:block;margin:5px 0;}

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
<input id="entrada" type="date">
<input id="saida" type="date">

<button onclick="addCar()">Salvar</button>
</div>

<div class="card">
<h3>Lista de Carros</h3>
<div id="lista"></div>
</div>

</div>

<!-- CHECKLIST -->
<div id="checklistView" class="hidden">
<div class="card">
<h3 id="tituloCarro"></h3>
<button onclick="voltar()">⬅ Voltar</button>
<button onclick="excluir()">🗑 Excluir veículo</button>
<div id="checklist"></div>
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

<input id="horasInput" type="number" placeholder="Horas trabalhadas">

<button onclick="addHoras()">Salvar</button>
</div>

<div class="card">
<h3>Histórico de Horas</h3>
<div id="resumoHoras"></div>
</div>

</div>

</div>

<script>

let carros = JSON.parse(localStorage.getItem("globe_carros")) || [];
let horas = JSON.parse(localStorage.getItem("globe_horas")) || [];

let atual = null;

/* ================= MENU ================= */
function show(t){
document.getElementById("carros").classList.add("hidden");
document.getElementById("horas").classList.add("hidden");
document.getElementById("checklistView").classList.add("hidden");

document.getElementById(t).classList.remove("hidden");

if(t==="horas") atualizarSelect();

render();
renderHoras();
}

/* ================= CHECKLIST BASE ================= */
function baseChecklist(){
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

/* ================= SALVAR CARRO ================= */
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
checklist:JSON.parse(JSON.stringify(baseChecklist()))
});

localStorage.setItem("globe_carros",JSON.stringify(carros));

render();
}

/* ================= ABRIR CHECKLIST ================= */
function abrir(i){

atual=i;
let c=carros[i];

document.getElementById("carros").classList.add("hidden");
document.getElementById("checklistView").classList.remove("hidden");

document.getElementById("tituloCarro").innerText=
`🚐 ${c.veiculo} - ${c.responsavel}`;

let html="";

for(let f in c.checklist){

html+=`<div class="card"><b>${f}</b>`;

c.checklist[f].forEach((item,idx)=>{

html+=`
<label class="item">
<input type="checkbox" ${item.done?"checked":""}
onchange="toggle(${i},'${f}',${idx})">
${item.item}
</label>`;
});

html+="</div>";
}

document.getElementById("checklist").innerHTML=html;

}

/* ================= TOGGLE ================= */
function toggle(i,f,idx){

carros[i].checklist[f][idx].done =
!carros[i].checklist[f][idx].done;

localStorage.setItem("globe_carros",JSON.stringify(carros));

abrir(i);
}

/* ================= EXCLUIR ================= */
function excluir(){

if(confirm("Excluir veículo?")){
carros.splice(atual,1);
localStorage.setItem("globe_carros",JSON.stringify(carros));
voltar();
}

}

/* ================= VOLTAR ================= */
function voltar(){
document.getElementById("checklistView").classList.add("hidden");
document.getElementById("carros").classList.remove("hidden");
render();
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

/* ================= RENDER CARROS ================= */
function render(){

let div=document.getElementById("lista");
div.innerHTML="";

carros.forEach((c,i)=>{

let p=progresso(c);

div.innerHTML+=`
<div class="vehicle" onclick="abrir(${i})">
<b>🚐 ${c.veiculo}</b><br>
<small>${c.responsavel}</small><br>

<div class="barbg">
<div class="bar" style="width:${p}%"></div>
</div>

<small>${p}% concluído</small>
</div>`;
});

}

/* ================= HORAS (COM DATA/HORA AUTOMÁTICA) ================= */
function addHoras(){

let carro=document.getElementById("carroSelect").value;
let ele=document.getElementById("eletricista").value;
let h=parseFloat(document.getElementById("horasInput").value);

if(!carro||!h) return;

let agora=new Date();

horas.push({
carro,
ele,
h,
data:agora.toLocaleDateString(),
hora:agora.toLocaleTimeString()
});

localStorage.setItem("globe_horas",JSON.stringify(horas));

document.getElementById("horasInput").value="";

renderHoras();
}

/* ================= SELECT ================= */
function atualizarSelect(){

let sel=document.getElementById("carroSelect");
sel.innerHTML="";

carros.forEach(c=>{
sel.innerHTML+=`<option>${c.veiculo}</option>`;
});

}

/* ================= HISTÓRICO HORAS ================= */
function renderHoras(){

let div=document.getElementById("resumoHoras");
div.innerHTML="";

horas.slice().reverse().forEach(h=>{

div.innerHTML+=`
<div class="card">
<b>🚐 ${h.carro}</b><br>
👷 ${h.ele}<br>
⏱ ${h.h}h<br>
📅 ${h.data} - ${h.hora}
</div>`;
});

}

show('carros');

</script>

</body>
</html>
