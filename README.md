<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Elétrica Globe - Painel Industrial</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{margin:0;font-family:Arial;background:#0f172a;color:#fff;}

header{background:#111827;padding:15px;font-size:20px;font-weight:bold;}

.menu{display:flex;background:#111827;}
.menu button{flex:1;padding:12px;border:none;background:#111827;color:#fff;cursor:pointer;}
.menu button:hover{background:#1f2937;}

.container{padding:15px;}

.card{
background:#1e293b;padding:15px;border-radius:10px;margin-bottom:10px;
}

.vehicle{
background:#1e293b;padding:10px;margin:8px 0;
border-left:5px solid #2563eb;cursor:pointer;
}

.barbg{height:10px;background:#334155;border-radius:5px;}
.bar{height:10px;background:#10b981;height:10px;}

.alert{
background:#ef4444;padding:10px;margin:10px 0;
border-radius:8px;font-weight:bold;animation:pulse 1s infinite;
}

@keyframes pulse{0%{opacity:1}50%{opacity:.5}100%{opacity:1}}

.hidden{display:none;}

input,select,button{width:100%;padding:10px;margin:5px 0;}
</style>
</head>

<body>

<header>🏭 Elétrica Globe - Painel Industrial</header>

<div class="menu">
<button onclick="show('dash')">📊 Dashboard</button>
<button onclick="show('carros')">🚐 Carros</button>
<button onclick="show('alertas')">🚨 Alertas</button>
<button onclick="show('horas')">⏱ Horas</button>
</div>

<div class="container">

<!-- DASH -->
<div id="dash">

<div class="card">
<h3>Resumo</h3>
<div id="resumo"></div>
</div>

</div>

<!-- CARROS -->
<div id="carros" class="hidden">

<div class="card">
<h3>Cadastro</h3>
<input id="v">
<input id="r">
<input id="e" type="date">
<input id="s" type="date">
<button onclick="add()">Salvar</button>
</div>

<div id="lista"></div>
</div>

<!-- CHECKLIST -->
<div id="checklistView" class="hidden">
<div class="card">
<h3 id="titulo"></h3>
<button onclick="back()">⬅ Voltar</button>
<button onclick="del()">🗑 Excluir</button>
<div id="checklist"></div>
</div>
</div>

<!-- ALERTAS -->
<div id="alertas" class="hidden">
<div class="card">
<h3>🚨 Atrasos</h3>
<div id="listaAlertas"></div>
</div>
</div>

<!-- HORAS -->
<div id="horas" class="hidden">

<div class="card">
<h3>Horas</h3>

<select id="sel"></select>
<select id="ele">
<option>Eletricista 1</option>
<option>Eletricista 2</option>
<option>Eletricista 3</option>
</select>

<input id="h" type="number">
<button onclick="addH()">Salvar</button>

</div>

<div id="hist"></div>

</div>

</div>

<script>

let carros = JSON.parse(localStorage.getItem("carros")) || [];
let horas = JSON.parse(localStorage.getItem("horas")) || [];

let atual = null;

function baseChecklist(){

return {

"FASE 1 - CA":[
"Circuito iluminação CA",
"Circuito tomadas CA"
],

"FASE 1 - CC":[
"Iluminação 12V",
"Placa solar",
"Ar-condicionado",
"Toldo e escada",
"Geladeira"
],

"FASE 2":[
"Tomadas 220V",
"Lâmpadas",
"Slide-out",
"Boias água"
],

"FASE 3":[
"Central elétrica",
"Inversor",
"Fusíveis",
"Quadro QBC"
]

};

}

function add(){

carros.push({
v:v.value,
r:r.value,
e:e.value,
s:s.value,
c:JSON.parse(JSON.stringify(baseChecklist()))
});

save();render();

}

function progresso(c){

let t=0,d=0;

for(let f in c.c){
c.c[f].forEach(i=>{
t++;
if(i.done)d++;
});
}

return t?Math.round(d/t*100):0;

}

function alerta(c){

let hoje=new Date();
let s=new Date(c.s);
let d=Math.ceil((s-hoje)/86400000);

if(d<0)return"red";
if(d<=2)return"yellow";
return"green";

}

function render(){

let list=document.getElementById("lista");
let al=document.getElementById("listaAlertas");
let res=document.getElementById("resumo");

list.innerHTML="";
al.innerHTML="";
res.innerHTML="";

let total=carros.length;
let atras=0;
let media=0;

carros.forEach((c,i)=>{

let p=progresso(c);
let st=alerta(c);
media+=p;

if(st==="red")atras++;

list.innerHTML+=`
<div class="vehicle" onclick="openC(${i})">
<b>${c.v}</b><br>
<small>${c.r}</small><br>
<div class="barbg"><div class="bar" style="width:${p}%"></div></div>
<small>${p}%</small>
</div>`;

if(st==="red"){
al.innerHTML+=`<div class="alert">🚨 ${c.v} ATRASADO</div>`;
}

});

res.innerHTML=`
Total: ${total}<br>
Atrasados: ${atras}<br>
Média: ${total?Math.round(media/total):0}%`;

}

function openC(i){

atual=i;

let c=carros[i];

document.getElementById("carros").classList.add("hidden");
document.getElementById("checklistView").classList.remove("hidden");

document.getElementById("titulo").innerText=
`🚐 ${c.v}`;

let html="";

for(let f in c.c){

html+=`<div class="card"><b>${f}</b>`;

c.c[f].forEach((i2,idx)=>{

html+=`
<label>
<input type="checkbox" ${i2.done?"checked":""}
onchange="toggle('${f}',${idx})">
${i2}
</label><br>
`;

});

html+="</div>";

}

document.getElementById("checklist").innerHTML=html;

}

function toggle(f,idx){

carros[atual].c[f][idx].done =
!carros[atual].c[f][idx].done;

save();
openC(atual);
render();

}

function del(){

if(confirm("Excluir?")){
carros.splice(atual,1);
save();
back();
}

}

function back(){
document.getElementById("checklistView").classList.add("hidden");
document.getElementById("carros").classList.remove("hidden");
render();
}

function addH(){

horas.push({
carro:sel.value,
ele:ele.value,
h:h.value,
d:new Date().toLocaleDateString(),
t:new Date().toLocaleTimeString()
});

save();

}

function save(){
localStorage.setItem("carros",JSON.stringify(carros));
localStorage.setItem("horas",JSON.stringify(horas));
}

function show(t){

["dash","carros","alertas","horas","checklistView"].forEach(x=>{
document.getElementById(x).classList.add("hidden");
});

document.getElementById(t).classList.remove("hidden");

render();
}

show("dash");

</script>

</body>
</html>
