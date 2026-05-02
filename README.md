<!doctype html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Mini Messenger PeerJS</title>
  <script src="https://unpkg.com/peerjs@1.5.5/dist/peerjs.min.js"></script>
  <style>
    :root {
      color-scheme: dark;
      --bg: #07111f;
      --panel: rgba(15, 23, 42, 0.9);
      --panel-2: #020617;
      --text: #f8fafc;
      --muted: #94a3b8;
      --soft: #cbd5e1;
      --line: rgba(255, 255, 255, 0.1);
      --green: #34d399;
      --green-text: #022c22;
      --red: #fca5a5;
      --yellow: #fde68a;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      min-height: 100vh;
      background: radial-gradient(circle at top left, #1f3b45 0, transparent 34%), var(--bg);
      color: var(--text);
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    button,
    input,
    textarea {
      font: inherit;
    }

    .app-shell {
      min-height: 100vh;
      padding: 24px;
    }

    .app-wrap {
      width: min(1180px, 100%);
      margin: 0 auto;
    }

    .app-header {
      display: flex;
      align-items: flex-end;
      justify-content: space-between;
      gap: 16px;
      margin-bottom: 18px;
    }

    .badge {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      color: #a7f3d0;
      background: rgba(16, 185, 129, 0.12);
      border: 1px solid rgba(16, 185, 129, 0.25);
      border-radius: 999px;
      padding: 6px 12px;
      font-size: 13px;
    }

    .title {
      font-size: clamp(30px, 5vw, 48px);
      line-height: 1.05;
      margin: 14px 0 8px;
      letter-spacing: -0.04em;
    }

    .subtitle {
      color: var(--soft);
      margin: 0;
      max-width: 760px;
      line-height: 1.55;
    }

    .layout {
      display: grid;
      grid-template-columns: 365px 1fr;
      gap: 16px;
      align-items: start;
    }

    .left-stack {
      display: grid;
      gap: 16px;
    }

    .panel,
    .status-card {
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 22px;
      box-shadow: 0 18px 60px rgba(0, 0, 0, 0.28);
    }

    .status-card {
      min-width: 210px;
      padding: 14px 16px;
    }

    .status-row {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 14px;
    }

    .muted-small,
    .tiny-note {
      color: var(--muted);
      font-size: 12px;
      line-height: 1.45;
    }

    .muted-small {
      margin-top: 5px;
    }

    .tiny-note {
      margin: 6px 0 0;
    }

    .panel-body {
      padding: 16px;
    }

    .panel-title {
      font-size: 18px;
      margin: 0 0 12px;
    }

    .stack {
      display: grid;
      gap: 10px;
    }

    .label {
      display: grid;
      gap: 7px;
      color: var(--soft);
      font-size: 13px;
    }

    .input,
    .textarea {
      width: 100%;
      border: 1px solid rgba(255, 255, 255, 0.12);
      border-radius: 14px;
      background: var(--panel-2);
      color: var(--text);
      padding: 11px 12px;
      outline: none;
    }

    .input:focus,
    .textarea:focus {
      border-color: var(--green);
      box-shadow: 0 0 0 3px rgba(52, 211, 153, 0.16);
    }

    .input:disabled,
    .textarea:disabled,
    .btn:disabled {
      opacity: 0.55;
      cursor: not-allowed;
    }

    .input-row,
    .composer-row {
      display: grid;
      grid-template-columns: 1fr auto;
      gap: 8px;
    }

    .button-grid,
    .friend-actions {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 8px;
    }

    .btn {
      border: 0;
      border-radius: 14px;
      padding: 10px 12px;
      font-weight: 750;
      font-size: 14px;
      cursor: pointer;
      transition: transform 0.12s ease, filter 0.12s ease, opacity 0.12s ease;
      white-space: nowrap;
    }

    .btn:hover:not(:disabled) {
      transform: translateY(-1px);
      filter: brightness(1.05);
    }

    .btn-primary {
      background: var(--green);
      color: var(--green-text);
    }

    .btn-secondary {
      background: #1e293b;
      color: var(--text);
      border: 1px solid rgba(255, 255, 255, 0.08);
    }

    .btn-danger {
      background: rgba(248, 113, 113, 0.14);
      color: #fecaca;
      border: 1px solid rgba(248, 113, 113, 0.22);
    }

    .notice,
    .empty,
    .test-box,
    .help-box {
      background: var(--panel-2);
      border: 1px solid var(--line);
      border-radius: 16px;
      color: var(--soft);
      padding: 12px;
      font-size: 14px;
      line-height: 1.5;
    }

    .help-box ol {
      margin: 8px 0 0 18px;
      padding: 0;
    }

    .help-box li {
      margin: 5px 0;
    }

    .friend-list {
      display: grid;
      gap: 9px;
    }

    .friend-card {
      border: 1px solid var(--line);
      border-radius: 16px;
      background: var(--panel-2);
      padding: 10px;
    }

    .friend-card.selected {
      border-color: var(--green);
      background: rgba(52, 211, 153, 0.1);
    }

    .friend-main {
      width: 100%;
      background: transparent;
      border: 0;
      color: inherit;
      padding: 0;
      text-align: left;
      cursor: pointer;
    }

    .friend-name {
      font-weight: 750;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .friend-id {
      display: flex;
      align-items: center;
      gap: 6px;
      color: var(--muted);
      font-size: 12px;
      margin-top: 4px;
      min-width: 0;
    }

    .truncate {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .chat-panel {
      min-height: 760px;
      display: flex;
      flex-direction: column;
    }

    .chat-head {
      display: flex;
      justify-content: space-between;
      gap: 14px;
      align-items: center;
      padding: 16px;
      border-bottom: 1px solid var(--line);
    }

    .chat-title {
      margin: 0;
      font-size: 22px;
    }

    .chat-subtitle {
      color: var(--muted);
      margin: 4px 0 0;
      font-size: 14px;
    }

    .chat-body {
      flex: 1;
      overflow-y: auto;
      padding: 16px;
    }

    .center-empty {
      min-height: 500px;
      display: grid;
      place-items: center;
      text-align: center;
      color: var(--muted);
    }

    .bubble-list {
      display: grid;
      gap: 11px;
    }

    .bubble-wrap {
      display: flex;
    }

    .bubble-wrap.mine {
      justify-content: flex-end;
    }

    .bubble-wrap.theirs {
      justify-content: flex-start;
    }

    .bubble {
      max-width: min(620px, 78%);
      border-radius: 20px;
      padding: 10px 13px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.22);
    }

    .bubble.mine {
      background: var(--green);
      color: var(--green-text);
    }

    .bubble.theirs {
      background: #1e293b;
      color: var(--text);
    }

    .bubble-text {
      white-space: pre-wrap;
      overflow-wrap: anywhere;
      line-height: 1.42;
    }

    .bubble-time {
      font-size: 11px;
      opacity: 0.72;
      text-align: right;
      margin-top: 4px;
    }

    .composer {
      border-top: 1px solid var(--line);
      padding: 16px;
    }

    .textarea {
      min-height: 52px;
      max-height: 150px;
      resize: vertical;
    }

    .test-pass,
    .env-ok {
      color: #86efac;
    }

    .test-fail {
      color: var(--red);
    }

    .env-warn {
      color: var(--yellow);
    }

    @media (max-width: 900px) {
      .app-shell {
        padding: 14px;
      }

      .app-header,
      .layout {
        display: grid;
        grid-template-columns: 1fr;
      }

      .status-card {
        min-width: 0;
      }

      .chat-panel {
        min-height: 660px;
      }
    }
  </style>
</head>
<body>
  <main class="app-shell">
    <div class="app-wrap">
      <header class="app-header">
        <div>
          <div class="badge"><span>✓</span><span>GitHub Pages friendly · HTML + PeerJS</span></div>
          <h1 class="title">Mini Messenger PeerJS</h1>
          <p class="subtitle">
            App nhắn tin 1-1 chạy bằng file HTML tĩnh. Deploy được lên GitHub Pages, không cần server riêng.
            Hai máy cần mở app cùng lúc để nhắn trực tiếp qua PeerJS/WebRTC.
          </p>
        </div>

        <div class="status-card">
          <div class="status-row">
            <span id="statusDot">⚪</span>
            <span>Trạng thái:</span>
            <strong id="statusText">Chưa bật</strong>
          </div>
          <div class="muted-small">Bạn online: <span id="onlineCount">0</span></div>
        </div>
      </header>

      <div class="layout">
        <div class="left-stack">
          <section class="panel">
            <div class="panel-body">
              <h2 class="panel-title">Hồ sơ & kết nối</h2>
              <div class="stack">
                <label class="label">
                  <span>Tên hiển thị</span>
                  <input id="displayNameInput" class="input" placeholder="Ví dụ: Hải" />
                </label>

                <label class="label">
                  <span>Peer ID của bạn</span>
                  <div class="input-row">
                    <input id="myPeerIdInput" class="input" placeholder="Để trống để app tự tạo ID" />
                    <button id="copyPeerIdBtn" class="btn btn-secondary">Copy</button>
                  </div>
                  <p class="tiny-note">Nếu nhập ID riêng, hãy nhập trước khi bấm “Bật kết nối”. Nếu ID bị trùng, hãy đổi ID khác.</p>
                </label>

                <div class="button-grid">
                  <button id="startPeerBtn" class="btn btn-primary">Bật kết nối</button>
                  <button id="stopPeerBtn" class="btn btn-secondary">Tắt</button>
                </div>

                <div id="notice" class="notice">Nhấn “Bật kết nối” để lấy mã Peer ID.</div>
              </div>
            </div>
          </section>

          <section class="panel">
            <div class="panel-body">
              <h2 class="panel-title">＋ Kết bạn bằng Peer ID</h2>
              <div class="stack">
                <input id="friendNameInput" class="input" placeholder="Tên bạn bè, không bắt buộc" />
                <input id="connectPeerIdInput" class="input" placeholder="Nhập Peer ID của máy kia" />
                <button id="connectFriendBtn" class="btn btn-primary">🔗 Kết nối / thêm bạn</button>
              </div>
            </div>
          </section>

          <section class="panel">
            <div class="panel-body">
              <h2 class="panel-title">💬 Bạn bè</h2>
              <div id="friendList" class="friend-list"></div>
            </div>
          </section>

          <section class="panel">
            <div class="panel-body">
              <h2 class="panel-title">🧪 Rà lỗi & môi trường</h2>
              <div class="button-grid">
                <button id="runTestsBtn" class="btn btn-secondary">Chạy test</button>
                <button id="clearAllBtn" class="btn btn-danger">Xóa dữ liệu</button>
              </div>
              <div id="environmentBox" class="test-box" style="margin-top: 10px;"></div>
              <div id="testBox" class="test-box" style="display: none; margin-top: 10px;"></div>
            </div>
          </section>

          <section class="panel">
            <div class="panel-body">
              <h2 class="panel-title">🚀 Deploy GitHub Pages</h2>
              <div class="help-box">
                <ol>
                  <li>Tạo repository trên GitHub.</li>
                  <li>Đặt file này tên là <strong>index.html</strong>.</li>
                  <li>Upload vào repository.</li>
                  <li>Vào <strong>Settings → Pages</strong>.</li>
                  <li>Chọn branch chứa file, thường là <strong>main</strong>, rồi Save.</li>
                </ol>
                <p class="tiny-note">GitHub Pages dùng HTTPS nên phù hợp với WebRTC. Không mở bằng đường dẫn file:// nếu muốn kết nối ổn định.</p>
              </div>
            </div>
          </section>
        </div>

        <section class="panel chat-panel">
          <div class="chat-head">
            <div style="min-width: 0;">
              <h2 id="chatTitle" class="chat-title truncate">Chọn một người bạn</h2>
              <p id="chatSubtitle" class="chat-subtitle truncate">Tin nhắn sẽ xuất hiện ở đây</p>
            </div>
            <button id="clearChatBtn" class="btn btn-secondary" disabled>Xóa chat</button>
          </div>

          <div id="chatBody" class="chat-body"></div>

          <div class="composer">
            <div class="composer-row">
              <textarea id="draftInput" class="textarea" placeholder="Chọn bạn bè trước" disabled></textarea>
              <button id="sendBtn" class="btn btn-primary" disabled>Gửi</button>
            </div>
            <p class="tiny-note">Enter để gửi, Shift + Enter để xuống dòng. Tin nhắn chỉ lưu trên trình duyệt hiện tại.</p>
          </div>
        </section>
      </div>
    </div>
  </main>

  <script>
    "use strict";

    const STORAGE_KEYS = {
      profile: "peer_chat_profile_html_v1",
      friends: "peer_chat_friends_html_v1",
      messages: "peer_chat_messages_html_v1"
    };

    const EMPTY_PROFILE = {
      displayName: "",
      peerId: ""
    };

    const state = {
      profile: loadStorage(STORAGE_KEYS.profile, EMPTY_PROFILE),
      friends: dedupeFriends(loadStorage(STORAGE_KEYS.friends, [])),
      messages: loadStorage(STORAGE_KEYS.messages, {}),
      peerStatus: "idle",
      myPeerId: "",
      selectedFriendId: "",
      onlineMap: {},
      peer: null,
      connections: {}
    };

    const els = {
      statusDot: document.getElementById("statusDot"),
      statusText: document.getElementById("statusText"),
      onlineCount: document.getElementById("onlineCount"),
      displayNameInput: document.getElementById("displayNameInput"),
      myPeerIdInput: document.getElementById("myPeerIdInput"),
      copyPeerIdBtn: document.getElementById("copyPeerIdBtn"),
      startPeerBtn: document.getElementById("startPeerBtn"),
      stopPeerBtn: document.getElementById("stopPeerBtn"),
      notice: document.getElementById("notice"),
      friendNameInput: document.getElementById("friendNameInput"),
      connectPeerIdInput: document.getElementById("connectPeerIdInput"),
      connectFriendBtn: document.getElementById("connectFriendBtn"),
      friendList: document.getElementById("friendList"),
      environmentBox: document.getElementById("environmentBox"),
      testBox: document.getElementById("testBox"),
      runTestsBtn: document.getElementById("runTestsBtn"),
      clearAllBtn: document.getElementById("clearAllBtn"),
      chatTitle: document.getElementById("chatTitle"),
      chatSubtitle: document.getElementById("chatSubtitle"),
      clearChatBtn: document.getElementById("clearChatBtn"),
      chatBody: document.getElementById("chatBody"),
      draftInput: document.getElementById("draftInput"),
      sendBtn: document.getElementById("sendBtn")
    };

    function safeJsonParse(value, fallback) {
      try {
        return value ? JSON.parse(value) : fallback;
      } catch {
        return fallback;
      }
    }

    function loadStorage(key, fallback) {
      if (typeof window === "undefined" || !window.localStorage) return fallback;
      return safeJsonParse(window.localStorage.getItem(key), fallback);
    }

    function saveStorage(key, value) {
      if (typeof window === "undefined" || !window.localStorage) return false;
      try {
        window.localStorage.setItem(key, JSON.stringify(value));
        return true;
      } catch {
        return false;
      }
    }

    function removeStorage(key) {
      if (typeof window === "undefined" || !window.localStorage) return false;
      try {
        window.localStorage.removeItem(key);
        return true;
      } catch {
        return false;
      }
    }

    function createId() {
      if (typeof crypto !== "undefined" && crypto.randomUUID) return crypto.randomUUID();
      return String(Date.now()) + "-" + Math.random().toString(16).slice(2);
    }

    function nowLabel() {
      return new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    }

    function normalizePeerId(value) {
      return String(value || "").trim().replace(/\s+/g, "");
    }

    function sanitizeText(value, maxLength = 4000) {
      return String(value || "").slice(0, maxLength);
    }

    function isValidIncomingData(data) {
      return data !== null && typeof data === "object" && typeof data.type === "string";
    }

    function isSecureEnoughForWebRTC() {
      const protocol = window.location.protocol;
      const hostname = window.location.hostname;
      return protocol === "https:" || hostname === "localhost" || hostname === "127.0.0.1";
    }

    function isLikelyGitHubPages() {
      return window.location.hostname.endsWith("github.io");
    }

    function getEnvironmentChecks() {
      const hasPeer = typeof window.Peer === "function";
      const hasLocalStorage = "localStorage" in window;
      const secure = isSecureEnoughForWebRTC();
      const githubPages = isLikelyGitHubPages();

      return [
        {
          name: "PeerJS CDN đã tải",
          ok: hasPeer,
          hint: hasPeer ? "OK" : "Không tải được PeerJS từ CDN"
        },
        {
          name: "localStorage có sẵn",
          ok: hasLocalStorage,
          hint: hasLocalStorage ? "OK" : "Trình duyệt đang chặn lưu dữ liệu cục bộ"
        },
        {
          name: "HTTPS hoặc localhost cho WebRTC",
          ok: secure,
          hint: secure ? "OK" : "GitHub Pages có HTTPS; không nên mở bằng file://"
        },
        {
          name: "Tương thích GitHub Pages",
          ok: secure && hasPeer,
          hint: githubPages ? "Đang chạy trên github.io" : "Có thể deploy file index.html lên GitHub Pages"
        }
      ];
    }

    function makeLocalMessage(text) {
      return {
        id: createId(),
        type: "message",
        text: sanitizeText(text),
        from: "me",
        time: nowLabel(),
        createdAt: Date.now()
      };
    }

    function makeRemoteMessage(data) {
      return {
        id: data.id || createId(),
        text: sanitizeText(data.text),
        from: "them",
        time: data.time || nowLabel(),
        createdAt: data.createdAt || Date.now()
      };
    }

    function makeHelloPayload(displayName, peerId) {
      return {
        type: "hello",
        displayName: sanitizeText(displayName || "Người dùng PeerJS", 80),
        peerId: normalizePeerId(peerId)
      };
    }

    function dedupeFriends(friends) {
      const seen = new Set();
      const clean = [];
      const list = Array.isArray(friends) ? friends : [];

      for (const friend of list) {
        const peerId = normalizePeerId(friend && friend.peerId);
        if (!peerId || seen.has(peerId)) continue;
        seen.add(peerId);
        clean.push({
          id: (friend && friend.id) || createId(),
          peerId,
          name: sanitizeText((friend && friend.name) || peerId, 80),
          createdAt: (friend && friend.createdAt) || Date.now()
        });
      }

      return clean;
    }

    function runSelfTests() {
      const results = [];
      const test = (name, condition) => results.push({ name, passed: Boolean(condition) });

      test("normalizePeerId xóa khoảng trắng", normalizePeerId("  abc 123 \n") === "abc123");
      test("safeJsonParse trả fallback khi JSON lỗi", safeJsonParse("{bad", { ok: true }).ok === true);
      test("safeJsonParse đọc JSON hợp lệ", safeJsonParse('{"ok":true}', {}).ok === true);
      test("isValidIncomingData nhận object có type", isValidIncomingData({ type: "message" }) === true);
      test("isValidIncomingData từ chối null", isValidIncomingData(null) === false);
      test("makeRemoteMessage ép text về string", makeRemoteMessage({ text: 123 }).text === "123");
      test("makeLocalMessage có type message", makeLocalMessage("hi").type === "message");
      test("sanitizeText giới hạn độ dài", sanitizeText("abcdef", 3) === "abc");
      test("dedupeFriends bỏ bạn trùng Peer ID", dedupeFriends([{ peerId: "a" }, { peerId: "a" }]).length === 1);
      test("makeHelloPayload có type hello", makeHelloPayload("Hải", "abc").type === "hello");
      test("Nút gửi tồn tại", Boolean(els.sendBtn));
      test("Khung chat tồn tại", Boolean(els.chatBody));

      for (const check of getEnvironmentChecks()) {
        test(check.name, check.ok);
      }

      return results;
    }

    function setNotice(text) {
      els.notice.textContent = text;
    }

    function persistAll() {
      saveStorage(STORAGE_KEYS.profile, state.profile);
      saveStorage(STORAGE_KEYS.friends, state.friends);
      saveStorage(STORAGE_KEYS.messages, state.messages);
    }

    function getSelectedFriend() {
      return state.friends.find((friend) => friend.peerId === state.selectedFriendId) || null;
    }

    function getStatusLabel(status) {
      const labels = {
        idle: "Chưa bật",
        connecting: "Đang bật",
        online: "Online",
        offline: "Offline",
        error: "Lỗi"
      };
      return labels[status] || "Không rõ";
    }

    function renderStatus() {
      const dots = {
        idle: "⚪",
        connecting: "🟡",
        online: "🟢",
        offline: "⚪",
        error: "🔴"
      };
      els.statusDot.textContent = dots[state.peerStatus] || "⚪";
      els.statusText.textContent = getStatusLabel(state.peerStatus);
      els.onlineCount.textContent = String(Object.values(state.onlineMap).filter(Boolean).length);
      els.startPeerBtn.disabled = state.peerStatus === "connecting" || state.peerStatus === "online";
      els.myPeerIdInput.disabled = state.peerStatus === "connecting" || state.peerStatus === "online";
    }

    function renderProfileInputs() {
      if (document.activeElement !== els.displayNameInput) {
        els.displayNameInput.value = state.profile.displayName || "";
      }
      if (document.activeElement !== els.myPeerIdInput) {
        els.myPeerIdInput.value = state.myPeerId || state.profile.peerId || "";
      }
    }

    function renderFriends() {
      if (state.friends.length === 0) {
        els.friendList.innerHTML = '<div class="empty">Chưa có bạn bè. Hãy nhập Peer ID của máy khác để bắt đầu.</div>';
        return;
      }

      els.friendList.innerHTML = "";
      for (const friend of state.friends) {
        const isSelected = state.selectedFriendId === friend.peerId;
        const isOnline = Boolean(state.onlineMap[friend.peerId]);
        const card = document.createElement("div");
        card.className = "friend-card" + (isSelected ? " selected" : "");

        const main = document.createElement("button");
        main.type = "button";
        main.className = "friend-main";
        main.addEventListener("click", () => {
          state.selectedFriendId = friend.peerId;
          render();
        });

        const name = document.createElement("div");
        name.className = "friend-name";
        name.textContent = friend.name || friend.peerId;

        const id = document.createElement("div");
        id.className = "friend-id";
        id.innerHTML = `<span>${isOnline ? "🟢" : "⚪"}</span>`;
        const idText = document.createElement("span");
        idText.className = "truncate";
        idText.textContent = friend.peerId;
        id.appendChild(idText);

        main.appendChild(name);
        main.appendChild(id);

        const actions = document.createElement("div");
        actions.className = "friend-actions";
        actions.style.marginTop = "9px";

        const reconnectBtn = document.createElement("button");
        reconnectBtn.type = "button";
        reconnectBtn.className = "btn btn-secondary";
        reconnectBtn.textContent = "Nối";
        reconnectBtn.addEventListener("click", () => connectToFriend(friend.peerId, friend.name));

        const removeBtn = document.createElement("button");
        removeBtn.type = "button";
        removeBtn.className = "btn btn-danger";
        removeBtn.textContent = "Xóa";
        removeBtn.addEventListener("click", () => removeFriend(friend.peerId));

        actions.appendChild(reconnectBtn);
        actions.appendChild(removeBtn);
        card.appendChild(main);
        card.appendChild(actions);
        els.friendList.appendChild(card);
      }
    }

    function renderChat() {
      const selectedFriend = getSelectedFriend();
      els.clearChatBtn.disabled = !selectedFriend;
      els.draftInput.disabled = !selectedFriend;
      els.sendBtn.disabled = !selectedFriend || !els.draftInput.value.trim();
      els.draftInput.placeholder = selectedFriend ? "Nhập tin nhắn..." : "Chọn bạn bè trước";

      if (!selectedFriend) {
        els.chatTitle.textContent = "Chọn một người bạn";
        els.chatSubtitle.textContent = "Tin nhắn sẽ xuất hiện ở đây";
        els.chatBody.innerHTML = '<div class="center-empty"><div><div style="font-size: 56px; margin-bottom: 12px;">💬</div><p>Chọn bạn bè hoặc thêm Peer ID để bắt đầu nhắn.</p></div></div>';
        return;
      }

      els.chatTitle.textContent = selectedFriend.name || selectedFriend.peerId;
      els.chatSubtitle.textContent = state.onlineMap[selectedFriend.peerId] ? "Đang kết nối trực tiếp" : "Chưa kết nối / offline";

      const list = state.messages[selectedFriend.peerId] || [];
      if (list.length === 0) {
        els.chatBody.innerHTML = '<div class="empty">Chưa có tin nhắn với người này. Cả hai máy cần đang mở app và đã kết nối.</div>';
        return;
      }

      els.chatBody.innerHTML = "";
      const bubbleList = document.createElement("div");
      bubbleList.className = "bubble-list";

      for (const message of list) {
        const mine = message.from === "me";
        const wrap = document.createElement("div");
        wrap.className = "bubble-wrap " + (mine ? "mine" : "theirs");

        const bubble = document.createElement("div");
        bubble.className = "bubble " + (mine ? "mine" : "theirs");

        const text = document.createElement("div");
        text.className = "bubble-text";
        text.textContent = message.text;

        const time = document.createElement("div");
        time.className = "bubble-time";
        time.textContent = message.time || "";

        bubble.appendChild(text);
        bubble.appendChild(time);
        wrap.appendChild(bubble);
        bubbleList.appendChild(wrap);
      }

      els.chatBody.appendChild(bubbleList);
      els.chatBody.scrollTop = els.chatBody.scrollHeight;
    }

    function renderEnvironmentChecks() {
      const checks = getEnvironmentChecks();
      els.environmentBox.innerHTML = "";
      for (const check of checks) {
        const row = document.createElement("div");
        row.className = check.ok ? "env-ok" : "env-warn";
        row.textContent = `${check.ok ? "✓" : "!"} ${check.name}: ${check.hint}`;
        els.environmentBox.appendChild(row);
      }
    }

    function render() {
      renderStatus();
      renderProfileInputs();
      renderFriends();
      renderChat();
      renderEnvironmentChecks();
    }

    function upsertFriend(peerId, name = "") {
      const cleanPeerId = normalizePeerId(peerId);
      if (!cleanPeerId) return;

      const existing = state.friends.find((friend) => friend.peerId === cleanPeerId);
      if (existing) {
        existing.name = sanitizeText(name || existing.name || cleanPeerId, 80);
      } else {
        state.friends.push({
          id: createId(),
          peerId: cleanPeerId,
          name: sanitizeText(name || `Bạn ${state.friends.length + 1}`, 80),
          createdAt: Date.now()
        });
      }

      state.friends = dedupeFriends(state.friends);
      state.selectedFriendId = cleanPeerId;
      persistAll();
      render();
    }

    function appendMessage(friendPeerId, message) {
      if (!state.messages[friendPeerId]) state.messages[friendPeerId] = [];
      state.messages[friendPeerId].push(message);
      saveStorage(STORAGE_KEYS.messages, state.messages);
      renderChat();
    }

    function sendHello(conn) {
      try {
        conn.send(makeHelloPayload(state.profile.displayName, state.myPeerId || (state.peer && state.peer.id) || ""));
      } catch {
        // Connection may close before hello is sent.
      }
    }

    function attachConnection(conn) {
      if (!conn || !conn.peer) return;

      state.connections[conn.peer] = conn;
      upsertFriend(conn.peer, (conn.metadata && conn.metadata.displayName) || "");

      conn.on("open", () => {
        state.connections[conn.peer] = conn;
        state.onlineMap[conn.peer] = true;
        setNotice(`Đã kết nối với ${(conn.metadata && conn.metadata.displayName) || conn.peer}.`);
        sendHello(conn);
        render();
      });

      conn.on("data", (data) => {
        if (!isValidIncomingData(data)) return;

        if (data.type === "hello") {
          state.onlineMap[conn.peer] = true;
          upsertFriend(conn.peer, data.displayName || conn.peer);
          render();
          return;
        }

        if (data.type === "message") {
          const text = sanitizeText(data.text).trim();
          if (!text) return;
          state.onlineMap[conn.peer] = true;
          upsertFriend(conn.peer, data.displayName || conn.peer);
          appendMessage(conn.peer, makeRemoteMessage({ ...data, text }));
          if (!state.selectedFriendId) state.selectedFriendId = conn.peer;
          render();
        }
      });

      conn.on("close", () => {
        state.onlineMap[conn.peer] = false;
        setNotice(`Kết nối với ${conn.peer} đã đóng.`);
        render();
      });

      conn.on("error", () => {
        state.onlineMap[conn.peer] = false;
        setNotice("Có lỗi kết nối. Hãy kiểm tra Peer ID hoặc thử bật lại kết nối.");
        render();
      });
    }

    function startPeer() {
      if (state.peer && !state.peer.destroyed) {
        setNotice("Kết nối đã bật rồi.");
        return;
      }

      if (typeof window.Peer !== "function") {
        state.peerStatus = "error";
        setNotice("PeerJS chưa tải được. Hãy kiểm tra mạng hoặc đổi CDN PeerJS.");
        render();
        return;
      }

      if (!isSecureEnoughForWebRTC()) {
        state.peerStatus = "error";
        setNotice("WebRTC cần HTTPS hoặc localhost. GitHub Pages dùng HTTPS nên deploy lên đó sẽ phù hợp hơn mở file trực tiếp.");
        render();
        return;
      }

      state.peerStatus = "connecting";
      setNotice("Đang tạo Peer ID...");
      render();

      try {
        const preferredId = normalizePeerId(state.profile.peerId);
        const peer = preferredId ? new Peer(preferredId, { debug: 1 }) : new Peer(undefined, { debug: 1 });
        state.peer = peer;

        peer.on("open", (id) => {
          state.myPeerId = id;
          state.peerStatus = "online";
          state.profile.peerId = id;
          saveStorage(STORAGE_KEYS.profile, state.profile);
          setNotice("Đã sẵn sàng. Gửi Peer ID này cho bạn bè để họ kết nối.");
          render();
        });

        peer.on("connection", (conn) => {
          attachConnection(conn);
          setNotice(`${conn.peer} đang kết nối tới bạn.`);
        });

        peer.on("disconnected", () => {
          state.peerStatus = "offline";
          setNotice("Peer bị ngắt. Hãy bấm “Tắt”, rồi “Bật kết nối” lại.");
          render();
        });

        peer.on("close", () => {
          state.peerStatus = "offline";
          setNotice("Peer đã đóng.");
          render();
        });

        peer.on("error", (error) => {
          state.peerStatus = "error";
          const message = error && error.type === "unavailable-id"
            ? "Peer ID này đang được dùng. Hãy đổi Peer ID khác hoặc để trống để app tự tạo ID."
            : error && error.type === "network"
              ? "Lỗi mạng PeerJS. Hãy kiểm tra Internet rồi thử lại."
              : error && error.type === "peer-unavailable"
                ? "Không tìm thấy Peer ID kia. Hãy kiểm tra máy kia đã bật kết nối chưa."
                : "Không tạo được kết nối PeerJS. Hãy thử lại hoặc kiểm tra mạng.";
          setNotice(message);
          render();
        });
      } catch {
        state.peerStatus = "error";
        setNotice("Không khởi tạo được PeerJS. Hãy thử tải lại app.");
        render();
      }
    }

    function stopPeer() {
      for (const conn of Object.values(state.connections)) {
        try {
          conn.close();
        } catch {
          // Ignore close errors.
        }
      }
      state.connections = {};
      state.onlineMap = {};

      try {
        if (state.peer) state.peer.destroy();
      } catch {
        // Ignore destroy errors.
      }

      state.peer = null;
      state.peerStatus = "offline";
      state.myPeerId = "";
      setNotice("Đã tắt kết nối.");
      render();
    }

    function connectToFriend(peerIdArg, nameArg) {
      const rawPeerId = peerIdArg || els.connectPeerIdInput.value;
      const rawName = typeof nameArg === "string" ? nameArg : els.friendNameInput.value;
      const cleanPeerId = normalizePeerId(rawPeerId);

      if (!state.peer || state.peer.destroyed || state.peerStatus !== "online") {
        setNotice("Bạn cần nhấn “Bật kết nối” trước.");
        return;
      }

      if (!cleanPeerId) {
        setNotice("Hãy nhập Peer ID của người bạn muốn kết nối.");
        return;
      }

      if (cleanPeerId === state.myPeerId) {
        setNotice("Không thể tự kết nối với chính mình.");
        return;
      }

      const existing = state.connections[cleanPeerId];
      if (existing && existing.open) {
        state.selectedFriendId = cleanPeerId;
        setNotice("Bạn đã kết nối với người này rồi.");
        render();
        return;
      }

      upsertFriend(cleanPeerId, rawName || cleanPeerId);
      setNotice("Đang kết nối...");

      try {
        const conn = state.peer.connect(cleanPeerId, {
          reliable: true,
          metadata: { displayName: sanitizeText(state.profile.displayName || "Người dùng PeerJS", 80) }
        });
        attachConnection(conn);
        els.connectPeerIdInput.value = "";
        els.friendNameInput.value = "";
      } catch {
        setNotice("Không thể bắt đầu kết nối. Hãy kiểm tra Peer ID.");
      }
    }

    function sendMessage() {
      const text = sanitizeText(els.draftInput.value).trim();
      if (!text) return;

      if (!state.selectedFriendId) {
        setNotice("Hãy chọn một người bạn trước khi nhắn.");
        return;
      }

      const conn = state.connections[state.selectedFriendId];
      if (!conn || !conn.open) {
        setNotice("Bạn này chưa online/kết nối. Nhấn “Nối” trước khi gửi.");
        return;
      }

      const localMessage = makeLocalMessage(text);
      const outgoingMessage = {
        ...localMessage,
        displayName: sanitizeText(state.profile.displayName || "Người dùng PeerJS", 80)
      };

      try {
        conn.send(outgoingMessage);
        appendMessage(state.selectedFriendId, {
          id: localMessage.id,
          text: localMessage.text,
          from: "me",
          time: localMessage.time,
          createdAt: localMessage.createdAt
        });
        els.draftInput.value = "";
        render();
      } catch {
        setNotice("Gửi tin nhắn thất bại. Hãy kết nối lại rồi thử gửi lần nữa.");
      }
    }

    function copyMyId() {
      const value = state.myPeerId || state.profile.peerId;
      if (!value) {
        setNotice("Chưa có Peer ID để sao chép.");
        return;
      }

      if (!navigator.clipboard) {
        setNotice("Trình duyệt không hỗ trợ copy tự động. Hãy bôi đen Peer ID và copy thủ công.");
        return;
      }

      navigator.clipboard
        .writeText(value)
        .then(() => setNotice("Đã sao chép Peer ID."))
        .catch(() => setNotice("Không sao chép được. Hãy bôi đen và copy thủ công."));
    }

    function removeFriend(peerId) {
      state.friends = state.friends.filter((friend) => friend.peerId !== peerId);
      delete state.messages[peerId];

      try {
        if (state.connections[peerId]) state.connections[peerId].close();
      } catch {
        // Ignore close errors.
      }
      delete state.connections[peerId];
      delete state.onlineMap[peerId];

      if (state.selectedFriendId === peerId) state.selectedFriendId = "";
      persistAll();
      setNotice("Đã xóa bạn và lịch sử chat trên máy này.");
      render();
    }

    function clearCurrentChat() {
      if (!state.selectedFriendId) return;
      state.messages[state.selectedFriendId] = [];
      saveStorage(STORAGE_KEYS.messages, state.messages);
      setNotice("Đã xóa lịch sử chat với bạn này trên máy này.");
      render();
    }

    function clearAllLocalData() {
      stopPeer();
      state.profile = { ...EMPTY_PROFILE };
      state.friends = [];
      state.messages = {};
      state.selectedFriendId = "";
      state.myPeerId = "";
      els.connectPeerIdInput.value = "";
      els.friendNameInput.value = "";
      els.draftInput.value = "";
      removeStorage(STORAGE_KEYS.profile);
      removeStorage(STORAGE_KEYS.friends);
      removeStorage(STORAGE_KEYS.messages);
      setNotice("Đã xóa toàn bộ dữ liệu cục bộ của app trên máy này.");
      render();
    }

    function handleRunTests() {
      const results = runSelfTests();
      const failed = results.filter((result) => !result.passed).length;
      els.testBox.style.display = "block";
      els.testBox.innerHTML = "";

      for (const result of results) {
        const row = document.createElement("div");
        row.className = result.passed ? "test-pass" : "test-fail";
        row.textContent = `${result.passed ? "✓" : "✕"} ${result.name}`;
        els.testBox.appendChild(row);
      }

      setNotice(failed === 0 ? "Tất cả kiểm tra cơ bản đều đạt." : `Có ${failed} kiểm tra chưa đạt.`);
      renderEnvironmentChecks();
    }

    function bindEvents() {
      els.displayNameInput.addEventListener("input", () => {
        state.profile.displayName = sanitizeText(els.displayNameInput.value, 80);
        saveStorage(STORAGE_KEYS.profile, state.profile);
      });

      els.myPeerIdInput.addEventListener("input", () => {
        state.profile.peerId = normalizePeerId(els.myPeerIdInput.value);
        saveStorage(STORAGE_KEYS.profile, state.profile);
      });

      els.copyPeerIdBtn.addEventListener("click", copyMyId);
      els.startPeerBtn.addEventListener("click", startPeer);
      els.stopPeerBtn.addEventListener("click", stopPeer);
      els.connectFriendBtn.addEventListener("click", () => connectToFriend());
      els.runTestsBtn.addEventListener("click", handleRunTests);
      els.clearAllBtn.addEventListener("click", clearAllLocalData);
      els.clearChatBtn.addEventListener("click", clearCurrentChat);
      els.sendBtn.addEventListener("click", sendMessage);

      els.connectPeerIdInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter") connectToFriend();
      });

      els.draftInput.addEventListener("input", () => {
        els.sendBtn.disabled = !getSelectedFriend() || !els.draftInput.value.trim();
      });

      els.draftInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter" && !event.shiftKey) {
          event.preventDefault();
          sendMessage();
        }
      });

      window.addEventListener("beforeunload", () => {
        persistAll();
        for (const conn of Object.values(state.connections)) {
          try {
            conn.close();
          } catch {
            // Ignore close errors.
          }
        }
      });
    }

    bindEvents();
    render();
  </script>
