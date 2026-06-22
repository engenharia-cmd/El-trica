<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>ERP Elétrica Motorhome</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    font-family: Arial;
    background:#f4f6f9;
    margin:0;
    padding:20px;
}

.container{
    max-width:1100px;
    margin:auto;
}

h1{ color:#1f2937; }

.card{
    background:#fff;
    padding:15px;
    margin-bottom:15px;
    border-radius:10px;
    box-shadow:0 2px 6px rgba(0,0,0,0.1);
}

input, button{
    width:100%;
    padding:10px;
    margin:5px 0;
}

button{
    border:none;
    border-radius:6px;
    cursor:pointer;
}

.btn{
    background:#2563eb;
    color:#fff;
}

.btn:hover{ background:#1d4ed8; }

.btn-danger{
    background:#ef4444;
    color:#fff;
}

.btn-gray{
    background:#6b7280;
    color:#fff;
}

.vehicle{
    padding:12px;
    margin:8px 0;
    background:#f9fafb;
    border-left:5px solid #2563eb;
    cursor:pointer;
}

.vehicle:hover{ background:#eef2ff; }

.checklist{
    background:#f1f5f9;
    padding:10px;
    border-radius:8px;
    margin-top:10px;
}

.item{ display:block; margin:5px 0; }

.hidden{ display:none; }

.year-title{
    margin-top:20px;
    font-weight:bold;
    color:#111827;
}
</style>
</head>

<body>

<div class="container">

<h1>🚐 ERP Elétrica Motorhome</h1>

<!-- CADASTRO -->
<div class="card" id="telaCadastro">

<h3>Cadastro de Veículo</h3>

<input id="veiculo" placeholder="Nome do veículo">
<input id="responsavel" placeholder="Responsável">

<button class="btn" onclick="salvar()">Salvar</button>

</div>

<!-- LISTA -->
<div class="card">
<h3>Veículos por Ano</h3>
<div id="lista"></div>
</div>

<!-- DETALHE -->
<div class="card hidden" id="telaDetalhe">

<h3 id="tituloDetalhe"></h3>

<button class="btn-gray" onclick="voltar()">⬅ Voltar</button>
<button class="btn" onclick="exportar()">📄 Exportar Relatório</button>
<button class="btn-danger" onclick="excluirAtual()">🗑 Excluir Veículo</button>

<div id="checklist"></div>

</div>

</div>

<script>

let dados = JSON.parse(localStorage.getItem("erp_motorhome")) || [];

let atual = null;

function salvar(){

let v = document.getElementById("veiculo").value.trim();
let r = document.getElementById("responsavel").value.trim();

if(!v || !r){
alert("Preencha tudo");
return;
}

if(dados.find(x => x.veiculo.toLowerCase() === v.toLowerCase())){
alert("Já existe esse veículo!");
return;
}

let ano = new Date().getFullYear();

dados.push({
veiculo:v,
responsavel:r,
ano:ano,
checklist:getChecklistCompleto(),
data:new Date().toLocaleString()
});

localStorage.setItem("erp_motorhome", JSON.stringify(dados));

document.getElementById("veiculo").value="";
document.getElementById("responsavel").value="";

render();

}

// abrir veículo
function abrir(i){

atual = i;

let v = dados[i];

document.getElementById("telaCadastro").classList.add("hidden");
document.getElementById("telaDetalhe").classList.remove("hidden");

document.getElementById("tituloDetalhe").innerText =
`🚐 ${v.veiculo} - ${v.responsavel}`;

let html = "";

for(let fase in v.checklist){

html += `<div class="checklist"><b>${fase}</b>`;

v.checklist[fase].forEach((c,idx)=>{
html += `
<label class="item">
<input type="checkbox" ${c.done?"checked":""}
onchange="toggle(${i},'${fase}',${idx})">
${c.item}
</label>
`;
});

html += `</div>`;
}

document.getElementById("checklist").innerHTML = html;

}

// toggle
function toggle(i,fase,idx){

dados[i].checklist[fase][idx].done =
!dados[i].checklist[fase][idx].done;

localStorage.setItem("erp_motorhome", JSON.stringify(dados));

abrir(i);

}

// voltar
function voltar(){
document.getElementById("telaCadastro").classList.remove("hidden");
document.getElementById("telaDetalhe").classList.add("hidden");
render();
}

// excluir veículo
function excluirAtual(){

if(confirm("Deseja excluir este veículo?")){

dados.splice(atual,1);
localStorage.setItem("erp_motorhome", JSON.stringify(dados));

voltar();
}
}

// exportar relatório
function exportar(){

let v = dados[atual];

let texto = `
RELATÓRIO ELÉTRICO MOTORHOME

Veículo: ${v.veiculo}
Responsável: ${v.responsavel}
Ano: ${v.ano}
Data: ${v.data}

-----------------------

`;

for(let f in v.checklist){
texto += `\n${f}\n`;

v.checklist[f].forEach(i=>{
texto += `[${i.done?"X":" "}] ${i.item}\n`;
});
}

let blob = new Blob([texto], {type:"text/plain"});
let link = document.createElement("a");

link.href = URL.createObjectURL(blob);
link.download = v.veiculo+"_relatorio.txt";
link.click();

}

// lista por ano
function render(){

let div = document.getElementById("lista");
div.innerHTML="";

let anos = {};

dados.forEach((d,i)=>{
if(!anos[d.ano]) anos[d.ano]=[];
anos[d.ano].push({d,i});
});

for(let ano in anos){

div.innerHTML += `<div class="year-title">📅 ${ano}</div>`;

anos[ano].forEach(obj=>{
div.innerHTML += `
<div class="vehicle" onclick="abrir(${obj.i})">
<b>🚐 ${obj.d.veiculo}</b><br>
<small>👤 ${obj.d.responsavel}</small>
</div>
`;
});

}

}

// checklist base completo
function getChecklistCompleto(){

return {
"FASE 1 - C.A":[
{item:"Iluminação fase/fase 2,5mm",done:false},
{item:"Tomadas fase/fase/terra 2,5mm",done:false}
],

"FASE 1 - C.C":[
{item:"Iluminação 12V",done:false},
{item:"Aquecedor passagem",done:false},
{item:"Placa solar",done:false},
{item:"Ar-condicionado",done:false},
{item:"Toldo e escada",done:false},
{item:"Geladeira",done:false}
],

"FASE 3":[
{item:"Organização cabos",done:false},
{item:"Baterias",done:false},
{item:"Inversor",done:false},
{item:"Fusíveis",done:false}
]

};

}

render();

</script>

</body>
</html>
