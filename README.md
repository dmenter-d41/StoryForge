<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>STORY FORGE — Collaborative Writing Game</title>
<link href="https://fonts.googleapis.com/css2?family=Permanent+Marker&family=Boogaloo&family=Nunito:wght@400;600;700;800;900&display=swap" rel="stylesheet">

<!-- Firebase: dynamically loaded to guarantee execution order -->
<script>
(function() {
  const FIREBASE_CONFIG = {
    apiKey: "AIzaSyBxLa6H-Mqe9X4ahcC1vZxy-GNqFWscPaw",
    authDomain: "co-writing-game.firebaseapp.com",
    databaseURL: "https://co-writing-game-default-rtdb.firebaseio.com",
    projectId: "co-writing-game",
    storageBucket: "co-writing-game.firebasestorage.app",
    messagingSenderId: "222987886578",
    appId: "1:222987886578:web:08ef1f38626493e384956b"
  };

  function loadScript(src, cb) {
    var s = document.createElement('script');
    s.src = src;
    s.onload = cb;
    s.onerror = function() {
      document.getElementById('loadError').style.display = 'block';
    };
    document.head.appendChild(s);
  }

  // Load app first, then database, then init — guaranteed order
  loadScript('https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js', function() {
    loadScript('https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js', function() {
      firebase.initializeApp(FIREBASE_CONFIG);
      window._firebaseDB = firebase.database();
      // Signal the game that Firebase is ready
      window.dispatchEvent(new Event('firebaseReady'));
    });
  });
})();
</script>

