<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Elétrica Globe - ERP Industrial</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    font-family: Arial;
    margin:0;
    background:#f4f6f9;
}

header{
    background:#111827;
    color:white;
    padding:15px;
    font-size:20px;
}

/* ABAS */
.tabs{
    display:flex;
    background:#1f2937;
}

.tabs button{
    flex:1;
    padding:12px;
    border:none;
    background:#1f2937;
    color:white;
    cursor:pointer;
}

.tabs button:hover{
    background:#374151;
}

.container{
    padding:15px;
}

.card{
    background:white;
    padding:15px;
    border-radius:10px;
    margin-bottom:10px;
    box-shadow:0 2px 6px rgba(0,0,0,0.1);
}

input,button{
    width:100%;
    padding:10px;
    margin:5px 0;
}

.vehicle{
    padding:10px;
    border-left:5px solid #2563eb;
    margin-bottom:8px;
    cursor:pointer;
    border-radius:6px;
}

.vehicle:hover{
    filter:brightness(0.97);
}

.barbg{
    width:100%;
    height:10px;
    background:#e5e7eb;
    border-radius:5px;
    margin-top:5px;
}

.bar{
    height:10px;
    background:#10b981;
    border-radius:5px;
}

.hidden{display:none;}

label{
    display:block;
    margin:5px 0;
}

#percentual{
    font-weight:bold;
    margin:8px 0;
}

/* 🔥 PISCAR ATRASADO */
@keyframes piscar {
0% { opacity: 1; }
50% { opacity: 0.2; }
100% { opacity: 1; }
}

.blink{
animation: piscar 1s infinite;
border: 2px solid #ef4444 !important;
}

/* TOAST */
#toast{
position:fixed;
bottom:20px;
left:50%;
transform:translateX(-50%);
background:#111827;
color:white;
padding:12px 20px;
border-radius:8px;
display:none;
font-weight:bold;
z-index:9999;
}
</style>
</head>

<body>

<header>⚡ Elétrica Globe - ERP Industrial</header>

<!-- ABAS -->
<div class="tabs">
<button onclick="openTab('cadastro')">➕ Cadastro</button>
<button onclick="openTab('visao')">📊 Visão Geral</button>
</div>

<div class="container">

<!-- CADASTRO -->
<div id="cadastro">

<div class="card">
<h3>Cadastrar Veículo</h3>

<input id="veiculo" placeholder="Nome do veículo">
<input id="responsavel" placeholder="Responsável">

<label>Data de entrada</label>
<input id="entrada" type="date">

<label>Data de saída</label>
<input id="saida" type="date">

<button onclick="addVehicle()">Salvar</button>
</div>

</div>

<!-- VISÃO GERAL -->
<div id="visao" class="hidden">

<div class="card">
<h3>Veículos em Produção</h3>
<div id="lista"></div>
</div>

</div>

<!-- CHECKLIST -->
<div id="checklistArea" class="hidden">

<div class="card">
<h3 id="titulo"></h3>

<button onclick="voltar()">⬅ Voltar</button>

<button onclick="deleteVehicle()" style="
background:#ef4444;
color:white;
font-weight:bold;
border:none;
padding:10px;
margin:10px 0;
border-radius:6px;
cursor:pointer;">
🗑 Excluir Veículo
</button>

<div class="barbg">
<div id="barra" class="bar" style="width:0%"></div>
</div>

<div id="percentual"></div>

<div id="checklist"></div>

</div>

</div>

</div>

<!-- TOAST -->
<div id="toast"></div>

<script>

let vehicles = JSON.parse(localStorage.getItem("vehicles")) || [];
let atual = null;

/* ================= ABAS ================= */
function openTab(tab){

document.getElementById("cadastro").classList.add("hidden");
document.getElementById("visao").classList.add("hidden");
document.getElementById("checklistArea").classList.add("hidden");

document.getElementById(tab).classList.remove("hidden");

if(tab === "visao") render();

}

/* ================= CHECKLIST ================= */
function baseChecklist(){
return [
"Iluminação 12V",
"Tomadas 220V",
"Sistema de baterias",
"Placa solar",
"Central elétrica",
"Inversor",
"Bombas de água",
"Teste final"
];
}

/* ================= SALVAR ================= */
function save(){
localStorage.setItem("vehicles",JSON.stringify(vehicles));
}

