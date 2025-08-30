<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>7D All-in-One App</title>
<style>
  *{box-sizing:border-box;margin:0;padding:0;font-family:Arial,sans-serif;}
  body{background:url('7d_background.jpg') center/cover no-repeat;color:#fff;font-size:30px;min-height:100vh;display:flex;flex-direction:column;}
  header{padding:10px;background:rgba(0,0,0,0.6);display:flex;justify-content:space-between;align-items:center;}
  header h1{font-size:32px;}
  nav button{margin:0 5px;padding:10px 15px;font-size:30px;border:none;border-radius:10px;cursor:pointer;background:rgba(255,255,255,0.1);color:#fff;}
  nav button:hover{background:rgba(255,255,255,0.2);}
  main{flex:1;padding:20px;display:none;}
  section{display:none;}
  .active{display:block;}
  input,select,textarea,button{font-size:30px;padding:8px;border-radius:8px;margin:5px 0;}
  #calGrid{display:grid;grid-template-columns:repeat(7,1fr);gap:5px;}
  .cal-day{text-align:center;padding:5px;background:rgba(255,255,255,0.1);border-radius:5px;}
  .today{background:linear-gradient(90deg,#7b61ff,#00d2ff);color:#000;font-weight:700;}
  #tttBoard{display:grid;grid-template-columns:repeat(3,1fr);gap:5px;margin-top:10px;}
  .ttt-cell{background:rgba(255,255,255,0.1);padding:20px;text-align:center;font-weight:700;cursor:pointer;border-radius:10px;}
  .file-item{display:flex;justify-content:space-between;background:rgba(255,255,255,0.1);padding:10px;margin:5px 0;border-radius:8px;}
  canvas.analog{background:rgba(0,0,0,0.1);border-radius:50%;margin-top:10px;}
</style>
</head>
<body>
<header>
  <h1>7D All-in-One</h1>
  <nav>
    <button onclick="openPanel('home')">Home</button>
    <button onclick="openPanel('weather')">Weather</button>
    <button onclick="openPanel('clock')">Clock</button>
    <button onclick="openPanel('calendar')">Calendar</button>
    <button onclick="openPanel('game')">Game</button>
    <button onclick="openPanel('locker')">Locker</button>
    <button onclick="openPanel('calculator')">Calculator</button>
    <button onclick="openPanel('ai')">AI</button>
    <button onclick="openPanel('account')">Account</button>
  </nav>
</header>
<main id="mainPanel" class="active">

<section id="home" class="active">
  <h2>Welcome</h2>
  <p>यह एक विस्तारित offline + online app है। सभी फीचर्स का आनंद लें।</p>
</section>

<section id="weather">
  <h2>Weather</h2>
  <input id="weatherCity" placeholder="City Name">
  <button onclick="searchWeather()">Search</button>
  <div id="weatherResult"></div>
</section>

<section id="clock">
  <h2>Clock</h2>
  <div id="digitalClock">00:00:00</div>
  <canvas id="analog" width="200" height="200" class="analog"></canvas>
</section>

<section id="calendar">
  <h2>Calendar</h2>
  <div>
    <button onclick="prevMonth()">◀</button>
    <span id="calMonthYear"></span>
    <button onclick="nextMonth()">▶</button>
    <button onclick="goToday()">Today</button>
  </div>
  <div id="calGrid"></div>
</section>

<section id="game">
  <h2>Tic-Tac-Toe</h2>
  <label><input type="checkbox" id="aiToggle"> Play vs AI</label>
  <div id="tttBoard"></div>
  <div id="tttMsg"></div>
  <div>Score X:<span id="scoreX">0</span> O:<span id="scoreO">0</span> Draw:<span id="scoreD">0</span></div>
  <button onclick="resetGame()">Restart</button>
</section>

<section id="locker">
  <h2>Locker</h2>
  <div id="lockerMsg">Login करें तभी फाइल दिखेंगी।</div>
  <input type="file" id="lockerFile">
  <button onclick="saveLockerFile()">Save File</button>
  <div id="lockerFiles"></div>
</section>

<section id="calculator">
  <h2>Calculator</h2>
  <input type="text" id="calcInput" placeholder="Expression">
  <button onclick="calculate()">Calculate</button>
  <div id="calcResult"></div>
</section>

<section id="ai">
  <h2>AI Assistant</h2>
  <input type="text" id="aiQuestion" placeholder="Ask me">
  <button onclick="askAI()">Search</button>
  <div id="aiAnswer"></div>
</section>

<section id="account">
  <h2>Account</h2>
  <input id="regUser" placeholder="Username">
  <input id="regPass" type="password" placeholder="Password">
  <button onclick="register()">Create Account</button>
  <hr>
  <select id="userList"></select>
  <input id="logPass" type="password" placeholder="Password">
  <button onclick="login()">Login</button>
  <button onclick="logout()">Logout</button>
</section>

</main>

<script>
// ---------------- Panels ----------------
function openPanel(id){
  document.querySelectorAll('main section').forEach(s=>s.style.display='none');
  document.getElementById(id).style.display='block';
}

// ---------------- User ----------------
function loadUsers(){return JSON.parse(localStorage.getItem('a1_users')||'{}');}
function saveUsers(u){localStorage.setItem('a1_users',JSON.stringify(u));}
async function sha256hex(msg){const data=new TextEncoder().encode(msg);const hash=await crypto.subtle.digest('SHA-256',data);return Array.from(new Uint8Array(hash)).map(b=>b.toString(16).padStart(2,'0')).join('');}

async function register(){
  const u=document.getElementById('regUser').value.trim();
  const p=document.getElementById('regPass').value;
  if(!u||!p){alert('Fill username & password');return;}
  const users=loadUsers();
  if(users[u]){alert('Username exists');return;}
  const passHash=await sha256hex(p);
  users[u]={passHash,locker:[]};
  saveUsers(users);
  alert('Account created!');
  populateUserList();
}

async function login(){
  const sel=document.getElementById('userList');
  const u=sel.value;
  const p=document.getElementById('logPass').value;
  if(!u||!p){alert('Fill credentials');return;}
  const users=loadUsers();
  if(!users[u]){alert('No user');return;}
  const hash=await sha256hex(p);
  if(hash===users[u].passHash){localStorage.setItem('a1_logged','1'); localStorage.setItem('a1_user',u); alert('Logged in'); renderLockerFiles();}
  else alert('Invalid credentials');
}

function logout(){localStorage.removeItem('a1_logged');localStorage.removeItem('a1_user');renderLockerFiles();}
function isLoggedIn(){return localStorage.getItem('a1_logged')==='1';}
function getCurrentUser(){return localStorage.getItem('a1_user')||null;}
function populateUserList(){
  const sel=document.getElementById('userList');
  const users=loadUsers();
  sel.innerHTML='<option value="">-- select --</option>';
  Object.keys(users).forEach(u=>{const opt=document.createElement('option');opt.value=u;opt.innerText=u;sel.appendChild(opt);});
}
populateUserList();

// ---------------- Locker ----------------
function fileToBase64(file){return new Promise((res,rej)=>{const r=new FileReader();r.onload=()=>res(r.result);r.onerror=e=>rej(e);r.readAsDataURL(file);});}
async function saveLockerFile(){
  if(!isLoggedIn()){alert('Login first');return;}
  const f=document.getElementById('lockerFile').files[0]; if(!f){alert('Select file');return;}
  const data=await fileToBase64(f); const user=getCurrentUser();
  const users=loadUsers(); users[user].locker.push({name:f.name,data}); saveUsers(users);
  renderLockerFiles();
}
function renderLockerFiles(){
  const list=document.getElementById('lockerFiles'); list.innerHTML='';
  if(!isLoggedIn()){document.getElementById('lockerMsg').innerText='Login first'; return;}
  const user=getCurrentUser(); const users=loadUsers(); const arr=users[user].locker;
  if(arr.length===0){list.innerText='No files'; return;}
  arr.forEach((f,i)=>{const div=document.createElement('div');div.className='file-item';div.innerHTML=`${f.name} <button onclick="downloadFile(${i})">Download</button> <button onclick="deleteFile(${i})">Delete</button>`;list.appendChild(div);});
}
function downloadFile(i){const user=getCurrentUser();const users=loadUsers();const f=users[user].locker[i];const a=document.createElement('a');a.href=f.data;a.download=f.name;a.click();}
function deleteFile(i){const user=getCurrentUser();const users=loadUsers();users[user].locker.splice(i,1);saveUsers(users);renderLockerFiles();}

// ---------------- Weather ----------------
function seeded(s){let h=0;for(let i=0;i<s.length;i++)h=(h*31+s.charCodeAt(i))|0;return Math.abs(h);}
function generateWeather(seed){const conds=['Sunny','Cloudy','Rainy','Windy']; const temps=[20,22,24,26,28]; return {cond:conds[seed%conds.length],temp:temps[seed%temps.length]};}
function searchWeather(){
  const city=document.getElementById('weatherCity').value.trim(); if(!city){alert('City?'); return;}
  const w=generateWeather(seeded(city));
  document.getElementById('weatherResult').innerText=`${city}: ${w.cond}, ${w.temp}°C`;
}

// ---------------- Clock ----------------
function startClock(){setInterval(()=>{const d=new Date();document.getElementById('digitalClock').innerText=d.toLocaleTimeString();drawAnalog(d);},1000);}
const a=document.getElementById('analog').getContext('2d');
function drawAnalog(d){const c=a.canvas;const ctx=a;const w=c.width,h=c.height,cx=w/2,cy=h/2,r=90;ctx.clearRect(0,0,w,h);ctx.beginPath();ctx.arc(cx,cy,r,0,2*Math.PI);ctx.strokeStyle="#fff";ctx.stroke();drawHand(ctx,(d.getHours()%12+d.getMinutes()/60)*30,r*0.5,cx,cy);drawHand(ctx,(d.getMinutes()+d.getSeconds()/60)*6,r*0.7,cx,cy);drawHand(ctx,d.getSeconds()*6,r*0.9,cx,cy,'red');}
function drawHand(ctx,deg,len,cx,cy,color="#fff"){const rad=(deg-90)*Math.PI/180;ctx.beginPath();ctx.moveTo(cx,cy);ctx.lineTo(cx+Math.cos(rad)*len,cy+Math.sin(rad)*len);ctx.strokeStyle=color;ctx.lineWidth=5;ctx.stroke();}

// ---------------- Calendar ----------------
let calDate=new Date();
function renderCalendar(){
  const grid=document.getElementById('calGrid'); grid.innerHTML='';
  const year=calDate.getFullYear(); const month=calDate.getMonth();
  const firstDay=new Date(year,month,1).getDay();
  const days=new Date(year,month+1,0).getDate();
  const headers=['Mon','Tue','Wed','Thu','Fri','Sat','Sun'];
  headers.forEach(h=>{const el=document.createElement('div');el.className='cal-day';el.style.fontWeight='700';el.innerText=h;grid.appendChild(el);});
  let offset=(firstDay+6)%7;
  for(let i=0;i<offset;i++){const el=document.createElement('div');el.className='cal-day';grid.appendChild(el);}
  const today=new Date();
  for(let d=1;d<=days;d++){const el=document.createElement('div');el.className='cal-day';if(d===today.getDate()&&month===today.getMonth()&&year===today.getFullYear())el.classList.add('today');el.innerText=d;grid.appendChild(el);}
  document.getElementById('calMonthYear').innerText=calDate.toLocaleString('default',{month:'long'})+' '+year;
}
function prevMonth(){calDate=new Date(calDate.getFullYear(),calDate.getMonth()-1,1);renderCalendar();}
function nextMonth(){calDate=new Date(calDate.getFullYear(),calDate.getMonth()+1,1);renderCalendar();}
function goToday(){calDate=new Date();renderCalendar();}
renderCalendar();

// ---------------- Tic-Tac-Toe ----------------
let board=Array(9).fill(''),turn='X',scores={X:0,O:0,D:0};
function renderBoard(){const root=document.getElementById('tttBoard');root.innerHTML='';board.forEach((v,i)=>{const cell=document.createElement('div');cell.className='ttt-cell';cell.innerText=v;cell.onclick=()=>playMove(i);root.appendChild(cell);});}
function playMove(i){if(board[i]||checkWin(board))return;board[i]=turn;const res=checkWin(board);if(res){handleResult(res);renderBoard();return;}turn=turn==='X'?'O':'X';renderBoard();if(document.getElementById('aiToggle').checked&&turn==='O'){setTimeout(()=>{aiMove();const r=checkWin(board);if(r)handleResult(r);else turn='X';renderBoard();},250);}}
function aiMove(){const empties=board.map((v,i)=>v?null:i).filter(x=>x!==null);for(const idx of empties){const copy=board.slice();copy[idx]='O';if(checkWin(copy)==='O'){board[idx]='O';return;}}for(const idx of empties){const copy=board.slice();copy[idx]='X';if(checkWin(copy)==='X'){board[idx]='O';return;}}if(board[4]===''){board[4]='O';return;}const corners=[0,2,6,8].filter(i=>board[i]==='');if(corners.length>0){board[corners[Math.floor(Math.random()*corners.length)]]='O';return;}board[empties[Math.floor(Math.random()*empties.length)]]='O';}
function checkWin(b){const wins=[[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];for(const w of wins){const [a,b1,c]=w;if(b[a]&&b[a]===b[b1]&&b[a]===b[c])return b[a];}if(b.every(x=>x))return 'Draw';return null;}
function handleResult(res){if(res==='Draw'){document.getElementById('tttMsg').innerText='Draw!';scores.D++;}else{document.getElementById('tttMsg').innerText=res+' wins!';scores[res]++;}document.getElementById('scoreX').innerText=scores.X;document.getElementById('scoreO').innerText=scores.O;document.getElementById('scoreD').innerText=scores.D;setTimeout(()=>{board=Array(9).fill('');turn='X';document.getElementById('tttMsg').innerText='';renderBoard();},1200);}
function resetGame(){board=Array(9).fill('');turn='X';scores={X:0,O:0,D:0};document.getElementById('tttMsg').innerText='';renderBoard();}
renderBoard();

// ---------------- Calculator ----------------
function calculate(){const expr=document.getElementById('calcInput').value;try{document.getElementById('calcResult').innerText=eval(expr);}catch(e){document.getElementById('calcResult').innerText='Error';}}

// ---------------- AI ----------------
async function askAI(){
  const q=document.getElementById('aiQuestion').value.trim();
  if(!q){alert('Type question');return;}
  const key=prompt('Enter OpenAI API key'); if(!key)return;
  document.getElementById('aiAnswer').innerText='Loading...';
  const resp=await fetch('https://api.openai.com/v1/chat/completions',{method:'POST',headers:{'Content-Type':'application/json','Authorization':'Bearer '+key},body:JSON.stringify({model:'gpt-3.5-turbo',messages:[{role:'user',content:q}],temperature:0.7,max_tokens:150})});
  const data=await resp.json();
  document.getElementById('aiAnswer').innerText=data.choices?.[0]?.message?.content||'No answer';
}

// ---------------- Init ----------------
startClock();
</script>
</body>
</html>