<style>
:root {
  --bg: #0d0d1a;
  --card: #1a1a2e;
  --card2: #16213e;
  --yellow: #FFD93D;
  --orange: #FF6B35;
  --pink: #FF3CAC;
  --cyan: #00F5FF;
  --green: #06FF7A;
  --purple: #784CF7;
  --white: #F0F0FF;
  --gray: #8888aa;
  --p1: #00F5FF;
  --p2: #FF3CAC;
  --p3: #06FF7A;
  --p4: #FFD93D;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
html, body { height: 100%; }
body {
  font-family: 'Nunito', sans-serif;
  background: var(--bg);
  color: var(--white);
  min-height: 100vh;
  overflow-x: hidden;
}

/* STARFIELD */
body::before {
  content: '';
  position: fixed; inset: 0;
  background-image:
    radial-gradient(1px 1px at 8%  18%, rgba(255,255,255,.4) 0%, transparent 100%),
    radial-gradient(1px 1px at 28% 62%, rgba(255,255,255,.3) 0%, transparent 100%),
    radial-gradient(1px 1px at 52% 9%,  rgba(255,255,255,.5) 0%, transparent 100%),
    radial-gradient(1px 1px at 71% 83%, rgba(255,255,255,.3) 0%, transparent 100%),
    radial-gradient(1px 1px at 91% 38%, rgba(255,255,255,.4) 0%, transparent 100%),
    radial-gradient(1px 1px at 18% 91%, rgba(255,255,255,.2) 0%, transparent 100%),
    radial-gradient(1px 1px at 82% 14%, rgba(255,255,255,.35) 0%,transparent 100%),
    radial-gradient(2px 2px at 44% 44%, rgba(255,217,61,.3)  0%, transparent 100%),
    radial-gradient(2px 2px at 14% 76%, rgba(0,245,255,.2)   0%, transparent 100%);
  pointer-events: none; z-index: 0;
}

.wrapper { position: relative; z-index: 1; max-width: 960px; margin: 0 auto; padding: 16px 20px 40px; }

/* ── HEADER ── */
.header { text-align: center; padding: 28px 0 16px; }
.logo {
  font-family: 'Permanent Marker', cursive;
  font-size: clamp(2.2rem,7vw,4.2rem);
  line-height: 1;
  background: linear-gradient(135deg, var(--cyan) 0%, var(--purple) 45%, var(--pink) 100%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
  filter: drop-shadow(0 0 18px rgba(120,76,247,.5));
  animation: pulse-glow 3s ease-in-out infinite;
}
@keyframes pulse-glow {
  0%,100%{filter:drop-shadow(0 0 18px rgba(120,76,247,.5))}
  50%{filter:drop-shadow(0 0 36px rgba(255,60,172,.65))}
}
.tagline { font-family:'Boogaloo',cursive; font-size:.95rem; color:var(--cyan); letter-spacing:3px; text-transform:uppercase; margin-top:6px; }

/* ── TOAST ── */
.toast {
  position:fixed; top:20px; left:50%;
  transform:translateX(-50%) translateY(-80px);
  background:var(--card); border-radius:14px;
  padding:12px 26px; font-family:'Boogaloo',cursive;
  font-size:1.3rem; letter-spacing:1px;
  z-index:9999; transition:transform .4s cubic-bezier(.34,1.56,.64,1);
  pointer-events:none; white-space:nowrap;
}
.toast.show { transform:translateX(-50%) translateY(0); }
.toast.correct { border:2px solid var(--green);  color:var(--green);  box-shadow:0 0 28px rgba(6,255,122,.3); }
.toast.wrong   { border:2px solid var(--pink);   color:var(--pink);   box-shadow:0 0 28px rgba(255,60,172,.3); }
.toast.bonus   { border:2px solid var(--yellow); color:var(--yellow); box-shadow:0 0 28px rgba(255,217,61,.4); }
.toast.info    { border:2px solid var(--cyan);   color:var(--cyan);   box-shadow:0 0 28px rgba(0,245,255,.3); }

/* ── CONFETTI ── */
.confetti-piece {
  position:fixed; width:9px; height:9px; top:-10px;
  pointer-events:none; z-index:9998; border-radius:2px;
  animation:confetti-fall linear forwards;
}
@keyframes confetti-fall { to{transform:translateY(110vh) rotate(720deg);opacity:0} }

/* ── CARD BASE ── */
.card {
  background:linear-gradient(145deg,#1a1a2e,#0f0f20);
  border-radius:18px; padding:24px;
  border:1px solid rgba(255,255,255,.07);
  box-shadow:0 16px 50px rgba(0,0,0,.5);
}

/* ════════════════════════════════
   SCREEN: FIREBASE SETUP WARNING
   ════════════════════════════════ */
#setupScreen {
  display:none;
  text-align:center; padding:40px 24px;
}
#setupScreen h2 { font-family:'Permanent Marker',cursive; font-size:1.8rem; color:var(--yellow); margin-bottom:14px; }
.setup-steps { text-align:left; max-width:520px; margin:0 auto 20px; }
.setup-step { display:flex; gap:12px; padding:10px 0; border-bottom:1px solid rgba(255,255,255,.06); font-size:.9rem; line-height:1.5; }
.step-num { font-family:'Boogaloo',cursive; font-size:1.3rem; color:var(--cyan); flex-shrink:0; width:28px; }
.setup-link { color:var(--cyan); text-decoration:underline; }
pre.config-block {
  background:#0a0a14; border:1px solid rgba(0,245,255,.2); border-radius:10px;
  padding:16px; font-size:.75rem; color:var(--cyan); text-align:left;
  overflow-x:auto; margin:14px 0; line-height:1.6;
}

/* ════════════════════════════════
   SCREEN: JOIN / LOBBY
   ════════════════════════════════ */
#joinScreen { display:none; }
.join-wrap { max-width:480px; margin:0 auto; text-align:center; padding:10px 0 20px; }
.join-title { font-family:'Permanent Marker',cursive; font-size:2rem; color:var(--white); margin-bottom:6px; }
.join-sub { color:var(--gray); font-size:.9rem; margin-bottom:28px; }
.room-row { display:flex; gap:10px; margin-bottom:16px; }
.field-label { font-size:.72rem; text-transform:uppercase; letter-spacing:2px; color:var(--gray); margin-bottom:5px; text-align:left; }
.text-input {
  width:100%; padding:13px 16px;
  background:rgba(255,255,255,.05); border:2px solid rgba(255,255,255,.1);
  border-radius:12px; color:var(--white);
  font-family:'Nunito',sans-serif; font-weight:700; font-size:1rem;
  transition:border-color .2s;
}
.text-input:focus { outline:none; border-color:var(--cyan); background:rgba(0,245,255,.04); }
.text-input::placeholder { color:var(--gray); }
.color-row { display:flex; justify-content:center; gap:12px; margin:14px 0 20px; }
.color-dot {
  width:34px; height:34px; border-radius:50%; cursor:pointer;
  border:3px solid transparent; transition:all .2s;
}
.color-dot:hover,.color-dot.sel { border-color:white; transform:scale(1.15); box-shadow:0 0 14px rgba(255,255,255,.4); }
.join-btn {
  width:100%; padding:15px; border:none; border-radius:50px;
  background:linear-gradient(135deg,var(--cyan),var(--purple));
  color:white; font-family:'Boogaloo',cursive; font-size:1.3rem;
  letter-spacing:1px; cursor:pointer; transition:all .25s;
  box-shadow:0 4px 20px rgba(120,76,247,.4);
}
.join-btn:hover { transform:translateY(-2px); box-shadow:0 8px 30px rgba(120,76,247,.6); }
.join-btn:disabled { opacity:.5; cursor:not-allowed; transform:none; }

/* Lobby waiting */
.lobby-card { text-align:center; padding:28px; }
.lobby-title { font-family:'Boogaloo',cursive; font-size:1.6rem; color:var(--cyan); margin-bottom:4px; }
.lobby-room { font-size:.8rem; color:var(--gray); letter-spacing:2px; text-transform:uppercase; margin-bottom:24px; }
.players-grid { display:flex; justify-content:center; gap:16px; flex-wrap:wrap; margin-bottom:24px; }
.player-slot {
  width:110px; padding:14px 10px; border-radius:14px;
  border:2px dashed rgba(255,255,255,.12);
  text-align:center; transition:all .3s;
}
.player-slot.filled { border-style:solid; box-shadow:0 0 20px rgba(255,255,255,.08); }
.player-avatar { font-size:1.8rem; margin-bottom:5px; }
.player-name { font-weight:800; font-size:.85rem; margin-bottom:2px; }
.player-status { font-size:.7rem; color:var(--gray); letter-spacing:1px; }
.start-btn {
  padding:14px 44px; border:none; border-radius:50px;
  background:linear-gradient(135deg,var(--yellow),var(--orange));
  color:#1a1a00; font-family:'Boogaloo',cursive; font-size:1.2rem;
  cursor:pointer; transition:all .25s;
  box-shadow:0 4px 20px rgba(255,217,61,.35);
}
.start-btn:hover { transform:translateY(-2px); box-shadow:0 8px 28px rgba(255,217,61,.55); }
.start-btn:disabled { opacity:.45; cursor:not-allowed; transform:none; }
.waiting-txt { color:var(--gray); font-size:.88rem; margin-top:10px; font-style:italic; }

/* ════════════════════════════════
   SCREEN: GAME
   ════════════════════════════════ */
#gameScreen { display:none; }

/* Top bar */
.top-bar {
  display:flex; align-items:center; justify-content:space-between;
  background:linear-gradient(135deg,var(--card),var(--card2));
  border:2px solid var(--purple); border-radius:18px;
  padding:12px 20px; margin-bottom:16px;
  box-shadow:0 0 28px rgba(120,76,247,.25);
  flex-wrap:wrap; gap:10px;
}
.top-bar-section { display:flex; align-items:center; gap:10px; }
.tb-label { font-size:.68rem; text-transform:uppercase; letter-spacing:2px; color:var(--gray); }
.tb-value { font-family:'Boogaloo',cursive; font-size:1.7rem; color:var(--yellow); text-shadow:0 0 12px rgba(255,217,61,.5); }
.round-badge {
  background:linear-gradient(135deg,var(--purple),var(--pink));
  padding:6px 18px; border-radius:20px;
  font-family:'Boogaloo',cursive; font-size:1rem; letter-spacing:1px;
}
.phase-badge {
  padding:5px 14px; border-radius:20px; font-family:'Boogaloo',cursive; font-size:.85rem; letter-spacing:1px;
  transition:all .3s;
}
.phase-write  { background:rgba(0,245,255,.15); border:1px solid var(--cyan);   color:var(--cyan); }
.phase-vote   { background:rgba(255,217,61,.15); border:1px solid var(--yellow); color:var(--yellow); }
.phase-reveal { background:rgba(6,255,122,.15);  border:1px solid var(--green);  color:var(--green); }

/* Players scoreboard strip */
.players-strip { display:flex; gap:10px; margin-bottom:16px; flex-wrap:wrap; }
.ps-card {
  flex:1; min-width:90px; padding:10px 14px; border-radius:12px;
  background:rgba(255,255,255,.04); border:2px solid rgba(255,255,255,.08);
  transition:all .3s;
}
.ps-card.me { border-color:rgba(255,255,255,.25); box-shadow:0 0 16px rgba(255,255,255,.06); }
.ps-card.submitted { opacity:.7; }
.ps-name { font-weight:800; font-size:.82rem; margin-bottom:2px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
.ps-pts { font-family:'Boogaloo',cursive; font-size:1.4rem; }
.ps-dot { width:8px; height:8px; border-radius:50%; display:inline-block; margin-right:4px; }
.ps-tick { font-size:.75rem; color:var(--green); margin-top:3px; }

/* Story scroll */
.story-scroll {
  background:linear-gradient(145deg,#12102a,#0c0a1e);
  border:1px solid rgba(120,76,247,.2);
  border-radius:14px; padding:20px 22px; margin-bottom:16px;
  min-height:100px; max-height:260px; overflow-y:auto;
  line-height:1.8; font-size:1rem;
  box-shadow:inset 0 2px 20px rgba(0,0,0,.4);
}
.story-scroll::-webkit-scrollbar { width:5px; }
.story-scroll::-webkit-scrollbar-track { background:rgba(255,255,255,.03); }
.story-scroll::-webkit-scrollbar-thumb { background:var(--purple); border-radius:3px; }
.story-sentence { display:inline; }
.story-sentence.new-winner { animation:sentence-pop .6s ease-out; }
@keyframes sentence-pop { 0%{opacity:0;background:rgba(6,255,122,.15)} 100%{opacity:1;background:transparent} }
.story-empty { color:var(--gray); font-style:italic; font-size:.9rem; }

/* Prompt card */
.prompt-card {
  background:linear-gradient(135deg,rgba(120,76,247,.1),rgba(255,60,172,.06));
  border:1px solid rgba(120,76,247,.25); border-radius:14px;
  padding:16px 20px; margin-bottom:16px;
}
.prompt-label { font-size:.68rem; text-transform:uppercase; letter-spacing:3px; color:var(--purple); margin-bottom:6px; }
.prompt-tags { display:flex; gap:8px; flex-wrap:wrap; margin-bottom:8px; }
.prompt-tag {
  padding:3px 12px; border-radius:20px; font-family:'Boogaloo',cursive; font-size:.85rem;
}
.tag-genre   { background:rgba(0,245,255,.12);  border:1px solid var(--cyan);   color:var(--cyan); }
.tag-char    { background:rgba(255,217,61,.12);  border:1px solid var(--yellow); color:var(--yellow); }
.tag-setting { background:rgba(255,107,53,.12);  border:1px solid var(--orange); color:var(--orange); }
.prompt-text { font-size:1rem; font-weight:700; color:var(--white); line-height:1.5; }
.prompt-loading { color:var(--gray); font-style:italic; font-size:.9rem; }

/* Write phase */
.write-area { margin-bottom:14px; }
.write-label { font-size:.72rem; text-transform:uppercase; letter-spacing:2px; color:var(--gray); margin-bottom:7px; }
.write-hint {
  background:rgba(255,255,255,.03); border-radius:10px;
  padding:10px 14px; font-size:.82rem; color:var(--gray);
  margin-bottom:10px; line-height:1.5; border-left:3px solid var(--purple);
}
.write-hint strong { color:var(--white); }
textarea.story-input {
  width:100%; min-height:90px;
  background:rgba(255,255,255,.04); border:2px solid rgba(255,255,255,.1);
  border-radius:12px; padding:14px 16px;
  color:var(--white); font-family:'Nunito',sans-serif;
  font-weight:700; font-size:1rem; resize:vertical; line-height:1.6;
  transition:border-color .2s;
}
textarea.story-input:focus { outline:none; border-color:var(--purple); background:rgba(120,76,247,.04); }
textarea.story-input::placeholder { color:var(--gray); }
textarea.story-input:disabled { opacity:.5; cursor:not-allowed; }
.write-toolbar { display:flex; align-items:center; justify-content:space-between; margin-top:10px; flex-wrap:wrap; gap:8px; }
.wc { font-size:.78rem; color:var(--gray); }
.wc span { color:var(--yellow); font-weight:800; }
.submit-sentence-btn {
  padding:12px 30px; border:none; border-radius:50px;
  background:linear-gradient(135deg,var(--purple),var(--pink));
  color:white; font-family:'Boogaloo',cursive; font-size:1.1rem;
  letter-spacing:1px; cursor:pointer; transition:all .25s;
  box-shadow:0 4px 18px rgba(120,76,247,.35);
}
.submit-sentence-btn:hover:not(:disabled) { transform:translateY(-2px); box-shadow:0 8px 26px rgba(120,76,247,.55); }
.submit-sentence-btn:disabled { opacity:.5; cursor:not-allowed; }

/* Score chips shown after submit */
.score-chips { display:flex; flex-wrap:wrap; gap:7px; margin-top:10px; }
.score-chip { padding:3px 12px; border-radius:8px; font-size:.78rem; font-weight:700; }
.chip-met  { background:rgba(6,255,122,.12);  color:var(--green); border:1px solid var(--green); }
.chip-miss { background:rgba(255,60,172,.1);  color:var(--pink);  border:1px solid var(--pink); }
.my-score-banner {
  margin-top:10px; padding:10px 16px; border-radius:10px;
  font-family:'Boogaloo',cursive; font-size:1.1rem; text-align:center;
  background:rgba(120,76,247,.15); border:1px solid rgba(120,76,247,.3); color:var(--purple);
}
.waiting-others { text-align:center; padding:14px; color:var(--gray); font-style:italic; font-size:.9rem; }

/* Vote phase */
.vote-section { margin-top:4px; }
.vote-title { font-family:'Boogaloo',cursive; font-size:1.3rem; color:var(--yellow); margin-bottom:4px; }
.vote-sub { color:var(--gray); font-size:.82rem; margin-bottom:14px; }
.vote-cards { display:flex; flex-direction:column; gap:10px; }
.vote-card {
  background:rgba(255,255,255,.04); border:2px solid rgba(255,255,255,.1);
  border-radius:14px; padding:16px 18px; cursor:pointer; transition:all .22s;
  position:relative;
}
.vote-card:hover:not(.voted):not(.own-card) { border-color:var(--yellow); background:rgba(255,217,61,.06); transform:translateX(4px); }
.vote-card.selected { border-color:var(--yellow); background:rgba(255,217,61,.1); box-shadow:0 0 20px rgba(255,217,61,.2); }
.vote-card.own-card { opacity:.75; cursor:default; border-color:rgba(120,76,247,.3); }
.vote-card.winner-card { border-color:var(--green); background:rgba(6,255,122,.08); box-shadow:0 0 25px rgba(6,255,122,.2); }
.vc-header { display:flex; align-items:center; justify-content:space-between; margin-bottom:8px; }
.vc-author { font-size:.72rem; text-transform:uppercase; letter-spacing:2px; color:var(--gray); }
.vc-score-badge {
  font-family:'Boogaloo',cursive; font-size:.9rem; padding:2px 10px; border-radius:8px;
  background:rgba(255,255,255,.08); color:var(--white);
}
.vc-text { font-size:.95rem; line-height:1.55; }
.vc-vote-count { margin-top:8px; font-family:'Boogaloo',cursive; font-size:.95rem; color:var(--yellow); }
.winner-crown { font-size:1.2rem; margin-right:4px; }
.vote-btn {
  margin-top:8px; padding:7px 20px; border:none; border-radius:20px;
  background:linear-gradient(135deg,var(--yellow),var(--orange));
  color:#1a0a00; font-family:'Boogaloo',cursive; font-size:.9rem;
  cursor:pointer; transition:all .2s;
}
.vote-btn:hover { transform:scale(1.04); }
.already-voted { color:var(--gray); font-size:.82rem; font-style:italic; margin-top:4px; }

/* Reveal phase */
.reveal-section { text-align:center; padding:16px 0; }
.reveal-title { font-family:'Permanent Marker',cursive; font-size:1.8rem; color:var(--green); margin-bottom:6px; }
.reveal-winner-name { font-family:'Boogaloo',cursive; font-size:1.3rem; margin-bottom:10px; }
.reveal-sentence {
  background:rgba(6,255,122,.07); border:1px solid rgba(6,255,122,.25);
  border-radius:12px; padding:16px 20px; font-size:1.05rem;
  font-style:italic; margin-bottom:16px; line-height:1.6;
}
.pts-earned { font-family:'Boogaloo',cursive; font-size:1.5rem; color:var(--yellow); margin-bottom:16px; }
.next-round-btn {
  padding:13px 40px; border:none; border-radius:50px;
  background:linear-gradient(135deg,var(--cyan),var(--purple));
  color:white; font-family:'Boogaloo',cursive; font-size:1.2rem;
  cursor:pointer; transition:all .25s;
  box-shadow:0 4px 20px rgba(120,76,247,.4);
}
.next-round-btn:hover { transform:translateY(-2px); box-shadow:0 8px 28px rgba(120,76,247,.6); }

/* Timer ring */
.phase-timer {
  display:flex; align-items:center; gap:10px;
  background:rgba(255,255,255,.04); border-radius:10px;
  padding:8px 14px; margin-bottom:14px;
}
.pt-label { font-size:.68rem; text-transform:uppercase; letter-spacing:2px; color:var(--gray); }
.pt-digits { font-family:'Boogaloo',cursive; font-size:1.4rem; color:var(--orange); min-width:44px; }
.pt-bar-outer { flex:1; height:5px; background:rgba(255,255,255,.07); border-radius:3px; overflow:hidden; }
.pt-bar-inner { height:100%; border-radius:3px; transition:width .9s linear, background .4s; }
.ptb-safe   { background:linear-gradient(90deg,var(--cyan),var(--green)); }
.ptb-warn   { background:linear-gradient(90deg,var(--yellow),var(--orange)); }
.ptb-danger { background:linear-gradient(90deg,var(--orange),var(--pink)); }

/* END SCREEN */
#endScreen { display:none; text-align:center; padding:30px 20px; }
.end-title { font-family:'Permanent Marker',cursive; font-size:2.4rem;
  background:linear-gradient(135deg,var(--yellow),var(--orange));
  -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text;
  margin-bottom:10px;
}
.podium { display:flex; justify-content:center; align-items:flex-end; gap:14px; margin:24px 0; flex-wrap:wrap; }
.podium-place { text-align:center; }
.podium-bar { border-radius:8px 8px 0 0; width:80px; display:flex; align-items:flex-end; justify-content:center; padding-bottom:6px; font-size:1.5rem; }
.p1-bar { height:120px; background:linear-gradient(180deg,var(--yellow),var(--orange)); }
.p2-bar { height:80px;  background:linear-gradient(180deg,#aaa,#666); }
.p3-bar { height:55px;  background:linear-gradient(180deg,#cd7f32,#8b4513); }
.podium-name { font-family:'Boogaloo',cursive; font-size:.95rem; margin-top:6px; }
.podium-pts  { font-size:.8rem; color:var(--gray); }
.story-final {
  background:rgba(255,255,255,.04); border:1px solid rgba(255,255,255,.08);
  border-radius:14px; padding:20px 24px; text-align:left;
  line-height:1.9; font-size:1rem; margin:16px 0;
  max-height:280px; overflow-y:auto;
}
.copy-story-btn {
  padding:13px 36px; border:none; border-radius:50px;
  background:linear-gradient(135deg,var(--green),#04cc60);
  color:#06200f; font-family:'Boogaloo',cursive; font-size:1.1rem;
  cursor:pointer; transition:all .25s; box-shadow:0 4px 18px rgba(6,255,122,.3);
}
.copy-story-btn:hover { transform:translateY(-2px); box-shadow:0 8px 26px rgba(6,255,122,.5); }
.play-again-btn {
  margin-left:12px; padding:13px 36px; border:none; border-radius:50px;
  background:linear-gradient(135deg,var(--yellow),var(--orange));
  color:#1a1a00; font-family:'Boogaloo',cursive; font-size:1.1rem;
  cursor:pointer; transition:all .25s; box-shadow:0 4px 18px rgba(255,107,53,.3);
}
.play-again-btn:hover { transform:translateY(-2px); box-shadow:0 8px 26px rgba(255,107,53,.5); }

.hidden { display:none !important; }
</style>
</head>
<body>
<div id="toast" class="toast"></div>

<!-- Loading screen -->
<div id="loadingScreen" style="display:flex;position:fixed;inset:0;background:#0d0d1a;z-index:9999;flex-direction:column;align-items:center;justify-content:center;gap:16px;">
  <div style="font-family:'Permanent Marker',cursive;font-size:2.5rem;background:linear-gradient(135deg,#00F5FF,#784CF7,#FF3CAC);-webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text;">Story Forge</div>
  <div style="font-family:'Boogaloo',cursive;font-size:1.1rem;color:#8888aa;letter-spacing:3px;text-transform:uppercase;">Connecting to Firebase...</div>
  <div style="width:200px;height:4px;background:rgba(255,255,255,.08);border-radius:2px;overflow:hidden;">
    <div style="height:100%;background:linear-gradient(90deg,#784CF7,#FF3CAC);border-radius:2px;animation:loading-bar 1.5s ease-in-out infinite;"></div>
  </div>
  <style>@keyframes loading-bar{0%{width:0%}50%{width:80%}100%{width:100%}}</style>
</div>

<!-- Firebase load error -->
<div id="loadError" style="display:none;position:fixed;inset:0;background:#0d0d1a;z-index:9998;flex-direction:column;align-items:center;justify-content:center;gap:12px;padding:30px;text-align:center;flex-direction:column;align-items:center;justify-content:center;gap:12px;padding:30px;text-align:center;">
  <div style="font-size:3rem;">⚠️</div>
  <div style="font-family:'Boogaloo',cursive;font-size:1.6rem;color:#FF3CAC;">Connection Failed</div>
  <div style="color:#8888aa;font-size:.9rem;max-width:400px;line-height:1.6;">Could not connect to Firebase. This usually means the page is being blocked from loading external scripts.<br><br>Try opening the file <strong style="color:#F0F0FF;">directly in Chrome</strong> instead of through Google Sites, or check your internet connection.</div>
  <button onclick="location.reload()" style="margin-top:10px;padding:12px 30px;border:none;border-radius:50px;background:linear-gradient(135deg,#00F5FF,#784CF7);color:white;font-family:'Boogaloo',cursive;font-size:1.1rem;cursor:pointer;">Try Again</button>
</div>

<div class="wrapper">
  <div class="header">
    <div class="logo">Story Forge</div>
    <div class="tagline">⚡ Collaborative Show vs. Tell Writing Game ⚡</div>
  </div>

  <!-- SETUP WARNING (shown if Firebase not configured) -->
  <div id="setupScreen" class="card">
    <h2>🔥 Firebase Setup Required</h2>
    <p style="color:var(--gray);margin-bottom:20px;font-size:.9rem;">This game needs a free Firebase Realtime Database to sync players in real time. Follow these steps (5 minutes):</p>
    <div class="setup-steps">
      <div class="setup-step"><span class="step-num">1</span><span>Go to <a class="setup-link" href="https://console.firebase.google.com" target="_blank">console.firebase.google.com</a> and create a free project.</span></div>
      <div class="setup-step"><span class="step-num">2</span><span>Inside your project, click <strong>Build → Realtime Database → Create Database</strong>. Choose "Start in test mode" (allows read/write for 30 days — enough for class use).</span></div>
      <div class="setup-step"><span class="step-num">3</span><span>Go to <strong>Project Settings (gear icon) → Your apps → Add app (web)</strong>. Copy the <code>firebaseConfig</code> object shown.</span></div>
      <div class="setup-step"><span class="step-num">4</span><span>Open this HTML file in a text editor and replace the <code>firebaseConfig</code> block near the top with your values:</span></div>
    </div>
    <pre class="config-block">const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  databaseURL: "https://your-project-default-rtdb.firebaseio.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123:web:abc123"
};</pre>
    <p style="color:var(--gray);font-size:.85rem;">5. Save the file and re-open it. All students open the same file (or host it on Google Sites). They join the same Room Code to play together.</p>
  </div>

  <!-- JOIN SCREEN -->
  <div id="joinScreen">
    <div id="joinForm" class="card join-wrap">
      <div class="join-title">Join the Forge</div>
      <div class="join-sub">Enter a room code to play with your classmates</div>
      <div class="field-label">Your Name</div>
      <input class="text-input" id="nameInput" placeholder="e.g. Alex" maxlength="16" style="margin-bottom:14px;" />
      <div class="field-label">Room Code</div>
      <input class="text-input" id="roomInput" placeholder="e.g. STORM" maxlength="8" style="margin-bottom:14px;text-transform:uppercase;" oninput="this.value=this.value.toUpperCase()" />
      <div class="field-label" style="text-align:center;">Pick Your Color</div>
      <div class="color-row" id="colorRow"></div>
      <button class="join-btn" id="joinBtn" onclick="joinRoom()">Enter the Forge →</button>
      <div id="joinError" style="color:var(--pink);font-size:.85rem;margin-top:10px;display:none;"></div>
    </div>

    <!-- LOBBY -->
    <div id="lobbyCard" class="card lobby-card hidden" style="margin-top:0;">
      <div class="lobby-title">⚔️ Waiting for Writers...</div>
      <div class="lobby-room" id="lobbyRoomLabel"></div>
      <div class="players-grid" id="lobbyPlayers"></div>
      <button class="start-btn" id="startBtn" onclick="startGame()" disabled>Start Game</button>
      <div class="waiting-txt" id="waitingTxt">Need at least 2 players to start</div>
    </div>
  </div>

  <!-- GAME SCREEN -->
  <div id="gameScreen">
    <!-- Top bar -->
    <div class="top-bar">
      <div class="top-bar-section">
        <div><div class="tb-label">My Score</div><div class="tb-value" id="myScore">0</div></div>
      </div>
      <div class="round-badge" id="roundBadge">Round 1</div>
      <div class="top-bar-section" style="gap:8px;">
        <div class="phase-badge phase-write" id="phaseBadge">✍️ Writing</div>
        <div><div class="tb-label">Streak</div><div class="tb-value" id="streakVal" style="font-size:1.4rem;">0</div></div>
      </div>
    </div>

    <!-- Players strip -->
    <div class="players-strip" id="playersStrip"></div>

    <!-- Story so far -->
    <div class="story-scroll" id="storyScroll">
      <span class="story-empty">The story begins with the first winning sentence...</span>
    </div>

    <!-- Prompt -->
    <div class="prompt-card" id="promptCard">
      <div class="prompt-label">🎲 This Round's Prompt</div>
      <div class="prompt-tags" id="promptTags"></div>
      <div class="prompt-text" id="promptText"><span class="prompt-loading">Generating your story prompt...</span></div>
    </div>

    <!-- Phase timer -->
    <div class="phase-timer">
      <div><div class="pt-label">Time</div><div class="pt-digits" id="ptDigits">—</div></div>
      <div class="pt-bar-outer"><div class="pt-bar-inner ptb-safe" id="ptBar" style="width:100%"></div></div>
    </div>

    <!-- WRITE PHASE -->
    <div id="writePhase">
      <div class="write-area">
        <div class="write-label">✍️ Your Sentence — SHOW, don't tell!</div>
        <div class="write-hint">
          <strong>Remember:</strong> Don't use boring "telling" words. Instead, use <strong>vivid verbs</strong>, <strong>body language</strong>, and <strong>sensory details</strong> to make the reader feel the scene.
        </div>
        <textarea class="story-input" id="sentenceInput" placeholder="Write a sentence that continues the story... make the reader SEE and FEEL it!" oninput="updateWC()"></textarea>
        <div class="write-toolbar">
          <div class="wc">Words: <span id="wcVal">0</span></div>
          <button class="submit-sentence-btn" id="submitSentBtn" onclick="submitSentence()">⚡ Submit Sentence</button>
        </div>
      </div>
      <div id="myScoreChips" class="hidden"></div>
      <div id="waitingOthers" class="waiting-others hidden">✅ Submitted! Waiting for other writers...</div>
    </div>

    <!-- VOTE PHASE -->
    <div id="votePhase" class="vote-section hidden">
      <div class="vote-title">🗳️ Vote for the Best Sentence</div>
      <div class="vote-sub">Pick the sentence that best SHOWS the story — not just tells it.</div>
      <div class="vote-cards" id="voteCards"></div>
    </div>

    <!-- REVEAL PHASE -->
    <div id="revealPhase" class="reveal-section hidden">
      <div class="reveal-title">🏆 Round Winner!</div>
      <div class="reveal-winner-name" id="revealWinnerName"></div>
      <div class="reveal-sentence" id="revealSentence"></div>
      <div class="pts-earned" id="revealPts"></div>
      <button class="next-round-btn" id="nextRoundBtn" onclick="advanceRound()">Next Round →</button>
      <div style="font-size:.8rem;color:var(--gray);margin-top:8px;" id="hostOnlyTxt"></div>
    </div>
  </div>

  <!-- END SCREEN -->
  <div id="endScreen" class="card">
    <div class="end-title">The Story is Complete!</div>
    <div class="podium" id="podium"></div>
    <div style="font-family:'Boogaloo',cursive;font-size:1.1rem;color:var(--cyan);margin-bottom:8px;text-align:left;">📖 Your Collaborative Story</div>
    <div class="story-final" id="storyFinal"></div>
    <div style="margin-top:16px;">
      <button class="copy-story-btn" onclick="copyStory()">📋 Copy Story</button>
      <button class="play-again-btn" onclick="location.reload()">▶ Play Again</button>
    </div>
  </div>
</div><!-- wrapper -->

<script>
// ════════════════════════════════════════════════════
// CONSTANTS & HELPERS
// ════════════════════════════════════════════════════
const WRITE_TIME  = 90;  // seconds to write a sentence
const VOTE_TIME   = 45;  // seconds to vote
const REVEAL_TIME = 12;  // seconds on reveal screen
const MAX_ROUNDS  = 6;
const PLAYER_COLORS = ['#00F5FF','#FF3CAC','#06FF7A','#FFD93D'];
const PLAYER_EMOJIS = ['🦊','🐺','🦅','🐉'];
const GENRES    = ['Mystery','Fantasy','Sci-Fi','Horror','Adventure','Romance','Historical Fiction','Thriller'];
const CHARS     = ['a nervous detective','a time-traveling student','a dragon with no fire','a ghost who forgot how to haunt','a soldier returning home','a girl who collects secrets','an inventor with no hands','a king who lost his voice'];
const SETTINGS  = ['in an abandoned library','on a sinking ship','inside a burning city','at the edge of a black hole','in a village that fears the dark','beneath a blood-red moon','in the last forest on Earth','inside a dream that won\'t end'];

function shuffle(a){ return [...a].sort(()=>Math.random()-.5); }
function rand(a){ return a[Math.floor(Math.random()*a.length)]; }
function formatTime(s){ return `${Math.floor(s/60)}:${String(s%60).padStart(2,'0')}`; }

// ════════════════════════════════════════════════════
// STATE
// ════════════════════════════════════════════════════
let myId       = 'p_'+Math.random().toString(36).slice(2,8);
let myName     = '';
let myColor    = PLAYER_COLORS[0];
let myEmoji    = PLAYER_EMOJIS[0];
let roomId     = '';
let isHost     = false;
let myScore    = 0;
let myStreak   = 0;
let currentRound = 0;
let currentPhase = 'lobby';  // lobby | write | vote | reveal | end
let phaseTimer   = null;
let phaseSecsLeft= 0;
let mySubmission = null;
let myVote       = null;
let selectedColor= PLAYER_COLORS[0];
let gameListener = null;

// ════════════════════════════════════════════════════
// FIREBASE INIT (compat SDK)
// ════════════════════════════════════════════════════
let db;

// Compat SDK wrapper helpers
function rref(path){ return db.ref(path); }
function rset(path, val){ return db.ref(path).set(val); }
function rget(path){ return db.ref(path).once('value'); }
function ronValue(path, cb){ db.ref(path).on('value', snap => cb(snap)); }
function rupdate(path, val){ return db.ref(path).update(val); }
function rremove(path){ return db.ref(path).remove(); }

// Show loading state immediately on DOM ready
document.addEventListener('DOMContentLoaded', ()=>{
  document.getElementById('loadingScreen').style.display = 'block';
});

// Firebase ready — fired after both compat scripts load and initializeApp succeeds
window.addEventListener('firebaseReady', ()=>{
  db = window._firebaseDB;
  document.getElementById('loadingScreen').style.display = 'none';
  if(!db){ showSetup(); return; }
  showJoin();
});

// Timeout fallback — if Firebase never loads after 8s, show error
setTimeout(()=>{
  if(!window._firebaseDB){
    document.getElementById('loadingScreen').style.display = 'none';
    document.getElementById('loadError').style.display = 'block';
  }
}, 8000);

function showSetup(){
  document.getElementById('setupScreen').style.display='block';
}
function showJoin(){
  document.getElementById('joinScreen').style.display='block';
  buildColorRow();
}

// ════════════════════════════════════════════════════
// COLOR PICKER
// ════════════════════════════════════════════════════
function buildColorRow(){
  const row = document.getElementById('colorRow');
  row.innerHTML='';
  PLAYER_COLORS.forEach((c,i)=>{
    const d=document.createElement('div');
    d.className='color-dot'+(i===0?' sel':'');
    d.style.background=c;
    d.onclick=()=>{
      row.querySelectorAll('.color-dot').forEach(x=>x.classList.remove('sel'));
      d.classList.add('sel');
      selectedColor=c;
      myEmoji=PLAYER_EMOJIS[i];
    };
    row.appendChild(d);
  });
}

// ════════════════════════════════════════════════════
// JOIN / LOBBY
// ════════════════════════════════════════════════════
async function joinRoom(){
  const name = document.getElementById('nameInput').value.trim();
  const room = document.getElementById('roomInput').value.trim().toUpperCase();
  if(!name||!room){ showJoinError('Enter your name and a room code.'); return; }
  myName=name; myColor=selectedColor; roomId=room;
  document.getElementById('joinBtn').disabled=true;

  const playersRef = rref(`rooms/${roomId}/players`);
  const snap = await rget(playersRef);
  const existing = snap.val()||{};
  const playerCount = Object.keys(existing).length;

  if(playerCount===0) isHost=true;
  if(playerCount>=4){ showJoinError('Room is full (max 4 players).'); document.getElementById('joinBtn').disabled=false; return; }

  const existingGame = (await rget(rref(`rooms/${roomId}/phase`))).val();
  if(existingGame && existingGame!=='lobby'){ showJoinError('Game already in progress.'); document.getElementById('joinBtn').disabled=false; return; }

  await rset(rref(`rooms/${roomId}/players/${myId}`),{
    id:myId, name:myName, color:myColor, emoji:myEmoji,
    score:0, streak:0, isHost:isHost
  });
  if(isHost) await rset(rref(`rooms/${roomId}/phase`),'lobby');

  document.getElementById('joinForm').classList.add('hidden');
  document.getElementById('lobbyCard').classList.remove('hidden');
  document.getElementById('lobbyRoomLabel').textContent=`Room: ${roomId}`;

  listenLobby();
}
function showJoinError(msg){
  const el=document.getElementById('joinError');
  el.textContent=msg; el.style.display='block';
  document.getElementById('joinBtn').disabled=false;
}

function listenLobby(){
  ronValue(rref(`rooms/${roomId}`),(snap)=>{
    const data=snap.val(); if(!data) return;
    const players=Object.values(data.players||{});
    renderLobbyPlayers(players);
    if(data.phase==='write'||data.phase==='vote'||data.phase==='reveal') enterGame(data);
  });
}

function renderLobbyPlayers(players){
  const grid=document.getElementById('lobbyPlayers');
  grid.innerHTML='';
  for(let i=0;i<4;i++){
    const p=players[i];
    const slot=document.createElement('div');
    slot.className='player-slot'+(p?' filled':'');
    if(p) slot.style.borderColor=p.color;
    slot.innerHTML=p
      ?`<div class="player-avatar">${p.emoji}</div><div class="player-name" style="color:${p.color}">${p.name}</div><div class="player-status">Ready</div>`
      :`<div class="player-avatar" style="opacity:.3">❓</div><div class="player-status">Waiting...</div>`;
    grid.appendChild(slot);
  }
  const btn=document.getElementById('startBtn');
  const txt=document.getElementById('waitingTxt');
  const canStart=players.length>=2;
  btn.disabled=!isHost||!canStart;
  if(!isHost) btn.style.display='none';
  txt.textContent=isHost?(canStart?'Ready — start when everyone is here!':'Need at least 2 players to start'):'Waiting for host to start...';
}

async function startGame(){
  if(!isHost) return;
  await rset(rref(`rooms/${roomId}/currentRound`),1);
  await rset(rref(`rooms/${roomId}/phase`),'write');
  await rset(rref(`rooms/${roomId}/story`),[]);
  await rset(rref(`rooms/${roomId}/round1`),{submissions:{},votes:{},prompt:null});
  await generateAndStorePrompt(1);
  await rset(rref(`rooms/${roomId}/phaseStartedAt`),Date.now());
}

// ════════════════════════════════════════════════════
// ENTER GAME — set up full game listener
// ════════════════════════════════════════════════════
function enterGame(initialData){
  document.getElementById('joinScreen').style.display='none';
  document.getElementById('gameScreen').style.display='block';
  if(gameListener) return; // already listening

  ronValue(`rooms/${roomId}`,(snap)=>{
    const data=snap.val(); if(!data) return;
    renderGame(data);
  });
  gameListener = true;
}

function renderGame(data){
  const players  = Object.values(data.players||{});
  const phase    = data.phase||'lobby';
  const roundNum = data.currentRound||1;
  const roundKey = `round${roundNum}`;
  const roundData= data[roundKey]||{};
  const story    = data.story||[];

  currentRound = roundNum;
  currentPhase = phase;

  // Update top bar
  document.getElementById('roundBadge').textContent=`Round ${roundNum} of ${MAX_ROUNDS}`;
  document.getElementById('myScore').textContent=myScore;
  document.getElementById('streakVal').textContent=myStreak;

  // Players strip
  renderPlayersStrip(players, roundData, phase);

  // Story scroll
  renderStory(story);

  // Prompt
  if(roundData.prompt) renderPrompt(roundData.prompt);

  // Phase badge
  const pb=document.getElementById('phaseBadge');
  pb.className='phase-badge';
  if(phase==='write'){ pb.classList.add('phase-write'); pb.textContent='✍️ Writing'; }
  if(phase==='vote') { pb.classList.add('phase-vote');  pb.textContent='🗳️ Voting'; }
  if(phase==='reveal'){ pb.classList.add('phase-reveal');pb.textContent='🏆 Reveal'; }

  // Phase sections
  document.getElementById('writePhase').classList.toggle('hidden', phase!=='write');
  document.getElementById('votePhase').classList.toggle('hidden',  phase!=='vote');
  document.getElementById('revealPhase').classList.toggle('hidden',phase!=='reveal');

  // Phase-specific rendering
  if(phase==='write')  renderWritePhase(roundData, data.phaseStartedAt);
  if(phase==='vote')   renderVotePhase(roundData, players, data.phaseStartedAt);
  if(phase==='reveal') renderRevealPhase(roundData, players, story);
  if(phase==='end')    renderEndScreen(players, story);
}

// ════════════════════════════════════════════════════
// PLAYERS STRIP
// ════════════════════════════════════════════════════
function renderPlayersStrip(players, roundData, phase){
  const strip=document.getElementById('playersStrip');
  strip.innerHTML='';
  const subs=roundData.submissions||{};
  players.forEach(p=>{
    const card=document.createElement('div');
    card.className='ps-card'+(p.id===myId?' me':'');
    card.style.borderColor=p.id===myId?p.color:'rgba(255,255,255,.08)';
    const hasSubmitted=!!subs[p.id];
    card.innerHTML=`
      <div class="ps-name"><span class="ps-dot" style="background:${p.color}"></span>${p.name}${p.id===myId?' (you)':''}</div>
      <div class="ps-pts" style="color:${p.color}">${p.score||0}</div>
      ${phase==='write'&&hasSubmitted?'<div class="ps-tick">✅ submitted</div>':''}
    `;
    strip.appendChild(card);
  });
}

// ════════════════════════════════════════════════════
// STORY SCROLL
// ════════════════════════════════════════════════════
function renderStory(story){
  const el=document.getElementById('storyScroll');
  if(!story||story.length===0){
    el.innerHTML='<span class="story-empty">The story begins with the first winning sentence...</span>';
    return;
  }
  el.innerHTML=story.map((s,i)=>`<span class="story-sentence" style="color:${s.color||'var(--white)'}">${s.text} </span>`).join('');
  el.scrollTop=el.scrollHeight;
}

// ════════════════════════════════════════════════════
// PROMPT
// ════════════════════════════════════════════════════
function renderPrompt(prompt){
  const tagsEl=document.getElementById('promptTags');
  const textEl=document.getElementById('promptText');
  tagsEl.innerHTML=`
    <span class="prompt-tag tag-genre">${prompt.genre}</span>
    <span class="prompt-tag tag-char">${prompt.character}</span>
    <span class="prompt-tag tag-setting">${prompt.setting}</span>
  `;
  textEl.textContent=prompt.text||'Continue the story...';
}

async function generateAndStorePrompt(roundNum){
  const genre   =rand(GENRES);
  const character=rand(CHARS);
  const setting =rand(SETTINGS);

  // Generate via Anthropic API
  let promptText='Continue the story with one vivid, showing sentence.';
  try {
    const resp = await fetch('https://api.anthropic.com/v1/messages',{
      method:'POST',
      headers:{'Content-Type':'application/json'},
      body:JSON.stringify({
        model:'claude-sonnet-4-20250514',
        max_tokens:120,
        system:`You generate one-sentence story continuation prompts for a middle school creative writing game. 
The genre is ${genre}, the main character is ${character}, the setting is ${setting}.
${roundNum===1?'This is the OPENING sentence — set the scene.':`This is round ${roundNum} — the story must escalate.`}
Return ONLY a single, vivid, concrete writing prompt sentence of 10–20 words. 
Do NOT write the story sentence yourself — write an instruction for students to follow.
Example format: "Write what the character hears just before the lights go out."`,
        messages:[{role:'user',content:'Generate the prompt now.'}]
      })
    });
    const d=await resp.json();
    if(d.content?.[0]?.text) promptText=d.content[0].text.trim();
  } catch(e){ /* fallback below */ }

  // Fallback prompts if API fails
  if(promptText==='Continue the story with one vivid, showing sentence.'){
    const fallbacks=[
      `Describe what ${character} sees when they first arrive ${setting}.`,
      `Show ${character}'s reaction to an unexpected sound ${setting}.`,
      `Reveal what ${character} discovers hidden ${setting}.`,
      `Describe the moment everything changes for ${character}.`,
      `Show how ${character} tries to escape an impossible situation.`,
      `Reveal the secret that ${character} has been hiding.`,
    ];
    promptText=rand(fallbacks);
  }

  await rset(rref(`rooms/${roomId}/round${roundNum}/prompt`),{genre,character,setting,text:promptText});
}

// ════════════════════════════════════════════════════
// WRITE PHASE
// ════════════════════════════════════════════════════
let writePhaseSeenAt = null;

function renderWritePhase(roundData, phaseStartedAt){
  const subs=roundData.submissions||{};
  const myHasSubmitted=!!subs[myId];

  document.getElementById('sentenceInput').disabled=myHasSubmitted;
  document.getElementById('submitSentBtn').disabled=myHasSubmitted;

  if(myHasSubmitted){
    document.getElementById('waitingOthers').classList.remove('hidden');
    document.getElementById('submitSentBtn').classList.add('hidden');
  }

  // Phase timer (host drives transitions via Firebase, timer is visual)
  if(phaseStartedAt && writePhaseSeenAt!==phaseStartedAt){
    writePhaseSeenAt=phaseStartedAt;
    startPhaseTimer(WRITE_TIME, phaseStartedAt, ()=>{
      // When write timer expires, host advances
      if(isHost) advanceToVote(roundData);
    });
  }
}

async function submitSentence(){
  const text=document.getElementById('sentenceInput').value.trim();
  if(text.length<8){ showToast('Write a complete sentence first!','wrong'); return; }

  const score=scoreSentence(text);
  mySubmission={text, score:score.total, criteria:score.criteria};

  // Show score chips immediately
  showScoreChips(score);

  document.getElementById('submitSentBtn').disabled=true;
  document.getElementById('submitSentBtn').classList.add('hidden');
  document.getElementById('sentenceInput').disabled=true;
  document.getElementById('waitingOthers').classList.remove('hidden');

  await rset(rref(`rooms/${roomId}/round${currentRound}/submissions/${myId}`),{
    playerId:myId, text, score:score.total
  });

  // Check if all players submitted → host advances early
  if(isHost){
    const snap=await rget(rref(`rooms/${roomId}/players`));
    const players=Object.values(snap.val()||{});
    const subsSnap=await rget(rref(`rooms/${roomId}/round${currentRound}/submissions`));
    const subs=Object.keys(subsSnap.val()||{});
    if(subs.length>=players.length) advanceToVote(subsSnap.val());
  }
}

// ── SHOW VS TELL SCORING ──
function scoreSentence(text){
  const words=text.toLowerCase().split(/\s+/);
  const wc=words.length;
  const tellWords=['happy','sad','angry','scared','excited','nervous','brave','kind','mean','nice','good','bad','beautiful','ugly','strong','weak','big','small'];
  const noTell=!tellWords.some(w=>words.includes(w));
  const hasAction=wc>=7;
  const hasPowerVerb=/(\w+ed|\w+ing|slammed|raced|trembl|whirled|lurched|gasped|clutched|sprinted|froze|burst|staggered|plunged|soared|crawled|shuddered|wrenched|twisted|collapsed|lunged)/i.test(text);
  const hasSensory=/(sweat|blood|cold|heat|dark|light|sound|silence|shadow|glare|smoke|thunder|wind|rain|chill|pulse|breath|heartbeat|eyes|hands|teeth|skin|voice|ear|nose|smell|taste|touch|feel)/i.test(text);
  const criteria=[
    {label:'✅ Full sentence',        met:hasAction},
    {label:'✅ No "telling" words',   met:noTell},
    {label:'✅ Power verb',           met:hasPowerVerb},
    {label:'✅ Sensory detail',       met:hasSensory},
  ];
  const met=criteria.filter(c=>c.met).length;
  const total= met===4?50 : met>=2?30 : met>=1?10 : 0;
  return {criteria, total, met};
}

function showScoreChips(score){
  const el=document.getElementById('myScoreChips');
  el.classList.remove('hidden');
  el.innerHTML=`
    <div class="score-chips">${score.criteria.map(c=>`<span class="score-chip ${c.met?'chip-met':'chip-miss'}">${c.met?c.label:c.label.replace('✅','❌')}</span>`).join('')}</div>
    <div class="my-score-banner">Your sentence scored <strong>${score.total} pts</strong> — votes could earn more!</div>
  `;
}

// ════════════════════════════════════════════════════
// VOTE PHASE
// ════════════════════════════════════════════════════
let votePhaseSeenAt=null;

async function advanceToVote(submissions){
  if(!isHost) return;
  clearPhaseTimer();
  await rset(rref(`rooms/${roomId}/phase`),'vote');
  await rset(rref(`rooms/${roomId}/phaseStartedAt`),Date.now());
}

function renderVotePhase(roundData, players, phaseStartedAt){
  const subs=Object.values(roundData.submissions||{});
  const votes=roundData.votes||{};
  const myHasVoted=!!votes[myId];

  const container=document.getElementById('voteCards');
  container.innerHTML='';

  if(subs.length===0){
    container.innerHTML='<div style="color:var(--gray);font-style:italic;">No sentences submitted this round.</div>';
    return;
  }

  subs.forEach(sub=>{
    const player=players.find(p=>p.id===sub.playerId)||{name:'Unknown',color:'#fff'};
    const isOwn=sub.playerId===myId;
    const voteCount=Object.values(votes).filter(v=>v===sub.playerId).length;
    const card=document.createElement('div');
    card.className='vote-card'+(isOwn?' own-card':'')+(myHasVoted&&votes[myId]===sub.playerId?' selected':'');
    card.innerHTML=`
      <div class="vc-header">
        <div class="vc-author" style="color:${player.color}">${isOwn?'Your sentence':player.name}</div>
        <div class="vc-score-badge">⚡ ${sub.score} pts</div>
      </div>
      <div class="vc-text">${sub.text}</div>
      ${myHasVoted?`<div class="vc-vote-count">${voteCount>0?`👍 ${voteCount} vote${voteCount!==1?'s':''}`:'No votes yet'}</div>`:''}
      ${!isOwn&&!myHasVoted?`<button class="vote-btn" onclick="castVote('${sub.playerId}','${sub.text}',${sub.score})">👍 Vote</button>`:''}
      ${isOwn&&!myHasVoted?'<div class="already-voted">You can\'t vote for your own sentence</div>':''}
    `;
    container.appendChild(card);
  });

  // Timer
  if(phaseStartedAt && votePhaseSeenAt!==phaseStartedAt){
    votePhaseSeenAt=phaseStartedAt;
    startPhaseTimer(VOTE_TIME, phaseStartedAt, ()=>{
      if(isHost) advanceToReveal(roundData);
    });
  }
}

async function castVote(playerId, text, score){
  if(myVote) return;
  myVote=playerId;
  await rset(rref(`rooms/${roomId}/round${currentRound}/votes/${myId}`),playerId);
  showToast('🗳️ Vote cast!','info');
  // Check if all voted
  if(isHost){
    const snap=await rget(rref(`rooms/${roomId}/players`));
    const players=Object.values(snap.val()||{});
    const votesSnap=await rget(rref(`rooms/${roomId}/round${currentRound}/votes`));
    const voteCount=Object.keys(votesSnap.val()||{}).length;
    // Advance if everyone (except those who couldn't vote for themselves) has voted
    if(voteCount>=players.length-1) advanceToReveal(votesSnap.val());
  }
}

// ════════════════════════════════════════════════════
// REVEAL PHASE
// ════════════════════════════════════════════════════
let revealPhaseSeenAt=null;
let revealHandled=false;

async function advanceToReveal(roundData){
  if(!isHost) return;
  clearPhaseTimer();
  // Tally votes
  const votes=roundData.votes||typeof roundData==='object'?roundData:{};
  const tally={};
  Object.values(votes).forEach(pid=>{ tally[pid]=(tally[pid]||0)+1; });
  let winnerId=null; let maxVotes=0;
  Object.entries(tally).forEach(([pid,v])=>{ if(v>maxVotes){ maxVotes=v; winnerId=pid; }});
  // Tie-break: pick highest auto-score
  if(!winnerId){
    const subs=Object.values((await rget(rref(`rooms/${roomId}/round${currentRound}/submissions`))).val()||{});
    subs.sort((a,b)=>b.score-a.score);
    winnerId=subs[0]?.playerId||null;
  }
  await rset(rref(`rooms/${roomId}/round${currentRound}/winner`),winnerId);
  await rset(rref(`rooms/${roomId}/phase`),'reveal');
  await rset(rref(`rooms/${roomId}/phaseStartedAt`),Date.now());
}

async function renderRevealPhase(roundData, players, story){
  const winnerId=roundData.winner;
  const subs=roundData.submissions||{};
  const winnerSub=subs[winnerId];
  if(!winnerSub) return;

  const winnerPlayer=players.find(p=>p.id===winnerId)||{name:'Unknown',color:'var(--yellow)'};
  document.getElementById('revealWinnerName').innerHTML=`<span style="color:${winnerPlayer.color}">${winnerPlayer.name}</span>'s sentence wins!`;
  document.getElementById('revealSentence').textContent=`"${winnerSub.text}"`;

  // Calculate total pts (auto-score + 20 bonus for winning vote)
  const totalPts=winnerSub.score+20;
  document.getElementById('revealPts').textContent=`+${winnerSub.score} writing pts + 20 vote bonus = ${totalPts} pts!`;

  // Apply score to winner in Firebase (once per reveal, host does it)
  if(isHost && !revealHandled){
    revealHandled=true;
    const winPlayer=players.find(p=>p.id===winnerId);
    const newScore=(winPlayer?.score||0)+totalPts;
    await rset(rref(`rooms/${roomId}/players/${winnerId}/score`),newScore);

    // Add to story
    const newStory=[...(story||[]),{text:winnerSub.text,color:winnerPlayer.color,author:winnerPlayer.name}];
    await rset(rref(`rooms/${roomId}/story`),newStory);
  }

  if(winnerId===myId){
    myScore+=totalPts; myStreak++;
    document.getElementById('myScore').textContent=myScore;
    document.getElementById('streakVal').textContent=myStreak;
    launchConfetti();
    showToast(`🏆 You won! +${totalPts} pts`,'bonus');
  } else {
    myStreak=0; document.getElementById('streakVal').textContent=0;
  }

  const nextBtn=document.getElementById('nextRoundBtn');
  const hostTxt=document.getElementById('hostOnlyTxt');
  if(isHost){
    nextBtn.classList.remove('hidden');
    hostTxt.textContent='(Advance when everyone is ready)';
  } else {
    nextBtn.classList.add('hidden');
    hostTxt.textContent='Waiting for host to advance...';
  }

  // Reveal timer
  if(isHost){
    startPhaseTimer(REVEAL_TIME, Date.now(), ()=>{ advanceRound(); });
  }
}

async function advanceRound(){
  if(!isHost) return;
  clearPhaseTimer();
  revealHandled=false;
  myVote=null; mySubmission=null;
  writePhaseSeenAt=null; votePhaseSeenAt=null; revealPhaseSeenAt=null;
  document.getElementById('sentenceInput').value='';
  document.getElementById('myScoreChips').classList.add('hidden');
  document.getElementById('waitingOthers').classList.add('hidden');
  document.getElementById('submitSentBtn').classList.remove('hidden');

  const nextRound=currentRound+1;
  if(nextRound>MAX_ROUNDS){
    await rset(rref(`rooms/${roomId}/phase`),'end');
    return;
  }
  await rset(rref(`rooms/${roomId}/currentRound`),nextRound);
  await rset(rref(`rooms/${roomId}/round${nextRound}`),{submissions:{},votes:{},prompt:null});
  await generateAndStorePrompt(nextRound);
  await rset(rref(`rooms/${roomId}/phase`),'write');
  await rset(rref(`rooms/${roomId}/phaseStartedAt`),Date.now());
}

// ════════════════════════════════════════════════════
// END SCREEN
// ════════════════════════════════════════════════════
function renderEndScreen(players, story){
  document.getElementById('gameScreen').style.display='none';
  document.getElementById('endScreen').style.display='block';
  clearPhaseTimer();

  const sorted=[...players].sort((a,b)=>(b.score||0)-(a.score||0));
  const podiumEl=document.getElementById('podium');
  const podiumOrder=sorted.length>=3?[sorted[1],sorted[0],sorted[2]]:sorted;
  const barClasses=['p2-bar','p1-bar','p3-bar'];
  const medals=['🥈','🥇','🥉'];
  podiumEl.innerHTML=podiumOrder.map((p,i)=>`
    <div class="podium-place">
      <div class="podium-bar ${barClasses[i]}">${medals[i]}</div>
      <div class="podium-name" style="color:${p.color}">${p.name}</div>
      <div class="podium-pts">${p.score||0} pts</div>
    </div>
  `).join('');

  const storyEl=document.getElementById('storyFinal');
  storyEl.innerHTML=(story||[]).map(s=>`<span style="color:${s.color||'var(--white)'}">${s.text} </span>`).join('');
  launchConfetti();
}

async function copyStory(){
  const story=(await rget(`rooms/${roomId}/story`)).val()||[];
  const text=story.map(s=>s.text).join(' ');
  navigator.clipboard.writeText(text).then(()=>showToast('📋 Story copied!','bonus'));
}

// ════════════════════════════════════════════════════
// PHASE TIMER
// ════════════════════════════════════════════════════
function startPhaseTimer(totalSecs, startedAt, onExpire){
  clearPhaseTimer();
  const elapsed=Math.floor((Date.now()-startedAt)/1000);
  let secsLeft=Math.max(0,totalSecs-elapsed);
  renderPT(secsLeft,totalSecs);
  if(secsLeft<=0){ onExpire(); return; }
  phaseTimer=setInterval(()=>{
    secsLeft--;
    renderPT(secsLeft,totalSecs);
    if(secsLeft<=0){ clearPhaseTimer(); onExpire(); }
  },1000);
}
function renderPT(s,total){
  document.getElementById('ptDigits').textContent=formatTime(s);
  const pct=(s/total)*100;
  const bar=document.getElementById('ptBar');
  bar.style.width=pct+'%';
  bar.className='pt-bar-inner '+(pct>50?'ptb-safe':pct>25?'ptb-warn':'ptb-danger');
}
function clearPhaseTimer(){ if(phaseTimer){ clearInterval(phaseTimer); phaseTimer=null; } }

// ════════════════════════════════════════════════════
// UTILS
// ════════════════════════════════════════════════════
function updateWC(){
  const w=document.getElementById('sentenceInput').value.trim().split(/\s+/).filter(Boolean);
  document.getElementById('wcVal').textContent=w.length;
}
function showToast(msg,type){
  const t=document.getElementById('toast');
  t.textContent=msg; t.className=`toast ${type} show`;
  setTimeout(()=>t.classList.remove('show'),2200);
}
function launchConfetti(){
  const cols=['#FFD93D','#FF6B35','#FF3CAC','#00F5FF','#06FF7A','#784CF7'];
  for(let i=0;i<30;i++){
    const el=document.createElement('div');
    el.className='confetti-piece';
    el.style.cssText=`left:${Math.random()*100}vw;background:${cols[Math.floor(Math.random()*cols.length)]};width:${5+Math.random()*8}px;height:${5+Math.random()*8}px;border-radius:${Math.random()>.5?'50%':'2px'};animation-duration:${1.5+Math.random()*1.5}s;animation-delay:${Math.random()*.5}s;`;
    document.body.appendChild(el);
    setTimeout(()=>el.remove(),3000);
  }
}

// Cleanup on leave
window.addEventListener('beforeunload',()=>{
  if(db&&roomId&&myId) rremove(`rooms/${roomId}/players/${myId}`);
});
</script>
</body>
</html>
