import React, { useEffect, useRef, useState } from "react";
import { motion, AnimatePresence } from "framer-motion";

const weirdIcons = [
  "🪩", "🧿", "🫧", "🛸", "🪬", "🧃", "🦄", "🐉", "🦖", "🦕", "🫨", "🤯",
  "😈", "👽", "🤖", "🧌", "🧙", "🧛", "🧟", "🥷", "🫰", "🫶", "🫵", "🤌",
  "💥", "💫", "🌪️", "🔥", "⚡", "☄️", "🌈", "🌙", "⭐", "🍄", "🌵", "🌊",
  "🍕", "🍟", "🍜", "🍣", "🍩", "🍭", "🥤", "🧋", "🎮", "🎧", "🎲", "🎯",
  "🚀", "🏆", "💎", "🔮", "🧸", "🎁", "🪄", "🧨", "🦾", "👑", "🕶️", "💀",
];

const stickerPacks = [
  { id: "cool", name: "Ngầu", items: ["😎🔥", "👑✨", "🕶️💥", "💎🧊", "🚀🌙", "⚡😈"] },
  { id: "cute", name: "Dễ thương", items: ["🥺👉👈", "🐣💛", "🧸💕", "🌷😊", "🐰🍓", "🫶✨"] },
  { id: "meme", name: "Meme", items: ["💀💀💀", "🤡🎪", "🗿☕", "🤯📈", "😭👌", "😼📸"] },
  { id: "vibe", name: "Vibe", items: ["🌌🪐", "🌊🫧", "🍄🌈", "☄️💫", "🪩🎧", "🌙⭐"] },
  { id: "battle", name: "Chiến", items: ["⚔️🔥", "🛡️👊", "🐉⚡", "🥷🌑", "🏆💥", "🧨😈"] },
  { id: "love", name: "Tim", items: ["💖✨", "💘🥺", "💞🫶", "❤️‍🔥😳", "💕🌷", "💝🎁"] },
];

const quickTexts = [
  "Hello cả phòng 👋",
  "Ai online không? 👀",
  "Quá cháy luôn 🔥",
  "Từ từ để mình rep 😭",
  "Ok nè ✅",
  "Haha vui quá 🤣",
];

const TEST_CASES = [
  { input: " Lớp 7A!! ", expected: "lp7a", reason: "lọc ký tự tiếng Việt/dấu và ký tự đặc biệt để PeerJS ID an toàn" },
  { input: "room_123-abc", expected: "room_123-abc", reason: "giữ chữ thường, số, gạch dưới và gạch ngang" },
  { input: "ABC DEF", expected: "abcdef", reason: "xóa khoảng trắng và chuyển về chữ thường" },
  { input: "012345678901234567890123456789", expected: "012345678901234567890123", reason: "giới hạn mã phòng 24 ký tự" },
];

function normalizeRoomCode(value) {
  return String(value || "")
    .trim()
    .toLowerCase()
    .replace(/[^a-z0-9-_]/g, "")
    .slice(0, 24);
}

function makeHostId(code) {
  return `room-${code}-host`;
}

function uid() {
  if (typeof crypto !== "undefined" && crypto.randomUUID) return crypto.randomUUID();
  return `msg-${Date.now()}-${Math.random().toString(16).slice(2)}`;
}

function Icon({ children, className = "", size = "text-base" }) {
  return <span className={`inline-flex items-center justify-center leading-none ${size} ${className}`}>{children}</span>;
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
