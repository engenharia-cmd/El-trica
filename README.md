<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Painel Industrial - Elétrica Globe</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{margin:0;font-family:Arial;background:#0f172a;color:#fff;}

header{
background:#111827;
padding:15px;
font-size:20px;
font-weight:bold;
}

.grid{
display:grid;
grid-template-columns:repeat(4,1fr);
gap:10px;
padding:10px;
}

.card{
background:#1e293b;
padding:15px;
border-radius:10px;
}

.big{
font-size:22px;
font-weight:bold;
}

.red{color:#ef4444;}
.yellow{color:#f59e0b;}
.green{color:#10b981;}

.menu{
display:flex;
background:#111827;
}

.menu button{
flex:1;
padding:12px;
border:none;
background:#111827;
color:#fff;
cursor:pointer;
}

.menu button:hover{background:#1f2937;}

.container{padding:15px;}

.vehicle{
background:#1e293b;
padding:10px;
margin:8px 0;
border-left:5px solid #2563eb;
cursor:pointer;
}

.barbg{height:10px;background:#334155;border-radius:5px;}
.bar{height:10px;background:#10b981;height:10px;}

.alert{
background:#ef4444;
padding:10px;
margin:10px 0;
font-weight:bold;
border-radius:8px;
animation:pulse 1s infinite;
}

@keyframes pulse{
0%{opacity:1;}
50%{opacity:0.5;}
100%{opacity:1;}
}

.hidden{display:none;}

</style>
</head>

<body>

<header>🏭 Elétrica Globe - Painel Industrial</header>

<div class="menu">
<button onclick="show('dashboard')">📊 Dashboard</button>
<button onclick="show('carros')">🚐 Carros</button>
<button onclick="show('alertas')">🚨 Alertas</button>
<button onclick="show('horas')">⏱ Horas</button>
</div>

<div class="container">

<!-- DASHBOARD -->
<div id="dashboard">

<div class="grid">

<div class="card">
<div>Total</div>
<div class="big" id="total">0</div>
</div>

<div class="card">
<div>Em andamento</div>
<div class="big yellow" id="andamento">0</div>
</div>

<div class="card">
<div>Concluídos</div>
<div class="big green" id="concluido">0</div>
</div>

<div class="card">
<div>Atrasados</div>
<div class="big red" id="atrasado">0</div>
</div>

</div>

<div class="card">
<h3>Progresso Médio da Oficina</h3>
<div class="barbg">
<div class="bar" id="mediaBar"></div>
</div>
</div>

</div>

<!-- CARROS -->
<div id="carros" class="hidden">

<div class="card">
<h3>Cadastrar Veículo</h3>
<input id="v" placeholder="Veículo">
<input id="r" placeholder="Responsável">
<input id="e" type="date">
<input id="s" type="date">
<button onclick="add()">Salvar</button>
</div>

<div id="lista"></div>

</div>

<!-- ALERTAS -->
<div id="alertas" class="hidden">
<div class="card">
<h3>🚨 URGÊNCIA DA OFICINA</h3>
<div id="listaAlertas"></div>
</div>
</div>

<!-- HORAS -->
<div id="horas" class="hidden">

<div class="card">
<h3>Registrar Horas</h3>

<select id="carroSel"></select>
<select id="ele">
<option>Eletricista 1</option>
<option>Eletricista 2</option>
<option>Eletricista 3</option>
</select>

<input id="h" type="number" placeholder="Horas">
<button onclick="addH()">Salvar</button>

</div>

<div id="hist"></div>

</div>

</div>

<script>

let carros = JSON.parse(localStorage.getItem("g_carros")) || [];
let horas = JSON.parse(localStorage.getItem("g_horas")) || [];

function show(t){
["dashboard","carros","alertas","horas"].forEach(x=>{
document.getElementById(x).classList.add("hidden");
});
document.getElementById(t).classList.remove("hidden");

render();
}

function add(){

carros.push({
v:v.value,
r:r.value,
e:e.value,
s:s.value,
c:base()
});

save();
render();
}

function base(){
return {
"A":[{t:"C.A",d:false}],
"B":[{t:"C.C",d:false}],
"C":[{t:"Baterias",d:false}]
};
}

function progresso(c){
let t=0,d=0;
for(let f in c.c){
c.c[f].forEach(i=>{
t++;if(i.d)d++;
});
}
return t?Math.round(d/t*100):0;
}

function alerta(c){

let hoje=new Date();
let s=new Date(c.s);
let diff=Math.ceil((s-hoje)/86400000);

if(diff<0)return"red";
if(diff<=2)return"yellow";
return"green";

}

function render(){

let total=carros.length;
let a=0,b=0,c=0;
let media=0;

let list=document.getElementById("lista");
let al=document.getElementById("listaAlertas");

list.innerHTML="";
al.innerHTML="";

carros.forEach((x,i)=>{

let p=progresso(x);
media+=p;

let st=alerta(x);

if(st==="red")a++;
if(p===100)b++;
if(p>0&&p<100)c++;

let alertaHTML="";

if(st==="red"){
alertaHTML=`<div class="alert">🚨 ATRASO CRÍTICO</div>`;
}

list.innerHTML+=`
<div class="vehicle" onclick="openC(${i})">
${alertaHTML}
<b>${x.v}</b><br>
<small>${x.r}</small><br>
<div class="barbg"><div class="bar" style="width:${p}%"></div></div>
<small>${p}%</small>
</div>`;

if(st==="red"){
al.innerHTML+=`<div class="alert">${x.v} ATRASADO</div>`;
}

});

document.getElementById("total").innerText=total;
document.getElementById("atrasado").innerText=a;
document.getElementById("concluido").innerText=b;
document.getElementById("andamento").innerText=c;

document.getElementById("mediaBar").style.width=
(total?media/total:0)+"%";

atualizarSel();
}

function atualizarSel(){

carroSel.innerHTML="";
carros.forEach(c=>{
carroSel.innerHTML+=`<option>${c.v}</option>`;
});

}

function addH(){

horas.push({
carro:carroSel.value,
ele:ele.value,
h:h.value,
d:new Date().toLocaleDateString(),
t:new Date().toLocaleTimeString()
});

save();
render();
}

function save(){
localStorage.setItem("g_carros",JSON.stringify(carros));
localStorage.setItem("g_horas",JSON.stringify(horas));
}

function openC(i){
alert("Checklist aqui (expansão futura)");
}

render();

</script>

</body>
</html>
