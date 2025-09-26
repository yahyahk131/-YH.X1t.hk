<!doctype html>
<html lang="ar">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>دردشة لشخصين</title>
<style>
  body{background:#000;color:#caffd6;font-family:Segoe UI,Roboto,Arial;display:flex;flex-direction:column;height:100vh;margin:0}
  header{padding:12px;background:#02120f;border-bottom:1px solid #0f0;display:flex;gap:8px;align-items:center}
  #roomInput,#nameInput{padding:8px;border-radius:6px;border:1px solid #0f0;background:#00120a;color:#caffd6}
  #chat{flex:1;overflow:auto;padding:12px;display:flex;flex-direction:column;gap:8px}
  .msg{max-width:75%;padding:10px;border-radius:12px;word-break:break-word}
  .me{align-self:flex-end;background:rgba(0,255,144,0.16);text-align:right}
  .them{align-self:flex-start;background:rgba(0,255,144,0.06);text-align:left}
  footer{display:flex;gap:8px;padding:10px;background:#02120f;border-top:1px solid #0f0}
  textarea{flex:1;padding:10px;border-radius:8px;background:#010f0a;color:#caffd6;border:1px solid #0f0;resize:none}
  button{background:#00ff90;color:#000;padding:8px 12px;border-radius:8px;border:none;font-weight:700}
  .sys{color:#9fffb8;font-size:13px;opacity:0.8;text-align:center}
</style>
</head>
<body>
  <header>
    <div style="flex:1;display:flex;gap:8px;align-items:center">
      <input id="nameInput" placeholder="اسمك (مثال: احمد)" />
      <input id="roomInput" placeholder="اسم الغرفة (مثال: room1)" />
      <button id="joinBtn">انضم</button>
    </div>
    <div style="font-weight:700">دردشة خاصة — YAHYA.HK</div>
  </header>

  <div id="chat"></div>

  <footer>
    <textarea id="msgInput" rows="2" placeholder="اكتب رسالة..."></textarea>
    <button id="sendBtn">إرسال</button>
  </footer>

  <!-- Socket.IO client (CDN) -->
  <script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>
  <script>
    // ====== عدّل السطر التالي إلى عنوان سيرفرك ======
    const SERVER = 'http://REPLACE_WITH_SERVER_IP:3000'; // مثال: http://192.168.1.110:3000 أو http://abc123.ngrok.io

    const socket = io(SERVER, { transports: ['websocket', 'polling'] });

    const joinBtn = document.getElementById('joinBtn');
    const sendBtn = document.getElementById('sendBtn');
    const chat = document.getElementById('chat');
    const msgInput = document.getElementById('msgInput');
    const roomInput = document.getElementById('roomInput');
    const nameInput = document.getElementById('nameInput');

    let currentRoom = null;
    let myName = null;

    function addMessage({user, text, time}, klass){
      const d = new Date(time || Date.now());
      const hh = d.getHours().toString().padStart(2,'0');
      const mm = d.getMinutes().toString().padStart(2,'0');
      const el = document.createElement('div');
      el.className = 'msg ' + klass;
      el.innerHTML = '<div style="font-size:12px;opacity:.8;">' + (user||'مستخدم') + ' • ' + hh+':'+mm + '</div><div style="margin-top:6px;">' + escapeHtml(text) + '</div>';
      chat.appendChild(el);
      chat.scrollTop = chat.scrollHeight;
    }

    function addSystem(text){
      const el = document.createElement('div');
      el.className = 'sys';
      el.innerText = text;
      chat.appendChild(el);
      chat.scrollTop = chat.scrollHeight;
    }

    joinBtn.onclick = () => {
      const room = roomInput.value.trim();
      const name = nameInput.value.trim();
      if(!room || !name){ alert('حط اسمك واسم الغرفة'); return; }
      currentRoom = room;
      myName = name;
      socket.emit('join-room', { room, user: name });
      addSystem('انضممت إلى ' + room);
    };

    sendBtn.onclick = sendMsg;
    msgInput.addEventListener('keydown', e=>{
      if(e.key === 'Enter' && !e.shiftKey){ e.preventDefault(); sendMsg(); }
    });

    function sendMsg(){
      const text = msgInput.value.trim();
      if(!text || !currentRoom) return alert('انضم للغرفة أولاً');
      socket.emit('chat-message', { room: currentRoom, text });
      msgInput.value = '';
    }

    socket.on('connect', ()=> addSystem('متصل بالسيرفر'));
    socket.on('disconnect', ()=> addSystem('انقطع الاتصال'));
    socket.on('system', (d) => addSystem(d.text));
    socket.on('chat-message', (d) => {
      // d = {user,text,time}
      const klass = (d.user === myName) ? 'me' : 'them';
      addMessage(d, klass);
    });

    function escapeHtml(unsafe) {
      return unsafe.replace(/[&<"'>]/g, function(m){ return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#039;'}[m]; });
    }
  </script>
</body>
</html>
