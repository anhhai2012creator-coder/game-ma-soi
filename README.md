<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Ma Sói Online - PeerJS</title>
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: radial-gradient(circle at top, #26314d, #0b1020 60%);
      color: #f4f6ff;
      min-height: 100vh;
    }
    .app {
      max-width: 1100px;
      margin: 0 auto;
      padding: 18px;
    }
    h1, h2, h3 { margin: 0 0 12px; }
    .card {
      background: rgba(255, 255, 255, 0.08);
      border: 1px solid rgba(255, 255, 255, 0.14);
      border-radius: 18px;
      padding: 16px;
      box-shadow: 0 16px 40px rgba(0, 0, 0, 0.24);
      backdrop-filter: blur(10px);
    }
    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 14px;
    }
    .row {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      align-items: center;
    }
    input, select, button, textarea {
      font: inherit;
      border: 0;
      border-radius: 12px;
      padding: 11px 12px;
    }
    input, select, textarea {
      background: rgba(255, 255, 255, 0.92);
      color: #111827;
      min-width: 0;
    }
    input { flex: 1; }
    button {
      background: #ffd166;
      color: #231f20;
      cursor: pointer;
      font-weight: 700;
      transition: transform 0.12s, opacity 0.12s;
    }
    button:hover { transform: translateY(-1px); }
    button:disabled { opacity: 0.45; cursor: not-allowed; transform: none; }
    .danger { background: #ff6b6b; color: white; }
    .ok { background: #6ee7b7; color: #063828; }
    .blue { background: #93c5fd; color: #0b2850; }
    .muted { color: #b9c1d9; }
    .hidden { display: none !important; }
    .room-code {
      font-size: 24px;
      letter-spacing: 2px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 14px;
      padding: 10px 14px;
      display: inline-block;
    }
    .players {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
      gap: 10px;
    }
    .player {
      background: rgba(255, 255, 255, 0.09);
      border-radius: 14px;
      padding: 12px;
      border: 1px solid rgba(255, 255, 255, 0.12);
    }
    .dead { opacity: 0.45; filter: grayscale(1); }
    .tag {
      display: inline-block;
      padding: 4px 8px;
      border-radius: 999px;
      background: rgba(255, 255, 255, 0.16);
      font-size: 12px;
      margin-top: 6px;
    }
    .role-box {
      background: linear-gradient(135deg, #ffd166, #fca311);
      color: #1f2937;
      border-radius: 18px;
      padding: 16px;
      font-weight: 700;
    }
    .log, .chat {
      height: 230px;
      overflow-y: auto;
      background: rgba(0, 0, 0, 0.24);
      border-radius: 14px;
      padding: 12px;
      line-height: 1.45;
    }
    .log p, .chat p { margin: 0 0 8px; }
    .chat-input { display: flex; gap: 8px; margin-top: 8px; }
    .phase {
      font-size: 18px;
      font-weight: 800;
      padding: 8px 12px;
      border-radius: 999px;
      background: rgba(255, 255, 255, 0.16);
      display: inline-block;
    }
    .vote-list {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
      gap: 8px;
    }
    .small { font-size: 13px; }
    @media (max-width: 800px) {
      .grid { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <div class="app">
    <h1>🐺 Ma Sói Online</h1>
    <p class="muted">Bản HTML một file, chơi bằng mã phòng PeerJS. Người tạo phòng là quản trò/host.</p>

    <div id="setup" class="card">
      <h2>Vào game</h2>
      <div class="row">
        <input id="nameInput" placeholder="Tên của bạn" maxlength="18" />
      </div>
      <div class="row" style="margin-top: 10px;">
        <button id="createBtn">Tạo phòng</button>
        <input id="roomInput" placeholder="Nhập mã phòng" maxlength="30" />
        <button id="joinBtn" class="blue">Tham gia</button>
      </div>
      <p id="setupMsg" class="muted small"></p>
    </div>

    <div id="game" class="hidden">
      <div class="grid">
        <div class="card">
          <h2>Phòng</h2>
          <p>Mã phòng:</p>
          <div id="roomCode" class="room-code">...</div>
          <p id="connectionInfo" class="muted small"></p>
          <div style="margin-top: 12px;" class="row">
            <button id="copyBtn" class="blue">Copy mã</button>
            <button id="startBtn" class="ok hidden">Bắt đầu game</button>
            <button id="nextPhaseBtn" class="hidden">Sang pha tiếp</button>
          </div>
        </div>

        <div class="card">
          <h2>Trạng thái</h2>
          <p><span id="phaseLabel" class="phase">Lobby</span></p>
          <div id="roleBox" class="role-box hidden"></div>
          <p id="turnHint" class="muted"></p>
        </div>
      </div>

      <div class="grid" style="margin-top: 14px;">
        <div class="card">
          <h2>Người chơi</h2>
          <div id="players" class="players"></div>
        </div>

        <div class="card">
          <h2>Hành động</h2>
          <div id="actionArea"></div>
        </div>
      </div>

      <div class="grid" style="margin-top: 14px;">
        <div class="card">
          <h2>Nhật ký</h2>
          <div id="log" class="log"></div>
        </div>
        <div class="card">
          <h2>Chat</h2>
          <div id="chat" class="chat"></div>
          <div class="chat-input">
            <input id="chatInput" placeholder="Nhắn tin..." />
            <button id="sendChatBtn">Gửi</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <script>
    const $ = (id) => document.getElementById(id);

    const ROLE_INFO = {
      werewolf: { name: 'Ma sói', emoji: '🐺', desc: 'Ban đêm chọn một người để cắn.' },
      seer: { name: 'Tiên tri', emoji: '🔮', desc: 'Ban đêm soi vai trò của một người.' },
      doctor: { name: 'Bảo vệ', emoji: '🛡️', desc: 'Ban đêm cứu một người khỏi bị cắn.' },
      villager: { name: 'Dân làng', emoji: '👨‍🌾', desc: 'Ban ngày thảo luận và bỏ phiếu tìm ma sói.' }
    };

    const state = {
      isHost: false,
      peer: null,
      hostConn: null,
      conns: {},
      myId: '',
      myName: '',
      roomId: '',
      game: {
        phase: 'lobby',
        day: 0,
        players: {},
        logs: [],
        chat: [],
        night: { wolfTarget: null, doctorTarget: null, seerChecks: {} },
        votes: {},
        winner: null
      }
    };

    function shortId() {
      return Math.random().toString(36).slice(2, 8).toUpperCase();
    }

    function addLog(text) {
      state.game.logs.push(text);
      if (state.game.logs.length > 80) state.game.logs.shift();
    }

    function broadcast() {
      if (!state.isHost) return;
      const payload = { type: 'state', game: state.game };
      Object.values(state.conns).forEach(conn => conn.open && conn.send(payload));
      render();
    }

    function sendToHost(payload) {
      if (state.isHost) handleClientMessage({ ...payload, from: state.myId });
      else if (state.hostConn && state.hostConn.open) state.hostConn.send(payload);
    }

    function alivePlayers() {
      return Object.values(state.game.players).filter(p => p.alive);
    }

    function aliveWerewolves() {
      return alivePlayers().filter(p => p.role === 'werewolf');
    }

    function aliveVillagers() {
      return alivePlayers().filter(p => p.role !== 'werewolf');
    }

    function makeRoles(count) {
      let roles = [];
      if (count === 3) roles = ['werewolf', 'seer', 'villager'];
      else if (count === 4) roles = ['werewolf', 'seer', 'doctor', 'villager'];
      else if (count <= 6) roles = ['werewolf', 'werewolf', 'seer', 'doctor', 'villager', 'villager'];
      else roles = ['werewolf', 'werewolf', 'seer', 'doctor', 'villager', 'villager', 'villager', 'villager'];
      roles = roles.slice(0, count);
      while (roles.length < count) roles.push('villager');
      return roles.sort(() => Math.random() - 0.5);
    }

    function checkWin() {
      const wolves = aliveWerewolves().length;
      const villagers = aliveVillagers().length;
      if (wolves === 0) return 'Dân làng thắng!';
      if (wolves >= villagers) return 'Ma sói thắng!';
      return null;
    }

    function startGame() {
      const players = Object.values(state.game.players);
      if (players.length < 3) {
        alert('Cần ít nhất 3 người để chơi bản này.');
        return;
      }
      const roles = makeRoles(players.length);
      players.forEach((p, i) => {
        p.role = roles[i];
        p.alive = true;
      });
      state.game.phase = 'night';
      state.game.day = 1;
      state.game.winner = null;
      state.game.night = { wolfTarget: null, doctorTarget: null, seerChecks: {} };
      state.game.votes = {};
      state.game.logs = [];
      addLog('Game bắt đầu. Đêm đầu tiên buông xuống.');
      broadcast();
    }

    function nextPhase() {
      if (state.game.winner) return;
      if (state.game.phase === 'night') {
        resolveNight();
        state.game.phase = 'day';
        addLog('Trời sáng. Mọi người thảo luận và bỏ phiếu.');
      } else if (state.game.phase === 'day') {
        resolveVotes();
        if (!state.game.winner) {
          state.game.phase = 'night';
          state.game.day += 1;
          state.game.night = { wolfTarget: null, doctorTarget: null, seerChecks: {} };
          state.game.votes = {};
          addLog('Đêm lại đến.');
        }
      }
      state.game.winner = checkWin();
      if (state.game.winner) {
        state.game.phase = 'ended';
        addLog(state.game.winner);
      }
      broadcast();
    }

    function resolveNight() {
      const wolfTarget = state.game.night.wolfTarget;
      const doctorTarget = state.game.night.doctorTarget;
      if (wolfTarget && wolfTarget !== doctorTarget && state.game.players[wolfTarget]) {
        state.game.players[wolfTarget].alive = false;
        addLog(`${state.game.players[wolfTarget].name} đã bị ma sói cắn.`);
      } else if (wolfTarget && wolfTarget === doctorTarget) {
        addLog('Đêm qua có người bị cắn nhưng đã được bảo vệ cứu.');
      } else {
        addLog('Đêm qua không ai bị cắn.');
      }
    }

    function resolveVotes() {
      const counts = {};
      Object.values(state.game.votes).forEach(target => {
        counts[target] = (counts[target] || 0) + 1;
      });
      let top = null;
      let topCount = 0;
      let tie = false;
      Object.entries(counts).forEach(([target, count]) => {
        if (count > topCount) {
          top = target;
          topCount = count;
          tie = false;
        } else if (count === topCount) {
          tie = true;
        }
      });
      if (!top) {
        addLog('Không ai bị treo cổ vì chưa có phiếu.');
        return;
      }
      if (tie) {
        addLog('Hòa phiếu. Không ai bị treo cổ.');
        return;
      }
      if (state.game.players[top]) {
        state.game.players[top].alive = false;
        addLog(`${state.game.players[top].name} bị treo cổ.`);
      }
    }

    function handleClientMessage(msg) {
      if (!state.isHost) return;
      const from = msg.from;
      if (msg.type === 'join') {
        if (state.game.phase !== 'lobby') return;
        state.game.players[from] = {
          id: from,
          name: msg.name || 'Người chơi',
          role: null,
          alive: true
        };
        addLog(`${state.game.players[from].name} đã vào phòng.`);
        broadcast();
      }
      if (msg.type === 'chat') {
        const player = state.game.players[from];
        if (!player) return;
        state.game.chat.push({ name: player.name, text: msg.text, time: new Date().toLocaleTimeString('vi-VN') });
        if (state.game.chat.length > 80) state.game.chat.shift();
        broadcast();
      }
      if (msg.type === 'nightAction') {
        const player = state.game.players[from];
        if (!player || !player.alive || state.game.phase !== 'night') return;
        if (player.role === 'werewolf') state.game.night.wolfTarget = msg.target;
        if (player.role === 'doctor') state.game.night.doctorTarget = msg.target;
        if (player.role === 'seer') {
          const target = state.game.players[msg.target];
          if (target) state.game.night.seerChecks[from] = { target: msg.target, role: target.role };
        }
        broadcast();
      }
      if (msg.type === 'vote') {
        const player = state.game.players[from];
        if (!player || !player.alive || state.game.phase !== 'day') return;
        state.game.votes[from] = msg.target;
        broadcast();
      }
    }

    function setupConn(conn) {
      state.conns[conn.peer] = conn;
      conn.on('data', (msg) => handleClientMessage({ ...msg, from: conn.peer }));
      conn.on('close', () => {
        const p = state.game.players[conn.peer];
        if (p) addLog(`${p.name} đã mất kết nối.`);
        delete state.conns[conn.peer];
        broadcast();
      });
    }

    function createRoom() {
      const name = $('nameInput').value.trim() || 'Host';
      state.isHost = true;
      state.myName = name;
      state.roomId = 'MS-' + shortId();
      state.peer = new Peer(state.roomId);
      $('setupMsg').textContent = 'Đang tạo phòng...';

      state.peer.on('open', (id) => {
        state.myId = id;
        state.game.players[id] = { id, name, role: null, alive: true };
        addLog(`${name} đã tạo phòng.`);
        $('setup').classList.add('hidden');
        $('game').classList.remove('hidden');
        render();
      });
      state.peer.on('connection', setupConn);
      state.peer.on('error', (err) => $('setupMsg').textContent = 'Lỗi tạo phòng: ' + err.type);
    }

    function joinRoom() {
      const name = $('nameInput').value.trim() || 'Người chơi';
      const room = $('roomInput').value.trim();
      if (!room) return alert('Nhập mã phòng trước.');
      state.isHost = false;
      state.myName = name;
      state.roomId = room;
      state.peer = new Peer();
      $('setupMsg').textContent = 'Đang kết nối...';

      state.peer.on('open', (id) => {
        state.myId = id;
        state.hostConn = state.peer.connect(room);
        state.hostConn.on('open', () => {
          state.hostConn.send({ type: 'join', name });
          $('setup').classList.add('hidden');
          $('game').classList.remove('hidden');
          render();
        });
        state.hostConn.on('data', (msg) => {
          if (msg.type === 'state') {
            state.game = msg.game;
            render();
          }
        });
      });
      state.peer.on('error', (err) => $('setupMsg').textContent = 'Lỗi kết nối: ' + err.type);
    }

    function roleText(role) {
      if (!role) return 'Chưa chia vai';
      const info = ROLE_INFO[role];
      return `${info.emoji} ${info.name}`;
    }

    function renderPlayers() {
      const me = state.game.players[state.myId];
      $('players').innerHTML = Object.values(state.game.players).map(p => {
        const canSeeRole = p.id === state.myId || state.game.phase === 'ended';
        const voteCount = Object.values(state.game.votes).filter(v => v === p.id).length;
        return `
          <div class="player ${p.alive ? '' : 'dead'}">
            <strong>${p.name}${p.id === state.myId ? ' (bạn)' : ''}</strong><br>
            <span class="muted small">${p.alive ? 'Còn sống' : 'Đã chết'}</span><br>
            <span class="tag">${canSeeRole ? roleText(p.role) : 'Vai trò ẩn'}</span>
            ${state.game.phase === 'day' && p.alive ? `<span class="tag">${voteCount} phiếu</span>` : ''}
          </div>
        `;
      }).join('');
    }

    function renderActionArea() {
      const me = state.game.players[state.myId];
      const alive = alivePlayers();
      const area = $('actionArea');
      if (!me) {
        area.innerHTML = '<p class="muted">Đang chờ dữ liệu phòng...</p>';
        return;
      }
      if (state.game.phase === 'lobby') {
        area.innerHTML = '<p class="muted">Cần ít nhất 3 người. Host bấm bắt đầu để chia vai.</p>';
        return;
      }
      if (state.game.phase === 'ended') {
        area.innerHTML = `<h3>${state.game.winner}</h3><p class="muted">Tải lại trang để tạo ván mới.</p>`;
        return;
      }
      if (!me.alive) {
        area.innerHTML = '<p class="muted">Bạn đã chết. Bạn có thể xem tiếp nhưng không thể hành động.</p>';
        return;
      }
      if (state.game.phase === 'night') {
        if (me.role === 'villager') {
          area.innerHTML = '<p class="muted">Bạn là dân làng. Ban đêm bạn ngủ.</p>';
          return;
        }
        const targets = alive.filter(p => p.id !== state.myId || me.role === 'doctor');
        const options = targets.map(p => `<option value="${p.id}">${p.name}</option>`).join('');
        let title = 'Chọn mục tiêu';
        if (me.role === 'werewolf') title = 'Ma sói chọn người để cắn';
        if (me.role === 'doctor') title = 'Bảo vệ chọn người để cứu';
        if (me.role === 'seer') title = 'Tiên tri chọn người để soi';
        const checked = state.game.night.seerChecks[state.myId];
        area.innerHTML = `
          <h3>${title}</h3>
          <div class="row">
            <select id="targetSelect">${options}</select>
            <button onclick="submitNightAction()">Xác nhận</button>
          </div>
          ${checked ? `<p class="tag">Kết quả soi: ${state.game.players[checked.target].name} là ${roleText(checked.role)}</p>` : ''}
          <p class="muted small">Host bấm sang pha tiếp sau khi mọi vai quan trọng đã chọn.</p>
        `;
        return;
      }
      if (state.game.phase === 'day') {
        const options = alive.filter(p => p.id !== state.myId).map(p => `<option value="${p.id}">${p.name}</option>`).join('');
        const voted = state.game.votes[state.myId];
        area.innerHTML = `
          <h3>Bỏ phiếu treo cổ</h3>
          <div class="row">
            <select id="voteSelect">${options}</select>
            <button onclick="submitVote()">Bỏ phiếu</button>
          </div>
          <p class="muted small">Phiếu của bạn: ${voted ? state.game.players[voted].name : 'chưa bỏ'}</p>
        `;
      }
    }

    window.submitNightAction = function() {
      const select = $('targetSelect');
      if (!select) return;
      sendToHost({ type: 'nightAction', target: select.value });
    };

    window.submitVote = function() {
      const select = $('voteSelect');
      if (!select) return;
      sendToHost({ type: 'vote', target: select.value });
    };

    function render() {
      $('roomCode').textContent = state.roomId || '...';
      $('connectionInfo').textContent = state.isHost
        ? 'Bạn là host. Hãy gửi mã phòng cho bạn bè.'
        : 'Bạn là người chơi. Đang kết nối tới host.';

      $('startBtn').classList.toggle('hidden', !state.isHost || state.game.phase !== 'lobby');
      $('nextPhaseBtn').classList.toggle('hidden', !state.isHost || !['night', 'day'].includes(state.game.phase));

      const phaseName = {
        lobby: 'Lobby',
        night: `Đêm ${state.game.day}`,
        day: `Ngày ${state.game.day}`,
        ended: 'Kết thúc'
      }[state.game.phase] || state.game.phase;
      $('phaseLabel').textContent = phaseName;

      const me = state.game.players[state.myId];
      if (me && me.role) {
        const info = ROLE_INFO[me.role];
        $('roleBox').classList.remove('hidden');
        $('roleBox').innerHTML = `Vai của bạn: ${info.emoji} ${info.name}<br><span class="small">${info.desc}</span>`;
      } else {
        $('roleBox').classList.add('hidden');
      }

      if (state.game.phase === 'night') $('turnHint').textContent = 'Ban đêm: ma sói cắn, bảo vệ cứu, tiên tri soi.';
      else if (state.game.phase === 'day') $('turnHint').textContent = 'Ban ngày: thảo luận trong chat rồi bỏ phiếu.';
      else $('turnHint').textContent = 'Mời người chơi vào phòng.';

      renderPlayers();
      renderActionArea();

      $('log').innerHTML = state.game.logs.map(x => `<p>• ${x}</p>`).join('');
      $('log').scrollTop = $('log').scrollHeight;
      $('chat').innerHTML = state.game.chat.map(m => `<p><strong>${m.name}</strong> <span class="muted small">${m.time}</span><br>${escapeHtml(m.text)}</p>`).join('');
      $('chat').scrollTop = $('chat').scrollHeight;
    }

    function escapeHtml(text) {
      return String(text).replace(/[&<>'"]/g, ch => ({
        '&': '&amp;', '<': '&lt;', '>': '&gt;', "'": '&#039;', '"': '&quot;'
      }[ch]));
    }

    $('createBtn').onclick = createRoom;
    $('joinBtn').onclick = joinRoom;
    $('startBtn').onclick = startGame;
    $('nextPhaseBtn').onclick = nextPhase;
    $('copyBtn').onclick = () => navigator.clipboard.writeText(state.roomId);
    $('sendChatBtn').onclick = () => {
      const text = $('chatInput').value.trim();
      if (!text) return;
      $('chatInput').value = '';
      sendToHost({ type: 'chat', text });
    };
    $('chatInput').addEventListener('keydown', e => {
      if (e.key === 'Enter') $('sendChatBtn').click();
    });
  </script>
</body>
</html>
