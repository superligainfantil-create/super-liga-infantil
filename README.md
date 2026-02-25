<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Super Liga Infantil</title>

<style>
body{
    font-family:Arial;
    margin:0;
    background:#f4f6f9;
}
header{
    background:linear-gradient(90deg,#0d47a1,#1976d2);
    color:white;
    padding:15px;
    text-align:center;
    font-size:22px;
}
nav{
    display:flex;
    background:white;
}
nav button{
    flex:1;
    padding:10px;
    border:none;
    background:white;
    font-weight:bold;
    cursor:pointer;
}
nav button:hover{
    background:#1976d2;
    color:white;
}
.section{
    display:none;
    padding:15px;
}
.active{
    display:block;
}
.card{
    background:white;
    padding:15px;
    margin-bottom:15px;
    border-radius:8px;
    box-shadow:0 2px 5px rgba(0,0,0,0.1);
}
button.action{
    background:#ff9800;
    color:white;
    border:none;
    padding:6px 10px;
    border-radius:4px;
    cursor:pointer;
}
table{
    width:100%;
    border-collapse:collapse;
    margin-top:10px;
}
th,td{
    border:1px solid #ddd;
    padding:6px;
    text-align:center;
}
th{
    background:#1976d2;
    color:white;
}
</style>
</head>
<body>

<header>⚽ SUPER LIGA INFANTIL</header>

<nav>
<button onclick="show('categorias')">Categorías</button>
<button onclick="show('fixture')">Fixture</button>
<button onclick="show('tabla')">Tabla</button>
</nav>

<div id="categorias" class="section active">
<div class="card">
<input id="catInput" placeholder="Nueva categoría">
<button class="action" onclick="crearCategoria()">Crear</button>
</div>
<div id="listaCategorias"></div>
</div>

<div id="fixture" class="section">
<select id="fixtureSelect"></select>
<button class="action" onclick="generarFixture()">Generar</button>
<div id="fixtureContainer"></div>
</div>

<div id="tabla" class="section">
<select id="tablaSelect"></select>
<button class="action" onclick="mostrarTabla()">Mostrar</button>
<div id="tablaContainer"></div>
</div>

<script>
let data = JSON.parse(localStorage.getItem("ligaData")) || {categorias:{}};
function save(){localStorage.setItem("ligaData",JSON.stringify(data));}
function show(id){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.getElementById(id).classList.add("active");
actualizarSelects();
}

/* =======================
   CATEGORÍAS
======================= */
function crearCategoria(){
let nombre=catInput.value.trim();
if(!nombre) return;
if(!data.categorias[nombre]){
data.categorias[nombre]={equipos:[],fixture:[]};
save();render();
}
catInput.value="";
}

function agregarEquipo(cat){
let nombre=prompt("Nombre equipo:");
if(!nombre)return;
data.categorias[cat].equipos.push({
nombre:nombre,
pj:0,pg:0,pe:0,pp:0,gf:0,gc:0,pts:0
});
save();render();
}

function render(){
listaCategorias.innerHTML="";
for(let c in data.categorias){
let html=`<div class="card"><b>${c}</b>
<button class="action" onclick="agregarEquipo('${c}')">+ Equipo</button><br><br>`;
data.categorias[c].equipos.forEach(e=>{
html+=`• ${e.nombre}<br>`;
});
html+="</div>";
listaCategorias.innerHTML+=html;
}
actualizarSelects();
}
render();

/* =======================
   FIXTURE
======================= */
function generarFixture(){
let cat=fixtureSelect.value;
let equipos=[...data.categorias[cat].equipos];
if(equipos.length<2) return alert("Mínimo 2 equipos");

if(equipos.length%2!==0) equipos.push({nombre:"LIBRE"});
let jornadas=equipos.length-1;
let mitad=equipos.length/2;
let fixture=[];

for(let j=0;j<jornadas;j++){
let ronda=[];
for(let i=0;i<mitad;i++){
let home=equipos[i];
let away=equipos[equipos.length-1-i];
if(home.nombre!="LIBRE"&&away.nombre!="LIBRE"){
ronda.push({home:home.nombre,away:away.nombre,golesHome:null,golesAway:null});
}}
equipos.splice(1,0,equipos.pop());
fixture.push(ronda);
}
data.categorias[cat].fixture=fixture;
save();
mostrarFixture(cat);
}

function mostrarFixture(cat){
fixtureContainer.innerHTML="";
data.categorias[cat].fixture.forEach((j,i)=>{
fixtureContainer.innerHTML+=`<div class="card"><b>Jornada ${i+1}</b><br>`+
j.map((p,idx)=>`
${p.home}
<input type="number" style="width:40px" onchange="guardarResultado('${cat}',${i},${idx},this.value,'home')">
-
<input type="number" style="width:40px" onchange="guardarResultado('${cat}',${i},${idx},this.value,'away')">
${p.away}<br>`).join("")+"</div>";
});
}

function guardarResultado(cat,j,i,val,side){
let p=data.categorias[cat].fixture[j][i];
if(side==="home")p.golesHome=parseInt(val);
else p.golesAway=parseInt(val);
actualizarTabla(cat);
save();
}

/* =======================
   TABLA
======================= */
function actualizarTabla(cat){
let equipos=data.categorias[cat].equipos;
equipos.forEach(e=>{e.pj=e.pg=e.pe=e.pp=e.gf=e.gc=e.pts=0;});
data.categorias[cat].fixture.forEach(j=>{
j.forEach(p=>{
if(p.golesHome!=null&&p.golesAway!=null){
let home=equipos.find(e=>e.nombre==p.home);
let away=equipos.find(e=>e.nombre==p.away);
home.pj++;away.pj++;
home.gf+=p.golesHome;home.gc+=p.golesAway;
away.gf+=p.golesAway;away.gc+=p.golesHome;
if(p.golesHome>p.golesAway){home.pg++;home.pts+=3;away.pp++;}
else if(p.golesHome<p.golesAway){away.pg++;away.pts+=3;home.pp++;}
else{home.pe++;away.pe++;home.pts++;away.pts++;}
}});});
}

function mostrarTabla(){
let cat=tablaSelect.value;
actualizarTabla(cat);
let equipos=[...data.categorias[cat].equipos];
equipos.sort((a,b)=>b.pts-a.pts||(b.gf-b.gc)-(a.gf-a.gc)||b.gf-a.gf);
tablaContainer.innerHTML="<table><tr><th>Equipo</th><th>PJ</th><th>PG</th><th>PE</th><th>PP</th><th>GF</th><th>GC</th><th>PTS</th></tr>"+
equipos.map(e=>`<tr><td>${e.nombre}</td><td>${e.pj}</td><td>${e.pg}</td><td>${e.pe}</td><td>${e.pp}</td><td>${e.gf}</td><td>${e.gc}</td><td>${e.pts}</td></tr>`).join("")+"</table>";
}

/* =======================
   SELECTS
======================= */
function actualizarSelects(){
fixtureSelect.innerHTML="";
tablaSelect.innerHTML="";
for(let c in data.categorias){
fixtureSelect.innerHTML+=`<option>${c}</option>`;
tablaSelect.innerHTML+=`<option>${c}</option>`;
}
}
</script>

</body>
</html>
