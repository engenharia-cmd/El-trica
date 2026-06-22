<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>ERP Oficina Elétrica Motorhome</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    font-family: Arial;
    background:#f4f6f9;
    margin:0;
    padding:20px;
}

.container{
    max-width:1200px;
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
    background:#2563eb;
    color:#fff;
    border:none;
    border-radius:6px;
}

.vehicle{
    padding:12px;
    margin:8px 0;
    background:#f9fafb;
    border-left:5px solid #2563eb;
    cursor:pointer;
}

.vehicle:hover{ background:#eef2ff; }

.status{
    font-size:12px;
    padding:3px 8px;
    border-radius:5px;
    color:#fff;
}

.red{ background:#ef4444; }
.yellow{ background:#f59e0b; }
.green{ background:#10b981; }
.gray{ background:#6b7280; }

.checklist{
    background:#f1f5f9;
    padding:10px;
    border-radius:8px;
    margin-top:10px;
}

.item{ display:block; margin:5px 0; }

.hidden{ display:none; }

.bar-bg{
    background:#e5e7eb;
    height:10px;
    border-radius:5px;
    overflow:hidden;
}

.bar{
    height:10px;
    background:#10b981;
}
</style>
</head>

<body>

<div class="container">

<h1>🚐 ERP Oficina Elétrica Motorhome</h1>

<div class="card">
<h3>Cadastro de Veículo</h3>
<input id="veiculo" placeholder="Nome do veículo">
<input id="responsavel" placeholder="Responsável">
<button onclick="salvar()">Salvar</button>
</div>

<div class="card">
<h3>📊 Progresso da Oficina</h3>
<div id="painel"></div>
</div>

<div class="card">
<h3>Veículos</h3>
<div id="lista"></div>
</div>

<div class="card hidden" id="detalhe">
<h3 id="titulo"></h3>
<button onclick="voltar()">⬅ Voltar</button>
<div id="checklist"></div>
</div>

</div>

<script>

let dados = JSON.parse(localStorage.getItem("erp_motorhome")) || [];
let atual = null;

/* =========================
CHECKLIST COMPLETO ORIGINAL
========================= */
function checklistBase(){

return {

"FASE ELÉTRICA — PASSAGEM DE CABOS C.A.":[
"Circuito de iluminação fase/fase (amarelo/cinza – 2,5mm)",
"Circuito de tomadas fase/fase/terra (azul/branco/verde – 2,5mm)"
],

"FASE ELÉTRICA — PASSAGEM DE CABOS C.C.":[
"Circuito iluminação 12V (preto/vermelho – 2,5mm)",
"Cabo aquecedor de passagem (PP 4x1 dupla isolação)",
"Fiação da placa solar do teto até a central (6mm)",
"Fiação do ar-condicionado (conforme manual/dimensionamento)",
"Fiação do toldo e escada (2,5mm)",
"Tomada 12V base banco motorista (4mm)",
"Fiação da geladeira (6mm)",
"LED da pia (2,5mm quando aplicável)",
"Motor do slide-out até central (6mm)",
"Válvulas de detritos/servida (PP 2x1 quando aplicável)",
"Boias de água limpa/suja/detritos (PP 2x0,75)",
"Estabilizador entrada (3 cabos 2,5mm)",
"Estabilizador saída (2 cabos 2,5mm)",
"Circuito iluminação/tomada slide-out (7 cores 2,5mm)",
"Tomada externa (PP 3x4mm)",
"Chicote elétrico automotivo (PP 0,5)"
],

"SISTEMA DE BATERIAS":[
"Cabo do gerenciador de bateria (2,5mm vermelho)",
"Cabos entre baterias e barramento central",
"Cabo de rede do painel até central (CAT6)",
"Interruptor da usina (2,5mm)"
],

"SISTEMAS HIDRÁULICOS ELÉTRICOS":[
"Cabo da bomba de água limpa (6mm)",
"Cabo 6mm vermelho (1 por bomba)",
"Cabo do inversor (PP 0,75 botão)"
],

"FASE 2 — LIGAÇÕES GERAIS":[
"Ligação de tomadas 220V e 12V",
"Ligação de lâmpadas e LEDs",
"Ligação de luz externa (abaixo do toldo)",
"Ligação da escada",
"Ligação do toldo",
"Ligação do slide-out",
"Ligação do estabilizador",
"Ligação do sistema de boia",
"Ligação do ar-condicionado",
"Ligação da placa aquecedora de passagem",
"Ligação do LED da pia",
"Ligação da iluminação de leitura"
],

"FASE 3 — CENTRAL ELETROELETRÔNICA":[
"Organização e roteamento de todos os cabos",
"Fixação dos equipamentos",
"Ligação da usina",
"Estabilizador de tensão",
"Canaletas e passa-fios",
"Ligação das baterias e barramento CC",
"Carregador DC-DC",
"Controlador solar",
"Chave geral",
"Bloco de fusíveis",
"Inversor",
"Quadro QBC",
"Detector de gás",
"Banco elétrico"
],

"PAINEL DE COMANDO":[
"Botões das bombas",
"Botão do inversor",
"Botão do carregador",
"Painel de nível de água",
"Conferência geral final"
]

};

}

/* ========================= */

function salvar(){

let v=document.getElementById("veiculo").value.trim();
let r=document.getElementById("responsavel").value.trim();

if(!v||!r) return alert("Preencha tudo");

if(dados.find(x=>x.veiculo.toLowerCase()===v.toLowerCase()))
return alert("Já existe");

dados.push({
veiculo:v,
responsavel:r,
checklist:JSON.parse(JSON.stringify(checklistBase()))
});

localStorage.setItem("erp_motorhome",JSON.stringify(dados));

render();

}

function progresso(v){

let total=0,done=0;

for(let f in v.checklist){
v.checklist[f].forEach(i=>{
total++;
if(i.done) done++;
});
}

return total?Math.round((done/total)*100):0;

}

function status(p){
if(p===100) return {t:"Concluído",c:"gray"};
if(p>=81) return {t:"Quase finalizado",c:"green"};
if(p>=21) return {t:"Em andamento",c:"yellow"};
return {t:"Iniciado",c:"red"};
}

function abrir(i){

atual=i;
let v=dados[i];

document.getElementById("detalhe").classList.remove("hidden");

document.getElementById("titulo").innerText=
`🚐 ${v.veiculo} - ${v.responsavel}`;

let html="";

for(let f in v.checklist){

html+=`<div class="checklist"><b>${f}</b>`;

v.checklist[f].forEach((c,idx)=>{
html+=`
<label class="item">
<input type="checkbox" ${c.done?"checked":""}
onchange="toggle(${i},'${f}',${idx})">
${c.item}
</label>`;
});

html+="</div>";
}

document.getElementById("checklist").innerHTML=html;

}

function toggle(i,f,idx){

dados[i].checklist[f][idx].done =
!dados[i].checklist[f][idx].done;

localStorage.setItem("erp_motorhome",JSON.stringify(dados));

render();
abrir(i);

}

function voltar(){
document.getElementById("detalhe").classList.add("hidden");
render();
}

function render(){

let div=document.getElementById("lista");
let painel=document.getElementById("painel");

div.innerHTML="";
painel.innerHTML="";

dados.forEach((d,i)=>{

let p=progresso(d);
let s=status(p);

div.innerHTML+=`
<div class="vehicle" onclick="abrir(${i})">
<b>🚐 ${d.veiculo}</b><br>
<small>👤 ${d.responsavel}</small><br>
<span class="status ${s.c}">${s.t} - ${p}%</span>

<div class="bar-bg">
<div class="bar" style="width:${p}%"></div>
</div>

</div>
`;

painel.innerHTML+=`
<div style="margin-bottom:10px;">
${d.veiculo}
<div class="bar-bg">
<div class="bar" style="width:${p}%"></div>
</div>
<small>${p}% concluído</small>
</div>
`;

});

}

render();

</script>

</body>
</html>
