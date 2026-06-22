<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Controle Elétrica Globe</title>
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
    background:#eef2ff;
    padding:10px;
    border-left:5px solid #2563eb;
    margin-bottom:8px;
    cursor:pointer;
}

.vehicle:hover{
    background:#e0e7ff;
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
</style>
</head>

<body>

<header>⚡ Controle Elétrica Globe</header>

<div class="container">

<!-- CADASTRO -->
<div class="card">
<h3>Cadastrar Veículo</h3>
<input id="veiculo" placeholder="Nome do veículo">
<input id="responsavel" placeholder="Responsável">
<button onclick="addVehicle()">Salvar</button>
</div>

<!-- LISTA -->
<div class="card">
<h3>Veículos</h3>
<div id="lista"></div>
</div>

<!-- CHECKLIST -->
<div id="checklistArea" class="hidden">

<div class="card">
<h3 id="titulo"></h3>
<button onclick="voltar()">⬅ Voltar</button>

<div class="barbg">
<div id="barra" class="bar" style="width:0%"></div>
</div>

<div id="checklist"></div>
</div>

</div>

</div>

<script>

let vehicles = JSON.parse(localStorage.getItem("vehicles")) || [];
let atual = null;

/* ================= CHECKLIST BASE ================= */
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
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    "Teste final"
    
];
}

/* ================= SALVAR ================= */
function save(){
localStorage.setItem("vehicles",JSON.stringify(vehicles));
}

/* ================= CADASTRAR ================= */
function addVehicle(){

let v = document.getElementById("veiculo").value.trim();
let r = document.getElementById("responsavel").value.trim();

if(!v || !r) return alert("Preencha tudo");

vehicles.push({
nome:v,
resp:r,
checklist:baseChecklist().map(i=>({item:i,done:false}))
});

save();
render();

document.getElementById("veiculo").value="";
document.getElementById("responsavel").value="";
}

/* ================= RENDER LISTA ================= */
function render(){

let div = document.getElementById("lista");
div.innerHTML="";

vehicles.forEach((v,i)=>{

let p = progress(v);

div.innerHTML+=`
<div class="vehicle" onclick="openVehicle(${i})">
<b>${v.nome}</b><br>
<small>${v.resp}</small>

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
function openVehicle(i){

atual = i;

document.getElementById("checklistArea").classList.remove("hidden");

document.getElementById("lista").parentElement.classList.add("hidden");

let v = vehicles[i];

document.getElementById("titulo").innerText =
v.nome + " - " + v.resp;

renderChecklist();

}

/* ================= CHECKLIST RENDER ================= */
function renderChecklist(){

let v = vehicles[atual];

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

updateBar();

}

/* ================= TOGGLE ================= */
function toggle(i){

vehicles[atual].checklist[i].done =
!vehicles[atual].checklist[i].done;

save();

renderChecklist();

render();

}

/* ================= BARRA ================= */
function updateBar(){

let v = vehicles[atual];

let p = progress(v);

document.getElementById("barra").style.width = p+"%";

}

/* ================= VOLTAR ================= */
function voltar(){

document.getElementById("checklistArea").classList.add("hidden");

document.getElementById("lista").parentElement.classList.remove("hidden");

render();

}

render();

</script>

</body>
</html>
