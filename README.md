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

.container{
    padding:20px;
}

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

.vehicle:hover{
    background:#eef2ff;
}

.hidden{display:none;}

.checklist{
    background:#f1f5f9;
    padding:10px;
    margin-top:10px;
    border-radius:8px;
}

.item{display:block;margin:5px 0;}

.barbg{height:10px;background:#e5e7eb;border-radius:5px;}
.bar{height:10px;background:#10b981;border-radius:5px;}
.status{
    font-size:12px;
    padding:3px 8px;
    color:#fff;
    border-radius:5px;
}
.red{background:#ef4444;}
.yellow{background:#f59e0b;}
.green{background:#10b981;}
.gray{background:#6b7280;}
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
<h3>Cadastrar Veículo</h3>
<input id="veiculo" placeholder="Nome do veículo">
<input id="responsavel" placeholder="Responsável">
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

<input id="horasInput" type="number" placeholder="Horas trabalhadas">

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
let atual = null;

/* ================= MENU ================= */
function show(t){
document.getElementById("carros").classList.add("hidden");
document.getElementById("horas").classList.add("hidden");
document.getElementById(t).classList.remove("hidden");

if(t==="horas") atualizarSelect();
renderHoras();
renderCarros();
}

/* ================= CARROS ================= */
function addCar(){

let v=document.getElementById("veiculo").value.trim();
let r=document.getElementById("responsavel").value.trim();

if(!v||!r) return alert("Preencha tudo");

if(carros.find(c=>c.veiculo.toLowerCase()===v.toLowerCase()))
return alert("Já existe");

carros.push({
veiculo:v,
responsavel:r,
checklist:getChecklist()
});

localStorage.setItem("globe_carros",JSON.stringify(carros));

document.getElementById("veiculo").value="";
document.getElementById("responsavel").value="";

renderCarros();
}

function renderCarros(){

let div=document.getElementById("lista");
div.innerHTML="";

carros.forEach((c,i)=>{

let p=progresso(c);

div.innerHTML+=`
<div class="vehicle">
<b>🚐 ${c.veiculo}</b><br>
<small>${c.responsavel}</small><br>

<div class="barbg">
<div class="bar" style="width:${p}%"></div>
</div>

<small>${p}% concluído</small>

<button onclick="deletar(${i})" style="background:#ef4444;color:#fff;margin-top:5px;">
Excluir
</button>
</div>`;
});

}

function deletar(i){

if(confirm("Excluir veículo?")){
carros.splice(i,1);
localStorage.setItem("globe_carros",JSON.stringify(carros));
renderCarros();
}

}

/* ================= CHECKLIST ================= */
function getChecklist(){

return {
"FASE 1 C.A":[
{item:"Iluminação fase/fase 2,5mm",done:false},
{item:"Tomadas fase/fase/terra 2,5mm",done:false}
],
"FASE 1 C.C":[
{item:"Iluminação 12V",done:false},
{item:"Placa solar",done:false},
{item:"Geladeira",done:false},
{item:"Ar-condicionado",done:false},
{item:"Toldo e escada",done:false}
],
"FASE 2":[
{item:"Tomadas 220V/12V",done:false},
{item:"Lâmpadas",done:false},
{item:"Slide-out",done:false}
],
"FASE 3":[
{item:"Central elétrica",done:false},
{item:"Fusíveis",done:false},
{item:"Inversor",done:false}
]
};

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

document.getElementById("horasInput").value="";

renderHoras();
}

function renderHoras(){

let div=document.getElementById("resumoHoras");
div.innerHTML="";

let mapa={};

horas.forEach(h=>{
if(!mapa[h.carro]) mapa[h.carro]=0;
mapa[h.carro]+=h.h;
});

for(let c in mapa){
div.innerHTML+=`
<div class="card">
<b>🚐 ${c}</b><br>
<div class="barbg">
<div class="bar" style="width:${mapa[c]*5}%"></div>
</div>
<small>${mapa[c]} horas totais</small>
</div>`;
}

}

show('carros');

</script>

</body>
</html>
