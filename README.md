// shadow-reapers.js
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom/client";
import { initializeApp } from "firebase/app";
import { getDatabase, ref, push, onValue, set, remove, update } from "firebase/database";
import { getAuth } from "firebase/auth";
import './styles.css'; // Use the same neon-dark CSS from previous code

// ===== Firebase Setup =====
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_DOMAIN",
  databaseURL: "YOUR_DB_URL",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_BUCKET",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
const app = initializeApp(firebaseConfig);
const db = getDatabase(app);
const auth = getAuth(app);

// ===== Landing Page =====
function LandingPage() {
  return <div className="landing"><div className="neon-logo">Shadow Reapers</div></div>;
}

// ===== Home Page =====
function HomePage() {
  return (
    <div className="home">
      <section className="section">
        <h1>Welcome to Shadow Reapers</h1>
        <p>Play international tournaments and become a pro gamer!</p>
        <p>Current / Upcoming Tournaments:</p>
        <TournamentPage />
      </section>
    </div>
  );
}

// ===== Tournament Page (Filter + Join) =====
function TournamentPage() {
  const [tournaments, setTournaments] = useState([
    { id: 1, name: "FreeFire Summer Cup", game: "FreeFire", date: "2025-09-15", fee: 50 },
    { id: 2, name: "BGMI Pro League", game: "BGMI", date: "2025-09-20", fee: 100 },
    { id: 3, name: "COD Royale", game: "COD", date: "2025-09-18", fee: 70 },
    { id: 4, name: "Valorant Masters", game: "Valorant", date: "2025-09-25", fee: 120 }
  ]);

  const [selectedGame, setSelectedGame] = useState("All");
  const [selectedTournament, setSelectedTournament] = useState(null);
  const [joined, setJoined] = useState(false);

  const filteredTournaments = tournaments
    .filter(t => selectedGame === "All" || t.game === selectedGame)
    .sort((a, b) => new Date(a.date) - new Date(b.date));

  const games = ["All", "FreeFire", "BGMI", "COD", "PUBG", "Valorant", "COC"];

  return (
    <div>
      <div style={{ marginBottom: "20px" }}>
        {games.map(game => (
          <button key={game} className="button" style={{ marginRight: "10px" }} onClick={() => setSelectedGame(game)}>{game}</button>
        ))}
      </div>

      {!selectedTournament && filteredTournaments.map(t => (
        <div key={t.id} className="tournament-box">
          <h2>{t.name} [{t.game}]</h2>
          <p>Date: {t.date}</p>
          <p>Entry Fee: ₹{t.fee}</p>
          <button className="button" onClick={() => setSelectedTournament(t)}>Join Tournament</button>
        </div>
      ))}

      {selectedTournament && !joined && <PaymentPage tournament={selectedTournament} onSuccess={() => setJoined(true)} />}
      {joined && selectedTournament && <TournamentRoom tournament={selectedTournament} />}
    </div>
  );
}

// ===== Payment Page =====
function PaymentPage({ tournament, onSuccess }) {
  const [info, setInfo] = useState({ name: '', gameID: '', age: '', dob: '', maxKills: '', totalMatches: '' });

  const handleChange = e => setInfo({ ...info, [e.target.name]: e.target.value });

  const handlePayment = () => {
    alert(`Payment placeholder. UPI ID integration coming soon.`);
    onSuccess();
  }

  return (
    <div>
      <h2>Join {tournament.name}</h2>
      <input type="text" name="name" placeholder="Name" onChange={handleChange} />
      <input type="text" name="gameID" placeholder="Game ID" onChange={handleChange} />
      <input type="number" name="age" placeholder="Age" onChange={handleChange} />
      <input type="date" name="dob" onChange={handleChange} />
      <input type="number" name="maxKills" placeholder="Max Kills (optional)" onChange={handleChange} />
      <input type="number" name="totalMatches" placeholder="Total Matches (optional)" onChange={handleChange} />
      <button className="button" onClick={handlePayment}>Pay ₹{tournament.fee} & Join</button>
    </div>
  )
}

