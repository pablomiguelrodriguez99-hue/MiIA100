# MiIA100
https://tu_usuario.github.io/MiIA.html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Okan Meta AI Dashboard v3.0</title>
<style>
body {margin:0; padding:0; font-family:sans-serif; background:#0F0F11; color:#fff;}
.container {display:flex; flex-wrap: wrap; gap:20px; padding:20px;}
.column {flex:1; min-width:300px; background: rgba(28,28,30,0.9); padding:15px; border-radius:10px; max-height:90vh; overflow-y:auto;}
h2 {color:#69C9D0; font-size:1.3em; margin-bottom:10px;}
.agent, .account {padding:8px; margin:5px 0; border-radius:5px; cursor:pointer; transition: all 0.3s ease;}
.agent.running {background:#69C9D0; animation: blink 1s infinite;}
.agent.idle {background:#555;}
.agent.sleeping {background:#FFA500;}
.account.active {background:#4EC6CD;}
.account.processing {background:#FF2D55;}
.account.idle {background:#555;}
button {margin-top:10px; padding:8px 12px; border:none; border-radius:5px; background:#FF2D55; color:#fff; cursor:pointer;}
input, select {width:100%; padding:8px; margin-top:5px; margin-bottom:10px; border-radius:5px; border:none;}
#log, #history {background:#111; height:150px; overflow-y:auto; margin-top:10px; padding:10px; border-radius:5px; font-size:12px;}
canvas {background:#111; border-radius:10px; margin-top:10px;}
@keyframes blink {0%,100%{opacity:1;}50%{opacity:0.4;}}
</style>
</head>
<body>

<div class="container">
  <div class="column">
    <h2>Agentes</h2>
    <div id="agentsList"></div>
  </div>
  <div class="column">
    <h2>Cuentas</h2>
    <div id="accountsList"></div>
    <div><strong>Historial:</strong>
      <div id="history"></div>
    </div>
  </div>
  <div class="column">
    <h2>Tareas</h2>
    <input type="text" id="taskInput" placeholder="Escribe una tarea">
    <select id="targetSelect">
      <option value="">Seleccionar Agente o Cuenta</option>
    </select>
    <button onclick="assignTask()">Asignar</button>
    <div id="log"></div>
    <canvas id="activityChart" width="300" height="200"></canvas>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
// Roles de agentes
const ROLES = { MANAGER:"Manager", ANALYST:"Analyst", CREATOR:"Creator", PLANNER:"Planner", EXECUTOR:"Executor" };

// Crear 100 agentes
const AI = { agents: [] };
for(let i=1;i<=100;i++){
    let role = i<=20?ROLES.ANALYST:i<=50?ROLES.CREATOR:i<=80?ROLES.PLANNER:i<=100?ROLES.EXECUTOR:ROLES.MANAGER;
    AI.agents.push({id:i, role, state:"idle", tasks:[], energy:1, totalActions:0});
}

// Crear 200 cuentas
const accounts = [];
for(let i=1;i<=100;i++) accounts.push({id:i, platform:"TikTok", name:`TikTok${i}`, tasks:[], completedTasks:[], status:"idle"});
for(let i=101;i<=200;i++) accounts.push({id:i, platform:"YouTube", name:`YT${i-100}`, tasks:[], completedTasks:[], status:"idle"});

// Render agentes
function renderAgents(){
    const el = document.getElementById("agentsList");
    el.innerHTML="";
    AI.agents.forEach(a=>{
        const div = document.createElement("div");
        div.className="agent "+a.state;
        div.textContent=`Agente ${a.id} | ${a.role} | ${a.state} | Energía: ${a.energy.toFixed(2)} | Acciones: ${a.totalActions}`;
        el.appendChild(div);
    });
}

// Render cuentas
function renderAccounts(){
    const el = document.getElementById("accountsList");
    el.innerHTML="";
    accounts.forEach(c=>{
        const div = document.createElement("div");
        div.className="account "+c.status;
        div.textContent=`${c.platform} | ${c.name} | ${c.status} | Pendientes: ${c.tasks.length}`;
        div.onclick = () => showHistory(c.id);
        el.appendChild(div);
    });
}

// Historial
function showHistory(accountId){
    const acc = accounts.find(a => a.id === accountId);
    const histEl = document.getElementById("history");
    histEl.innerHTML = `<strong>${acc.platform} ${acc.name}:</strong><br>`;
    if(acc.completedTasks.length===0) histEl.innerHTML += "Sin tareas completadas";
    else acc.completedTasks.forEach(t => histEl.innerHTML += `• ${t}<br>`);
}

// Actualizar select
function updateTargetSelect(){
    const sel=document.getElementById("targetSelect");
    sel.innerHTML='<option value="">Seleccionar Agente o Cuenta</option>';
    AI.agents.forEach(a=>sel.innerHTML+=`<option value="agent-${a.id}">Agente ${a.id}</option>`);
    accounts.forEach(c=>sel.innerHTML+=`<option value="account-${c.id}">${c.platform} | ${c.name}</option>`);
}

// Log
function log(msg){
    const el=document.getElementById("log");
    const entry=document.createElement("div");
    entry.textContent=`[${new Date().toLocaleTimeString()}] ${msg}`;
    el.appendChild(entry);
    el.scrollTop = el.scrollHeight;
}

// Asignar Tarea
function assignTask(){
    const task=document.getElementById("taskInput").value.trim();
    const target=document.getElementById("targetSelect").value;
    if(!task||!target) return log("❌ Escribe tarea y selecciona destino");
    const [type,id]=target.split("-");
    if(type==="agent"){
        const agent=AI.agents.find(a=>a.id==id);
        agent.tasks.push(task);
        agent.state="running";
        log(`✅ Tarea enviada a Agente ${id}: ${task}`);
    } else if(type==="account"){
        const acc=accounts.find(c=>c.id==id);
        acc.tasks.push(task);
        acc.status="active";
        log(`✅ Tarea enviada a ${acc.platform} ${acc.name}: ${task}`);
    }
    renderAgents();
    renderAccounts();
    document.getElementById("taskInput").value="";
}

// Loop agentes
function agentLoop() {
    AI.agents.forEach(agent => {
        if(agent.tasks.length > 0 && agent.energy>0){
            const task = agent.tasks.shift();
            agent.totalActions++;
            agent.energy -= 0.05 + Math.random()*0.05;

            // Escoger cuenta aleatoria
            const acc = accounts[Math.floor(Math.random() * accounts.length)];
            acc.tasks.push(task);
            acc.status="processing";

            // Completar tarea tras 1-3s
            setTimeout(() => {
                const completedTask = acc.tasks.shift();
                if(completedTask) acc.completedTasks.push(completedTask);
                if(acc.tasks.length===0) acc.status="idle";
                renderAccounts();
            }, 1000 + Math.random()*2000);

            agent.state="running";
        } else if(agent.tasks.length===0 && agent.energy>0){
            agent.state="idle";
        } else if(agent.energy<=0){
            agent.state="sleeping";
            agent.energy += 0.02;
        }
    });
    renderAgents();
    renderAccounts();
    updateChart();
}
setInterval(agentLoop,2000);

// Inicializar
renderAgents();
renderAccounts();
updateTargetSelect();
log("🧠 Okan Meta AI v3.0 listo con 100 agentes y 200 cuentas.");

// === Gráfica de actividad ===
const ctx = document.getElementById('activityChart').getContext('2d');
const chart = new Chart(ctx, {
    type:'bar',
    data:{
        labels:['Running','Idle','Sleeping','Active Accounts','Processing Accounts'],
        datasets:[{
            label:'Actividad en tiempo real',
            data:[0,0,0,0,0],
            backgroundColor:['#69C9D0','#555','#FFA500','#4EC6CD','#FF2D55']
        }]
    },
    options:{responsive:true, animation:{duration:500}, plugins:{legend:{display:false}}}
});

function updateChart(){
    const running = AI.agents.filter(a=>a.state==='running').length;
    const idle = AI.agents.filter(a=>a.state==='idle').length;
    const sleeping = AI.agents.filter(a=>a.state==='sleeping').length;
    const activeAcc = accounts.filter(a=>a.status==='active').length;
    const processingAcc = accounts.filter(a=>a.status==='processing').length;
    chart.data.datasets[0].data = [running,idle,sleeping,activeAcc,processingAcc];
    chart.update();
}
</script>

</body>
</html>