</body>
</html>    border-radius: 22px;
    box-shadow: 0 18px 60px rgba(0, 0, 0, 0.28);
  }

  .status-card {
    min-width: 210px;
    padding: 14px 16px;
  }

  .status-row {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 14px;
  }

  .muted-small {
    color: #94a3b8;
    font-size: 12px;
    margin-top: 5px;
  }

  .layout {
    display: grid;
    grid-template-columns: 365px 1fr;
    gap: 16px;
    align-items: start;
  }

  .left-stack {
    display: grid;
    gap: 16px;
  }

  .panel-body {
    padding: 16px;
  }

  .panel-title {
    font-size: 18px;
    margin: 0 0 12px;
  }

  .stack {
    display: grid;
    gap: 10px;
  }

  .label {
    display: grid;
    gap: 7px;
    color: #cbd5e1;
    font-size: 13px;
  }

  .input,
  .textarea {
    width: 100%;
    box-sizing: border-box;
    border: 1px solid rgba(255, 255, 255, 0.12);
    border-radius: 14px;
    background: #020617;
    color: #f8fafc;
    padding: 11px 12px;
    outline: none;
    font: inherit;
  }

  .input:focus,
  .textarea:focus {
    border-color: #34d399;
    box-shadow: 0 0 0 3px rgba(52, 211, 153, 0.16);
  }

  .input:disabled,
  .textarea:disabled {
    opacity: 0.62;
    cursor: not-allowed;
  }

  .input-row,
  .button-grid,
  .friend-actions {
    display: grid;
    grid-template-columns: 1fr auto;
    gap: 8px;
  }

  .button-grid,
  .friend-actions {
    grid-template-columns: 1fr 1fr;
  }

  .btn {
    border: 0;
    border-radius: 14px;
    padding: 10px 12px;
    font-weight: 700;
    font-size: 14px;
    cursor: pointer;
    transition: transform 0.12s ease, filter 0.12s ease, opacity 0.12s ease;
    white-space: nowrap;
  }

  .btn:hover:not(:disabled) {
    transform: translateY(-1px);
    filter: brightness(1.05);
  }

  .btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .btn-primary {
    background: #34d399;
    color: #022c22;
  }

  .btn-secondary {
    background: #1e293b;
    color: #f8fafc;
    border: 1px solid rgba(255, 255, 255, 0.08);
  }

  .btn-danger {
    background: rgba(248, 113, 113, 0.14);
    color: #fecaca;
    border: 1px solid rgba(248, 113, 113, 0.22);
  }

  .notice,
  .empty,
  .test-box,
  .help-box {
    background: #020617;
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 16px;
    color: #cbd5e1;
    padding: 12px;
    font-size: 14px;
    line-height: 1.5;
  }

  .help-box ul,
  .help-box ol {
    margin: 8px 0 0 18px;
    padding: 0;
  }

  .help-box li {
    margin: 5px 0;
  }

  .tiny-note {
    color: #64748b;
    font-size: 12px;
    margin: 6px 0 0;
    line-height: 1.45;
  }

  .friend-list {
    display: grid;
    gap: 9px;
  }

  .friend-card {
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 16px;
    background: #020617;
    padding: 10px;
  }

  .friend-card.selected {
    border-color: #34d399;
    background: rgba(52, 211, 153, 0.1);
  }

  .friend-main {
    width: 100%;
    background: transparent;
    border: 0;
    color: inherit;
    padding: 0;
    text-align: left;
    cursor: pointer;
  }

  .friend-name {
    font-weight: 750;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .friend-id {
    display: flex;
    align-items: center;
    gap: 6px;
    color: #94a3b8;
    font-size: 12px;
    margin-top: 4px;
    min-width: 0;
  }

  .truncate {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .chat-panel {
    min-height: 760px;
    display: flex;
    flex-direction: column;
  }

  .chat-head {
    display: flex;
    justify-content: space-between;
    gap: 14px;
    align-items: center;
    padding: 16px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }

  .chat-title {
    margin: 0;
    font-size: 22px;
  }

  .chat-subtitle {
    color: #94a3b8;
    margin: 4px 0 0;
    font-size: 14px;
  }

  .chat-body {
    flex: 1;
    overflow-y: auto;
    padding: 16px;
  }

  .center-empty {
    min-height: 500px;
    display: grid;
    place-items: center;
    text-align: center;
    color: #94a3b8;
  }

  .bubble-list {
    display: grid;
    gap: 11px;
  }

  .bubble-wrap {
    display: flex;
  }

  .bubble-wrap.mine {
    justify-content: flex-end;
  }

  .bubble-wrap.theirs {
    justify-content: flex-start;
  }

  .bubble {
    max-width: min(620px, 78%);
    border-radius: 20px;
    padding: 10px 13px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.22);
  }

  .bubble.mine {
    background: #34d399;
    color: #022c22;
  }

  .bubble.theirs {
    background: #1e293b;
    color: #f8fafc;
  }

  .bubble-text {
    white-space: pre-wrap;
    overflow-wrap: anywhere;
    line-height: 1.42;
  }

  .bubble-time {
    font-size: 11px;
    opacity: 0.72;
    text-align: right;
    margin-top: 4px;
  }

  .composer {
    border-top: 1px solid rgba(255, 255, 255, 0.1);
    padding: 16px;
  }

  .composer-row {
    display: grid;
    grid-template-columns: 1fr auto;
    gap: 9px;
  }

  .textarea {
    min-height: 52px;
    max-height: 150px;
    resize: vertical;
  }

  .test-pass {
    color: #86efac;
  }

  .test-fail {
    color: #fca5a5;
  }

  .env-ok {
    color: #86efac;
  }

  .env-warn {
    color: #fde68a;
  }

  @media (max-width: 900px) {
    .app-shell {
      padding: 14px;
    }

    .app-header,
    .layout {
      grid-template-columns: 1fr;
      display: grid;
    }

    .status-card {
      min-width: 0;
    }

    .chat-panel {
      min-height: 660px;
    }
  }
`;

function safeJsonParse(value, fallback) {
  try {
    return value ? JSON.parse(value) : fallback;
  } catch {
    return fallback;
  }
}

function loadStorage(key, fallback) {
  if (typeof window === "undefined") return fallback;
  return safeJsonParse(window.localStorage.getItem(key), fallback);
}

function saveStorage(key, value) {
  if (typeof window === "undefined") return false;
  try {
    window.localStorage.setItem(key, JSON.stringify(value));
    return true;
  } catch {
    return false;
  }
}

function removeStorage(key) {
  if (typeof window === "undefined") return false;
  try {
    window.localStorage.removeItem(key);
    return true;
  } catch {
    return false;
  }
}

function createId() {
  if (typeof crypto !== "undefined" && crypto.randomUUID) return crypto.randomUUID();
  return `${Date.now()}-${Math.random().toString(16).slice(2)}`;
}

function nowLabel() {
  return new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
}

function normalizePeerId(value) {
  return String(value || "").trim().replace(/\s+/g, "");
}

function sanitizeText(value, maxLength = 4000) {
  return String(value || "").slice(0, maxLength);
}

function isValidIncomingData(data) {
  return data !== null && typeof data === "object" && typeof data.type === "string";
}

function isSecureEnoughForWebRTC() {
  if (typeof window === "undefined") return false;
  const { protocol, hostname } = window.location;
  return protocol === "https:" || hostname === "localhost" || hostname === "127.0.0.1";
}

function isLikelyGitHubPages() {
  if (typeof window === "undefined") return false;
  return window.location.hostname.endsWith("github.io");
}

function getEnvironmentChecks() {
  const hasPeer = typeof Peer === "function";
  const hasLocalStorage = typeof window !== "undefined" && "localStorage" in window;
  const secure = isSecureEnoughForWebRTC();
  const githubPages = isLikelyGitHubPages();

  return [
    {
      name: "PeerJS dependency đã tải",
      ok: hasPeer,
      hint: hasPeer ? "OK" : "Cần chạy: npm install peerjs",
    },
    {
      name: "localStorage có sẵn",
      ok: hasLocalStorage,
      hint: hasLocalStorage ? "OK" : "Trình duyệt đang chặn lưu dữ liệu cục bộ",
    },
    {
      name: "HTTPS hoặc localhost cho WebRTC",
      ok: secure,
      hint: secure ? "OK" : "GitHub Pages có HTTPS; khi test local hãy dùng localhost",
    },
    {
      name: "Tương thích GitHub Pages",
      ok: secure && hasPeer,
      hint: githubPages ? "Đang chạy trên github.io" : "Có thể deploy lên GitHub Pages nếu build bằng Vite/React",
    },
  ];
}

function makeLocalMessage(text) {
  return {
    id: createId(),
    type: "message",
    text: sanitizeText(text),
    from: "me",
    time: nowLabel(),
    createdAt: Date.now(),
  };
}

function makeRemoteMessage(data) {
  return {
    id: data.id || createId(),
    text: sanitizeText(data.text),
    from: "them",
    time: data.time || nowLabel(),
    createdAt: data.createdAt || Date.now(),
  };
}

function makeHelloPayload(displayName, peerId) {
  return {
    type: "hello",
    displayName: sanitizeText(displayName || "Người dùng PeerJS", 80),
    peerId: normalizePeerId(peerId),
  };
}

function dedupeFriends(friends) {
  const seen = new Set();
  const clean = [];

  for (const friend of Array.isArray(friends) ? friends : []) {
    const peerId = normalizePeerId(friend?.peerId);
    if (!peerId || seen.has(peerId)) continue;
    seen.add(peerId);
    clean.push({
      id: friend?.id || createId(),
      peerId,
      name: sanitizeText(friend?.name || peerId, 80),
      createdAt: friend?.createdAt || Date.now(),
    });
  }

  return clean;
}

function runSelfTests() {
  const results = [];

  function test(name, condition) {
    results.push({ name, passed: Boolean(condition) });
  }

  test("normalizePeerId xóa khoảng trắng", normalizePeerId("  abc 123 \n") === "abc123");
  test("safeJsonParse trả fallback khi JSON lỗi", safeJsonParse("{bad", { ok: true }).ok === true);
  test("safeJsonParse đọc JSON hợp lệ", safeJsonParse('{"ok":true}', {}).ok === true);
  test("isValidIncomingData nhận object có type", isValidIncomingData({ type: "message" }) === true);
  test("isValidIncomingData từ chối null", isValidIncomingData(null) === false);
  test("makeRemoteMessage ép text về string", makeRemoteMessage({ text: 123 }).text === "123");
  test("makeLocalMessage có type message", makeLocalMessage("hi").type === "message");
  test("sanitizeText giới hạn độ dài", sanitizeText("abcdef", 3) === "abc");
  test("dedupeFriends bỏ bạn trùng Peer ID", dedupeFriends([{ peerId: "a" }, { peerId: "a" }]).length === 1);
  test("makeHelloPayload có type hello", makeHelloPayload("Hải", "abc").type === "hello");

  for (const check of getEnvironmentChecks()) {
    test(check.name, check.ok);
  }

  return results;
}

function Button({ children, onClick, disabled, variant = "primary", title, type = "button" }) {
  return (
    <button type={type} title={title} className={`btn btn-${variant}`} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

function Panel({ children, className = "" }) {
  return <section className={`panel ${className}`}>{children}</section>;
}

function Field({ label, children }) {
  return (
    <label className="label">
      <span>{label}</span>
      {children}
    </label>
  );
}

function TextInput(props) {
  return <input {...props} className={`input ${props.className || ""}`} />;
}

export default function PeerJsMessengerApp() {
  const [profile, setProfile] = useState(() => loadStorage(STORAGE_KEYS.profile, EMPTY_PROFILE));
  const [friends, setFriends] = useState(() => dedupeFriends(loadStorage(STORAGE_KEYS.friends, [])));
  const [messages, setMessages] = useState(() => loadStorage(STORAGE_KEYS.messages, {}));

  const [peerStatus, setPeerStatus] = useState("idle");
  const [myPeerId, setMyPeerId] = useState("");
  const [selectedFriendId, setSelectedFriendId] = useState("");
  const [connectPeerId, setConnectPeerId] = useState("");
  const [friendName, setFriendName] = useState("");
  const [draft, setDraft] = useState("");
  const [notice, setNotice] = useState("Nhấn “Bật kết nối” để lấy mã Peer ID.");
  const [onlineMap, setOnlineMap] = useState({});
  const [testResults, setTestResults] = useState([]);
  const [environmentChecks, setEnvironmentChecks] = useState([]);

  const peerRef = useRef(null);
  const connectionsRef = useRef({});
  const bottomRef = useRef(null);
  const profileRef = useRef(profile);
  const myPeerIdRef = useRef(myPeerId);

  useEffect(() => {
    setEnvironmentChecks(getEnvironmentChecks());
  }, []);

  useEffect(() => {
    profileRef.current = profile;
  }, [profile]);

  useEffect(() => {
    myPeerIdRef.current = myPeerId;
  }, [myPeerId]);

  const selectedFriend = useMemo(
    () => friends.find((friend) => friend.peerId === selectedFriendId) || null,
    [friends, selectedFriendId]
  );

  const selectedMessages = selectedFriendId ? messages[selectedFriendId] || [] : [];

  useEffect(() => saveStorage(STORAGE_KEYS.profile, profile), [profile]);
  useEffect(() => saveStorage(STORAGE_KEYS.friends, friends), [friends]);
  useEffect(() => saveStorage(STORAGE_KEYS.messages, messages), [messages]);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [selectedFriendId, selectedMessages.length]);

  useEffect(() => {
    return () => {
      Object.values(connectionsRef.current).forEach((conn) => {
        try {
          conn.close();
        } catch {
          // Ignore close errors.
        }
      });
      try {
        peerRef.current?.destroy();
      } catch {
        // Ignore destroy errors.
      }
    };
  }, []);

  function addNotice(text) {
    setNotice(text);
  }

  function upsertFriend(peerId, name = "") {
    const cleanPeerId = normalizePeerId(peerId);
    if (!cleanPeerId) return;

    setFriends((current) => {
      const exists = current.some((friend) => friend.peerId === cleanPeerId);
      if (exists) {
        return current.map((friend) =>
          friend.peerId === cleanPeerId
            ? { ...friend, name: sanitizeText(name || friend.name || cleanPeerId, 80) }
            : friend
        );
      }

      return [
        ...current,
        {
          id: createId(),
          peerId: cleanPeerId,
          name: sanitizeText(name || `Bạn ${current.length + 1}`, 80),
          createdAt: Date.now(),
        },
      ];
    });

    setSelectedFriendId(cleanPeerId);
  }

  function appendMessage(friendPeerId, message) {
    setMessages((current) => ({
      ...current,
      [friendPeerId]: [...(current[friendPeerId] || []), message],
    }));
  }

  function sendHello(conn) {
    try {
      conn.send(makeHelloPayload(profileRef.current.displayName, myPeerIdRef.current || peerRef.current?.id || ""));
    } catch {
      // Connection may close before hello is sent.
    }
  }

  function attachConnection(conn) {
    if (!conn || !conn.peer) return;

    connectionsRef.current[conn.peer] = conn;
    upsertFriend(conn.peer, conn.metadata?.displayName || "");

    conn.on("open", () => {
      connectionsRef.current[conn.peer] = conn;
      setOnlineMap((current) => ({ ...current, [conn.peer]: true }));
      addNotice(`Đã kết nối với ${conn.metadata?.displayName || conn.peer}.`);
      sendHello(conn);
    });

    conn.on("data", (data) => {
      if (!isValidIncomingData(data)) return;

      if (data.type === "hello") {
        setOnlineMap((current) => ({ ...current, [conn.peer]: true }));
        upsertFriend(conn.peer, data.displayName || conn.peer);
        return;
      }

      if (data.type === "message") {
        const text = sanitizeText(data.text).trim();
        if (!text) return;
        setOnlineMap((current) => ({ ...current, [conn.peer]: true }));
        upsertFriend(conn.peer, data.displayName || conn.peer);
        appendMessage(conn.peer, makeRemoteMessage({ ...data, text }));
        setSelectedFriendId((current) => current || conn.peer);
      }
    });

    conn.on("close", () => {
      setOnlineMap((current) => ({ ...current, [conn.peer]: false }));
      addNotice(`Kết nối với ${conn.peer} đã đóng.`);
    });

    conn.on("error", () => {
      setOnlineMap((current) => ({ ...current, [conn.peer]: false }));
      addNotice("Có lỗi kết nối. Hãy kiểm tra Peer ID hoặc thử bật lại kết nối.");
    });
  }

  function startPeer() {
    if (peerRef.current && !peerRef.current.destroyed) {
      addNotice("Kết nối đã bật rồi.");
      return;
    }

    if (typeof Peer !== "function") {
      setPeerStatus("error");
      addNotice("PeerJS chưa tải được. Trên GitHub/Vite hãy chạy: npm install peerjs");
      return;
    }

    if (!isSecureEnoughForWebRTC()) {
      setPeerStatus("error");
      addNotice("WebRTC cần HTTPS hoặc localhost. GitHub Pages dùng HTTPS nên deploy lên đó sẽ phù hợp hơn mở file trực tiếp.");
      return;
    }

    setPeerStatus("connecting");
    addNotice("Đang tạo Peer ID...");

    try {
      const preferredId = normalizePeerId(profile.peerId);
      const peer = preferredId ? new Peer(preferredId, { debug: 1 }) : new Peer(undefined, { debug: 1 });
      peerRef.current = peer;

      peer.on("open", (id) => {
        setMyPeerId(id);
        setPeerStatus("online");
        setProfile((current) => ({ ...current, peerId: id }));
        addNotice("Đã sẵn sàng. Gửi Peer ID này cho bạn bè để họ kết nối.");
      });

      peer.on("connection", (conn) => {
        attachConnection(conn);
        addNotice(`${conn.peer} đang kết nối tới bạn.`);
      });

      peer.on("disconnected", () => {
        setPeerStatus("offline");
        addNotice("Peer bị ngắt. Hãy bấm “Tắt”, rồi “Bật kết nối” lại.");
      });

      peer.on("close", () => {
        setPeerStatus("offline");
        addNotice("Peer đã đóng.");
      });

      peer.on("error", (error) => {
        setPeerStatus("error");
        const message = error?.type === "unavailable-id"
          ? "Peer ID này đang được dùng. Hãy đổi Peer ID khác hoặc để trống để app tự tạo ID."
          : error?.type === "network"
            ? "Lỗi mạng PeerJS. Hãy kiểm tra Internet rồi thử lại."
            : error?.type === "peer-unavailable"
              ? "Không tìm thấy Peer ID kia. Hãy kiểm tra máy kia đã bật kết nối chưa."
              : "Không tạo được kết nối PeerJS. Hãy thử lại hoặc kiểm tra mạng.";
        addNotice(message);
      });
    } catch {
      setPeerStatus("error");
      addNotice("Không khởi tạo được PeerJS. Hãy thử tải lại app.");
    }
  }

  function stopPeer() {
    Object.values(connectionsRef.current).forEach((conn) => {
      try {
        conn.close();
      } catch {
        // Ignore close errors.
      }
    });
    connectionsRef.current = {};
    setOnlineMap({});

    try {
      peerRef.current?.destroy();
    } catch {
      // Ignore destroy errors.
    }

    peerRef.current = null;
    setPeerStatus("offline");
    setMyPeerId("");
    addNotice("Đã tắt kết nối.");
  }

  function connectToFriend(peerIdArg = connectPeerId, nameArg = friendName) {
    const cleanPeerId = normalizePeerId(peerIdArg);

    if (!peerRef.current || peerRef.current.destroyed || peerStatus !== "online") {
      addNotice("Bạn cần nhấn “Bật kết nối” trước.");
      return;
    }

    if (!cleanPeerId) {
      addNotice("Hãy nhập Peer ID của người bạn muốn kết nối.");
      return;
    }

    if (cleanPeerId === myPeerId) {
      addNotice("Không thể tự kết nối với chính mình.");
      return;
    }

    const existing = connectionsRef.current[cleanPeerId];
    if (existing && existing.open) {
      setSelectedFriendId(cleanPeerId);
      addNotice("Bạn đã kết nối với người này rồi.");
      return;
    }

    upsertFriend(cleanPeerId, nameArg || cleanPeerId);
    addNotice("Đang kết nối...");

    try {
      const conn = peerRef.current.connect(cleanPeerId, {
        reliable: true,
        metadata: { displayName: sanitizeText(profile.displayName || "Người dùng PeerJS", 80) },
      });
      attachConnection(conn);
      setConnectPeerId("");
      setFriendName("");
    } catch {
      addNotice("Không thể bắt đầu kết nối. Hãy kiểm tra Peer ID.");
    }
  }

  function sendMessage() {
    const text = sanitizeText(draft).trim();
    if (!text) return;

    if (!selectedFriendId) {
      addNotice("Hãy chọn một người bạn trước khi nhắn.");
      return;
    }

    const conn = connectionsRef.current[selectedFriendId];
    if (!conn || !conn.open) {
      addNotice("Bạn này chưa online/kết nối. Nhấn “Nối” trước khi gửi.");
      return;
    }

    const localMessage = makeLocalMessage(text);
    const outgoingMessage = {
      ...localMessage,
      displayName: sanitizeText(profile.displayName || "Người dùng PeerJS", 80),
    };

    try {
      conn.send(outgoingMessage);
      appendMessage(selectedFriendId, {
        id: localMessage.id,
        text: localMessage.text,
        from: "me",
        time: localMessage.time,
        createdAt: localMessage.createdAt,
      });
      setDraft("");
    } catch {
      addNotice("Gửi tin nhắn thất bại. Hãy kết nối lại rồi thử gửi lần nữa.");
    }
  }

  function copyMyId() {
    const value = myPeerId || profile.peerId;
    if (!value) {
      addNotice("Chưa có Peer ID để sao chép.");
      return;
    }

    if (!navigator.clipboard) {
      addNotice("Trình duyệt không hỗ trợ copy tự động. Hãy bôi đen Peer ID và copy thủ công.");
      return;
    }

    navigator.clipboard
      .writeText(value)
      .then(() => addNotice("Đã sao chép Peer ID."))
      .catch(() => addNotice("Không sao chép được. Hãy bôi đen và copy thủ công."));
  }

  function removeFriend(peerId) {
    setFriends((current) => current.filter((friend) => friend.peerId !== peerId));
    setMessages((current) => {
      const next = { ...current };
      delete next[peerId];
      return next;
    });

    try {
      connectionsRef.current[peerId]?.close();
    } catch {
      // Ignore close errors.
    }
    delete connectionsRef.current[peerId];

    setOnlineMap((current) => {
      const next = { ...current };
      delete next[peerId];
      return next;
    });

    if (selectedFriendId === peerId) setSelectedFriendId("");
    addNotice("Đã xóa bạn và lịch sử chat trên máy này.");
  }

  function clearCurrentChat() {
    if (!selectedFriendId) return;
    setMessages((current) => ({ ...current, [selectedFriendId]: [] }));
    addNotice("Đã xóa lịch sử chat với bạn này trên máy này.");
  }

  function clearAllLocalData() {
    stopPeer();
    setProfile(EMPTY_PROFILE);
    setFriends([]);
    setMessages({});
    setSelectedFriendId("");
    setConnectPeerId("");
    setFriendName("");
    setDraft("");
    removeStorage(STORAGE_KEYS.profile);
    removeStorage(STORAGE_KEYS.friends);
    removeStorage(STORAGE_KEYS.messages);
    addNotice("Đã xóa toàn bộ dữ liệu cục bộ của app trên máy này.");
  }

  function handleRunTests() {
    const results = runSelfTests();
    setTestResults(results);
    setEnvironmentChecks(getEnvironmentChecks());
    const failed = results.filter((result) => !result.passed).length;
    addNotice(failed === 0 ? "Tất cả kiểm tra cơ bản đều đạt." : `Có ${failed} kiểm tra chưa đạt.`);
  }

  const statusText = {
    idle: "Chưa bật",
    connecting: "Đang bật",
    online: "Online",
    offline: "Offline",
    error: "Lỗi",
  }[peerStatus] || "Không rõ";

  const onlineCount = Object.values(onlineMap).filter(Boolean).length;

  return (
    <div className="app-shell">
      <style>{APP_CSS}</style>
      <div className="app-wrap">
        <header className="app-header">
          <div>
            <div className="badge">
              <span>✓</span>
              <span>GitHub Pages friendly · React + PeerJS</span>
            </div>
            <h1 className="title">Mini Messenger PeerJS</h1>
            <p className="subtitle">
              App nhắn tin 1-1 chạy frontend tĩnh. Deploy được lên GitHub Pages, không cần server riêng. Hai máy cần mở app cùng lúc để nhắn trực tiếp qua PeerJS/WebRTC.
            </p>
          </div>

          <div className="status-card">
            <div className="status-row">
              <span>{peerStatus === "online" ? "🟢" : peerStatus === "connecting" ? "🟡" : peerStatus === "error" ? "🔴" : "⚪"}</span>
              <span>Trạng thái:</span>
              <strong>{statusText}</strong>
            </div>
            <div className="muted-small">Bạn online: {onlineCount}</div>
          </div>
        </header>

        <div className="layout">
          <div className="left-stack">
            <Panel>
              <div className="panel-body">
                <h2 className="panel-title">Hồ sơ & kết nối</h2>

                <div className="stack">
                  <Field label="Tên hiển thị">
                    <TextInput
                      value={profile.displayName}
                      onChange={(event) => setProfile((current) => ({ ...current, displayName: sanitizeText(event.target.value, 80) }))}
                      placeholder="Ví dụ: Hải"
                    />
                  </Field>

                  <Field label="Peer ID của bạn">
                    <div className="input-row">
                      <TextInput
                        value={myPeerId || profile.peerId}
                        onChange={(event) => setProfile((current) => ({ ...current, peerId: normalizePeerId(event.target.value) }))}
                        disabled={peerStatus === "online" || peerStatus === "connecting"}
                        placeholder="Để trống để app tự tạo ID"
                      />
                      <Button variant="secondary" onClick={copyMyId} title="Sao chép Peer ID">
                        Copy
                      </Button>
                    </div>
                    <p className="tiny-note">Nếu nhập ID riêng, hãy nhập trước khi bấm “Bật kết nối”. Nếu ID bị trùng, hãy đổi ID khác.</p>
                  </Field>

                  <div className="button-grid">
                    <Button onClick={startPeer} disabled={peerStatus === "connecting" || peerStatus === "online"}>
                      Bật kết nối
                    </Button>
                    <Button variant="secondary" onClick={stopPeer}>
                      Tắt
                    </Button>
                  </div>

                  <div className="notice">{notice}</div>
                </div>
              </div>
            </Panel>

            <Panel>
              <div className="panel-body">
                <h2 className="panel-title">＋ Kết bạn bằng Peer ID</h2>

                <div className="stack">
                  <TextInput value={friendName} onChange={(event) => setFriendName(sanitizeText(event.target.value, 80))} placeholder="Tên bạn bè, không bắt buộc" />
                  <TextInput
                    value={connectPeerId}
                    onChange={(event) => setConnectPeerId(event.target.value)}
                    onKeyDown={(event) => {
                      if (event.key === "Enter") connectToFriend();
                    }}
                    placeholder="Nhập Peer ID của máy kia"
                  />
                  <Button onClick={() => connectToFriend()}>🔗 Kết nối / thêm bạn</Button>
                </div>
              </div>
            </Panel>

            <Panel>
              <div className="panel-body">
                <h2 className="panel-title">💬 Bạn bè</h2>

                {friends.length === 0 ? (
                  <div className="empty">Chưa có bạn bè. Hãy nhập Peer ID của máy khác để bắt đầu.</div>
                ) : (
                  <div className="friend-list">
                    {friends.map((friend) => {
                      const isSelected = selectedFriendId === friend.peerId;
                      const isOnline = Boolean(onlineMap[friend.peerId]);

                      return (
                        <div key={friend.peerId} className={`friend-card ${isSelected ? "selected" : ""}`}>
                          <button type="button" className="friend-main" onClick={() => setSelectedFriendId(friend.peerId)}>
                            <div className="friend-name">{friend.name || friend.peerId}</div>
                            <div className="friend-id">
                              <span>{isOnline ? "🟢" : "⚪"}</span>
                              <span className="truncate">{friend.peerId}</span>
                            </div>
                          </button>

                          <div className="friend-actions" style={{ marginTop: 9 }}>
                            <Button variant="secondary" onClick={() => connectToFriend(friend.peerId, friend.name)}>
                              Nối
                            </Button>
                            <Button variant="danger" onClick={() => removeFriend(friend.peerId)}>
                              Xóa
                            </Button>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                )}
              </div>
            </Panel>

            <Panel>
              <div className="panel-body">
                <h2 className="panel-title">🧪 Rà lỗi & môi trường</h2>
                <div className="button-grid">
                  <Button variant="secondary" onClick={handleRunTests}>Chạy test</Button>
                  <Button variant="danger" onClick={clearAllLocalData}>Xóa dữ liệu</Button>
                </div>

                <div className="test-box" style={{ marginTop: 10 }}>
                  {environmentChecks.length === 0 ? (
                    <div>Đang kiểm tra môi trường...</div>
                  ) : (
                    environmentChecks.map((check) => (
                      <div key={check.name} className={check.ok ? "env-ok" : "env-warn"}>
                        {check.ok ? "✓" : "!"} {check.name}: {check.hint}
                      </div>
                    ))
                  )}
                </div>

                {testResults.length > 0 && (
                  <div className="test-box" style={{ marginTop: 10 }}>
                    {testResults.map((result) => (
                      <div key={result.name} className={result.passed ? "test-pass" : "test-fail"}>
                        {result.passed ? "✓" : "✕"} {result.name}
                      </div>
                    ))}
                  </div>
                )}
              </div>
            </Panel>

            <Panel>
              <div className="panel-body">
                <h2 className="panel-title">🚀 Deploy GitHub Pages</h2>
                <div className="help-box">
                  <ol>
                    <li>Tạo app bằng Vite React.</li>
                    <li>Cài dependency: <strong>npm install peerjs</strong>.</li>
                    <li>Đưa component này vào <strong>src/App.jsx</strong>.</li>
                    <li>Build: <strong>npm run build</strong>.</li>
                    <li>Deploy thư mục <strong>dist</strong> lên GitHub Pages.</li>
                  </ol>
                  <p className="tiny-note">GitHub Pages dùng HTTPS nên phù hợp với WebRTC. Không mở file index.html trực tiếp bằng đường dẫn file://.</p>
                </div>
              </div>
            </Panel>
          </div>

          <Panel className="chat-panel">
            <div className="chat-head">
              <div style={{ minWidth: 0 }}>
                <h2 className="chat-title truncate">{selectedFriend ? selectedFriend.name || selectedFriend.peerId : "Chọn một người bạn"}</h2>
                <p className="chat-subtitle truncate">
                  {selectedFriend ? (onlineMap[selectedFriend.peerId] ? "Đang kết nối trực tiếp" : "Chưa kết nối / offline") : "Tin nhắn sẽ xuất hiện ở đây"}
                </p>
              </div>

              <Button variant="secondary" onClick={clearCurrentChat} disabled={!selectedFriendId}>
                Xóa chat
              </Button>
            </div>

            <div className="chat-body">
              {!selectedFriend ? (
                <div className="center-empty">
                  <div>
                    <div style={{ fontSize: 56, marginBottom: 12 }}>💬</div>
                    <p>Chọn bạn bè hoặc thêm Peer ID để bắt đầu nhắn.</p>
                  </div>
                </div>
              ) : selectedMessages.length === 0 ? (
                <div className="empty">Chưa có tin nhắn với người này. Cả hai máy cần đang mở app và đã kết nối.</div>
              ) : (
                <div className="bubble-list">
                  {selectedMessages.map((message) => {
                    const mine = message.from === "me";
                    return (
                      <div key={message.id} className={`bubble-wrap ${mine ? "mine" : "theirs"}`}>
                        <div className={`bubble ${mine ? "mine" : "theirs"}`}>
                          <div className="bubble-text">{message.text}</div>
                          <div className="bubble-time">{message.time}</div>
                        </div>
                      </div>
                    );
                  })}
                  <div ref={bottomRef} />
                </div>
              )}
            </div>

            <div className="composer">
              <div className="composer-row">
                <textarea
                  className="textarea"
                  value={draft}
                  onChange={(event) => setDraft(event.target.value)}
                  onKeyDown={(event) => {
                    if (event.key === "Enter" && !event.shiftKey) {
                      event.preventDefault();
                      sendMessage();
                    }
                  }}
                  placeholder={selectedFriend ? "Nhập tin nhắn..." : "Chọn bạn bè trước"}
                  disabled={!selectedFriend}
                />
                <Button onClick={sendMessage} disabled={!selectedFriend || !draft.trim()}>
                  Gửi
                </Button>
              </div>
              <p className="tiny-note">Enter để gửi, Shift + Enter để xuống dòng. Tin nhắn chỉ lưu trên trình duyệt hiện tại.</p>
            </div>
          </Panel>
        </div>
      </div>
    </div>
  );
}
  } catch {
    // Local storage can fail in private mode or when storage is full.
  }
}

function createId() {
  if (typeof crypto !== "undefined" && crypto.randomUUID) return crypto.randomUUID();
  return `${Date.now()}-${Math.random().toString(16).slice(2)}`;
}

function nowLabel() {
  return new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
}

function normalizePeerId(value) {
  return String(value || "").trim().replace(/\s+/g, "");
}

function isValidIncomingData(data) {
  return data !== null && typeof data === "object" && typeof data.type === "string";
}

function makeLocalMessage(text) {
  return {
    id: createId(),
    type: "message",
    text,
    from: "me",
    time: nowLabel(),
    createdAt: Date.now(),
  };
}

function makeRemoteMessage(data) {
  return {
    id: data.id || createId(),
    text: String(data.text || ""),
    from: "them",
    time: data.time || nowLabel(),
    createdAt: data.createdAt || Date.now(),
  };
}

function runSelfTests() {
  const results = [];

  function test(name, condition) {
    results.push({ name, passed: Boolean(condition) });
  }

  test("normalizePeerId xóa khoảng trắng", normalizePeerId("  abc 123 \n") === "abc123");
  test("safeJsonParse trả fallback khi JSON lỗi", safeJsonParse("{bad", { ok: true }).ok === true);
  test("safeJsonParse đọc JSON hợp lệ", safeJsonParse('{"ok":true}', {}).ok === true);
  test("isValidIncomingData nhận object có type", isValidIncomingData({ type: "message" }) === true);
  test("isValidIncomingData từ chối null", isValidIncomingData(null) === false);
  test("makeRemoteMessage ép text về string", makeRemoteMessage({ text: 123 }).text === "123");
  test("makeLocalMessage có type message", makeLocalMessage("hi").type === "message");

  return results;
}

function Button({ children, onClick, disabled, variant = "primary", title, type = "button" }) {
  const base = "rounded-xl px-3 py-2 text-sm font-semibold transition focus:outline-none focus:ring-2 focus:ring-emerald-400 disabled:cursor-not-allowed disabled:opacity-50";
  const styles = {
    primary: "bg-emerald-400 text-slate-950 hover:bg-emerald-300",
    secondary: "bg-slate-800 text-slate-100 hover:bg-slate-700 ring-1 ring-white/10",
    danger: "bg-red-500/15 text-red-200 hover:bg-red-500/25 ring-1 ring-red-400/20",
    ghost: "bg-transparent text-slate-300 hover:bg-slate-800",
  };

  return (
    <button type={type} title={title} className={`${base} ${styles[variant] || styles.primary}`} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

function Panel({ children, className = "" }) {
  return <section className={`rounded-2xl bg-slate-900/90 shadow-xl ring-1 ring-white/10 ${className}`}>{children}</section>;
}

function Field({ label, children }) {
  return (
    <label className="block">
      <span className="mb-2 block text-sm text-slate-300">{label}</span>
      {children}
    </label>
  );
}

function TextInput(props) {
  return (
    <input
      {...props}
      className={`w-full rounded-xl border border-white/10 bg-slate-950 px-3 py-2 text-slate-100 outline-none placeholder:text-slate-600 focus:ring-2 focus:ring-emerald-400 ${props.className || ""}`}
    />
  );
}

export default function PeerJsMessengerApp() {
  const [profile, setProfile] = useState(() => loadStorage(STORAGE_KEYS.profile, EMPTY_PROFILE));
  const [friends, setFriends] = useState(() => loadStorage(STORAGE_KEYS.friends, []));
  const [messages, setMessages] = useState(() => loadStorage(STORAGE_KEYS.messages, {}));

  const [peerStatus, setPeerStatus] = useState("idle");
  const [myPeerId, setMyPeerId] = useState("");
  const [selectedFriendId, setSelectedFriendId] = useState("");
  const [connectPeerId, setConnectPeerId] = useState("");
  const [friendName, setFriendName] = useState("");
  const [draft, setDraft] = useState("");
  const [notice, setNotice] = useState("Nhấn “Bật kết nối” để lấy mã Peer ID.");
  const [onlineMap, setOnlineMap] = useState({});
  const [testResults, setTestResults] = useState([]);

  const peerRef = useRef(null);
  const connectionsRef = useRef({});
  const bottomRef = useRef(null);
  const profileRef = useRef(profile);
  const myPeerIdRef = useRef(myPeerId);

  useEffect(() => {
    profileRef.current = profile;
  }, [profile]);

  useEffect(() => {
    myPeerIdRef.current = myPeerId;
  }, [myPeerId]);

  const selectedFriend = useMemo(
    () => friends.find((friend) => friend.peerId === selectedFriendId) || null,
    [friends, selectedFriendId]
  );

  const selectedMessages = selectedFriendId ? messages[selectedFriendId] || [] : [];

  useEffect(() => saveStorage(STORAGE_KEYS.profile, profile), [profile]);
  useEffect(() => saveStorage(STORAGE_KEYS.friends, friends), [friends]);
  useEffect(() => saveStorage(STORAGE_KEYS.messages, messages), [messages]);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [selectedFriendId, selectedMessages.length]);

  useEffect(() => {
    return () => {
      Object.values(connectionsRef.current).forEach((conn) => {
        try {
          conn.close();
        } catch {
          // Ignore close errors.
        }
      });
      try {
        peerRef.current?.destroy();
      } catch {
        // Ignore destroy errors.
      }
    };
  }, []);

  function addNotice(text) {
    setNotice(text);
  }

  function upsertFriend(peerId, name = "") {
    const cleanPeerId = normalizePeerId(peerId);
    if (!cleanPeerId) return;

    setFriends((current) => {
      const exists = current.some((friend) => friend.peerId === cleanPeerId);
      if (exists) {
        return current.map((friend) =>
          friend.peerId === cleanPeerId ? { ...friend, name: name || friend.name || cleanPeerId } : friend
        );
      }

      return [
        ...current,
        {
          id: createId(),
          peerId: cleanPeerId,
          name: name || `Bạn ${current.length + 1}`,
          createdAt: Date.now(),
        },
      ];
    });

    setSelectedFriendId(cleanPeerId);
  }

  function appendMessage(friendPeerId, message) {
    setMessages((current) => ({
      ...current,
      [friendPeerId]: [...(current[friendPeerId] || []), message],
    }));
  }

  function sendHello(conn) {
    try {
      conn.send({
        type: "hello",
        displayName: profileRef.current.displayName || "Người dùng PeerJS",
        peerId: myPeerIdRef.current || peerRef.current?.id || "",
      });
    } catch {
      // Connection may close before hello is sent.
    }
  }

  function attachConnection(conn) {
    if (!conn || !conn.peer) return;

    connectionsRef.current[conn.peer] = conn;
    upsertFriend(conn.peer, conn.metadata?.displayName || "");

    conn.on("open", () => {
      connectionsRef.current[conn.peer] = conn;
      setOnlineMap((current) => ({ ...current, [conn.peer]: true }));
      addNotice(`Đã kết nối với ${conn.metadata?.displayName || conn.peer}.`);
      sendHello(conn);
    });

    conn.on("data", (data) => {
      if (!isValidIncomingData(data)) return;

      if (data.type === "hello") {
        setOnlineMap((current) => ({ ...current, [conn.peer]: true }));
        upsertFriend(conn.peer, data.displayName || conn.peer);
        return;
      }

      if (data.type === "message") {
        setOnlineMap((current) => ({ ...current, [conn.peer]: true }));
        upsertFriend(conn.peer, data.displayName || conn.peer);
        appendMessage(conn.peer, makeRemoteMessage(data));
        setSelectedFriendId((current) => current || conn.peer);
      }
    });

    conn.on("close", () => {
      setOnlineMap((current) => ({ ...current, [conn.peer]: false }));
      addNotice(`Kết nối với ${conn.peer} đã đóng.`);
    });

    conn.on("error", () => {
      setOnlineMap((current) => ({ ...current, [conn.peer]: false }));
      addNotice("Có lỗi kết nối. Hãy kiểm tra Peer ID hoặc thử bật lại kết nối.");
    });
  }

  function startPeer() {
    if (peerRef.current && !peerRef.current.destroyed) {
      addNotice("Kết nối đã bật rồi.");
      return;
    }

    if (typeof Peer !== "function") {
      setPeerStatus("error");
      addNotice("PeerJS chưa tải được. Hãy kiểm tra dependency peerjs trong môi trường chạy.");
      return;
    }

    setPeerStatus("connecting");
    addNotice("Đang tạo Peer ID...");

    try {
      const preferredId = normalizePeerId(profile.peerId);
      const peer = preferredId ? new Peer(preferredId) : new Peer();
      peerRef.current = peer;

      peer.on("open", (id) => {
        setMyPeerId(id);
        setPeerStatus("online");
        setProfile((current) => ({ ...current, peerId: id }));
        addNotice("Đã sẵn sàng. Gửi Peer ID này cho bạn bè để họ kết nối.");
      });

      peer.on("connection", (conn) => {
        attachConnection(conn);
        addNotice(`${conn.peer} đang kết nối tới bạn.`);
      });

      peer.on("disconnected", () => {
        setPeerStatus("offline");
        addNotice("Peer bị ngắt. Có thể thử bấm “Bật kết nối” lại.");
      });

      peer.on("close", () => {
        setPeerStatus("offline");
        addNotice("Peer đã đóng.");
      });

      peer.on("error", (error) => {
        setPeerStatus("error");
        const message = error?.type === "unavailable-id"
          ? "Peer ID này đang được dùng. Hãy đổi Peer ID khác hoặc để trống để app tự tạo ID."
          : error?.type === "network"
            ? "Lỗi mạng PeerJS. Hãy kiểm tra Internet rồi thử lại."
            : "Không tạo được kết nối PeerJS. Hãy thử lại hoặc kiểm tra mạng.";
        addNotice(message);
      });
    } catch {
      setPeerStatus("error");
      addNotice("Không khởi tạo được PeerJS. Hãy thử tải lại app.");
    }
  }

  function stopPeer() {
    Object.values(connectionsRef.current).forEach((conn) => {
      try {
        conn.close();
      } catch {
        // Ignore close errors.
      }
    });
    connectionsRef.current = {};
    setOnlineMap({});

    try {
      peerRef.current?.destroy();
    } catch {
      // Ignore destroy errors.
    }

    peerRef.current = null;
    setPeerStatus("offline");
    setMyPeerId("");
    addNotice("Đã tắt kết nối.");
  }

  function connectToFriend(peerIdArg = connectPeerId, nameArg = friendName) {
    const cleanPeerId = normalizePeerId(peerIdArg);

    if (!peerRef.current || peerRef.current.destroyed || peerStatus !== "online") {
      addNotice("Bạn cần nhấn “Bật kết nối” trước.");
      return;
    }

    if (!cleanPeerId) {
      addNotice("Hãy nhập Peer ID của người bạn muốn kết nối.");
      return;
    }

    if (cleanPeerId === myPeerId) {
      addNotice("Không thể tự kết nối với chính mình.");
      return;
    }

    const existing = connectionsRef.current[cleanPeerId];
    if (existing && existing.open) {
      setSelectedFriendId(cleanPeerId);
      addNotice("Bạn đã kết nối với người này rồi.");
      return;
    }

    upsertFriend(cleanPeerId, nameArg || cleanPeerId);
    addNotice("Đang kết nối...");

    try {
      const conn = peerRef.current.connect(cleanPeerId, {
        reliable: true,
        metadata: { displayName: profile.displayName || "Người dùng PeerJS" },
      });
      attachConnection(conn);
      setConnectPeerId("");
      setFriendName("");
    } catch {
      addNotice("Không thể bắt đầu kết nối. Hãy kiểm tra Peer ID.");
    }
  }

  function sendMessage() {
    const text = draft.trim();
    if (!text) return;

    if (!selectedFriendId) {
      addNotice("Hãy chọn một người bạn trước khi nhắn.");
      return;
    }

    const conn = connectionsRef.current[selectedFriendId];
    if (!conn || !conn.open) {
      addNotice("Bạn này chưa online/kết nối. Nhấn “Nối” trước khi gửi.");
      return;
    }

    const localMessage = makeLocalMessage(text);
    const outgoingMessage = {
      ...localMessage,
      displayName: profile.displayName || "Người dùng PeerJS",
    };

    try {
      conn.send(outgoingMessage);
      appendMessage(selectedFriendId, {
        id: localMessage.id,
        text: localMessage.text,
        from: "me",
        time: localMessage.time,
        createdAt: localMessage.createdAt,
      });
      setDraft("");
    } catch {
      addNotice("Gửi tin nhắn thất bại. Hãy kết nối lại rồi thử gửi lần nữa.");
    }
  }

  function copyMyId() {
    const value = myPeerId || profile.peerId;
    if (!value) {
      addNotice("Chưa có Peer ID để sao chép.");
      return;
    }

    if (!navigator.clipboard) {
      addNotice("Trình duyệt không hỗ trợ copy tự động. Hãy bôi đen Peer ID và copy thủ công.");
      return;
    }

    navigator.clipboard
      .writeText(value)
      .then(() => addNotice("Đã sao chép Peer ID."))
      .catch(() => addNotice("Không sao chép được. Hãy bôi đen và copy thủ công."));
  }

  function removeFriend(peerId) {
    setFriends((current) => current.filter((friend) => friend.peerId !== peerId));
    setMessages((current) => {
      const next = { ...current };
      delete next[peerId];
      return next;
    });

    try {
      connectionsRef.current[peerId]?.close();
    } catch {
      // Ignore close errors.
    }
    delete connectionsRef.current[peerId];

    setOnlineMap((current) => {
      const next = { ...current };
      delete next[peerId];
      return next;
    });

    if (selectedFriendId === peerId) setSelectedFriendId("");
    addNotice("Đã xóa bạn và lịch sử chat trên máy này.");
  }

  function clearCurrentChat() {
    if (!selectedFriendId) return;
    setMessages((current) => ({ ...current, [selectedFriendId]: [] }));
    addNotice("Đã xóa lịch sử chat với bạn này trên máy này.");
  }

  function clearAllLocalData() {
    stopPeer();
    setProfile(EMPTY_PROFILE);
    setFriends([]);
    setMessages({});
    setSelectedFriendId("");
    setConnectPeerId("");
    setFriendName("");
    setDraft("");
    addNotice("Đã xóa toàn bộ dữ liệu cục bộ của app trên máy này.");
  }

  function handleRunTests() {
    const results = runSelfTests();
    setTestResults(results);
    const failed = results.filter((result) => !result.passed).length;
    addNotice(failed === 0 ? "Tất cả kiểm tra cơ bản đều đạt." : `Có ${failed} kiểm tra chưa đạt.`);
  }

  const statusText = {
    idle: "Chưa bật",
    connecting: "Đang bật",
    online: "Online",
    offline: "Offline",
    error: "Lỗi",
  }[peerStatus] || "Không rõ";

  const onlineCount = Object.values(onlineMap).filter(Boolean).length;

  return (
    <div className="min-h-screen bg-slate-950 p-4 text-slate-100 md:p-6">
      <div className="mx-auto max-w-6xl">
        <header className="mb-5 flex flex-col gap-3 md:flex-row md:items-end md:justify-between">
          <div>
            <div className="inline-flex items-center gap-2 rounded-full bg-emerald-500/10 px-3 py-1 text-sm text-emerald-300 ring-1 ring-emerald-500/20">
              <span>✓</span>
              <span>PeerJS trực tiếp, lưu cục bộ</span>
            </div>
            <h1 className="mt-3 text-3xl font-bold tracking-tight md:text-4xl">Mini Messenger PeerJS</h1>
            <p className="mt-2 max-w-2xl text-slate-300">
              Nhắn tin 1-1 giữa hai máy đang mở app. Bạn bè và lịch sử chat được lưu bằng localStorage trên trình duyệt hiện tại.
            </p>
          </div>

          <div className="rounded-2xl bg-slate-900/80 px-4 py-3 shadow-xl ring-1 ring-white/10">
            <div className="flex items-center gap-2 text-sm">
              <span>{peerStatus === "online" ? "🟢" : peerStatus === "connecting" ? "🟡" : "⚪"}</span>
              <span>Trạng thái:</span>
              <strong>{statusText}</strong>
            </div>
            <div className="mt-1 text-xs text-slate-400">Bạn online: {onlineCount}</div>
          </div>
        </header>

        <div className="grid gap-4 lg:grid-cols-[360px_1fr]">
          <div className="space-y-4">
            <Panel>
              <div className="p-4">
                <h2 className="mb-3 text-lg font-semibold">Hồ sơ & kết nối</h2>

                <div className="space-y-3">
                  <Field label="Tên hiển thị">
                    <TextInput
                      value={profile.displayName}
                      onChange={(event) => setProfile((current) => ({ ...current, displayName: event.target.value }))}
                      placeholder="Ví dụ: Hải"
                    />
                  </Field>

                  <Field label="Peer ID của bạn">
                    <div className="flex gap-2">
                      <TextInput
                        value={myPeerId || profile.peerId}
                        onChange={(event) => setProfile((current) => ({ ...current, peerId: normalizePeerId(event.target.value) }))}
                        disabled={peerStatus === "online" || peerStatus === "connecting"}
                        placeholder="Để trống để app tự tạo ID"
                      />
                      <Button variant="secondary" onClick={copyMyId} title="Sao chép Peer ID">
                        Copy
                      </Button>
                    </div>
                    <p className="mt-2 text-xs text-slate-500">Nếu nhập ID riêng, hãy nhập trước khi bấm “Bật kết nối”.</p>
                  </Field>

                  <div className="grid grid-cols-2 gap-2">
                    <Button onClick={startPeer} disabled={peerStatus === "connecting" || peerStatus === "online"}>
                      Bật kết nối
                    </Button>
                    <Button variant="secondary" onClick={stopPeer}>
                      Tắt
                    </Button>
                  </div>

                  <div className="rounded-xl bg-slate-950 p-3 text-sm text-slate-300 ring-1 ring-white/10">{notice}</div>
                </div>
              </div>
            </Panel>

            <Panel>
              <div className="p-4">
                <h2 className="mb-3 text-lg font-semibold">＋ Kết bạn bằng Peer ID</h2>

                <div className="space-y-2">
                  <TextInput value={friendName} onChange={(event) => setFriendName(event.target.value)} placeholder="Tên bạn bè, không bắt buộc" />
                  <TextInput
                    value={connectPeerId}
                    onChange={(event) => setConnectPeerId(event.target.value)}
                    onKeyDown={(event) => {
                      if (event.key === "Enter") connectToFriend();
                    }}
                    placeholder="Nhập Peer ID của máy kia"
                  />
                  <Button onClick={() => connectToFriend()}>🔗 Kết nối / thêm bạn</Button>
                </div>
              </div>
            </Panel>

            <Panel>
              <div className="p-4">
                <h2 className="mb-3 text-lg font-semibold">💬 Bạn bè</h2>

                {friends.length === 0 ? (
                  <div className="rounded-xl bg-slate-950 p-4 text-sm text-slate-400 ring-1 ring-white/10">
                    Chưa có bạn bè. Hãy nhập Peer ID của máy khác để bắt đầu.
                  </div>
                ) : (
                  <div className="space-y-2">
                    {friends.map((friend) => {
                      const isSelected = selectedFriendId === friend.peerId;
                      const isOnline = Boolean(onlineMap[friend.peerId]);

                      return (
                        <div
                          key={friend.peerId}
                          className={`rounded-xl border p-2 transition ${
                            isSelected ? "border-emerald-400 bg-emerald-400/10" : "border-white/10 bg-slate-950 hover:bg-slate-800"
                          }`}
                        >
                          <button type="button" className="w-full text-left" onClick={() => setSelectedFriendId(friend.peerId)}>
                            <div className="truncate font-medium">{friend.name || friend.peerId}</div>
                            <div className="mt-1 flex items-center gap-2 text-xs text-slate-400">
                              <span>{isOnline ? "🟢" : "⚪"}</span>
                              <span className="truncate">{friend.peerId}</span>
                            </div>
                          </button>

                          <div className="mt-2 grid grid-cols-2 gap-2">
                            <Button variant="secondary" onClick={() => connectToFriend(friend.peerId, friend.name)}>
                              Nối
                            </Button>
                            <Button variant="danger" onClick={() => removeFriend(friend.peerId)}>
                              Xóa
                            </Button>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                )}
              </div>
            </Panel>

            <Panel>
              <div className="p-4">
                <h2 className="mb-3 text-lg font-semibold">🧪 Kiểm tra lỗi cơ bản</h2>
                <div className="grid grid-cols-2 gap-2">
                  <Button variant="secondary" onClick={handleRunTests}>Chạy test</Button>
                  <Button variant="danger" onClick={clearAllLocalData}>Xóa dữ liệu</Button>
                </div>

                {testResults.length > 0 && (
                  <div className="mt-3 space-y-1 rounded-xl bg-slate-950 p-3 text-sm ring-1 ring-white/10">
                    {testResults.map((result) => (
                      <div key={result.name} className={result.passed ? "text-emerald-300" : "text-red-300"}>
                        {result.passed ? "✓" : "✕"} {result.name}
                      </div>
                    ))}
                  </div>
                )}
              </div>
            </Panel>
          </div>

          <Panel className="min-h-[720px]">
            <div className="flex min-h-[720px] flex-col">
              <div className="flex items-center justify-between border-b border-white/10 p-4">
                <div className="min-w-0">
                  <h2 className="truncate text-xl font-semibold">{selectedFriend ? selectedFriend.name || selectedFriend.peerId : "Chọn một người bạn"}</h2>
                  <p className="truncate text-sm text-slate-400">
                    {selectedFriend ? (onlineMap[selectedFriend.peerId] ? "Đang kết nối trực tiếp" : "Chưa kết nối / offline") : "Tin nhắn sẽ xuất hiện ở đây"}
                  </p>
                </div>

                <Button variant="secondary" onClick={clearCurrentChat} disabled={!selectedFriendId}>
                  Xóa chat
                </Button>
              </div>

              <div className="flex-1 overflow-y-auto p-4">
                {!selectedFriend ? (
                  <div className="flex h-full items-center justify-center text-center text-slate-400">
                    <div>
                      <div className="mb-3 text-5xl">💬</div>
                      <p>Chọn bạn bè hoặc thêm Peer ID để bắt đầu nhắn.</p>
                    </div>
                  </div>
                ) : selectedMessages.length === 0 ? (
                  <div className="rounded-2xl bg-slate-950 p-5 text-center text-sm text-slate-400 ring-1 ring-white/10">
                    Chưa có tin nhắn với người này. Cả hai máy cần đang mở app và đã kết nối.
                  </div>
                ) : (
                  <div className="space-y-3">
                    {selectedMessages.map((message) => {
                      const mine = message.from === "me";
                      return (
                        <div key={message.id} className={`flex ${mine ? "justify-end" : "justify-start"}`}>
                          <div className={`max-w-[78%] rounded-2xl px-4 py-2 shadow ${mine ? "bg-emerald-400 text-slate-950" : "bg-slate-800 text-slate-100"}`}>
                            <div className="whitespace-pre-wrap break-words text-sm md:text-base">{message.text}</div>
                            <div className={`mt-1 text-right text-xs ${mine ? "text-slate-800" : "text-slate-400"}`}>{message.time}</div>
                          </div>
                        </div>
                      );
                    })}
                    <div ref={bottomRef} />
                  </div>
                )}
              </div>

              <div className="border-t border-white/10 p-4">
                <div className="flex gap-2">
                  <textarea
                    className="max-h-32 min-h-[48px] flex-1 resize-none rounded-2xl border border-white/10 bg-slate-950 px-4 py-3 text-slate-100 outline-none placeholder:text-slate-600 focus:ring-2 focus:ring-emerald-400"
                    value={draft}
                    onChange={(event) => setDraft(event.target.value)}
                    onKeyDown={(event) => {
                      if (event.key === "Enter" && !event.shiftKey) {
                        event.preventDefault();
                        sendMessage();
                      }
                    }}
                    placeholder={selectedFriend ? "Nhập tin nhắn..." : "Chọn bạn bè trước"}
                    disabled={!selectedFriend}
                  />
                  <Button onClick={sendMessage} disabled={!selectedFriend || !draft.trim()}>
                    Gửi
                  </Button>
                </div>
                <p className="mt-2 text-xs text-slate-500">Enter để gửi, Shift + Enter để xuống dòng. Tin nhắn chỉ lưu trên trình duyệt hiện tại.</p>
              </div>
            </div>
          </Panel>
        </div>
      </div>
    </div>
  );
}
      outline: none;
      padding: 13px 14px;
      font-size: 16px;
    }

    input:focus {
      border-color: var(--cyan);
    }

    input:disabled {
      opacity: 0.6;
    }

    .app {
      width: 100%;
      height: 100dvh;
      min-height: 100dvh;
      display: grid;
      grid-template-columns: 340px minmax(0, 1fr);
      gap: 14px;
      padding: 14px;
      max-width: 1200px;
      margin: 0 auto;
    }

    .card {
      min-width: 0;
      background: rgba(15, 23, 42, 0.88);
      border: 1px solid var(--border);
      border-radius: 24px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.32);
      backdrop-filter: blur(12px);
    }

    .sidebar {
      padding: 18px;
      overflow: hidden;
    }

    .brand {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 18px;
    }

    .logo {
      flex: 0 0 auto;
      width: 44px;
      height: 44px;
      border-radius: 17px;
      background: rgba(34, 211, 238, 0.14);
      display: grid;
      place-items: center;
      font-size: 23px;
    }

    h1,
    h2,
    p {
      margin: 0;
    }

    h1 {
      font-size: 21px;
      line-height: 1.15;
    }

    h2 {
      font-size: 18px;
    }

    .muted {
      color: var(--muted);
      font-size: 14px;
    }

    label {
      display: block;
      color: #cbd5e1;
      font-size: 14px;
      margin: 12px 0 7px;
    }

    .row {
      display: flex;
      gap: 8px;
      min-width: 0;
    }

    .btn {
      flex: 0 0 auto;
      border-radius: 15px;
      padding: 12px 14px;
      color: var(--text);
      background: #1e293b;
      transition: 0.18s;
      white-space: nowrap;
    }

    .btn:hover {
      background: #334155;
    }

    .btn-primary,
    .btn-danger {
      width: 100%;
      margin-top: 12px;
      border-radius: 16px;
      padding: 13px 14px;
      font-weight: 700;
    }

    .btn-primary {
      background: var(--cyan);
      color: #020617;
    }

    .btn-danger {
      background: var(--rose);
      color: white;
    }

    .status-box,
    .users-box,
    .feature-box {
      margin-top: 14px;
      border-radius: 20px;
      padding: 14px;
      background: #020617;
      border: 1px solid var(--border);
      overflow-wrap: anywhere;
    }

    .status-head,
    .user-item,
    .chat-head,
    .head-actions {
      display: flex;
      align-items: center;
    }

    .status-head,
    .user-item,
    .chat-head {
      justify-content: space-between;
      gap: 10px;
    }

    .pill {
      flex: 0 0 auto;
      font-size: 12px;
      padding: 4px 9px;
      border-radius: 999px;
      color: #cbd5e1;
      background: #334155;
    }

    .pill.online {
      background: rgba(16, 185, 129, 0.16);
      color: #6ee7b7;
    }

    .small-id {
      color: #64748b;
      font-size: 12px;
      word-break: break-all;
      margin-top: 8px;
    }

    .user-list {
      display: grid;
      gap: 8px;
      margin-top: 10px;
    }

    .user-item {
      padding: 9px 11px;
      border-radius: 14px;
      background: #0f172a;
      font-size: 14px;
      min-width: 0;
    }

    .user-item span:first-child {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .host-tag {
      color: #67e8f9;
      font-size: 11px;
      flex: 0 0 auto;
    }

    .feature-box {
      background: linear-gradient(135deg, rgba(217, 70, 239, 0.12), rgba(34, 211, 238, 0.1));
      border-color: rgba(217, 70, 239, 0.22);
    }

    .chat {
      min-width: 0;
      height: calc(100dvh - 28px);
      min-height: 0;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    .chat-head {
      flex: 0 0 auto;
      padding: 15px 16px;
      border-bottom: 1px solid var(--border);
    }

    .chat-title {
      min-width: 0;
    }

    .chat-title p {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      max-width: 100%;
    }

    .head-actions {
      gap: 10px;
      flex: 0 0 auto;
    }

    .dot {
      width: 12px;
      height: 12px;
      border-radius: 999px;
      background: #475569;
      flex: 0 0 auto;
    }

    .dot.online {
      background: var(--green);
      box-shadow: 0 0 18px rgba(52, 211, 153, 0.75);
    }

    .panel {
      flex: 0 0 auto;
      display: none;
      padding: 13px;
      border-bottom: 1px solid var(--border);
      background: rgba(2, 6, 23, 0.75);
      max-height: 30dvh;
      overflow-y: auto;
      overflow-x: hidden;
    }

    .panel.open {
      display: block;
    }

    .tabs,
    .pack-row {
      display: flex;
      gap: 8px;
      flex-wrap: nowrap;
      overflow-x: auto;
      padding-bottom: 3px;
      scrollbar-width: none;
    }

    .tabs::-webkit-scrollbar,
    .pack-row::-webkit-scrollbar {
      display: none;
    }

    .tabs {
      margin-bottom: 13px;
    }

    .tab,
    .pack-btn {
      flex: 0 0 auto;
      border-radius: 999px;
      padding: 9px 12px;
      color: #cbd5e1;
      background: #1e293b;
      white-space: nowrap;
      font-size: 14px;
    }

    .tab.active.cyan {
      background: var(--cyan);
      color: #020617;
      font-weight: 700;
    }

    .tab.active.pink,
    .pack-btn.active {
      background: var(--pink);
      color: white;
      font-weight: 700;
    }

    .tab.active.amber {
      background: var(--amber);
      color: #020617;
      font-weight: 700;
    }

    .icon-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(42px, 1fr));
      gap: 8px;
      width: 100%;
    }

    .icon-btn {
      min-width: 0;
      height: 43px;
      border-radius: 14px;
      background: #0f172a;
      border: 1px solid var(--border);
      font-size: 23px;
      color: white;
    }

    .icon-btn:hover {
      border-color: var(--cyan);
      background: rgba(34, 211, 238, 0.14);
    }

    .sticker-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(92px, 1fr));
      gap: 9px;
      width: 100%;
    }

    .sticker-btn {
      min-width: 0;
      min-height: 82px;
      border-radius: 20px;
      color: white;
      font-size: 34px;
      border: 1px solid #334155;
      background: linear-gradient(135deg, #0f172a, #1e293b);
      box-shadow: 0 10px 24px rgba(0, 0, 0, 0.22);
    }

    .sticker-btn:hover {
      border-color: #f0abfc;
      background: linear-gradient(135deg, rgba(217, 70, 239, 0.22), rgba(34, 211, 238, 0.14));
    }

    .quick-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
      gap: 8px;
    }

    .quick-btn {
      min-width: 0;
      text-align: left;
      border-radius: 14px;
      padding: 13px;
      color: var(--text);
      background: #0f172a;
      border: 1px solid var(--border);
      overflow-wrap: anywhere;
    }

    .messages {
      flex: 1 1 0;
      min-height: 0;
      overflow-y: auto;
      overflow-x: hidden;
      -webkit-overflow-scrolling: touch;
      padding: 16px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      overscroll-behavior: contain;
    }

    .empty {
      flex: 1;
      min-height: 260px;
      display: grid;
      place-items: center;
      text-align: center;
      color: #64748b;
      padding: 24px;
    }

    .msg-wrap {
      display: flex;
      max-width: 100%;
      animation: pop 0.18s ease-out;
    }

    .msg-wrap.mine {
      justify-content: flex-end;
    }

    .msg-wrap.other {
      justify-content: flex-start;
    }

    .msg-wrap.system {
      justify-content: center;
    }

    @keyframes pop {
      from {
        opacity: 0;
        transform: translateY(8px) scale(0.98);
      }
      to {
        opacity: 1;
        transform: translateY(0) scale(1);
      }
    }

    .bubble,
    .sticker-bubble {
      max-width: min(78%, 520px);
      min-width: 0;
      overflow-wrap: anywhere;
      word-break: break-word;
    }

    .bubble {
      border-radius: 20px;
      padding: 11px 14px;
      box-shadow: 0 10px 26px rgba(0, 0, 0, 0.2);
      background: #1e293b;
      color: var(--text);
      white-space: pre-wrap;
    }

    .mine .bubble {
      background: var(--cyan);
      color: #020617;
    }

    .meta {
      display: flex;
      gap: 8px;
      align-items: center;
      margin-bottom: 5px;
      font-size: 12px;
      font-weight: 700;
      opacity: 0.78;
      min-width: 0;
    }

    .meta span:first-child {
      min-width: 0;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .time {
      flex: 0 0 auto;
      font-size: 11px;
      opacity: 0.65;
      font-weight: 400;
    }

    .system-text {
      display: inline-block;
      max-width: 92%;
      border-radius: 999px;
      padding: 6px 11px;
      font-size: 12px;
      color: var(--muted);
      background: #020617;
      border: 1px solid var(--border);
      overflow-wrap: anywhere;
    }

    .sticker-bubble {
      border-radius: 30px;
      padding: 14px 16px;
      border: 1px solid rgba(240, 171, 252, 0.35);
      background: linear-gradient(135deg, #1e293b, #312e81);
      box-shadow: 0 16px 36px rgba(0, 0, 0, 0.28);
    }

    .mine .sticker-bubble {
      border-color: rgba(103, 232, 249, 0.45);
      background: linear-gradient(135deg, rgba(34, 211, 238, 0.28), rgba(217, 70, 239, 0.24));
    }

    .sticker-big {
      font-size: clamp(42px, 13vw, 68px);
      line-height: 1.05;
      text-align: center;
      padding: 7px 10px;
      filter: drop-shadow(0 8px 10px rgba(0, 0, 0, 0.25));
    }

    .composer {
      flex: 0 0 auto;
      padding: 12px;
      padding-bottom: max(12px, env(safe-area-inset-bottom));
      border-top: 1px solid var(--border);
      background: rgba(2, 6, 23, 0.88);
    }

    .composer .row {
      align-items: center;
    }

    .send-btn {
      flex: 0 0 auto;
      min-width: 74px;
      border-radius: 15px;
      padding: 13px 14px;
      color: #020617;
      background: var(--cyan);
      font-weight: 700;
      white-space: nowrap;
    }

    .hidden {
      display: none !important;
    }

    .desktop-only {
      display: inline;
    }

    @media (max-width: 760px) {
      html,
      body {
        height: 100%;
        position: fixed;
        inset: 0;
      }

      body {
        overflow: hidden;
      }

      .app {
        height: 100dvh;
        min-height: 0;
        display: flex;
        flex-direction: column;
        gap: 6px;
        padding: 6px;
        overflow: hidden;
      }

      .card {
        border-radius: 20px;
      }

      .sidebar {
        flex: 0 0 auto;
        padding: 9px;
        border-radius: 18px;
      }

      .brand {
        margin-bottom: 6px;
      }

      .logo {
        width: 38px;
        height: 38px;
        border-radius: 14px;
        font-size: 20px;
      }

      h1 {
        font-size: 18px;
      }

      label {
        margin-top: 7px;
        margin-bottom: 5px;
        font-size: 13px;
      }

      input {
        padding: 11px 12px;
      }

      .btn-primary,
      .btn-danger {
        margin-top: 8px;
        padding: 11px 12px;
      }

      .status-box,
      .users-box,
      .feature-box {
        margin-top: 7px;
        padding: 8px 10px;
        border-radius: 15px;
      }

      .status-box p,
      .users-box p {
        font-size: 13px;
      }

      .feature-box,
      .small-id {
        display: none;
      }

      .users-box {
        display: none;
      }

      .chat {
        flex: 1 1 0;
        height: auto;
        min-height: 0;
        border-radius: 18px;
      }

      .chat-head {
        padding: 9px 10px;
      }

      .head-actions {
        gap: 7px;
      }

      .desktop-only,
      .panel-word {
        display: none;
      }

      .btn,
      .send-btn {
        padding: 12px;
      }

      .panel {
        max-height: 24dvh;
        padding: 8px;
      }

      .tab,
      .pack-btn {
        font-size: 13px;
        padding: 8px 10px;
      }

      .icon-grid {
        grid-template-columns: repeat(auto-fill, minmax(39px, 1fr));
        gap: 7px;
      }

      .icon-btn {
        height: 40px;
        font-size: 22px;
      }

      .sticker-grid {
        grid-template-columns: repeat(auto-fill, minmax(82px, 1fr));
        gap: 8px;
      }

      .sticker-btn {
        min-height: 76px;
        font-size: 31px;
      }

      .quick-grid {
        grid-template-columns: 1fr;
      }

      .messages {
        flex: 1 1 0;
        padding: 9px;
        gap: 8px;
        min-height: 0;
        overflow-y: auto;
      }

      .bubble,
      .sticker-bubble {
        max-width: 86%;
      }

      .bubble {
        padding: 10px 12px;
        border-radius: 18px;
      }

      .sticker-bubble {
        padding: 12px 13px;
        border-radius: 24px;
      }

      .composer {
        padding: 7px;
        padding-bottom: max(7px, env(safe-area-inset-bottom));
      }

      .composer .row {
        gap: 6px;
      }

      .composer input {
        padding: 10px 11px;
      }

      .send-btn {
        min-width: 52px;
      }

      .send-text {
        display: none;
      }
    }

    @media (max-width: 390px) {
      .app {
        padding: 5px;
        gap: 5px;
      }

      .sidebar {
        padding: 8px;
      }

      .chat-head {
        padding: 8px;
      }

      .panel {
        padding: 9px;
      }

      .messages {
        padding: 8px;
      }

      .bubble,
      .sticker-bubble {
        max-width: 90%;
      }

      .icon-grid {
        grid-template-columns: repeat(auto-fill, minmax(36px, 1fr));
      }

      .icon-btn {
        height: 38px;
        font-size: 20px;
      }

      .sticker-grid {
        grid-template-columns: repeat(auto-fill, minmax(76px, 1fr));
      }
    }
  </style>
</head>
<body>
  <div class="app">
    <aside class="card sidebar">
      <div class="brand">
        <div class="logo" id="wifiIcon">📴</div>
        <div>
          <h1>PeerJS Room Chat</h1>
          <p class="muted">Chat bằng mã phòng</p>
        </div>
      </div>

      <label for="nameInput">Tên của bạn</label>
      <input id="nameInput" placeholder="Nhập tên" />

      <label for="roomInput">Mã phòng</label>
      <div class="row">
        <input id="roomInput" placeholder="vd: lop7a" />
        <button class="btn" id="copyBtn" title="Copy mã phòng">📋</button>
      </div>

      <button class="btn-primary" id="joinBtn">🚪 Vào / tạo phòng</button>
      <button class="btn-danger hidden" id="leaveBtn">🏃 Rời phòng</button>

      <div class="status-box">
        <div class="status-head">
          <span class="muted">Trạng thái</span>
          <span class="pill" id="onlinePill">Offline</span>
        </div>
        <p id="statusText">Chưa vào phòng</p>
        <p class="small-id hidden" id="peerIdText"></p>
        <p class="small-id hidden" id="hostIdText"></p>
      </div>

      <div class="users-box">
        <p>👥 Online (<span id="userCount">0</span>)</p>
        <div class="user-list" id="userList">
          <p class="muted">Chưa có ai trong phòng.</p>
        </div>
      </div>

      <div class="feature-box">
        <p><b>✨ Tính năng</b></p>
        <p class="muted" style="margin-top: 8px;">Gửi icon độc lạ, sticker lớn, tin nhắn nhanh và bong bóng đẹp.</p>
      </div>
    </aside>

    <main class="card chat">
      <div class="chat-head">
        <div class="chat-title">
          <h2>Tin nhắn</h2>
          <p class="muted">Phòng: <span id="roomLabel">chưa có</span></p>
        </div>
        <div class="head-actions">
          <button class="btn" id="panelBtn">✨ <span class="panel-word">Sticker</span></button>
          <div class="dot" id="onlineDot"></div>
        </div>
      </div>

      <section class="panel" id="panel">
        <div class="tabs">
          <button class="tab active cyan" data-tab="icons">😊 Icon</button>
          <button class="tab" data-tab="stickers">🏷️ Sticker</button>
          <button class="tab" data-tab="quick">🪄 Nhanh</button>
        </div>

        <div id="iconsPanel" class="icon-grid"></div>
        <div id="stickersPanel" class="hidden">
          <div class="pack-row" id="packRow"></div>
          <div class="sticker-grid" id="stickerGrid"></div>
        </div>
        <div id="quickPanel" class="quick-grid hidden"></div>
      </section>

      <section class="messages" id="messages">
        <div class="empty" id="emptyState">Nhập mã phòng rồi bắt đầu nhắn tin.</div>
      </section>

      <section class="composer">
        <div class="row">
          <button class="btn" id="emojiBtn">😊</button>
          <input id="messageInput" placeholder="Vào phòng trước" disabled />
          <button class="send-btn" id="sendBtn" disabled>➤ <span class="send-text">Gửi</span></button>
        </div>
      </section>
    </main>
  </div>

  <script>
    const weirdIcons = [
      "🪩", "🧿", "🫧", "🛸", "🪬", "🧃", "🦄", "🐉", "🦖", "🦕", "🫨", "🤯",
      "😈", "👽", "🤖", "🧌", "🧙", "🧛", "🧟", "🥷", "🫰", "🫶", "🫵", "🤌",
      "💥", "💫", "🌪️", "🔥", "⚡", "☄️", "🌈", "🌙", "⭐", "🍄", "🌵", "🌊",
      "🍕", "🍟", "🍜", "🍣", "🍩", "🍭", "🥤", "🧋", "🎮", "🎧", "🎲", "🎯",
      "🚀", "🏆", "💎", "🔮", "🧸", "🎁", "🪄", "🧨", "🦾", "👑", "🕶️", "💀"
    ];

    const stickerPacks = [
      { id: "cool", name: "Ngầu", items: ["😎🔥", "👑✨", "🕶️💥", "💎🧊", "🚀🌙", "⚡😈"] },
      { id: "cute", name: "Dễ thương", items: ["🥺👉👈", "🐣💛", "🧸💕", "🌷😊", "🐰🍓", "🫶✨"] },
      { id: "meme", name: "Meme", items: ["💀💀💀", "🤡🎪", "🗿☕", "🤯📈", "😭👌", "😼📸"] },
      { id: "vibe", name: "Vibe", items: ["🌌🪐", "🌊🫧", "🍄🌈", "☄️💫", "🪩🎧", "🌙⭐"] },
      { id: "battle", name: "Chiến", items: ["⚔️🔥", "🛡️👊", "🐉⚡", "🥷🌑", "🏆💥", "🧨😈"] },
      { id: "love", name: "Tim", items: ["💖✨", "💘🥺", "💞🫶", "❤️‍🔥😳", "💕🌷", "💝🎁"] }
    ];

    const quickTexts = [
      "Hello cả phòng 👋",
      "Ai online không? 👀",
      "Quá cháy luôn 🔥",
      "Từ từ để mình rep 😭",
      "Ok nè ✅",
      "Haha vui quá 🤣"
    ];

    const state = {
      peer: null,
      conns: {},
      isHost: false,
      joined: false,
      online: false,
      activePack: "cool",
      myName: "User-" + Math.floor(Math.random() * 900 + 100),
      roomCode: ""
    };

    const els = {
      wifiIcon: document.getElementById("wifiIcon"),
      nameInput: document.getElementById("nameInput"),
      roomInput: document.getElementById("roomInput"),
      copyBtn: document.getElementById("copyBtn"),
      joinBtn: document.getElementById("joinBtn"),
      leaveBtn: document.getElementById("leaveBtn"),
      onlinePill: document.getElementById("onlinePill"),
      statusText: document.getElementById("statusText"),
      peerIdText: document.getElementById("peerIdText"),
      hostIdText: document.getElementById("hostIdText"),
      userCount: document.getElementById("userCount"),
      userList: document.getElementById("userList"),
      roomLabel: document.getElementById("roomLabel"),
      panelBtn: document.getElementById("panelBtn"),
      panel: document.getElementById("panel"),
      onlineDot: document.getElementById("onlineDot"),
      iconsPanel: document.getElementById("iconsPanel"),
      stickersPanel: document.getElementById("stickersPanel"),
      quickPanel: document.getElementById("quickPanel"),
      packRow: document.getElementById("packRow"),
      stickerGrid: document.getElementById("stickerGrid"),
      messages: document.getElementById("messages"),
      emojiBtn: document.getElementById("emojiBtn"),
      messageInput: document.getElementById("messageInput"),
      sendBtn: document.getElementById("sendBtn")
    };

    function normalizeRoomCode(value) {
      return String(value || "").trim().toLowerCase().replace(/[^a-z0-9-_]/g, "").slice(0, 24);
    }

    function makeHostId(code) {
      return "room-" + code + "-host";
    }

    function uid() {
      if (window.crypto && crypto.randomUUID) return crypto.randomUUID();
      return "msg-" + Date.now() + "-" + Math.random().toString(16).slice(2);
    }

    function escapeHtml(value) {
      return String(value)
        .replaceAll("&", "&amp;")
        .replaceAll("<", "&lt;")
        .replaceAll(">", "&gt;")
        .replaceAll('"', "&quot;")
        .replaceAll("'", "&#039;");
    }

    function runSelfTests() {
      const tests = [
        { input: " Lớp 7A!! ", expected: "lp7a" },
        { input: "room_123-abc", expected: "room_123-abc" },
        { input: "ABC DEF", expected: "abcdef" },
        { input: "012345678901234567890123456789", expected: "012345678901234567890123" }
      ];

      tests.forEach((test) => {
        const actual = normalizeRoomCode(test.input);
        console.assert(actual === test.expected, "normalizeRoomCode failed", test, actual);
      });

      console.assert(makeHostId("abc") === "room-abc-host", "makeHostId failed");
      console.assert(stickerPacks.every((pack) => pack.items.length >= 6), "Sticker packs need at least 6 stickers");
      console.assert(document.querySelector(".app"), "App container should exist");
      console.assert(getComputedStyle(document.documentElement).overflowX !== "scroll", "Page should not force horizontal scroll");
    }

    function setStatus(text) {
      els.statusText.textContent = text;
    }

    function setOnline(value) {
      state.online = value;
      els.wifiIcon.textContent = value ? "📶" : "📴";
      els.onlinePill.textContent = value ? "Online" : "Offline";
      els.onlinePill.classList.toggle("online", value);
      els.onlineDot.classList.toggle("online", value);
    }

    function setJoined(value) {
      state.joined = value;
      els.joinBtn.classList.toggle("hidden", value);
      els.leaveBtn.classList.toggle("hidden", !value);
      els.nameInput.disabled = value;
      els.roomInput.disabled = value;
      els.messageInput.disabled = !value;
      els.sendBtn.disabled = !value || !els.messageInput.value.trim();
      els.messageInput.placeholder = value ? "Nhập tin nhắn..." : "Vào phòng trước";
      renderStickers();
      renderQuickTexts();
    }

    function setPeerId(id) {
      if (id) {
        els.peerIdText.textContent = "ID: " + id;
        els.peerIdText.classList.remove("hidden");
      } else {
        els.peerIdText.textContent = "";
        els.peerIdText.classList.add("hidden");
      }
    }

    function setHostId(id) {
      if (id) {
        els.hostIdText.textContent = "Host: " + id;
        els.hostIdText.classList.remove("hidden");
      } else {
        els.hostIdText.textContent = "";
        els.hostIdText.classList.add("hidden");
      }
    }

    function clearMessages() {
      els.messages.innerHTML = '<div class="empty" id="emptyState">Nhập mã phòng rồi bắt đầu nhắn tin.</div>';
    }

    function addMessage(msg) {
      const empty = document.getElementById("emptyState");
      if (empty) empty.remove();

      const wrap = document.createElement("div");
      wrap.className = "msg-wrap " + (msg.type === "system" ? "system" : msg.mine ? "mine" : "other");
      wrap.dataset.id = uid();

      if (msg.type === "system") {
        wrap.innerHTML = `<span class="system-text">${escapeHtml(msg.text)}</span>`;
      } else if (msg.type === "sticker") {
        wrap.innerHTML = `
          <div class="sticker-bubble">
            <div class="meta">
              <span>${escapeHtml(msg.from)}</span>
              <span class="time">${escapeHtml(msg.time || currentTime())}</span>
              <span>💖</span>
            </div>
            <div class="sticker-big">${escapeHtml(msg.text)}</div>
          </div>
        `;
      } else {
        wrap.innerHTML = `
          <div class="bubble">
            <div class="meta">
              <span>${escapeHtml(msg.from)}</span>
              <span class="time">${escapeHtml(msg.time || currentTime())}</span>
            </div>
            <div>${escapeHtml(msg.text)}</div>
          </div>
        `;
      }

      els.messages.appendChild(wrap);
      requestAnimationFrame(() => {
        els.messages.scrollTo({ top: els.messages.scrollHeight, behavior: "smooth" });
      });
    }

    function currentTime() {
      return new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    }

    function broadcast(data, exceptPeerId) {
      Object.entries(state.conns).forEach(([id, conn]) => {
        if (id !== exceptPeerId && conn.open) conn.send(data);
      });
    }

    function getUsers() {
      return [
        { id: state.peer ? state.peer.id : "", name: state.myName, host: state.isHost },
        ...Object.values(state.conns)
          .filter((conn) => conn.open && conn.metadata && conn.metadata.name)
          .map((conn) => ({ id: conn.peer, name: conn.metadata.name, host: false }))
      ];
    }

    function renderUsers(users) {
      els.userCount.textContent = users.length;
      if (!users.length) {
        els.userList.innerHTML = `<p class="muted">Chưa có ai trong phòng.</p>`;
        return;
      }

      els.userList.innerHTML = users.map((u) => `
        <div class="user-item">
          <span>${escapeHtml(u.name)}</span>
          ${u.host ? '<span class="host-tag">host</span>' : ''}
        </div>
      `).join("");
    }

    function updateUsers() {
      const users = getUsers();
      renderUsers(users);
      if (state.isHost) broadcast({ type: "users", users }, null);
    }

    function setupConnection(conn) {
      state.conns[conn.peer] = conn;

      conn.on("open", () => {
        setOnline(true);
        addMessage({ type: "system", text: (conn.metadata && conn.metadata.name ? conn.metadata.name : conn.peer) + " đã online." });
        updateUsers();

        if (!state.isHost) {
          conn.send({ type: "hello", name: state.myName });
        } else {
          conn.send({ type: "users", users: getUsers() });
        }
      });

      conn.on("data", (data) => {
        if (!data || typeof data !== "object") return;

        if (data.type === "chat" || data.type === "sticker") {
          addMessage({
            type: data.type,
            from: data.from,
            text: data.text,
            mine: false,
            time: currentTime()
          });

          if (state.isHost) broadcast(data, conn.peer);
        }

        if (data.type === "hello") {
          conn.metadata = { ...(conn.metadata || {}), name: data.name };
          updateUsers();
        }

        if (data.type === "users") {
          renderUsers(data.users || []);
        }
      });

      conn.on("close", () => {
        addMessage({ type: "system", text: (conn.metadata && conn.metadata.name ? conn.metadata.name : conn.peer) + " đã offline." });
        delete state.conns[conn.peer];
        updateUsers();
      });

      conn.on("error", () => {
        addMessage({ type: "system", text: "Kết nối gặp lỗi." });
      });
    }

    function joinRoom() {
      const code = normalizeRoomCode(els.roomInput.value);
      if (!code) {
        setStatus("Nhập mã phòng trước đã nhé.");
        return;
      }

      if (!window.Peer) {
        setStatus("Không tải được PeerJS. Hãy kiểm tra mạng hoặc CDN.");
        return;
      }

      state.roomCode = code;
      state.myName = els.nameInput.value.trim() || state.myName;
      els.nameInput.value = state.myName;
      els.roomInput.value = code;
      els.roomLabel.textContent = code;
      clearMessages();
      renderUsers([]);
      state.conns = {};

      const fixedHostId = makeHostId(code);
      setHostId(fixedHostId);
      setStatus("Đang vào phòng...");

      const peer = new Peer(fixedHostId, { debug: 1 });
      state.peer = peer;

      peer.on("open", (id) => {
        state.isHost = true;
        setPeerId(id);
        setJoined(true);
        setOnline(true);
        setStatus("Bạn đang là chủ phòng: " + code);
        addMessage({ type: "system", text: "Đã tạo phòng " + code + ". Người khác nhập cùng mã để chat." });
        updateUsers();
        els.messageInput.focus({ preventScroll: true });
      });

      peer.on("connection", setupConnection);

      peer.on("error", (err) => {
        if (err.type === "unavailable-id") {
          peer.destroy();
          state.isHost = false;

          const clientPeer = new Peer(undefined, { debug: 1 });
          state.peer = clientPeer;

          clientPeer.on("open", (id) => {
            setPeerId(id);
            setJoined(true);
            setOnline(true);
            setStatus("Đã vào phòng: " + code);
            const conn = clientPeer.connect(fixedHostId, {
              reliable: true,
              metadata: { name: state.myName }
            });
            setupConnection(conn);
            addMessage({ type: "system", text: "Đã kết nối tới phòng " + code + "." });
            els.messageInput.focus({ preventScroll: true });
          });

          clientPeer.on("error", () => {
            setOnline(false);
            setStatus("Không kết nối được phòng. Hãy thử lại hoặc tạo mã khác.");
          });
        } else {
          setOnline(false);
          setStatus("Lỗi PeerJS: " + (err.type || "không rõ"));
        }
      });
    }

    function leaveRoom() {
      Object.values(state.conns).forEach((conn) => conn.close());
      state.conns = {};
      if (state.peer) state.peer.destroy();
      state.peer = null;
      state.isHost = false;
      setJoined(false);
      setOnline(false);
      setPeerId("");
      setHostId("");
      renderUsers([]);
      els.panel.classList.remove("open");
      setStatus("Đã rời phòng");
    }

    function sendPayload(payload) {
      if (!state.joined) return;

      addMessage({ ...payload, mine: true, time: currentTime() });

      const networkPayload = {
        type: payload.type,
        from: state.myName,
        text: payload.text,
        stickerPack: payload.stickerPack
      };

      if (state.isHost) {
        broadcast(networkPayload, null);
      } else {
        const hostConn = Object.values(state.conns)[0];
        if (hostConn && hostConn.open) {
          hostConn.send(networkPayload);
        } else {
          addMessage({ type: "system", text: "Chưa có kết nối tới chủ phòng." });
        }
      }
    }

    function sendMessage() {
      const text = els.messageInput.value.trim();
      if (!text || !state.joined) return;
      sendPayload({ type: "chat", from: state.myName, text });
      els.messageInput.value = "";
      els.sendBtn.disabled = true;
    }

    function sendSticker(text, packId) {
      if (!state.joined) return;
      sendPayload({ type: "sticker", from: state.myName, text, stickerPack: packId });
    }

    function renderIcons() {
      els.iconsPanel.innerHTML = weirdIcons.map((icon) => `
        <button class="icon-btn" data-icon="${escapeHtml(icon)}">${escapeHtml(icon)}</button>
      `).join("");

      els.iconsPanel.querySelectorAll(".icon-btn").forEach((btn) => {
        btn.addEventListener("click", () => {
          els.messageInput.value += (els.messageInput.value ? " " : "") + btn.dataset.icon;
          els.messageInput.focus({ preventScroll: true });
          els.sendBtn.disabled = !state.joined || !els.messageInput.value.trim();
        });
      });
    }

    function renderPacks() {
      els.packRow.innerHTML = stickerPacks.map((pack) => `
        <button class="pack-btn ${pack.id === state.activePack ? "active" : ""}" data-pack="${pack.id}">${escapeHtml(pack.name)}</button>
      `).join("");

      els.packRow.querySelectorAll(".pack-btn").forEach((btn) => {
        btn.addEventListener("click", () => {
          state.activePack = btn.dataset.pack;
          renderPacks();
          renderStickers();
        });
      });
    }

    function renderStickers() {
      const pack = stickerPacks.find((item) => item.id === state.activePack) || stickerPacks[0];
      els.stickerGrid.innerHTML = pack.items.map((sticker) => `
        <button class="sticker-btn" data-sticker="${escapeHtml(sticker)}" ${state.joined ? "" : "disabled"}>${escapeHtml(sticker)}</button>
      `).join("");

      els.stickerGrid.querySelectorAll(".sticker-btn").forEach((btn) => {
        btn.addEventListener("click", () => sendSticker(btn.dataset.sticker, pack.id));
      });
    }

    function renderQuickTexts() {
      els.quickPanel.innerHTML = quickTexts.map((text) => `
        <button class="quick-btn" data-text="${escapeHtml(text)}" ${state.joined ? "" : "disabled"}>${escapeHtml(text)}</button>
      `).join("");

      els.quickPanel.querySelectorAll(".quick-btn").forEach((btn) => {
        btn.addEventListener("click", () => sendPayload({ type: "chat", from: state.myName, text: btn.dataset.text }));
      });
    }

    function showTab(tabName) {
      document.querySelectorAll(".tab").forEach((tab) => {
        tab.classList.remove("active", "cyan", "pink", "amber");
        if (tab.dataset.tab === tabName) {
          tab.classList.add("active");
          if (tabName === "icons") tab.classList.add("cyan");
          if (tabName === "stickers") tab.classList.add("pink");
          if (tabName === "quick") tab.classList.add("amber");
        }
      });

      els.iconsPanel.classList.toggle("hidden", tabName !== "icons");
      els.stickersPanel.classList.toggle("hidden", tabName !== "stickers");
      els.quickPanel.classList.toggle("hidden", tabName !== "quick");
    }

    function copyRoom() {
      const code = normalizeRoomCode(els.roomInput.value);
      if (!code) return;

      if (navigator.clipboard) {
        navigator.clipboard.writeText(code)
          .then(() => addMessage({ type: "system", text: "Đã copy mã phòng: " + code }))
          .catch(() => addMessage({ type: "system", text: "Mã phòng của bạn: " + code }));
      } else {
        addMessage({ type: "system", text: "Mã phòng của bạn: " + code });
      }
    }

    function boot() {
      runSelfTests();
      els.nameInput.value = state.myName;

      renderIcons();
      renderPacks();
      renderStickers();
      renderQuickTexts();

      els.roomInput.addEventListener("input", () => {
        els.roomInput.value = normalizeRoomCode(els.roomInput.value);
        els.roomLabel.textContent = els.roomInput.value || "chưa có";
      });

      els.roomInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter" && !state.joined) joinRoom();
      });

      els.messageInput.addEventListener("input", () => {
        els.sendBtn.disabled = !state.joined || !els.messageInput.value.trim();
      });

      els.messageInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter" && !event.shiftKey) {
          event.preventDefault();
          sendMessage();
        }
      });

      els.joinBtn.addEventListener("click", joinRoom);
      els.leaveBtn.addEventListener("click", leaveRoom);
      els.sendBtn.addEventListener("click", sendMessage);
      els.copyBtn.addEventListener("click", copyRoom);
      els.panelBtn.addEventListener("click", () => els.panel.classList.toggle("open"));
      els.emojiBtn.addEventListener("click", () => els.panel.classList.toggle("open"));

      document.querySelectorAll(".tab").forEach((tab) => {
        tab.addEventListener("click", () => showTab(tab.dataset.tab));
      });
    }

    boot();
  </script>
</body>
</html>
      font-size: 16px;
    }

    input:focus {
      border-color: var(--cyan);
    }

    input:disabled {
      opacity: 0.6;
    }

    .app {
      width: 100%;
      min-height: 100dvh;
      display: grid;
      grid-template-columns: 340px minmax(0, 1fr);
      gap: 14px;
      padding: 14px;
      max-width: 1200px;
      margin: 0 auto;
    }

    .card {
      min-width: 0;
      background: rgba(15, 23, 42, 0.88);
      border: 1px solid var(--border);
      border-radius: 24px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.32);
      backdrop-filter: blur(12px);
    }

    .sidebar {
      padding: 18px;
      overflow: hidden;
    }

    .brand {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 18px;
    }

    .logo {
      flex: 0 0 auto;
      width: 44px;
      height: 44px;
      border-radius: 17px;
      background: rgba(34, 211, 238, 0.14);
      display: grid;
      place-items: center;
      font-size: 23px;
    }

    h1,
    h2,
    p {
      margin: 0;
    }

    h1 {
      font-size: 21px;
      line-height: 1.15;
    }

    h2 {
      font-size: 18px;
    }

    .muted {
      color: var(--muted);
      font-size: 14px;
    }

    label {
      display: block;
      color: #cbd5e1;
      font-size: 14px;
      margin: 12px 0 7px;
    }

    .row {
      display: flex;
      gap: 8px;
      min-width: 0;
    }

    .btn {
      flex: 0 0 auto;
      border-radius: 15px;
      padding: 12px 14px;
      color: var(--text);
      background: #1e293b;
      transition: 0.18s;
      white-space: nowrap;
    }

    .btn:hover {
      background: #334155;
    }

    .btn-primary,
    .btn-danger {
      width: 100%;
      margin-top: 12px;
      border-radius: 16px;
      padding: 13px 14px;
      font-weight: 700;
    }

    .btn-primary {
      background: var(--cyan);
      color: #020617;
    }

    .btn-danger {
      background: var(--rose);
      color: white;
    }

    .status-box,
    .users-box,
    .feature-box {
      margin-top: 14px;
      border-radius: 20px;
      padding: 14px;
      background: #020617;
      border: 1px solid var(--border);
      overflow-wrap: anywhere;
    }

    .status-head,
    .user-item,
    .chat-head,
    .head-actions {
      display: flex;
      align-items: center;
    }

    .status-head,
    .user-item,
    .chat-head {
      justify-content: space-between;
      gap: 10px;
    }

    .pill {
      flex: 0 0 auto;
      font-size: 12px;
      padding: 4px 9px;
      border-radius: 999px;
      color: #cbd5e1;
      background: #334155;
    }

    .pill.online {
      background: rgba(16, 185, 129, 0.16);
      color: #6ee7b7;
    }

    .small-id {
      color: #64748b;
      font-size: 12px;
      word-break: break-all;
      margin-top: 8px;
    }

    .user-list {
      display: grid;
      gap: 8px;
      margin-top: 10px;
    }

    .user-item {
      padding: 9px 11px;
      border-radius: 14px;
      background: #0f172a;
      font-size: 14px;
      min-width: 0;
    }

    .user-item span:first-child {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .host-tag {
      color: #67e8f9;
      font-size: 11px;
      flex: 0 0 auto;
    }

    .feature-box {
      background: linear-gradient(135deg, rgba(217, 70, 239, 0.12), rgba(34, 211, 238, 0.1));
      border-color: rgba(217, 70, 239, 0.22);
    }

    .chat {
      min-width: 0;
      height: calc(100dvh - 28px);
      min-height: 620px;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    .chat-head {
      flex: 0 0 auto;
      padding: 15px 16px;
      border-bottom: 1px solid var(--border);
    }

    .chat-title {
      min-width: 0;
    }

    .chat-title p {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      max-width: 100%;
    }

    .head-actions {
      gap: 10px;
      flex: 0 0 auto;
    }

    .dot {
      width: 12px;
      height: 12px;
      border-radius: 999px;
      background: #475569;
      flex: 0 0 auto;
    }

    .dot.online {
      background: var(--green);
      box-shadow: 0 0 18px rgba(52, 211, 153, 0.75);
    }

    .panel {
      flex: 0 0 auto;
      display: none;
      padding: 13px;
      border-bottom: 1px solid var(--border);
      background: rgba(2, 6, 23, 0.75);
      max-height: 38dvh;
      overflow-y: auto;
      overflow-x: hidden;
    }

    .panel.open {
      display: block;
    }

    .tabs,
    .pack-row {
      display: flex;
      gap: 8px;
      flex-wrap: nowrap;
      overflow-x: auto;
      padding-bottom: 3px;
      scrollbar-width: none;
    }

    .tabs::-webkit-scrollbar,
    .pack-row::-webkit-scrollbar {
      display: none;
    }

    .tabs {
      margin-bottom: 13px;
    }

    .tab,
    .pack-btn {
      flex: 0 0 auto;
      border-radius: 999px;
      padding: 9px 12px;
      color: #cbd5e1;
      background: #1e293b;
      white-space: nowrap;
      font-size: 14px;
    }

    .tab.active.cyan {
      background: var(--cyan);
      color: #020617;
      font-weight: 700;
    }

    .tab.active.pink,
    .pack-btn.active {
      background: var(--pink);
      color: white;
      font-weight: 700;
    }

    .tab.active.amber {
      background: var(--amber);
      color: #020617;
      font-weight: 700;
    }

    .icon-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(42px, 1fr));
      gap: 8px;
      width: 100%;
    }

    .icon-btn {
      min-width: 0;
      height: 43px;
      border-radius: 14px;
      background: #0f172a;
      border: 1px solid var(--border);
      font-size: 23px;
      color: white;
    }

    .icon-btn:hover {
      border-color: var(--cyan);
      background: rgba(34, 211, 238, 0.14);
    }

    .sticker-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(92px, 1fr));
      gap: 9px;
      width: 100%;
    }

    .sticker-btn {
      min-width: 0;
      min-height: 82px;
      border-radius: 20px;
      color: white;
      font-size: 34px;
      border: 1px solid #334155;
      background: linear-gradient(135deg, #0f172a, #1e293b);
      box-shadow: 0 10px 24px rgba(0, 0, 0, 0.22);
    }

    .sticker-btn:hover {
      border-color: #f0abfc;
      background: linear-gradient(135deg, rgba(217, 70, 239, 0.22), rgba(34, 211, 238, 0.14));
    }

    .quick-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
      gap: 8px;
    }

    .quick-btn {
      min-width: 0;
      text-align: left;
      border-radius: 14px;
      padding: 13px;
      color: var(--text);
      background: #0f172a;
      border: 1px solid var(--border);
      overflow-wrap: anywhere;
    }

    .messages {
      flex: 1 1 auto;
      min-height: 0;
      overflow-y: auto;
      overflow-x: hidden;
      padding: 16px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      overscroll-behavior: contain;
    }

    .empty {
      flex: 1;
      min-height: 260px;
      display: grid;
      place-items: center;
      text-align: center;
      color: #64748b;
      padding: 24px;
    }

    .msg-wrap {
      display: flex;
      max-width: 100%;
      animation: pop 0.18s ease-out;
    }

    .msg-wrap.mine {
      justify-content: flex-end;
    }

    .msg-wrap.other {
      justify-content: flex-start;
    }

    .msg-wrap.system {
      justify-content: center;
    }

    @keyframes pop {
      from {
        opacity: 0;
        transform: translateY(8px) scale(0.98);
      }
      to {
        opacity: 1;
        transform: translateY(0) scale(1);
      }
    }

    .bubble,
    .sticker-bubble {
      max-width: min(78%, 520px);
      min-width: 0;
      overflow-wrap: anywhere;
      word-break: break-word;
    }

    .bubble {
      border-radius: 20px;
      padding: 11px 14px;
      box-shadow: 0 10px 26px rgba(0, 0, 0, 0.2);
      background: #1e293b;
      color: var(--text);
      white-space: pre-wrap;
    }

    .mine .bubble {
      background: var(--cyan);
      color: #020617;
    }

    .meta {
      display: flex;
      gap: 8px;
      align-items: center;
      margin-bottom: 5px;
      font-size: 12px;
      font-weight: 700;
      opacity: 0.78;
      min-width: 0;
    }

    .meta span:first-child {
      min-width: 0;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .time {
      flex: 0 0 auto;
      font-size: 11px;
      opacity: 0.65;
      font-weight: 400;
    }

    .system-text {
      display: inline-block;
      max-width: 92%;
      border-radius: 999px;
      padding: 6px 11px;
      font-size: 12px;
      color: var(--muted);
      background: #020617;
      border: 1px solid var(--border);
      overflow-wrap: anywhere;
    }

    .sticker-bubble {
      border-radius: 30px;
      padding: 14px 16px;
      border: 1px solid rgba(240, 171, 252, 0.35);
      background: linear-gradient(135deg, #1e293b, #312e81);
      box-shadow: 0 16px 36px rgba(0, 0, 0, 0.28);
    }

    .mine .sticker-bubble {
      border-color: rgba(103, 232, 249, 0.45);
      background: linear-gradient(135deg, rgba(34, 211, 238, 0.28), rgba(217, 70, 239, 0.24));
    }

    .sticker-big {
      font-size: clamp(42px, 13vw, 68px);
      line-height: 1.05;
      text-align: center;
      padding: 7px 10px;
      filter: drop-shadow(0 8px 10px rgba(0, 0, 0, 0.25));
    }

    .composer {
      flex: 0 0 auto;
      padding: 12px;
      padding-bottom: max(12px, env(safe-area-inset-bottom));
      border-top: 1px solid var(--border);
      background: rgba(2, 6, 23, 0.88);
    }

    .composer .row {
      align-items: center;
    }

    .send-btn {
      flex: 0 0 auto;
      min-width: 74px;
      border-radius: 15px;
      padding: 13px 14px;
      color: #020617;
      background: var(--cyan);
      font-weight: 700;
      white-space: nowrap;
    }

    .hidden {
      display: none !important;
    }

    .desktop-only {
      display: inline;
    }

    @media (max-width: 760px) {
      .app {
        min-height: 100dvh;
        display: flex;
        flex-direction: column;
        gap: 10px;
        padding: 8px;
      }

      .card {
        border-radius: 20px;
      }

      .sidebar {
        padding: 12px;
      }

      .brand {
        margin-bottom: 10px;
      }

      .logo {
        width: 38px;
        height: 38px;
        border-radius: 14px;
        font-size: 20px;
      }

      h1 {
        font-size: 18px;
      }

      label {
        margin-top: 9px;
      }

      .status-box,
      .users-box,
      .feature-box {
        margin-top: 10px;
        padding: 11px;
        border-radius: 17px;
      }

      .feature-box,
      .small-id {
        display: none;
      }

      .users-box {
        max-height: 110px;
        overflow-y: auto;
      }

      .chat {
        flex: 1 1 auto;
        height: auto;
        min-height: 0;
        border-radius: 20px;
      }

      .chat-head {
        padding: 12px;
      }

      .head-actions {
        gap: 7px;
      }

      .desktop-only,
      .panel-word {
        display: none;
      }

      .btn,
      .send-btn {
        padding: 12px;
      }

      .panel {
        max-height: 34dvh;
        padding: 11px;
      }

      .tab,
      .pack-btn {
        font-size: 13px;
        padding: 8px 10px;
      }

      .icon-grid {
        grid-template-columns: repeat(auto-fill, minmax(39px, 1fr));
        gap: 7px;
      }

      .icon-btn {
        height: 40px;
        font-size: 22px;
      }

      .sticker-grid {
        grid-template-columns: repeat(auto-fill, minmax(82px, 1fr));
        gap: 8px;
      }

      .sticker-btn {
        min-height: 76px;
        font-size: 31px;
      }

      .quick-grid {
        grid-template-columns: 1fr;
      }

      .messages {
        padding: 12px;
        gap: 10px;
      }

      .bubble,
      .sticker-bubble {
        max-width: 86%;
      }

      .bubble {
        padding: 10px 12px;
        border-radius: 18px;
      }

      .sticker-bubble {
        padding: 12px 13px;
        border-radius: 24px;
      }

      .composer {
        padding: 9px;
        padding-bottom: max(9px, env(safe-area-inset-bottom));
      }

      .composer .row {
        gap: 7px;
      }

      .send-btn {
        min-width: 52px;
      }

      .send-text {
        display: none;
      }
    }

    @media (max-width: 390px) {
      .app {
        padding: 6px;
      }

      .sidebar {
        padding: 10px;
      }

      .chat-head {
        padding: 10px;
      }

      .panel {
        padding: 9px;
      }

      .messages {
        padding: 10px;
      }

      .bubble,
      .sticker-bubble {
        max-width: 90%;
      }

      .icon-grid {
        grid-template-columns: repeat(auto-fill, minmax(36px, 1fr));
      }

      .icon-btn {
        height: 38px;
        font-size: 20px;
      }

      .sticker-grid {
        grid-template-columns: repeat(auto-fill, minmax(76px, 1fr));
      }
    }
  </style>
</head>
<body>
  <div class="app">
    <aside class="card sidebar">
      <div class="brand">
        <div class="logo" id="wifiIcon">📴</div>
        <div>
          <h1>PeerJS Room Chat</h1>
          <p class="muted">Chat bằng mã phòng</p>
        </div>
      </div>

      <label for="nameInput">Tên của bạn</label>
      <input id="nameInput" placeholder="Nhập tên" />

      <label for="roomInput">Mã phòng</label>
      <div class="row">
        <input id="roomInput" placeholder="vd: lop7a" />
        <button class="btn" id="copyBtn" title="Copy mã phòng">📋</button>
      </div>

      <button class="btn-primary" id="joinBtn">🚪 Vào / tạo phòng</button>
      <button class="btn-danger hidden" id="leaveBtn">🏃 Rời phòng</button>

      <div class="status-box">
        <div class="status-head">
          <span class="muted">Trạng thái</span>
          <span class="pill" id="onlinePill">Offline</span>
        </div>
        <p id="statusText">Chưa vào phòng</p>
        <p class="small-id hidden" id="peerIdText"></p>
        <p class="small-id hidden" id="hostIdText"></p>
      </div>

      <div class="users-box">
        <p>👥 Online (<span id="userCount">0</span>)</p>
        <div class="user-list" id="userList">
          <p class="muted">Chưa có ai trong phòng.</p>
        </div>
      </div>

      <div class="feature-box">
        <p><b>✨ Tính năng</b></p>
        <p class="muted" style="margin-top: 8px;">Gửi icon độc lạ, sticker lớn, tin nhắn nhanh và bong bóng đẹp.</p>
      </div>
    </aside>

    <main class="card chat">
      <div class="chat-head">
        <div class="chat-title">
          <h2>Tin nhắn</h2>
          <p class="muted">Phòng: <span id="roomLabel">chưa có</span></p>
        </div>
        <div class="head-actions">
          <button class="btn" id="panelBtn">✨ <span class="panel-word">Sticker</span></button>
          <div class="dot" id="onlineDot"></div>
        </div>
      </div>

      <section class="panel" id="panel">
        <div class="tabs">
          <button class="tab active cyan" data-tab="icons">😊 Icon</button>
          <button class="tab" data-tab="stickers">🏷️ Sticker</button>
          <button class="tab" data-tab="quick">🪄 Nhanh</button>
        </div>

        <div id="iconsPanel" class="icon-grid"></div>
        <div id="stickersPanel" class="hidden">
          <div class="pack-row" id="packRow"></div>
          <div class="sticker-grid" id="stickerGrid"></div>
        </div>
        <div id="quickPanel" class="quick-grid hidden"></div>
      </section>

      <section class="messages" id="messages">
        <div class="empty" id="emptyState">Nhập mã phòng rồi bắt đầu nhắn tin.</div>
      </section>

      <section class="composer">
        <div class="row">
          <button class="btn" id="emojiBtn">😊</button>
          <input id="messageInput" placeholder="Vào phòng trước" disabled />
          <button class="send-btn" id="sendBtn" disabled>➤ <span class="send-text">Gửi</span></button>
        </div>
      </section>
    </main>
  </div>

  <script>
    const weirdIcons = [
      "🪩", "🧿", "🫧", "🛸", "🪬", "🧃", "🦄", "🐉", "🦖", "🦕", "🫨", "🤯",
      "😈", "👽", "🤖", "🧌", "🧙", "🧛", "🧟", "🥷", "🫰", "🫶", "🫵", "🤌",
      "💥", "💫", "🌪️", "🔥", "⚡", "☄️", "🌈", "🌙", "⭐", "🍄", "🌵", "🌊",
      "🍕", "🍟", "🍜", "🍣", "🍩", "🍭", "🥤", "🧋", "🎮", "🎧", "🎲", "🎯",
      "🚀", "🏆", "💎", "🔮", "🧸", "🎁", "🪄", "🧨", "🦾", "👑", "🕶️", "💀"
    ];

    const stickerPacks = [
      { id: "cool", name: "Ngầu", items: ["😎🔥", "👑✨", "🕶️💥", "💎🧊", "🚀🌙", "⚡😈"] },
      { id: "cute", name: "Dễ thương", items: ["🥺👉👈", "🐣💛", "🧸💕", "🌷😊", "🐰🍓", "🫶✨"] },
      { id: "meme", name: "Meme", items: ["💀💀💀", "🤡🎪", "🗿☕", "🤯📈", "😭👌", "😼📸"] },
      { id: "vibe", name: "Vibe", items: ["🌌🪐", "🌊🫧", "🍄🌈", "☄️💫", "🪩🎧", "🌙⭐"] },
      { id: "battle", name: "Chiến", items: ["⚔️🔥", "🛡️👊", "🐉⚡", "🥷🌑", "🏆💥", "🧨😈"] },
      { id: "love", name: "Tim", items: ["💖✨", "💘🥺", "💞🫶", "❤️‍🔥😳", "💕🌷", "💝🎁"] }
    ];

    const quickTexts = [
      "Hello cả phòng 👋",
      "Ai online không? 👀",
      "Quá cháy luôn 🔥",
      "Từ từ để mình rep 😭",
      "Ok nè ✅",
      "Haha vui quá 🤣"
    ];

    const state = {
      peer: null,
      conns: {},
      isHost: false,
      joined: false,
      online: false,
      activePack: "cool",
      myName: "User-" + Math.floor(Math.random() * 900 + 100),
      roomCode: ""
    };

    const els = {
      wifiIcon: document.getElementById("wifiIcon"),
      nameInput: document.getElementById("nameInput"),
      roomInput: document.getElementById("roomInput"),
      copyBtn: document.getElementById("copyBtn"),
      joinBtn: document.getElementById("joinBtn"),
      leaveBtn: document.getElementById("leaveBtn"),
      onlinePill: document.getElementById("onlinePill"),
      statusText: document.getElementById("statusText"),
      peerIdText: document.getElementById("peerIdText"),
      hostIdText: document.getElementById("hostIdText"),
      userCount: document.getElementById("userCount"),
      userList: document.getElementById("userList"),
      roomLabel: document.getElementById("roomLabel"),
      panelBtn: document.getElementById("panelBtn"),
      panel: document.getElementById("panel"),
      onlineDot: document.getElementById("onlineDot"),
      iconsPanel: document.getElementById("iconsPanel"),
      stickersPanel: document.getElementById("stickersPanel"),
      quickPanel: document.getElementById("quickPanel"),
      packRow: document.getElementById("packRow"),
      stickerGrid: document.getElementById("stickerGrid"),
      messages: document.getElementById("messages"),
      emojiBtn: document.getElementById("emojiBtn"),
      messageInput: document.getElementById("messageInput"),
      sendBtn: document.getElementById("sendBtn")
    };

    function normalizeRoomCode(value) {
      return String(value || "").trim().toLowerCase().replace(/[^a-z0-9-_]/g, "").slice(0, 24);
    }

    function makeHostId(code) {
      return "room-" + code + "-host";
    }

    function uid() {
      if (window.crypto && crypto.randomUUID) return crypto.randomUUID();
      return "msg-" + Date.now() + "-" + Math.random().toString(16).slice(2);
    }

    function escapeHtml(value) {
      return String(value)
        .replaceAll("&", "&amp;")
        .replaceAll("<", "&lt;")
        .replaceAll(">", "&gt;")
        .replaceAll('"', "&quot;")
        .replaceAll("'", "&#039;");
    }

    function runSelfTests() {
      const tests = [
        { input: " Lớp 7A!! ", expected: "lp7a" },
        { input: "room_123-abc", expected: "room_123-abc" },
        { input: "ABC DEF", expected: "abcdef" },
        { input: "012345678901234567890123456789", expected: "012345678901234567890123" }
      ];

      tests.forEach((test) => {
        const actual = normalizeRoomCode(test.input);
        console.assert(actual === test.expected, "normalizeRoomCode failed", test, actual);
      });

      console.assert(makeHostId("abc") === "room-abc-host", "makeHostId failed");
      console.assert(stickerPacks.every((pack) => pack.items.length >= 6), "Sticker packs need at least 6 stickers");
      console.assert(document.querySelector(".app"), "App container should exist");
      console.assert(getComputedStyle(document.documentElement).overflowX !== "scroll", "Page should not force horizontal scroll");
    }

    function setStatus(text) {
      els.statusText.textContent = text;
    }

    function setOnline(value) {
      state.online = value;
      els.wifiIcon.textContent = value ? "📶" : "📴";
      els.onlinePill.textContent = value ? "Online" : "Offline";
      els.onlinePill.classList.toggle("online", value);
      els.onlineDot.classList.toggle("online", value);
    }

    function setJoined(value) {
      state.joined = value;
      els.joinBtn.classList.toggle("hidden", value);
      els.leaveBtn.classList.toggle("hidden", !value);
      els.nameInput.disabled = value;
      els.roomInput.disabled = value;
      els.messageInput.disabled = !value;
      els.sendBtn.disabled = !value || !els.messageInput.value.trim();
      els.messageInput.placeholder = value ? "Nhập tin nhắn..." : "Vào phòng trước";
      renderStickers();
      renderQuickTexts();
    }

    function setPeerId(id) {
      if (id) {
        els.peerIdText.textContent = "ID: " + id;
        els.peerIdText.classList.remove("hidden");
      } else {
        els.peerIdText.textContent = "";
        els.peerIdText.classList.add("hidden");
      }
    }

    function setHostId(id) {
      if (id) {
        els.hostIdText.textContent = "Host: " + id;
        els.hostIdText.classList.remove("hidden");
      } else {
        els.hostIdText.textContent = "";
        els.hostIdText.classList.add("hidden");
      }
    }

    function clearMessages() {
      els.messages.innerHTML = '<div class="empty" id="emptyState">Nhập mã phòng rồi bắt đầu nhắn tin.</div>';
    }

    function addMessage(msg) {
      const empty = document.getElementById("emptyState");
      if (empty) empty.remove();

      const wrap = document.createElement("div");
      wrap.className = "msg-wrap " + (msg.type === "system" ? "system" : msg.mine ? "mine" : "other");
      wrap.dataset.id = uid();

      if (msg.type === "system") {
        wrap.innerHTML = `<span class="system-text">${escapeHtml(msg.text)}</span>`;
      } else if (msg.type === "sticker") {
        wrap.innerHTML = `
          <div class="sticker-bubble">
            <div class="meta">
              <span>${escapeHtml(msg.from)}</span>
              <span class="time">${escapeHtml(msg.time || currentTime())}</span>
              <span>💖</span>
            </div>
            <div class="sticker-big">${escapeHtml(msg.text)}</div>
          </div>
        `;
      } else {
        wrap.innerHTML = `
          <div class="bubble">
            <div class="meta">
              <span>${escapeHtml(msg.from)}</span>
              <span class="time">${escapeHtml(msg.time || currentTime())}</span>
            </div>
            <div>${escapeHtml(msg.text)}</div>
          </div>
        `;
      }

      els.messages.appendChild(wrap);
      requestAnimationFrame(() => {
        els.messages.scrollTo({ top: els.messages.scrollHeight, behavior: "smooth" });
      });
    }

    function currentTime() {
      return new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    }

    function broadcast(data, exceptPeerId) {
      Object.entries(state.conns).forEach(([id, conn]) => {
        if (id !== exceptPeerId && conn.open) conn.send(data);
      });
    }

    function getUsers() {
      return [
        { id: state.peer ? state.peer.id : "", name: state.myName, host: state.isHost },
        ...Object.values(state.conns)
          .filter((conn) => conn.open && conn.metadata && conn.metadata.name)
          .map((conn) => ({ id: conn.peer, name: conn.metadata.name, host: false }))
      ];
    }

    function renderUsers(users) {
      els.userCount.textContent = users.length;
      if (!users.length) {
        els.userList.innerHTML = `<p class="muted">Chưa có ai trong phòng.</p>`;
        return;
      }

      els.userList.innerHTML = users.map((u) => `
        <div class="user-item">
          <span>${escapeHtml(u.name)}</span>
          ${u.host ? '<span class="host-tag">host</span>' : ''}
        </div>
      `).join("");
    }

    function updateUsers() {
      const users = getUsers();
      renderUsers(users);
      if (state.isHost) broadcast({ type: "users", users }, null);
    }

    function setupConnection(conn) {
      state.conns[conn.peer] = conn;

      conn.on("open", () => {
        setOnline(true);
        addMessage({ type: "system", text: (conn.metadata && conn.metadata.name ? conn.metadata.name : conn.peer) + " đã online." });
        updateUsers();

        if (!state.isHost) {
          conn.send({ type: "hello", name: state.myName });
        } else {
          conn.send({ type: "users", users: getUsers() });
        }
      });

      conn.on("data", (data) => {
        if (!data || typeof data !== "object") return;

        if (data.type === "chat" || data.type === "sticker") {
          addMessage({
            type: data.type,
            from: data.from,
            text: data.text,
            mine: false,
            time: currentTime()
          });

          if (state.isHost) broadcast(data, conn.peer);
        }

        if (data.type === "hello") {
          conn.metadata = { ...(conn.metadata || {}), name: data.name };
          updateUsers();
        }

        if (data.type === "users") {
          renderUsers(data.users || []);
        }
      });

      conn.on("close", () => {
        addMessage({ type: "system", text: (conn.metadata && conn.metadata.name ? conn.metadata.name : conn.peer) + " đã offline." });
        delete state.conns[conn.peer];
        updateUsers();
      });

      conn.on("error", () => {
        addMessage({ type: "system", text: "Kết nối gặp lỗi." });
      });
    }

    function joinRoom() {
      const code = normalizeRoomCode(els.roomInput.value);
      if (!code) {
        setStatus("Nhập mã phòng trước đã nhé.");
        return;
      }

      if (!window.Peer) {
        setStatus("Không tải được PeerJS. Hãy kiểm tra mạng hoặc CDN.");
        return;
      }

      state.roomCode = code;
      state.myName = els.nameInput.value.trim() || state.myName;
      els.nameInput.value = state.myName;
      els.roomInput.value = code;
      els.roomLabel.textContent = code;
      clearMessages();
      renderUsers([]);
      state.conns = {};

      const fixedHostId = makeHostId(code);
      setHostId(fixedHostId);
      setStatus("Đang vào phòng...");

      const peer = new Peer(fixedHostId, { debug: 1 });
      state.peer = peer;

      peer.on("open", (id) => {
        state.isHost = true;
        setPeerId(id);
        setJoined(true);
        setOnline(true);
        setStatus("Bạn đang là chủ phòng: " + code);
        addMessage({ type: "system", text: "Đã tạo phòng " + code + ". Người khác nhập cùng mã để chat." });
        updateUsers();
        els.messageInput.focus({ preventScroll: true });
      });

      peer.on("connection", setupConnection);

      peer.on("error", (err) => {
        if (err.type === "unavailable-id") {
          peer.destroy();
          state.isHost = false;

          const clientPeer = new Peer(undefined, { debug: 1 });
          state.peer = clientPeer;

          clientPeer.on("open", (id) => {
            setPeerId(id);
            setJoined(true);
            setOnline(true);
            setStatus("Đã vào phòng: " + code);
            const conn = clientPeer.connect(fixedHostId, {
              reliable: true,
              metadata: { name: state.myName }
            });
            setupConnection(conn);
            addMessage({ type: "system", text: "Đã kết nối tới phòng " + code + "." });
            els.messageInput.focus({ preventScroll: true });
          });

          clientPeer.on("error", () => {
            setOnline(false);
            setStatus("Không kết nối được phòng. Hãy thử lại hoặc tạo mã khác.");
          });
        } else {
          setOnline(false);
          setStatus("Lỗi PeerJS: " + (err.type || "không rõ"));
        }
      });
    }

    function leaveRoom() {
      Object.values(state.conns).forEach((conn) => conn.close());
      state.conns = {};
      if (state.peer) state.peer.destroy();
      state.peer = null;
      state.isHost = false;
      setJoined(false);
      setOnline(false);
      setPeerId("");
      setHostId("");
      renderUsers([]);
      els.panel.classList.remove("open");
      setStatus("Đã rời phòng");
    }

    function sendPayload(payload) {
      if (!state.joined) return;

      addMessage({ ...payload, mine: true, time: currentTime() });

      const networkPayload = {
        type: payload.type,
        from: state.myName,
        text: payload.text,
        stickerPack: payload.stickerPack
      };

      if (state.isHost) {
        broadcast(networkPayload, null);
      } else {
        const hostConn = Object.values(state.conns)[0];
        if (hostConn && hostConn.open) {
          hostConn.send(networkPayload);
        } else {
          addMessage({ type: "system", text: "Chưa có kết nối tới chủ phòng." });
        }
      }
    }

    function sendMessage() {
      const text = els.messageInput.value.trim();
      if (!text || !state.joined) return;
      sendPayload({ type: "chat", from: state.myName, text });
      els.messageInput.value = "";
      els.sendBtn.disabled = true;
    }

    function sendSticker(text, packId) {
      if (!state.joined) return;
      sendPayload({ type: "sticker", from: state.myName, text, stickerPack: packId });
    }

    function renderIcons() {
      els.iconsPanel.innerHTML = weirdIcons.map((icon) => `
        <button class="icon-btn" data-icon="${escapeHtml(icon)}">${escapeHtml(icon)}</button>
      `).join("");

      els.iconsPanel.querySelectorAll(".icon-btn").forEach((btn) => {
        btn.addEventListener("click", () => {
          els.messageInput.value += (els.messageInput.value ? " " : "") + btn.dataset.icon;
          els.messageInput.focus({ preventScroll: true });
          els.sendBtn.disabled = !state.joined || !els.messageInput.value.trim();
        });
      });
    }

    function renderPacks() {
      els.packRow.innerHTML = stickerPacks.map((pack) => `
        <button class="pack-btn ${pack.id === state.activePack ? "active" : ""}" data-pack="${pack.id}">${escapeHtml(pack.name)}</button>
      `).join("");

      els.packRow.querySelectorAll(".pack-btn").forEach((btn) => {
        btn.addEventListener("click", () => {
          state.activePack = btn.dataset.pack;
          renderPacks();
          renderStickers();
        });
      });
    }

    function renderStickers() {
      const pack = stickerPacks.find((item) => item.id === state.activePack) || stickerPacks[0];
      els.stickerGrid.innerHTML = pack.items.map((sticker) => `
        <button class="sticker-btn" data-sticker="${escapeHtml(sticker)}" ${state.joined ? "" : "disabled"}>${escapeHtml(sticker)}</button>
      `).join("");

      els.stickerGrid.querySelectorAll(".sticker-btn").forEach((btn) => {
        btn.addEventListener("click", () => sendSticker(btn.dataset.sticker, pack.id));
      });
    }

    function renderQuickTexts() {
      els.quickPanel.innerHTML = quickTexts.map((text) => `
        <button class="quick-btn" data-text="${escapeHtml(text)}" ${state.joined ? "" : "disabled"}>${escapeHtml(text)}</button>
      `).join("");

      els.quickPanel.querySelectorAll(".quick-btn").forEach((btn) => {
        btn.addEventListener("click", () => sendPayload({ type: "chat", from: state.myName, text: btn.dataset.text }));
      });
    }

    function showTab(tabName) {
      document.querySelectorAll(".tab").forEach((tab) => {
        tab.classList.remove("active", "cyan", "pink", "amber");
        if (tab.dataset.tab === tabName) {
          tab.classList.add("active");
          if (tabName === "icons") tab.classList.add("cyan");
          if (tabName === "stickers") tab.classList.add("pink");
          if (tabName === "quick") tab.classList.add("amber");
        }
      });

      els.iconsPanel.classList.toggle("hidden", tabName !== "icons");
      els.stickersPanel.classList.toggle("hidden", tabName !== "stickers");
      els.quickPanel.classList.toggle("hidden", tabName !== "quick");
    }

    function copyRoom() {
      const code = normalizeRoomCode(els.roomInput.value);
      if (!code) return;

      if (navigator.clipboard) {
        navigator.clipboard.writeText(code)
          .then(() => addMessage({ type: "system", text: "Đã copy mã phòng: " + code }))
          .catch(() => addMessage({ type: "system", text: "Mã phòng của bạn: " + code }));
      } else {
        addMessage({ type: "system", text: "Mã phòng của bạn: " + code });
      }
    }

    function boot() {
      runSelfTests();
      els.nameInput.value = state.myName;

      renderIcons();
      renderPacks();
      renderStickers();
      renderQuickTexts();

      els.roomInput.addEventListener("input", () => {
        els.roomInput.value = normalizeRoomCode(els.roomInput.value);
        els.roomLabel.textContent = els.roomInput.value || "chưa có";
      });

      els.roomInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter" && !state.joined) joinRoom();
      });

      els.messageInput.addEventListener("input", () => {
        els.sendBtn.disabled = !state.joined || !els.messageInput.value.trim();
      });

      els.messageInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter" && !event.shiftKey) {
          event.preventDefault();
          sendMessage();
        }
      });

      els.joinBtn.addEventListener("click", joinRoom);
      els.leaveBtn.addEventListener("click", leaveRoom);
      els.sendBtn.addEventListener("click", sendMessage);
      els.copyBtn.addEventListener("click", copyRoom);
      els.panelBtn.addEventListener("click", () => els.panel.classList.toggle("open"));
      els.emojiBtn.addEventListener("click", () => els.panel.classList.toggle("open"));

      document.querySelectorAll(".tab").forEach((tab) => {
        tab.addEventListener("click", () => showTab(tab.dataset.tab));
      });
    }

    boot();
  </script>
</body>
</html>
    }

    button:disabled {
      cursor: not-allowed;
      opacity: 0.45;
    }

    .app {
      width: 100%;
      max-width: 1180px;
      display: grid;
      grid-template-columns: 350px 1fr;
      gap: 16px;
    }

    .card {
      background: rgba(15, 23, 42, 0.88);
      border: 1px solid rgba(148, 163, 184, 0.18);
      border-radius: 24px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.35);
      backdrop-filter: blur(12px);
    }

    .sidebar {
      padding: 20px;
    }

    .brand {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 24px;
    }

    .logo {
      width: 46px;
      height: 46px;
      border-radius: 18px;
      background: rgba(34, 211, 238, 0.14);
      display: grid;
      place-items: center;
      font-size: 24px;
    }

    h1,
    h2,
    p {
      margin: 0;
    }

    h1 {
      font-size: 22px;
    }

    .muted {
      color: #94a3b8;
      font-size: 14px;
    }

    label {
      display: block;
      color: #cbd5e1;
      font-size: 14px;
      margin-bottom: 8px;
    }

    input {
      width: 100%;
      border-radius: 14px;
      background: #020617;
      color: #f8fafc;
      border: 1px solid #334155;
      outline: none;
      padding: 13px 14px;
      margin-bottom: 14px;
    }

    input:focus {
      border-color: #22d3ee;
    }

    input:disabled {
      opacity: 0.6;
    }

    .row {
      display: flex;
      gap: 8px;
    }

    .btn {
      border-radius: 14px;
      padding: 12px 14px;
      color: #f8fafc;
      background: #1e293b;
      transition: 0.18s;
    }

    .btn:hover {
      background: #334155;
    }

    .btn-primary {
      width: 100%;
      margin-top: 2px;
      background: #22d3ee;
      color: #020617;
      font-weight: 700;
    }

    .btn-primary:hover {
      background: #67e8f9;
    }

    .btn-danger {
      width: 100%;
      margin-top: 2px;
      background: #f43f5e;
      color: white;
      font-weight: 700;
    }

    .btn-danger:hover {
      background: #fb7185;
    }

    .status-box,
    .users-box,
    .feature-box {
      margin-top: 16px;
      border-radius: 20px;
      padding: 16px;
      background: #020617;
      border: 1px solid rgba(148, 163, 184, 0.16);
    }

    .status-head {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
    }

    .pill {
      font-size: 12px;
      padding: 4px 9px;
      border-radius: 999px;
      color: #cbd5e1;
      background: #334155;
    }

    .pill.online {
      background: rgba(16, 185, 129, 0.16);
      color: #6ee7b7;
    }

    .small-id {
      color: #64748b;
      font-size: 12px;
      word-break: break-all;
      margin-top: 8px;
    }

    .user-list {
      display: grid;
      gap: 8px;
      margin-top: 10px;
    }

    .user-item {
      display: flex;
      justify-content: space-between;
      gap: 10px;
      padding: 9px 11px;
      border-radius: 14px;
      background: #0f172a;
      font-size: 14px;
    }

    .host-tag {
      color: #67e8f9;
      font-size: 11px;
    }

    .feature-box {
      background: linear-gradient(135deg, rgba(217, 70, 239, 0.12), rgba(34, 211, 238, 0.1));
      border-color: rgba(217, 70, 239, 0.22);
    }

    .chat {
      min-height: calc(100vh - 36px);
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    .chat-head {
      padding: 16px 20px;
      border-bottom: 1px solid rgba(148, 163, 184, 0.16);
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
    }

    .head-actions {
      display: flex;
      align-items: center;
      gap: 12px;
    }

    .dot {
      width: 12px;
      height: 12px;
      border-radius: 999px;
      background: #475569;
    }

    .dot.online {
      background: #34d399;
      box-shadow: 0 0 18px rgba(52, 211, 153, 0.75);
    }

    .panel {
      display: none;
      padding: 16px;
      border-bottom: 1px solid rgba(148, 163, 184, 0.16);
      background: rgba(2, 6, 23, 0.72);
    }

    .panel.open {
      display: block;
    }

    .tabs {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
      margin-bottom: 14px;
    }

    .tab {
      border-radius: 14px;
      padding: 10px 13px;
      color: #cbd5e1;
      background: #1e293b;
    }

    .tab.active.cyan {
      background: #22d3ee;
      color: #020617;
      font-weight: 700;
    }

    .tab.active.pink {
      background: #d946ef;
      color: white;
      font-weight: 700;
    }

    .tab.active.amber {
      background: #fbbf24;
      color: #020617;
      font-weight: 700;
    }

    .icon-grid {
      display: grid;
      grid-template-columns: repeat(16, 1fr);
      gap: 8px;
    }

    .icon-btn {
      height: 44px;
      border-radius: 14px;
      background: #0f172a;
      border: 1px solid rgba(148, 163, 184, 0.16);
      font-size: 24px;
      color: white;
    }

    .icon-btn:hover {
      border-color: #22d3ee;
      background: rgba(34, 211, 238, 0.14);
    }

    .pack-row {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin-bottom: 14px;
    }

    .pack-btn {
      border-radius: 999px;
      padding: 8px 12px;
      color: #cbd5e1;
      background: #1e293b;
    }

    .pack-btn.active {
      color: white;
      background: #d946ef;
    }

    .sticker-grid {
      display: grid;
      grid-template-columns: repeat(6, 1fr);
      gap: 10px;
    }

    .sticker-btn {
      min-height: 92px;
      border-radius: 20px;
      color: white;
      font-size: 40px;
      border: 1px solid #334155;
      background: linear-gradient(135deg, #0f172a, #1e293b);
      box-shadow: 0 10px 24px rgba(0, 0, 0, 0.22);
    }

    .sticker-btn:hover {
      border-color: #f0abfc;
      background: linear-gradient(135deg, rgba(217, 70, 239, 0.22), rgba(34, 211, 238, 0.14));
    }

    .quick-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 8px;
    }

    .quick-btn {
      text-align: left;
      border-radius: 14px;
      padding: 13px;
      color: #f8fafc;
      background: #0f172a;
      border: 1px solid rgba(148, 163, 184, 0.16);
    }

    .quick-btn:hover {
      border-color: #fbbf24;
      background: rgba(251, 191, 36, 0.12);
    }

    .messages {
      flex: 1;
      overflow-y: auto;
      padding: 20px;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .empty {
      min-height: 360px;
      display: grid;
      place-items: center;
      text-align: center;
      color: #64748b;
    }

    .msg-wrap {
      display: flex;
      animation: pop 0.18s ease-out;
    }

    .msg-wrap.mine {
      justify-content: flex-end;
    }

    .msg-wrap.other {
      justify-content: flex-start;
    }

    .msg-wrap.system {
      justify-content: center;
    }

    @keyframes pop {
      from {
        opacity: 0;
        transform: translateY(8px) scale(0.98);
      }
      to {
        opacity: 1;
        transform: translateY(0) scale(1);
      }
    }

    .bubble {
      max-width: 78%;
      border-radius: 20px;
      padding: 11px 14px;
      box-shadow: 0 10px 26px rgba(0, 0, 0, 0.2);
      background: #1e293b;
      color: #f8fafc;
      word-break: break-word;
      white-space: pre-wrap;
    }

    .mine .bubble {
      background: #22d3ee;
      color: #020617;
    }

    .meta {
      display: flex;
      gap: 8px;
      align-items: center;
      margin-bottom: 5px;
      font-size: 12px;
      font-weight: 700;
      opacity: 0.78;
    }

    .time {
      font-size: 11px;
      opacity: 0.65;
      font-weight: 400;
    }

    .system-text {
      display: inline-block;
      border-radius: 999px;
      padding: 6px 11px;
      font-size: 12px;
      color: #94a3b8;
      background: #020617;
      border: 1px solid rgba(148, 163, 184, 0.16);
    }

    .sticker-bubble {
      max-width: 78%;
      border-radius: 30px;
      padding: 16px 20px;
      border: 1px solid rgba(240, 171, 252, 0.35);
      background: linear-gradient(135deg, #1e293b, #312e81);
      box-shadow: 0 16px 36px rgba(0, 0, 0, 0.28);
    }

    .mine .sticker-bubble {
      border-color: rgba(103, 232, 249, 0.45);
      background: linear-gradient(135deg, rgba(34, 211, 238, 0.28), rgba(217, 70, 239, 0.24));
    }

    .sticker-big {
      font-size: 68px;
      line-height: 1.05;
      text-align: center;
      padding: 8px 16px;
      filter: drop-shadow(0 8px 10px rgba(0, 0, 0, 0.25));
    }

    .composer {
      padding: 16px;
      border-top: 1px solid rgba(148, 163, 184, 0.16);
      background: rgba(2, 6, 23, 0.65);
    }

    .composer input {
      margin-bottom: 0;
    }

    .send-btn {
      min-width: 88px;
      border-radius: 14px;
      padding: 12px 18px;
      color: #020617;
      background: #22d3ee;
      font-weight: 700;
    }

    .send-btn:hover {
      background: #67e8f9;
    }

    .hidden {
      display: none !important;
    }

    @media (max-width: 900px) {
      body {
        padding: 10px;
      }

      .app {
        grid-template-columns: 1fr;
      }

      .chat {
        min-height: 680px;
      }

      .icon-grid {
        grid-template-columns: repeat(8, 1fr);
      }

      .sticker-grid,
      .quick-grid {
        grid-template-columns: repeat(2, 1fr);
      }
    }
  </style>
</head>
<body>
  <div class="app">
    <aside class="card sidebar">
      <div class="brand">
        <div class="logo" id="wifiIcon">📴</div>
        <div>
          <h1>PeerJS Room Chat</h1>
          <p class="muted">Chat P2P bằng mã phòng</p>
        </div>
      </div>

      <label for="nameInput">Tên của bạn</label>
      <input id="nameInput" placeholder="Nhập tên" />

      <label for="roomInput">Mã phòng</label>
      <div class="row">
        <input id="roomInput" placeholder="vd: lop7a" />
        <button class="btn" id="copyBtn" title="Copy mã phòng">📋</button>
      </div>

      <button class="btn-primary" id="joinBtn">🚪 Vào / tạo phòng</button>
      <button class="btn-danger hidden" id="leaveBtn">🏃 Rời phòng</button>

      <div class="status-box">
        <div class="status-head">
          <span class="muted">Trạng thái</span>
          <span class="pill" id="onlinePill">Offline</span>
        </div>
        <p id="statusText">Chưa vào phòng</p>
        <p class="small-id hidden" id="peerIdText"></p>
        <p class="small-id hidden" id="hostIdText"></p>
      </div>

      <div class="users-box">
        <p>👥 Người online (<span id="userCount">0</span>)</p>
        <div class="user-list" id="userList">
          <p class="muted">Chưa có ai trong phòng.</p>
        </div>
      </div>

      <div class="feature-box">
        <p><b>✨ Tính năng mới</b></p>
        <p class="muted" style="margin-top: 8px;">Gửi icon độc lạ, sticker cỡ lớn, tin nhắn nhanh và hiệu ứng bong bóng đẹp hơn.</p>
      </div>
    </aside>

    <main class="card chat">
      <div class="chat-head">
        <div>
          <h2>Tin nhắn</h2>
          <p class="muted">Mã phòng: <span id="roomLabel">chưa có</span></p>
        </div>
        <div class="head-actions">
          <button class="btn" id="panelBtn">✨ Icon & Sticker</button>
          <div class="dot" id="onlineDot"></div>
        </div>
      </div>

      <section class="panel" id="panel">
        <div class="tabs">
          <button class="tab active cyan" data-tab="icons">😊 Icon độc lạ</button>
          <button class="tab" data-tab="stickers">🏷️ Sticker đẹp</button>
          <button class="tab" data-tab="quick">🪄 Gửi nhanh</button>
        </div>

        <div id="iconsPanel" class="icon-grid"></div>
        <div id="stickersPanel" class="hidden">
          <div class="pack-row" id="packRow"></div>
          <div class="sticker-grid" id="stickerGrid"></div>
        </div>
        <div id="quickPanel" class="quick-grid hidden"></div>
      </section>

      <section class="messages" id="messages">
        <div class="empty" id="emptyState">Nhập mã phòng rồi bắt đầu nhắn tin. Bật bảng icon để gửi sticker siêu đẹp.</div>
      </section>

      <section class="composer">
        <div class="row">
          <button class="btn" id="emojiBtn">😊</button>
          <input id="messageInput" placeholder="Vào phòng trước để nhắn tin" disabled />
          <button class="send-btn" id="sendBtn" disabled>➤ Gửi</button>
        </div>
      </section>
    </main>
  </div>

  <script>
    const weirdIcons = [
      "🪩", "🧿", "🫧", "🛸", "🪬", "🧃", "🦄", "🐉", "🦖", "🦕", "🫨", "🤯",
      "😈", "👽", "🤖", "🧌", "🧙", "🧛", "🧟", "🥷", "🫰", "🫶", "🫵", "🤌",
      "💥", "💫", "🌪️", "🔥", "⚡", "☄️", "🌈", "🌙", "⭐", "🍄", "🌵", "🌊",
      "🍕", "🍟", "🍜", "🍣", "🍩", "🍭", "🥤", "🧋", "🎮", "🎧", "🎲", "🎯",
      "🚀", "🏆", "💎", "🔮", "🧸", "🎁", "🪄", "🧨", "🦾", "👑", "🕶️", "💀"
    ];

    const stickerPacks = [
      { id: "cool", name: "Ngầu", items: ["😎🔥", "👑✨", "🕶️💥", "💎🧊", "🚀🌙", "⚡😈"] },
      { id: "cute", name: "Dễ thương", items: ["🥺👉👈", "🐣💛", "🧸💕", "🌷😊", "🐰🍓", "🫶✨"] },
      { id: "meme", name: "Meme", items: ["💀💀💀", "🤡🎪", "🗿☕", "🤯📈", "😭👌", "😼📸"] },
      { id: "vibe", name: "Vibe", items: ["🌌🪐", "🌊🫧", "🍄🌈", "☄️💫", "🪩🎧", "🌙⭐"] },
      { id: "battle", name: "Chiến", items: ["⚔️🔥", "🛡️👊", "🐉⚡", "🥷🌑", "🏆💥", "🧨😈"] },
      { id: "love", name: "Tim", items: ["💖✨", "💘🥺", "💞🫶", "❤️‍🔥😳", "💕🌷", "💝🎁"] }
    ];

    const quickTexts = [
      "Hello cả phòng 👋",
      "Ai online không? 👀",
      "Quá cháy luôn 🔥",
      "Từ từ để mình rep 😭",
      "Ok nè ✅",
      "Haha vui quá 🤣"
    ];

    const state = {
      peer: null,
      conns: {},
      isHost: false,
      joined: false,
      online: false,
      activePack: "cool",
      myName: "User-" + Math.floor(Math.random() * 900 + 100),
      roomCode: ""
    };

    const els = {
      wifiIcon: document.getElementById("wifiIcon"),
      nameInput: document.getElementById("nameInput"),
      roomInput: document.getElementById("roomInput"),
      copyBtn: document.getElementById("copyBtn"),
      joinBtn: document.getElementById("joinBtn"),
      leaveBtn: document.getElementById("leaveBtn"),
      onlinePill: document.getElementById("onlinePill"),
      statusText: document.getElementById("statusText"),
      peerIdText: document.getElementById("peerIdText"),
      hostIdText: document.getElementById("hostIdText"),
      userCount: document.getElementById("userCount"),
      userList: document.getElementById("userList"),
      roomLabel: document.getElementById("roomLabel"),
      panelBtn: document.getElementById("panelBtn"),
      panel: document.getElementById("panel"),
      onlineDot: document.getElementById("onlineDot"),
      iconsPanel: document.getElementById("iconsPanel"),
      stickersPanel: document.getElementById("stickersPanel"),
      quickPanel: document.getElementById("quickPanel"),
      packRow: document.getElementById("packRow"),
      stickerGrid: document.getElementById("stickerGrid"),
      messages: document.getElementById("messages"),
      emptyState: document.getElementById("emptyState"),
      emojiBtn: document.getElementById("emojiBtn"),
      messageInput: document.getElementById("messageInput"),
      sendBtn: document.getElementById("sendBtn")
    };

    function normalizeRoomCode(value) {
      return String(value || "").trim().toLowerCase().replace(/[^a-z0-9-_]/g, "").slice(0, 24);
    }

    function makeHostId(code) {
      return "room-" + code + "-host";
    }

    function uid() {
      if (window.crypto && crypto.randomUUID) return crypto.randomUUID();
      return "msg-" + Date.now() + "-" + Math.random().toString(16).slice(2);
    }

    function escapeHtml(value) {
      return String(value)
        .replaceAll("&", "&amp;")
        .replaceAll("<", "&lt;")
        .replaceAll(">", "&gt;")
        .replaceAll('"', "&quot;")
        .replaceAll("'", "&#039;");
    }

    function runSelfTests() {
      const tests = [
        { input: " Lớp 7A!! ", expected: "lp7a" },
        { input: "room_123-abc", expected: "room_123-abc" },
        { input: "ABC DEF", expected: "abcdef" },
        { input: "012345678901234567890123456789", expected: "012345678901234567890123" }
      ];

      tests.forEach((test) => {
        const actual = normalizeRoomCode(test.input);
        console.assert(actual === test.expected, "normalizeRoomCode failed", test, actual);
      });

      console.assert(makeHostId("abc") === "room-abc-host", "makeHostId failed");
      console.assert(stickerPacks.every((pack) => pack.items.length >= 6), "Sticker packs need at least 6 stickers");
    }

    function setStatus(text) {
      els.statusText.textContent = text;
    }

    function setOnline(value) {
      state.online = value;
      els.wifiIcon.textContent = value ? "📶" : "📴";
      els.onlinePill.textContent = value ? "Online" : "Offline";
      els.onlinePill.classList.toggle("online", value);
      els.onlineDot.classList.toggle("online", value);
    }

    function setJoined(value) {
      state.joined = value;
      els.joinBtn.classList.toggle("hidden", value);
      els.leaveBtn.classList.toggle("hidden", !value);
      els.nameInput.disabled = value;
      els.roomInput.disabled = value;
      els.messageInput.disabled = !value;
      els.sendBtn.disabled = !value || !els.messageInput.value.trim();
      els.messageInput.placeholder = value ? "Nhập tin nhắn hoặc chọn icon..." : "Vào phòng trước để nhắn tin";
    }

    function setPeerId(id) {
      if (id) {
        els.peerIdText.textContent = "ID: " + id;
        els.peerIdText.classList.remove("hidden");
      } else {
        els.peerIdText.textContent = "";
        els.peerIdText.classList.add("hidden");
      }
    }

    function setHostId(id) {
      if (id) {
        els.hostIdText.textContent = "Host: " + id;
        els.hostIdText.classList.remove("hidden");
      } else {
        els.hostIdText.textContent = "";
        els.hostIdText.classList.add("hidden");
      }
    }

    function clearMessages() {
      els.messages.innerHTML = "";
      els.emptyState = document.createElement("div");
      els.emptyState.className = "empty";
      els.emptyState.id = "emptyState";
      els.emptyState.textContent = "Nhập mã phòng rồi bắt đầu nhắn tin. Bật bảng icon để gửi sticker siêu đẹp.";
      els.messages.appendChild(els.emptyState);
    }

    function addMessage(msg) {
      const empty = document.getElementById("emptyState");
      if (empty) empty.remove();

      const wrap = document.createElement("div");
      wrap.className = "msg-wrap " + (msg.type === "system" ? "system" : msg.mine ? "mine" : "other");
      wrap.dataset.id = uid();

      if (msg.type === "system") {
        wrap.innerHTML = `<span class="system-text">${escapeHtml(msg.text)}</span>`;
      } else if (msg.type === "sticker") {
        wrap.innerHTML = `
          <div class="sticker-bubble">
            <div class="meta">
              <span>${escapeHtml(msg.from)}</span>
              <span class="time">${escapeHtml(msg.time || currentTime())}</span>
              <span>💖</span>
            </div>
            <div class="sticker-big">${escapeHtml(msg.text)}</div>
          </div>
        `;
      } else {
        wrap.innerHTML = `
          <div class="bubble">
            <div class="meta">
              <span>${escapeHtml(msg.from)}</span>
              <span class="time">${escapeHtml(msg.time || currentTime())}</span>
            </div>
            <div>${escapeHtml(msg.text)}</div>
          </div>
        `;
      }

      els.messages.appendChild(wrap);
      els.messages.scrollTo({ top: els.messages.scrollHeight, behavior: "smooth" });
    }

    function currentTime() {
      return new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    }

    function broadcast(data, exceptPeerId) {
      Object.entries(state.conns).forEach(([id, conn]) => {
        if (id !== exceptPeerId && conn.open) conn.send(data);
      });
    }

    function getUsers() {
      const list = [
        { id: state.peer ? state.peer.id : "", name: state.myName, host: state.isHost },
        ...Object.values(state.conns)
          .filter((conn) => conn.open && conn.metadata && conn.metadata.name)
          .map((conn) => ({ id: conn.peer, name: conn.metadata.name, host: false }))
      ];
      return list;
    }

    function renderUsers(users) {
      els.userCount.textContent = users.length;
      if (!users.length) {
        els.userList.innerHTML = `<p class="muted">Chưa có ai trong phòng.</p>`;
        return;
      }

      els.userList.innerHTML = users.map((u) => `
        <div class="user-item">
          <span>${escapeHtml(u.name)}</span>
          ${u.host ? '<span class="host-tag">host</span>' : ''}
        </div>
      `).join("");
    }

    function updateUsers() {
      const users = getUsers();
      renderUsers(users);
      if (state.isHost) broadcast({ type: "users", users }, null);
    }

    function setupConnection(conn) {
      state.conns[conn.peer] = conn;

      conn.on("open", () => {
        setOnline(true);
        addMessage({ type: "system", text: (conn.metadata && conn.metadata.name ? conn.metadata.name : conn.peer) + " đã online." });
        updateUsers();

        if (!state.isHost) {
          conn.send({ type: "hello", name: state.myName });
        } else {
          conn.send({ type: "users", users: getUsers() });
        }
      });

      conn.on("data", (data) => {
        if (!data || typeof data !== "object") return;

        if (data.type === "chat" || data.type === "sticker") {
          addMessage({
            type: data.type,
            from: data.from,
            text: data.text,
            mine: false,
            time: currentTime()
          });

          if (state.isHost) broadcast(data, conn.peer);
        }

        if (data.type === "hello") {
          conn.metadata = { ...(conn.metadata || {}), name: data.name };
          updateUsers();
        }

        if (data.type === "users") {
          renderUsers(data.users || []);
        }
      });

      conn.on("close", () => {
        addMessage({ type: "system", text: (conn.metadata && conn.metadata.name ? conn.metadata.name : conn.peer) + " đã offline." });
        delete state.conns[conn.peer];
        updateUsers();
      });

      conn.on("error", () => {
        addMessage({ type: "system", text: "Kết nối gặp lỗi." });
      });
    }

    function joinRoom() {
      const code = normalizeRoomCode(els.roomInput.value);
      if (!code) {
        setStatus("Nhập mã phòng trước đã nhé.");
        return;
      }

      if (!window.Peer) {
        setStatus("Không tải được PeerJS. Hãy kiểm tra mạng hoặc CDN.");
        return;
      }

      state.roomCode = code;
      state.myName = els.nameInput.value.trim() || state.myName;
      els.nameInput.value = state.myName;
      els.roomInput.value = code;
      els.roomLabel.textContent = code;
      clearMessages();
      renderUsers([]);
      state.conns = {};

      const fixedHostId = makeHostId(code);
      setHostId(fixedHostId);
      setStatus("Đang vào phòng...");

      const peer = new Peer(fixedHostId, { debug: 1 });
      state.peer = peer;

      peer.on("open", (id) => {
        state.isHost = true;
        setPeerId(id);
        setJoined(true);
        setOnline(true);
        setStatus("Bạn đang là chủ phòng: " + code);
        addMessage({ type: "system", text: "Đã tạo phòng " + code + ". Người khác nhập cùng mã để chat." });
        updateUsers();
      });

      peer.on("connection", setupConnection);

      peer.on("error", (err) => {
        if (err.type === "unavailable-id") {
          peer.destroy();
          state.isHost = false;

          const clientPeer = new Peer(undefined, { debug: 1 });
          state.peer = clientPeer;

          clientPeer.on("open", (id) => {
            setPeerId(id);
            setJoined(true);
            setOnline(true);
            setStatus("Đã vào phòng: " + code);
            const conn = clientPeer.connect(fixedHostId, {
              reliable: true,
              metadata: { name: state.myName }
            });
            setupConnection(conn);
            addMessage({ type: "system", text: "Đã kết nối tới phòng " + code + "." });
          });

          clientPeer.on("error", () => {
            setOnline(false);
            setStatus("Không kết nối được phòng. Hãy thử lại hoặc tạo mã khác.");
          });
        } else {
          setOnline(false);
          setStatus("Lỗi PeerJS: " + (err.type || "không rõ"));
        }
      });
    }

    function leaveRoom() {
      Object.values(state.conns).forEach((conn) => conn.close());
      state.conns = {};
      if (state.peer) state.peer.destroy();
      state.peer = null;
      state.isHost = false;
      setJoined(false);
      setOnline(false);
      setPeerId("");
      setHostId("");
      renderUsers([]);
      els.panel.classList.remove("open");
      setStatus("Đã rời phòng");
    }

    function sendPayload(payload) {
      if (!state.joined) return;

      addMessage({ ...payload, mine: true, time: currentTime() });

      const networkPayload = {
        type: payload.type,
        from: state.myName,
        text: payload.text,
        stickerPack: payload.stickerPack
      };

      if (state.isHost) {
        broadcast(networkPayload, null);
      } else {
        const hostConn = Object.values(state.conns)[0];
        if (hostConn && hostConn.open) {
          hostConn.send(networkPayload);
        } else {
          addMessage({ type: "system", text: "Chưa có kết nối tới chủ phòng." });
        }
      }
    }

    function sendMessage() {
      const text = els.messageInput.value.trim();
      if (!text || !state.joined) return;
      sendPayload({ type: "chat", from: state.myName, text });
      els.messageInput.value = "";
      els.sendBtn.disabled = true;
    }

    function sendSticker(text, packId) {
      if (!state.joined) return;
      sendPayload({ type: "sticker", from: state.myName, text, stickerPack: packId });
    }

    function renderIcons() {
      els.iconsPanel.innerHTML = weirdIcons.map((icon) => `
        <button class="icon-btn" data-icon="${escapeHtml(icon)}">${escapeHtml(icon)}</button>
      `).join("");

      els.iconsPanel.querySelectorAll(".icon-btn").forEach((btn) => {
        btn.addEventListener("click", () => {
          els.messageInput.value += (els.messageInput.value ? " " : "") + btn.dataset.icon;
          els.messageInput.focus();
          els.sendBtn.disabled = !state.joined || !els.messageInput.value.trim();
        });
      });
    }

    function renderPacks() {
      els.packRow.innerHTML = stickerPacks.map((pack) => `
        <button class="pack-btn ${pack.id === state.activePack ? "active" : ""}" data-pack="${pack.id}">${escapeHtml(pack.name)}</button>
      `).join("");

      els.packRow.querySelectorAll(".pack-btn").forEach((btn) => {
        btn.addEventListener("click", () => {
          state.activePack = btn.dataset.pack;
          renderPacks();
          renderStickers();
        });
      });
    }

    function renderStickers() {
      const pack = stickerPacks.find((item) => item.id === state.activePack) || stickerPacks[0];
      els.stickerGrid.innerHTML = pack.items.map((sticker) => `
        <button class="sticker-btn" data-sticker="${escapeHtml(sticker)}" ${state.joined ? "" : "disabled"}>${escapeHtml(sticker)}</button>
      `).join("");

      els.stickerGrid.querySelectorAll(".sticker-btn").forEach((btn) => {
        btn.addEventListener("click", () => sendSticker(btn.dataset.sticker, pack.id));
      });
    }

    function renderQuickTexts() {
      els.quickPanel.innerHTML = quickTexts.map((text) => `
        <button class="quick-btn" data-text="${escapeHtml(text)}" ${state.joined ? "" : "disabled"}>${escapeHtml(text)}</button>
      `).join("");

      els.quickPanel.querySelectorAll(".quick-btn").forEach((btn) => {
        btn.addEventListener("click", () => sendPayload({ type: "chat", from: state.myName, text: btn.dataset.text }));
      });
    }

    function showTab(tabName) {
      document.querySelectorAll(".tab").forEach((tab) => {
        tab.classList.remove("active", "cyan", "pink", "amber");
        if (tab.dataset.tab === tabName) {
          tab.classList.add("active");
          if (tabName === "icons") tab.classList.add("cyan");
          if (tabName === "stickers") tab.classList.add("pink");
          if (tabName === "quick") tab.classList.add("amber");
        }
      });

      els.iconsPanel.classList.toggle("hidden", tabName !== "icons");
      els.stickersPanel.classList.toggle("hidden", tabName !== "stickers");
      els.quickPanel.classList.toggle("hidden", tabName !== "quick");
    }

    function copyRoom() {
      const code = normalizeRoomCode(els.roomInput.value);
      if (!code) return;

      if (navigator.clipboard) {
        navigator.clipboard.writeText(code)
          .then(() => addMessage({ type: "system", text: "Đã copy mã phòng: " + code }))
          .catch(() => addMessage({ type: "system", text: "Mã phòng của bạn: " + code }));
      } else {
        addMessage({ type: "system", text: "Mã phòng của bạn: " + code });
      }
    }

    function boot() {
      runSelfTests();
      els.nameInput.value = state.myName;

      renderIcons();
      renderPacks();
      renderStickers();
      renderQuickTexts();

      els.roomInput.addEventListener("input", () => {
        els.roomInput.value = normalizeRoomCode(els.roomInput.value);
        els.roomLabel.textContent = els.roomInput.value || "chưa có";
      });

      els.roomInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter" && !state.joined) joinRoom();
      });

      els.messageInput.addEventListener("input", () => {
        els.sendBtn.disabled = !state.joined || !els.messageInput.value.trim();
      });

      els.messageInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter" && !event.shiftKey) {
          event.preventDefault();
          sendMessage();
        }
      });

      els.joinBtn.addEventListener("click", joinRoom);
      els.leaveBtn.addEventListener("click", leaveRoom);
      els.sendBtn.addEventListener("click", sendMessage);
      els.copyBtn.addEventListener("click", copyRoom);
      els.panelBtn.addEventListener("click", () => els.panel.classList.toggle("open"));
      els.emojiBtn.addEventListener("click", () => els.panel.classList.toggle("open"));

      document.querySelectorAll(".tab").forEach((tab) => {
        tab.addEventListener("click", () => showTab(tab.dataset.tab));
      });
    }

    boot();
  </script>
</body>
</html>
}

function runSelfTests() {
  TEST_CASES.forEach((test) => {
    const actual = normalizeRoomCode(test.input);
    console.assert(actual === test.expected, `normalizeRoomCode failed: ${test.reason}. Expected ${test.expected}, got ${actual}`);
  });
  console.assert(makeHostId("abc") === "room-abc-host", "makeHostId should create the fixed host peer id");
  console.assert(stickerPacks.every((pack) => pack.items.length >= 6), "Each sticker pack should have at least 6 stickers");
}

export default function PeerRoomChatApp() {
  const [roomCode, setRoomCode] = useState("");
  const [myName, setMyName] = useState(() => `User-${Math.floor(Math.random() * 900 + 100)}`);
  const [joined, setJoined] = useState(false);
  const [status, setStatus] = useState("Chưa vào phòng");
  const [online, setOnline] = useState(false);
  const [peerId, setPeerId] = useState("");
  const [hostId, setHostId] = useState("");
  const [message, setMessage] = useState("");
  const [messages, setMessages] = useState([]);
  const [users, setUsers] = useState([]);
  const [panelOpen, setPanelOpen] = useState(false);
  const [panelTab, setPanelTab] = useState("icons");
  const [activePack, setActivePack] = useState("cool");

  const peerRef = useRef(null);
  const connsRef = useRef({});
  const isHostRef = useRef(false);
  const roomRef = useRef("");
  const messagesEndRef = useRef(null);

  useEffect(() => {
    runSelfTests();
  }, []);

  const addMessage = (msg) => {
    setMessages((prev) => [
      ...prev,
      {
        id: uid(),
        time: new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" }),
        ...msg,
      },
    ]);
  };

  const scrollToBottom = () => {
    setTimeout(() => messagesEndRef.current?.scrollIntoView({ behavior: "smooth" }), 50);
  };

  useEffect(scrollToBottom, [messages]);

  const broadcast = (data, exceptPeerId = null) => {
    Object.entries(connsRef.current).forEach(([id, conn]) => {
      if (id !== exceptPeerId && conn.open) conn.send(data);
    });
  };

  const updateUsers = () => {
    const list = [
      { id: peerRef.current?.id || "", name: myName, host: isHostRef.current },
      ...Object.values(connsRef.current)
        .filter((conn) => conn.open && conn.metadata?.name)
        .map((conn) => ({ id: conn.peer, name: conn.metadata.name, host: false })),
    ];

    setUsers(list);
    if (isHostRef.current) broadcast({ type: "users", users: list });
  };

  const setupConnection = (conn) => {
    connsRef.current[conn.peer] = conn;

    conn.on("open", () => {
      setOnline(true);
      addMessage({ type: "system", text: `${conn.metadata?.name || conn.peer} đã online.` });
      updateUsers();

      if (!isHostRef.current) {
        conn.send({ type: "hello", name: myName });
      } else {
        const hostUser = { id: peerRef.current?.id || "", name: myName, host: true };
        conn.send({ type: "users", users: [hostUser] });
      }
    });

    conn.on("data", (data) => {
      if (!data || typeof data !== "object") return;

      if (data.type === "chat" || data.type === "sticker") {
        addMessage({
          type: data.type,
          from: data.from,
          text: data.text,
          mine: false,
          stickerPack: data.stickerPack,
        });
        if (isHostRef.current) broadcast(data, conn.peer);
      }

      if (data.type === "hello") {
        conn.metadata = { ...conn.metadata, name: data.name };
        updateUsers();
      }

      if (data.type === "users") {
        setUsers(data.users || []);
      }
    });

    conn.on("close", () => {
      addMessage({ type: "system", text: `${conn.metadata?.name || conn.peer} đã offline.` });
      delete connsRef.current[conn.peer];
      updateUsers();
    });

    conn.on("error", () => {
      addMessage({ type: "system", text: "Kết nối gặp lỗi." });
    });
  };

  const loadPeerJs = async () => {
    if (window.Peer) return true;

    setStatus("Đang tải PeerJS...");

    const existingScript = document.querySelector('script[data-peerjs="true"]');
    if (existingScript) {
      await new Promise((resolve) => {
        existingScript.addEventListener("load", resolve, { once: true });
        existingScript.addEventListener("error", resolve, { once: true });
      });
      return Boolean(window.Peer);
    }

    const script = document.createElement("script");
    script.src = "https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js";
    script.async = true;
    script.dataset.peerjs = "true";
    document.body.appendChild(script);

    await new Promise((resolve) => {
      script.onload = resolve;
      script.onerror = resolve;
    });

    if (!window.Peer) {
      setStatus("Không tải được PeerJS. Kiểm tra mạng hoặc CDN.");
      return false;
    }

    return true;
  };

  const joinRoom = async () => {
    const code = normalizeRoomCode(roomCode);
    if (!code) {
      setStatus("Nhập mã phòng trước đã nhé.");
      return;
    }

    const peerReady = await loadPeerJs();
    if (!peerReady) return;

    roomRef.current = code;
    const fixedHostId = makeHostId(code);
    setHostId(fixedHostId);
    setMessages([]);
    setUsers([]);
    connsRef.current = {};

    const peer = new window.Peer(fixedHostId, { debug: 1 });
    peerRef.current = peer;
    setStatus("Đang vào phòng...");

    peer.on("open", (id) => {
      isHostRef.current = true;
      setPeerId(id);
      setJoined(true);
      setOnline(true);
      setStatus(`Bạn đang là chủ phòng: ${code}`);
      addMessage({ type: "system", text: `Đã tạo phòng ${code}. Người khác nhập cùng mã để chat.` });
      updateUsers();
    });

    peer.on("connection", setupConnection);

    peer.on("error", (err) => {
      if (err.type === "unavailable-id") {
        peer.destroy();
        isHostRef.current = false;

        const clientPeer = new window.Peer(undefined, { debug: 1 });
        peerRef.current = clientPeer;

        clientPeer.on("open", (id) => {
          setPeerId(id);
          setJoined(true);
          setOnline(true);
          setStatus(`Đã vào phòng: ${code}`);
          const conn = clientPeer.connect(fixedHostId, { reliable: true, metadata: { name: myName } });
          setupConnection(conn);
          addMessage({ type: "system", text: `Đã kết nối tới phòng ${code}.` });
        });

        clientPeer.on("error", () => {
          setOnline(false);
          setStatus("Không kết nối được phòng. Hãy thử lại hoặc tạo mã khác.");
        });
      } else {
        setOnline(false);
        setStatus(`Lỗi PeerJS: ${err.type || "không rõ"}`);
      }
    });
  };

  const leaveRoom = () => {
    Object.values(connsRef.current).forEach((conn) => conn.close());
    connsRef.current = {};
    peerRef.current?.destroy();
    peerRef.current = null;
    isHostRef.current = false;
    setJoined(false);
    setOnline(false);
    setPeerId("");
    setHostId("");
    setUsers([]);
    setPanelOpen(false);
    setStatus("Đã rời phòng");
  };

  const sendPayload = (payload) => {
    if (!joined) return;

    addMessage({ ...payload, mine: true });

    const networkPayload = {
      type: payload.type,
      from: myName,
      text: payload.text,
      stickerPack: payload.stickerPack,
    };

    if (isHostRef.current) {
      broadcast(networkPayload);
    } else {
      const hostConn = Object.values(connsRef.current)[0];
      if (hostConn?.open) hostConn.send(networkPayload);
      else addMessage({ type: "system", text: "Chưa có kết nối tới chủ phòng." });
    }
  };

  const sendMessage = () => {
    const text = message.trim();
    if (!text || !joined) return;
    sendPayload({ type: "chat", from: myName, text });
    setMessage("");
  };

  const sendSticker = (text, stickerPack = "custom") => {
    if (!joined) return;
    sendPayload({ type: "sticker", from: myName, text, stickerPack });
  };

  const addIconToInput = (icon) => {
    setMessage((prev) => `${prev}${prev ? " " : ""}${icon}`);
  };

  const copyRoom = async () => {
    const code = normalizeRoomCode(roomCode);
    if (!code) return;
    try {
      await navigator.clipboard?.writeText(code);
      addMessage({ type: "system", text: `Đã copy mã phòng: ${code}` });
    } catch {
      addMessage({ type: "system", text: `Mã phòng của bạn: ${code}` });
    }
  };

  const currentPack = stickerPacks.find((pack) => pack.id === activePack) || stickerPacks[0];

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-950 via-indigo-950 to-slate-950 text-slate-100 p-4 sm:p-6 flex items-center justify-center">
      <div className="w-full max-w-6xl grid lg:grid-cols-[350px_1fr] gap-4">
        <motion.aside
          initial={{ opacity: 0, y: 12 }}
          animate={{ opacity: 1, y: 0 }}
          className="bg-slate-900/80 border border-slate-800 rounded-2xl shadow-2xl p-5 backdrop-blur"
        >
          <div className="flex items-center gap-3 mb-6">
            <div className="h-11 w-11 rounded-2xl bg-cyan-500/15 flex items-center justify-center">
              {online ? <Icon size="text-xl">📶</Icon> : <Icon size="text-xl">📴</Icon>}
            </div>
            <div>
              <h1 className="text-xl font-bold">PeerJS Room Chat</h1>
              <p className="text-sm text-slate-400">Chat P2P bằng mã phòng</p>
            </div>
          </div>

          <label className="text-sm text-slate-300">Tên của bạn</label>
          <input
            value={myName}
            disabled={joined}
            onChange={(e) => setMyName(e.target.value)}
            className="mt-2 mb-4 w-full rounded-xl bg-slate-950 border border-slate-700 px-4 py-3 outline-none focus:border-cyan-400 disabled:opacity-60"
            placeholder="Nhập tên"
          />

          <label className="text-sm text-slate-300">Mã phòng</label>
          <div className="mt-2 flex gap-2">
            <input
              value={roomCode}
              disabled={joined}
              onChange={(e) => setRoomCode(normalizeRoomCode(e.target.value))}
              onKeyDown={(e) => e.key === "Enter" && !joined && joinRoom()}
              className="w-full rounded-xl bg-slate-950 border border-slate-700 px-4 py-3 outline-none focus:border-cyan-400 disabled:opacity-60"
              placeholder="vd: lop7a"
            />
            <button onClick={copyRoom} className="rounded-xl bg-slate-800 hover:bg-slate-700 px-3" title="Copy mã phòng">
              <Icon size="text-lg">📋</Icon>
            </button>
          </div>

          {!joined ? (
            <button
              onClick={joinRoom}
              className="mt-4 w-full rounded-xl bg-cyan-500 hover:bg-cyan-400 text-slate-950 font-bold py-3 flex items-center justify-center gap-2"
            >
              <Icon>🚪</Icon> Vào / tạo phòng
            </button>
          ) : (
            <button
              onClick={leaveRoom}
              className="mt-4 w-full rounded-xl bg-rose-500 hover:bg-rose-400 text-white font-bold py-3 flex items-center justify-center gap-2"
            >
              <Icon>🏃</Icon> Rời phòng
            </button>
          )}

          <div className="mt-5 rounded-2xl bg-slate-950 border border-slate-800 p-4">
            <div className="flex items-center justify-between gap-2">
              <span className="text-sm text-slate-400">Trạng thái</span>
              <span className={`text-xs px-2 py-1 rounded-full ${online ? "bg-emerald-500/15 text-emerald-300" : "bg-slate-700 text-slate-300"}`}>
                {online ? "Online" : "Offline"}
              </span>
            </div>
            <p className="mt-2 text-sm break-words">{status}</p>
            {peerId && <p className="mt-2 text-xs text-slate-500 break-all">ID: {peerId}</p>}
            {hostId && <p className="mt-1 text-xs text-slate-500 break-all">Host: {hostId}</p>}
          </div>

          <div className="mt-4 rounded-2xl bg-slate-950 border border-slate-800 p-4">
            <div className="flex items-center gap-2 mb-3 text-sm text-slate-300">
              <Icon>👥</Icon> Người online ({users.length})
            </div>
            <div className="space-y-2">
              {users.length === 0 ? (
                <p className="text-sm text-slate-500">Chưa có ai trong phòng.</p>
              ) : (
                users.map((u) => (
                  <div key={u.id || u.name} className="flex items-center justify-between rounded-xl bg-slate-900 px-3 py-2">
                    <span className="text-sm truncate">{u.name}</span>
                    {u.host && <span className="text-[11px] text-cyan-300">host</span>}
                  </div>
                ))
              )}
            </div>
          </div>

          <div className="mt-4 rounded-2xl bg-gradient-to-br from-fuchsia-500/10 to-cyan-500/10 border border-fuchsia-400/20 p-4">
            <div className="flex items-center gap-2 text-sm font-bold text-fuchsia-200 mb-2">
              <Icon>✨</Icon> Tính năng mới
            </div>
            <p className="text-sm text-slate-400">Gửi icon độc lạ, sticker cỡ lớn, tin nhắn nhanh và hiệu ứng bong bóng đẹp hơn.</p>
          </div>
        </motion.aside>

        <motion.main
          initial={{ opacity: 0, y: 12 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.08 }}
          className="bg-slate-900/80 border border-slate-800 rounded-2xl shadow-2xl flex flex-col min-h-[720px] overflow-hidden backdrop-blur"
        >
          <div className="px-5 py-4 border-b border-slate-800 flex items-center justify-between">
            <div>
              <h2 className="font-bold">Tin nhắn</h2>
              <p className="text-sm text-slate-400">Mã phòng: {normalizeRoomCode(roomCode) || "chưa có"}</p>
            </div>
            <div className="flex items-center gap-3">
              <button
                onClick={() => setPanelOpen((v) => !v)}
                className="rounded-xl bg-slate-800 hover:bg-slate-700 px-3 py-2 flex items-center gap-2 text-sm"
              >
                <Icon>{panelOpen ? "✖️" : "✨"}</Icon>
                <span className="hidden sm:inline">Icon & Sticker</span>
              </button>
              <div className={`h-3 w-3 rounded-full ${online ? "bg-emerald-400" : "bg-slate-600"}`} />
            </div>
          </div>

          <AnimatePresence>
            {panelOpen && (
              <motion.div
                initial={{ height: 0, opacity: 0 }}
                animate={{ height: "auto", opacity: 1 }}
                exit={{ height: 0, opacity: 0 }}
                className="border-b border-slate-800 bg-slate-950/80 overflow-hidden"
              >
                <div className="p-4">
                  <div className="flex flex-wrap gap-2 mb-4">
                    <button
                      onClick={() => setPanelTab("icons")}
                      className={`rounded-xl px-4 py-2 text-sm flex items-center gap-2 ${panelTab === "icons" ? "bg-cyan-500 text-slate-950 font-bold" : "bg-slate-800 text-slate-300"}`}
                    >
                      <Icon>😊</Icon> Icon độc lạ
                    </button>
                    <button
                      onClick={() => setPanelTab("stickers")}
                      className={`rounded-xl px-4 py-2 text-sm flex items-center gap-2 ${panelTab === "stickers" ? "bg-fuchsia-500 text-white font-bold" : "bg-slate-800 text-slate-300"}`}
                    >
                      <Icon>🏷️</Icon> Sticker đẹp
                    </button>
                    <button
                      onClick={() => setPanelTab("quick")}
                      className={`rounded-xl px-4 py-2 text-sm flex items-center gap-2 ${panelTab === "quick" ? "bg-amber-400 text-slate-950 font-bold" : "bg-slate-800 text-slate-300"}`}
                    >
                      <Icon>🪄</Icon> Gửi nhanh
                    </button>
                  </div>

                  {panelTab === "icons" && (
                    <div className="grid grid-cols-8 sm:grid-cols-12 md:grid-cols-16 gap-2">
                      {weirdIcons.map((icon) => (
                        <button
                          key={icon}
                          onClick={() => addIconToInput(icon)}
                          className="h-11 rounded-xl bg-slate-900 hover:bg-cyan-500/20 border border-slate-800 hover:border-cyan-400 text-2xl transition"
                          title="Thêm vào tin nhắn"
                        >
                          {icon}
                        </button>
                      ))}
                    </div>
                  )}

                  {panelTab === "stickers" && (
                    <div>
                      <div className="flex flex-wrap gap-2 mb-4">
                        {stickerPacks.map((pack) => (
                          <button
                            key={pack.id}
                            onClick={() => setActivePack(pack.id)}
                            className={`rounded-full px-3 py-1 text-sm ${activePack === pack.id ? "bg-fuchsia-500 text-white" : "bg-slate-800 text-slate-300"}`}
                          >
                            {pack.name}
                          </button>
                        ))}
                      </div>
                      <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-6 gap-3">
                        {currentPack.items.map((sticker) => (
                          <button
                            key={sticker}
                            disabled={!joined}
                            onClick={() => sendSticker(sticker, currentPack.id)}
                            className="rounded-2xl min-h-24 bg-gradient-to-br from-slate-900 to-slate-800 hover:from-fuchsia-500/20 hover:to-cyan-500/20 border border-slate-700 hover:border-fuchsia-300 text-4xl disabled:opacity-40 shadow-lg transition"
                            title="Gửi sticker"
                          >
                            {sticker}
                          </button>
                        ))}
                      </div>
                    </div>
                  )}

                  {panelTab === "quick" && (
                    <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-2">
                      {quickTexts.map((text) => (
                        <button
                          key={text}
                          disabled={!joined}
                          onClick={() => sendPayload({ type: "chat", from: myName, text })}
                          className="text-left rounded-xl bg-slate-900 hover:bg-amber-400/15 border border-slate-800 hover:border-amber-300 px-4 py-3 disabled:opacity-40"
                        >
                          {text}
                        </button>
                      ))}
                    </div>
                  )}
                </div>
              </motion.div>
            )}
          </AnimatePresence>

          <div className="flex-1 overflow-y-auto p-5 space-y-3">
            <AnimatePresence initial={false}>
              {messages.length === 0 ? (
                <div className="h-full flex items-center justify-center text-center text-slate-500">
                  <p>Nhập mã phòng rồi bắt đầu nhắn tin. Bật bảng icon để gửi sticker siêu đẹp.</p>
                </div>
              ) : (
                messages.map((msg) => (
                  <motion.div
                    key={msg.id}
                    initial={{ opacity: 0, y: 8, scale: 0.98 }}
                    animate={{ opacity: 1, y: 0, scale: 1 }}
                    exit={{ opacity: 0 }}
                    className={msg.type === "system" ? "text-center" : `flex ${msg.mine ? "justify-end" : "justify-start"}`}
                  >
                    {msg.type === "system" ? (
                      <span className="inline-block text-xs text-slate-400 bg-slate-950 border border-slate-800 rounded-full px-3 py-1">{msg.text}</span>
                    ) : msg.type === "sticker" ? (
                      <div className={`max-w-[78%] rounded-[2rem] px-5 py-4 border shadow-xl ${msg.mine ? "bg-gradient-to-br from-cyan-400/25 to-fuchsia-500/25 border-cyan-300/40" : "bg-gradient-to-br from-slate-800 to-indigo-900 border-fuchsia-300/30"}`}>
                        <div className="flex items-center gap-2 mb-2">
                          <span className="text-xs font-bold opacity-80">{msg.from}</span>
                          <span className="text-[11px] opacity-60">{msg.time}</span>
                          <Icon size="text-xs">💖</Icon>
                        </div>
                        <div className="text-6xl sm:text-7xl leading-tight text-center px-3 py-2 drop-shadow-lg">{msg.text}</div>
                      </div>
                    ) : (
                      <div className={`max-w-[78%] rounded-2xl px-4 py-3 shadow-lg ${msg.mine ? "bg-cyan-500 text-slate-950" : "bg-slate-800 text-slate-100"}`}>
                        <div className="flex items-center gap-2 mb-1">
                          <span className="text-xs font-bold opacity-80">{msg.from}</span>
                          <span className="text-[11px] opacity-60">{msg.time}</span>
                        </div>
                        <p className="whitespace-pre-wrap break-words">{msg.text}</p>
                      </div>
                    )}
                  </motion.div>
                ))
              )}
            </AnimatePresence>
            <div ref={messagesEndRef} />
          </div>

          <div className="p-4 border-t border-slate-800 bg-slate-950/60">
            <div className="flex gap-2">
              <button
                onClick={() => setPanelOpen((v) => !v)}
                className="rounded-xl bg-slate-800 hover:bg-slate-700 px-3 flex items-center justify-center"
                title="Mở icon và sticker"
              >
                <Icon size="text-xl">😊</Icon>
              </button>
              <input
                value={message}
                disabled={!joined}
                onChange={(e) => setMessage(e.target.value)}
                onKeyDown={(e) => e.key === "Enter" && !e.shiftKey && sendMessage()}
                className="flex-1 rounded-xl bg-slate-950 border border-slate-700 px-4 py-3 outline-none focus:border-cyan-400 disabled:opacity-50"
                placeholder={joined ? "Nhập tin nhắn hoặc chọn icon..." : "Vào phòng trước để nhắn tin"}
              />
              <button
                disabled={!joined || !message.trim()}
                onClick={sendMessage}
                className="rounded-xl bg-cyan-500 hover:bg-cyan-400 disabled:bg-slate-700 disabled:text-slate-500 text-slate-950 font-bold px-5 flex items-center gap-2"
              >
                <Icon>➤</Icon>
                <span className="hidden sm:inline">Gửi</span>
              </button>
            </div>
          </div>
        </motion.main>
      </div>
    </div>
  );
}
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