// ===== Tournament Room (Chat + Leaderboard) =====
function TournamentRoom({ tournament }) {
  const [message, setMessage] = useState("");
  const [messages, setMessages] = useState([]);
  const [players, setPlayers] = useState([]);
  const [user, setUser] = useState({ name: "", gameID: "", kills: 0 });

  useEffect(() => {
    const chatRef = ref(db, `tournaments/${tournament.id}/chat`);
    onValue(chatRef, snapshot => setMessages(Object.values(snapshot.val() || {})));

    const leaderboardRef = ref(db, `tournaments/${tournament.id}/leaderboard`);
    onValue(leaderboardRef, snapshot => setPlayers(Object.values(snapshot.val() || {}).sort((a, b) => b.kills - a.kills)));
  }, [tournament.id]);

  const sendMessage = () => {
    if (message.trim() === "") return;
    push(ref(db, `tournaments/${tournament.id}/chat`), { user: user.name || "Anonymous", message });
    setMessage("");
  }

  const updateKills = (kills) => set(ref(db, `tournaments/${tournament.id}/leaderboard/${user.gameID}`), { name: user.name, gameID: user.gameID, kills });

  return (
    <div style={{ display: "flex", gap: "50px", flexWrap: "wrap" }}>
      <div style={{ flex: 1, minWidth: "300px" }}>
        <h2 style={{ color: "#0ff" }}>Chat Room</h2>
        <div className="chat-room">{messages.map((msg, idx) => <div key={idx} className="chat-message"><b>{msg.user}:</b> {msg.message}</div>)}</div>
        <input type="text" value={message} onChange={e => setMessage(e.target.value)} placeholder="Type a message" style={{ width: "70%" }} />
        <button onClick={sendMessage} className="button">Send</button>
      </div>
      <div style={{ flex: 1, minWidth: "300px" }}>
        <h2 style={{ color: "#0ff" }}>Leaderboard</h2>
        <input type="text" placeholder="Your Name" onChange={e => setUser({ ...user, name: e.target.value })} />
        <input type="text" placeholder="Game ID" onChange={e => setUser({ ...user, gameID: e.target.value })} />
        <input type="number" placeholder="Kills" onChange={e => setUser({ ...user, kills: Number(e.target.value) })} />
        <button onClick={() => updateKills(user.kills)} className="button">Update Kills</button>
        <ol className="leaderboard">{players.map((p, idx) => <li key={idx}>{p.name} ({p.gameID}) - Kills: {p.kills}</li>)}</ol>
      </div>
    </div>
  );
}

// ===== Admin Panel =====
function AdminPanel() {
  const [tournaments, setTournaments] = useState([]);
  const [newTournament, setNewTournament] = useState({ name: "", game: "", date: "", fee: 0 });

  useEffect(() => {
    const tourRef = ref(db, "tournaments");
    onValue(tourRef, snapshot => setTournaments(Object.entries(snapshot.val() || {}).map(([id, val]) => ({ id, ...val })).sort((a, b) => new Date(a.date) - new Date(b.date))));
  }, []);

  const handleAdd = () => { push(ref(db, "tournaments"), newTournament); setNewTournament({ name: "", game: "", date: "", fee: 0 }); }
  const handleDelete = (id) => remove(ref(db, `tournaments/${id}`));
  const handleEdit = (id, field, value) => update(ref(db, `tournaments/${id}`), { [field]: value });

  return (
    <div style={{ padding: "20px" }}>
      <h2 style={{ color: "#0ff" }}>Admin Panel - Manage Tournaments</h2>
      <div style={{ marginBottom: "20px" }}>
        <input type="text" placeholder="Name" value={newTournament.name} onChange={e => setNewTournament({ ...newTournament, name: e.target.value })} />
        <input type="text" placeholder="Game" value={newTournament.game} onChange={e => setNewTournament({ ...newTournament, game: e.target.value })} />
        <input type="date" value={newTournament.date} onChange={e => setNewTournament({ ...newTournament, date: e.target.value })} />
        <input type="number" value={newTournament.fee} onChange={e => setNewTournament({ ...newTournament, fee: Number(e.target.value) })} />
        <button className="button" onClick={handleAdd}>Add Tournament</button>
      </div>
      <div>
        {tournaments.map(t => (
          <div key={t.id} className="tournament-box">
            <input type="text" value={t.name} onChange={e => handleEdit(t.id, "name", e.target.value)} />
            <input type="text" value={t.game} onChange={e => handleEdit(t.id, "game", e.target.value)} />
            <input type="date" value={t.date} onChange={e => handleEdit(t.id, "date", e.target.value)} />
            <input type="number" value={t.fee} onChange={e => handleEdit(t.id, "fee", Number(e.target.value))} />
            <button className="button" onClick={() => handleDelete(t.id)}>Delete</button>
          </div>
        ))}
      </div>
    </div>
  );
}

// ===== App Component =====
function App() {
  const [showHome, setShowHome] = useState(false);
  const [isAdmin, setIsAdmin] = useState(false);

  useEffect(() => { const timer = setTimeout(() => setShowHome(true), 2000); return () => clearTimeout(timer); }, []);

  return (
    <div>
      {!showHome && <LandingPage />}
      {showHome && !isAdmin && <HomePage />}
      {isAdmin && <AdminPanel />}
      <button className="button" style={{ position: "fixed", top: "10px", right: "10px" }} onClick={() => setIsAdmin(!isAdmin)}>
        {isAdmin ? "Exit Admin" : "Admin Login"}
      </button>
    </div>
  );
}

// ===== Render App =====
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);