/* ================= CADASTRO ================= */
function addVehicle(){

let v = document.getElementById("veiculo").value.trim();
let r = document.getElementById("responsavel").value.trim();
let e = document.getElementById("entrada").value;
let s = document.getElementById("saida").value;

if(!v || !r || !e || !s) return alert("Preencha tudo");

vehicles.push({
id: Date.now(), // 🔥 ID ÚNICO
nome:v,
resp:r,
entrada:e,
saida:s,
checklist:baseChecklist().map(i=>({item:i,done:false}))
});

save();

document.getElementById("veiculo").value="";
document.getElementById("responsavel").value="";
document.getElementById("entrada").value="";
document.getElementById("saida").value="";

notify("✔ Veículo cadastrado!");
}

/* ================= STATUS ================= */
function getStatus(v){

if(!v.saida) return "normal";

let hoje = new Date();
let saida = new Date(v.saida);

let diff = Math.ceil((saida - hoje) / (1000*60*60*24));

if(diff < 0) return "atrasado";
if(diff <= 10) return "alerta";
return "normal";
}

/* ================= LISTA ================= */
function render(){

let div = document.getElementById("lista");
div.innerHTML="";

vehicles.forEach((v,i)=>{

let p = progress(v);
let status = getStatus(v);

let bg = "#eef2ff";
let aviso = "";

if(status === "alerta"){
bg = "#fef9c3";
aviso = "⚠️ Prazo próximo";
}

if(status === "atrasado"){
bg = "#fee2e2";
aviso = "🚨 ATRASADO";
}

div.innerHTML+=`
<div class="vehicle ${status === "atrasado" ? "blink" : ""}"
onclick="openVehicle(${v.id})"
style="background:${bg}">

<b>${v.nome}</b><br>
<small>${v.resp}</small><br>

<small>Entrada: ${v.entrada} | Saída: ${v.saida}</small><br>

<small style="font-weight:bold;color:#b91c1c">${aviso}</small>

<div class="barbg">
<div class="bar" style="width:${p}%"></div>
</div>

<small>${p}% concluído</small>
</div>`;
});

}

/* ================= PROGRESSO ================= */
function progress(v){

let total = v.checklist.length;
let done = v.checklist.filter(i=>i.done).length;

return Math.round((done/total)*100);

}

/* ================= ABRIR CHECKLIST ================= */
function openVehicle(id){

atual = id;

document.getElementById("visao").classList.add("hidden");
document.getElementById("checklistArea").classList.remove("hidden");

let v = vehicles.find(x => x.id === id);

if(!v) return;

document.getElementById("titulo").innerText =
v.nome + " - " + v.resp;

renderChecklist(v);
updateBar(v);

}

/* ================= CHECKLIST ================= */
function renderChecklist(v){

let div = document.getElementById("checklist");
div.innerHTML="";

v.checklist.forEach((c,i)=>{

div.innerHTML+=`
<label>
<input type="checkbox" ${c.done?"checked":""}
onchange="toggle(${i})">
${c.item}
</label>
`;

});

}

/* ================= TOGGLE ================= */
function toggle(i){

let v = vehicles.find(x => x.id === atual);

v.checklist[i].done = !v.checklist[i].done;

save();

renderChecklist(v);
render();
updateBar(v);

}

/* ================= BARRA ================= */
function updateBar(v){

let p = progress(v);

document.getElementById("barra").style.width = p + "%";
document.getElementById("percentual").innerText =
p + "% concluído";

}

/* ================= EXCLUIR ================= */
function deleteVehicle(){

if(atual === null) return;

if(!confirm("Deseja excluir este veículo?")) return;

vehicles = vehicles.filter(v => v.id !== atual);

save();

atual = null;

document.getElementById("checklistArea").classList.add("hidden");
document.getElementById("visao").classList.remove("hidden");

render();

notify("🗑 Veículo excluído com sucesso!");
}

/* ================= VOLTAR ================= */
function voltar(){

document.getElementById("checklistArea").classList.add("hidden");
document.getElementById("visao").classList.remove("hidden");

render();

}

/* ================= NOTIFICAÇÃO ================= */
function notify(msg){

let t = document.getElementById("toast");
t.innerText = msg;
t.style.display = "block";

setTimeout(()=>{
t.style.display = "none";
},2500);

}

/* INICIAL */
openTab("cadastro");

</script>

</body>
</html>
