<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pete-tide — lo-fi chill rap</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,500;9..144,600;9..144,700&family=Manrope:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;700&display=swap');

  :root{
    --bg: #16121c;
    --surface: #1e1725;
    --surface-hi: #291f31;
    --line: #322639;
    --accent: #e0a458;
    --accent-dim: #a97c48;
    --secondary: #8a7ca8;
    --text-hi: #f2e9dc;
    --text-lo: #948aa0;
    --text-faint: #5f5669;
    --danger: #c96b6b;
  }

  *{ box-sizing:border-box; margin:0; padding:0; }
  html,body{ height:100%; }
  body{
    background:var(--bg);
    color:var(--text-hi);
    font-family:'Manrope',sans-serif;
    overflow:hidden;
    position:relative;
  }

  /* film-grain / tape hiss texture */
  body::before{
    content:"";
    position:fixed; inset:0;
    pointer-events:none;
    z-index:999;
    opacity:0.05;
    mix-blend-mode:overlay;
    background-image:url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='120' height='120'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2' stitchTiles='stitch'/></filter><rect width='100%25' height='100%25' filter='url(%23n)'/></svg>");
  }

  #app{ display:flex; flex-direction:column; height:100vh; }
  .shell{ display:flex; flex:1; min-height:0; }

  /* ---------- sidebar ---------- */
  .sidebar{
    width:250px; flex-shrink:0;
    background:var(--surface);
    border-right:1px solid var(--line);
    display:flex; flex-direction:column;
    padding:22px 16px;
  }
  .brand{ display:flex; align-items:center; gap:10px; padding:0 6px 22px; }
  .reel{ width:26px; height:26px; }
  .brand-name{ font-family:'Fraunces',serif; font-weight:600; font-size:20px; letter-spacing:0.5px; }
  .brand-name span{ color:var(--accent); }

  .nav-group{ margin-bottom:22px; }
  .nav-label{ font-size:10.5px; letter-spacing:1.5px; text-transform:uppercase; color:var(--text-faint); padding:0 10px 8px; font-weight:700; }
  .nav-item{
    display:flex; align-items:center; gap:10px;
    padding:9px 10px; border-radius:8px; cursor:pointer;
    color:var(--text-lo); font-size:14px; font-weight:600;
    transition:background .15s, color .15s;
  }
  .nav-item svg{ width:16px; height:16px; opacity:.85; flex-shrink:0; }
  .nav-item:hover{ background:var(--surface-hi); color:var(--text-hi); }
  .nav-item.active{ background:var(--surface-hi); color:var(--accent); }
  .nav-item .count{ margin-left:auto; font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--text-faint); }

  .library-list{ overflow-y:auto; flex:1; }
  .lib-item{
    display:flex; align-items:center; gap:10px;
    padding:8px 10px; border-radius:8px; cursor:pointer;
  }
  .lib-item:hover, .lib-item.active{ background:var(--surface-hi); }
  .lib-swatch{ width:34px; height:34px; border-radius:6px; flex-shrink:0; }
  .lib-meta .t{ font-size:13px; font-weight:700; color:var(--text-hi); line-height:1.2; }
  .lib-meta .s{ font-size:11px; color:var(--text-faint); }

  /* ---------- main ---------- */
  .main{ flex:1; min-width:0; display:flex; flex-direction:column; }
  .topbar{
    display:flex; align-items:center; gap:16px;
    padding:18px 32px; border-bottom:1px solid var(--line);
    background:linear-gradient(180deg, rgba(30,23,37,.9), rgba(22,18,28,.4));
  }
  .search-wrap{ position:relative; flex:1; max-width:420px; }
  .search-wrap svg{ position:absolute; left:12px; top:50%; transform:translateY(-50%); width:15px; height:15px; color:var(--text-faint); }
  .search-wrap input{
    width:100%; background:var(--surface); border:1px solid var(--line);
    border-radius:20px; padding:9px 14px 9px 34px; color:var(--text-hi);
    font-family:'Manrope',sans-serif; font-size:13.5px; outline:none;
  }
  .search-wrap input:focus{ border-color:var(--accent-dim); }
  .search-wrap input::placeholder{ color:var(--text-faint); }
  .topbar-tag{ margin-left:auto; font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--text-faint); letter-spacing:0.5px; }

  .content{ flex:1; overflow-y:auto; padding:28px 32px 40px; }
  .section-title{ font-family:'Fraunces',serif; font-size:26px; font-weight:600; margin-bottom:4px; }
  .section-sub{ color:var(--text-lo); font-size:13.5px; margin-bottom:22px; }

  .grid{ display:grid; grid-template-columns:repeat(auto-fill,minmax(190px,1fr)); gap:20px; }
  .card{ cursor:pointer; }
  .cover{
    position:relative; aspect-ratio:1/1; border-radius:10px; overflow:hidden;
    display:flex; flex-direction:column; justify-content:space-between;
    padding:14px; box-shadow:0 8px 20px rgba(0,0,0,.35);
  }
  .cover::after{
    content:""; position:absolute; inset:0; opacity:.12; mix-blend-mode:overlay;
    background-image:url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='90' height='90'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2'/></filter><rect width='100%25' height='100%25' filter='url(%23n)'/></svg>");
  }
  .cover-top{ display:flex; justify-content:space-between; align-items:flex-start; font-family:'JetBrains Mono',monospace; font-size:10px; letter-spacing:1px; color:rgba(242,233,220,.75); z-index:1; }
  .cover-title{ font-family:'Fraunces',serif; font-weight:600; font-size:17px; color:#faf5ec; line-height:1.15; z-index:1; text-shadow:0 2px 8px rgba(0,0,0,.3); }
  .play-fab{
    position:absolute; right:12px; bottom:12px; width:38px; height:38px; border-radius:50%;
    background:var(--accent); color:#20160c; display:flex; align-items:center; justify-content:center;
    opacity:0; transform:translateY(6px); transition:opacity .18s, transform .18s;
    box-shadow:0 6px 16px rgba(0,0,0,.4); z-index:2; border:none; cursor:pointer;
  }
  .card:hover .play-fab{ opacity:1; transform:translateY(0); }
  .play-fab svg{ width:15px; height:15px; }
  .card-meta{ padding-top:10px; }
  .card-meta .t{ font-size:14px; font-weight:700; }
  .card-meta .s{ font-size:12.5px; color:var(--text-faint); margin-top:2px; }

  /* detail / tracklist */
  .detail-header{ display:flex; gap:24px; align-items:flex-end; margin-bottom:26px; }
  .detail-cover{ width:170px; height:170px; border-radius:10px; flex-shrink:0; position:relative; overflow:hidden; box-shadow:0 10px 26px rgba(0,0,0,.4); }
  .detail-cover .cover-title{ font-size:20px; position:absolute; bottom:14px; left:14px; right:14px; }
  .detail-eyebrow{ font-family:'JetBrains Mono',monospace; font-size:11px; letter-spacing:2px; color:var(--accent); text-transform:uppercase; margin-bottom:8px; }
  .detail-title{ font-family:'Fraunces',serif; font-size:38px; font-weight:700; line-height:1.05; }
  .detail-sub{ color:var(--text-lo); font-size:13.5px; margin-top:10px; max-width:520px; }
  .detail-actions{ display:flex; align-items:center; gap:14px; margin-top:16px; }
  .btn-play-all{
    display:flex; align-items:center; gap:8px; background:var(--accent); color:#20160c;
    border:none; border-radius:22px; padding:10px 20px; font-weight:800; font-size:13.5px;
    cursor:pointer; font-family:'Manrope',sans-serif;
  }
  .btn-play-all svg{ width:14px; height:14px; }

  table.tracklist{ width:100%; border-collapse:collapse; }
  .tracklist thead th{
    text-align:left; font-size:10.5px; letter-spacing:1.2px; text-transform:uppercase;
    color:var(--text-faint); border-bottom:1px solid var(--line); padding:0 12px 10px; font-weight:700;
  }
  .tracklist tbody tr{ cursor:pointer; border-radius:6px; }
  .tracklist tbody tr:hover{ background:var(--surface-hi); }
  .tracklist tbody tr.playing .t-title{ color:var(--accent); }
  .tracklist td{ padding:10px 12px; font-size:13.5px; color:var(--text-hi); vertical-align:middle; }
  .t-idx{ width:34px; color:var(--text-faint); font-family:'JetBrains Mono',monospace; font-size:12px; }
  .t-title{ font-weight:700; }
  .t-artist{ color:var(--text-faint); font-size:12px; margin-top:1px; }
  .t-dur{ font-family:'JetBrains Mono',monospace; color:var(--text-faint); font-size:12px; text-align:right; width:60px; }
  .t-like{ width:34px; text-align:center; }
  .t-like svg{ width:15px; height:15px; color:var(--text-faint); cursor:pointer; }
  .t-like svg.liked{ color:var(--accent); fill:var(--accent); }

  .eq{ display:flex; align-items:flex-end; gap:2px; width:14px; height:14px; }
  .eq span{ width:3px; background:var(--accent); border-radius:1px; animation:eqbar 0.9s ease-in-out infinite; }
  .eq span:nth-child(2){ animation-delay:.2s; }
  .eq span:nth-child(3){ animation-delay:.4s; }
  @keyframes eqbar{ 0%,100%{ height:3px; } 50%{ height:13px; } }

  .empty{ padding:60px 0; text-align:center; color:var(--text-faint); }
  .empty svg{ width:34px; height:34px; margin-bottom:12px; opacity:.5; }

  /* ---------- now playing bar ---------- */
  .now-bar{
    height:82px; flex-shrink:0; border-top:1px solid var(--line);
    background:var(--surface); display:flex; align-items:center;
    padding:0 18px; gap:18px; position:relative;
  }
  .np-track{ display:flex; align-items:center; gap:12px; width:250px; flex-shrink:0; }
  .np-cover{
    width:52px; height:52px; border-radius:8px; position:relative; flex-shrink:0;
    display:flex; align-items:center; justify-content:center; overflow:hidden;
  }
  .reel-spin{ width:26px; height:26px; animation:spin 3s linear infinite; animation-play-state:paused; opacity:.9; }
  .np-cover.playing .reel-spin{ animation-play-state:running; }
  @keyframes spin{ to{ transform:rotate(360deg); } }
  .np-meta .t{ font-size:13.5px; font-weight:700; line-height:1.2; }
  .np-meta .s{ font-size:11.5px; color:var(--text-faint); }
  .np-like svg{ width:15px; height:15px; color:var(--text-faint); cursor:pointer; flex-shrink:0; }
  .np-like svg.liked{ color:var(--accent); fill:var(--accent); }

  .np-center{ flex:1; display:flex; flex-direction:column; align-items:center; gap:6px; min-width:0; }
  .transport{ display:flex; align-items:center; gap:18px; }
  .transport button{ background:none; border:none; color:var(--text-lo); cursor:pointer; display:flex; padding:4px; }
  .transport button:hover{ color:var(--text-hi); }
  .transport button.active{ color:var(--accent); }
  .transport svg{ width:16px; height:16px; }
  .play-toggle{
    width:34px; height:34px; border-radius:50%; background:var(--text-hi);
    display:flex; align-items:center; justify-content:center; color:var(--bg) !important;
  }
  .play-toggle svg{ width:14px; height:14px; }

  .progress-row{ display:flex; align-items:center; gap:10px; width:100%; max-width:520px; }
  .tape-counter{
    font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--accent);
    background:#100d15; border:1px solid var(--line); border-radius:4px; padding:2px 6px;
    letter-spacing:1px; min-width:44px; text-align:center;
  }
  .progress-track{ flex:1; height:4px; background:var(--line); border-radius:2px; position:relative; cursor:pointer; }
  .progress-fill{ position:absolute; left:0; top:0; height:100%; background:var(--accent); border-radius:2px; width:0%; }

  .np-right{ width:190px; flex-shrink:0; display:flex; align-items:center; justify-content:flex-end; gap:14px; }
  .np-right button{ background:none; border:none; color:var(--text-lo); cursor:pointer; display:flex; }
  .np-right button:hover{ color:var(--text-hi); }
  .np-right svg{ width:16px; height:16px; }
  .volume-row{ display:flex; align-items:center; gap:6px; width:90px; }
  input[type="range"]{
    -webkit-appearance:none; width:100%; height:3px; background:var(--line); border-radius:2px; outline:none;
  }
  input[type="range"]::-webkit-slider-thumb{
    -webkit-appearance:none; width:11px; height:11px; border-radius:50%; background:var(--accent); cursor:pointer;
    box-shadow:0 0 0 3px rgba(224,164,88,.15);
  }

  /* queue panel */
  .queue-panel{
    position:fixed; right:0; top:0; bottom:82px; width:320px; background:var(--surface);
    border-left:1px solid var(--line); transform:translateX(100%); transition:transform .22s ease;
    z-index:50; display:flex; flex-direction:column;
  }
  .queue-panel.open{ transform:translateX(0); }
  .queue-head{ display:flex; align-items:center; justify-content:space-between; padding:18px 18px 12px; border-bottom:1px solid var(--line); }
  .queue-head h3{ font-family:'Fraunces',serif; font-size:17px; font-weight:600; }
  .queue-head button{ background:none; border:none; color:var(--text-faint); cursor:pointer; }
  .queue-head svg{ width:16px; height:16px; }
  .queue-list{ overflow-y:auto; padding:8px 10px; flex:1; }
  .queue-item{ display:flex; align-items:center; gap:10px; padding:8px 8px; border-radius:8px; cursor:pointer; }
  .queue-item:hover{ background:var(--surface-hi); }
  .queue-item .qi-cover{ width:36px; height:36px; border-radius:5px; flex-shrink:0; }
  .queue-item .t{ font-size:12.5px; font-weight:700; }
  .queue-item .s{ font-size:11px; color:var(--text-faint); }
  .queue-empty{ padding:30px 18px; color:var(--text-faint); font-size:13px; text-align:center; }

  ::-webkit-scrollbar{ width:8px; }
  ::-webkit-scrollbar-thumb{ background:var(--line); border-radius:4px; }
</style>
</head>
<body>

<div id="app"></div>

<script>
/* ============================= DATA ============================= */
const MOODS = {
  moody:   { swing:.58, hatDensity:.55, filterFreq:1700, reverbWet:.24, chordDur:"2n",  detune:0  },
  warm:    { swing:.50, hatDensity:.48, filterFreq:2600, reverbWet:.16, chordDur:"2n",  detune:0  },
  boombap: { swing:.62, hatDensity:.70, filterFreq:2200, reverbWet:.10, chordDur:"4n",  detune:0  },
  dreamy:  { swing:.50, hatDensity:.32, filterFreq:3100, reverbWet:.34, chordDur:"1n",  detune:6  }
};
const CHORD_PROG = [0, 8, 3, 10]; // natural-minor i - VI - III - VII, semitone offsets

function mk(id,title,dur,bpm,root){ return {id,title,duration:dur,bpm,root}; }

const MIXTAPES = [
  {
    id:"m1", title:"3AM Static", artist:"Mellow Ghost", cat:"PT-01", mood:"moody",
    gradient:["#2b2038","#171019","135deg"],
    blurb:"Rain on the window, the city gone quiet, a verse whispered so it doesn't wake anyone.",
    tracks:[
      mk("t1","Insomnia Freestyle",158,78,45),
      mk("t2","Static in the Hallway",171,76,43),
      mk("t3","Empty Streetlight",149,80,45),
      mk("t4","Talk Slow",184,74,41),
      mk("t5","Fade to Grey",163,77,43)
    ]
  },
  {
    id:"m2", title:"Rooftop Haze", artist:"Nightbus", cat:"PT-02", mood:"warm",
    gradient:["#3a2a2c","#25161c","135deg"],
    blurb:"Golden-hour keys and a slow-rolling flow, recorded with the window cracked open.",
    tracks:[
      mk("t6","Skyline Exhale",176,84,48),
      mk("t7","Slow Down the Clock",165,82,45),
      mk("t8","Paper Cranes",188,80,48),
      mk("t9","Golden Hour Flow",152,85,50),
      mk("t10","Drift",170,81,45)
    ]
  },
  {
    id:"m3", title:"Velvet Concrete", artist:"Static Bloom", cat:"PT-03", mood:"boombap",
    gradient:["#26302c","#141a17","135deg"],
    blurb:"Dusty boom-bap drums and corner-store poetry — chill rap with its shoes still on.",
    tracks:[
      mk("t11","Corner Store Poetry",168,88,40),
      mk("t12","Concrete Garden",155,90,43),
      mk("t13","Loose Change",179,86,40),
      mk("t14","Velvet Static",161,89,42),
      mk("t15","Windowseat",173,87,40)
    ]
  },
  {
    id:"m4", title:"Low Tide", artist:"Low Tide Kid", cat:"PT-04", mood:"dreamy",
    gradient:["#1e2a33","#121a20","135deg"],
    blurb:"Tape-warped and half-asleep, verses that drift out with the tide.",
    tracks:[
      mk("t16","Tide Pool",192,70,38),
      mk("t17","Salt & Smoke",177,72,36),
      mk("t18","Analog Waves",203,68,38),
      mk("t19","Driftwood",166,71,40),
      mk("t20","Undertow Freestyle",181,69,36)
    ]
  }
];

const trackIndex = {}; // id -> {track, mixtape}
MIXTAPES.forEach(m => m.tracks.forEach(t => { trackIndex[t.id] = {track:t, mixtape:m}; }));

/* ============================= STATE ============================= */
const state = {
  view: {type:"home"},          // {type:"home"} | {type:"mixtape", id} | {type:"liked"} | {type:"search", q}
  queue: [],                    // array of track ids, current play order
  queuePos: -1,
  playing: false,
  liked: new Set(),
  shuffle: false,
  repeat: "off",                // off | all | one
  volume: 70,
  queueOpen: false,
  elapsed: 0,
  searchQuery: ""
};

/* ============================= ICONS ============================= */
const ICN = {
  search:`<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.3-4.3"/></svg>`,
  home:`<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 11l9-8 9 8"/><path d="M5 10v10h14V10"/></svg>`,
  heartOutline:`<svg class="i" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 21s-7.5-4.6-10-9.3C.4 8 2 4.5 5.6 4c2-.3 3.8.7 6.4 3.4C14.6 4.7 16.4 3.7 18.4 4c3.6.5 5.2 4 3.6 7.7C19.5 16.4 12 21 12 21z"/></svg>`,
  heartFilled:`<svg class="i liked" viewBox="0 0 24 24" fill="currentColor" stroke="currentColor" stroke-width="2"><path d="M12 21s-7.5-4.6-10-9.3C.4 8 2 4.5 5.6 4c2-.3 3.8.7 6.4 3.4C14.6 4.7 16.4 3.7 18.4 4c3.6.5 5.2 4 3.6 7.7C19.5 16.4 12 21 12 21z"/></svg>`,
  play:`<svg viewBox="0 0 24 24" fill="currentColor"><path d="M7 5v14l12-7z"/></svg>`,
  pause:`<svg viewBox="0 0 24 24" fill="currentColor"><rect x="6" y="5" width="4" height="14"/><rect x="14" y="5" width="4" height="14"/></svg>`,
  prev:`<svg viewBox="0 0 24 24" fill="currentColor"><path d="M6 5h2v14H6zM20 5v14L9 12z"/></svg>`,
  next:`<svg viewBox="0 0 24 24" fill="currentColor"><path d="M18 5h-2v14h2zM4 5v14l11-7z"/></svg>`,
  shuffle:`<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 5h3l9 14h4"/><path d="M4 19h3l4-6"/><path d="M16 5h4v4"/><path d="M20 5l-5.5 6.5"/><path d="M20 19l-3.5-4"/></svg>`,
  repeat:`<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 2l4 4-4 4"/><path d="M3 11V9a4 4 0 0 1 4-4h14"/><path d="M7 22l-4-4 4-4"/><path d="M21 13v2a4 4 0 0 1-4 4H3"/></svg>`,
  volume:`<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M11 5 6 9H2v6h4l5 4V5z"/><path d="M15.5 8.5a5 5 0 0 1 0 7"/><path d="M18.5 5.5a9 9 0 0 1 0 13"/></svg>`,
  queue:`<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 6h18M3 12h18M3 18h12"/></svg>`,
  x:`<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M18 6L6 18M6 6l12 12"/></svg>`,
  cassette:`<svg class="reel" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6"><rect x="2" y="5" width="20" height="14" rx="2"/><circle cx="8" cy="12" r="2.4"/><circle cx="16" cy="12" r="2.4"/><path d="M9.5 16h5"/></svg>`,
  reelspin:`<svg class="reel-spin" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6"><circle cx="12" cy="12" r="9"/><circle cx="12" cy="12" r="2"/><path d="M12 3v3M12 18v3M3 12h3M18 12h3M5.6 5.6l2 2M16.4 16.4l2 2M5.6 18.4l2-2M16.4 7.6l2-2"/></svg>`,
};

function fmtTime(s){
  s = Math.max(0, Math.floor(s));
  const m = Math.floor(s/60), r = s%60;
  return m + ":" + String(r).padStart(2,"0");
}

/* ============================= AUDIO ENGINE ============================= */
const Engine = (() => {
  let ready=false, nodes=null, barIndex=0, currentMood=null;

  function build(){
    const master = new Tone.Gain(state.volume/100);
    const filter = new Tone.Filter(2400,"lowpass");
    const distortion = new Tone.Distortion(0.05);
    const reverb = new Tone.Reverb({decay:2.2, wet:.2});
    master.chain(filter, distortion, reverb, Tone.Destination);

    const kick = new Tone.MembraneSynth({pitchDecay:0.05, octaves:4, envelope:{attack:0.001, decay:0.35, sustain:0}});
    const kickGain = new Tone.Gain(0.9).connect(master); kick.connect(kickGain);

    const snare = new Tone.NoiseSynth({noise:{type:"pink"}, envelope:{attack:0.001, decay:0.16, sustain:0}});
    const snareFilt = new Tone.Filter(1200,"bandpass");
    const snareGain = new Tone.Gain(0.5).connect(master);
    snare.chain(snareFilt, snareGain);

    const hat = new Tone.NoiseSynth({noise:{type:"white"}, envelope:{attack:0.001, decay:0.045, sustain:0}});
    const hatFilt = new Tone.Filter(7500,"highpass");
    const hatGain = new Tone.Gain(0.22).connect(master);
    hat.chain(hatFilt, hatGain);

    const bass = new Tone.MonoSynth({
      oscillator:{type:"sine"},
      envelope:{attack:0.02, decay:0.3, sustain:0.5, release:0.5},
      filterEnvelope:{attack:0.02, decay:0.2, sustain:0.3, release:0.4, baseFrequency:200, octaves:2}
    });
    const bassGain = new Tone.Gain(0.55).connect(master); bass.connect(bassGain);

    const chordVoice = new Tone.PolySynth(Tone.Synth, {
      oscillator:{type:"triangle"},
      envelope:{attack:0.4, decay:0.4, sustain:0.55, release:1.4}
    });
    const chorus = new Tone.Chorus(3.2, 2.0, 0.25).start();
    const chordGain = new Tone.Gain(0.28).connect(master);
    chordVoice.chain(chorus, chordGain);

    const vinylNoise = new Tone.Noise("pink");
    const vinylHP = new Tone.Filter(500,"highpass");
    const vinylLP = new Tone.Filter(4000,"lowpass");
    const vinylGain = new Tone.Gain(0.035).connect(master);
    vinylNoise.chain(vinylHP, vinylLP, vinylGain);

    return {master, filter, reverb, kick, snare, hat, bass, chordVoice, vinylNoise, barLoop:null, drumSeq:null};
  }

  async function ensureStarted(){
    if (Tone.context.state !== "running") await Tone.start();
    if (!ready){ nodes = build(); ready = true; }
  }

  function disposePattern(){
    if (nodes.barLoop){ nodes.barLoop.dispose(); nodes.barLoop=null; }
    if (nodes.drumSeq){ nodes.drumSeq.dispose(); nodes.drumSeq=null; }
  }

  async function loadTrack(track){
    await ensureStarted();
    Tone.Transport.stop();
    Tone.Transport.cancel();
    disposePattern();
    barIndex = 0;
    currentMood = MOODS[trackIndex[track.id].mixtape.mood];

    Tone.Transport.bpm.value = track.bpm;
    Tone.Transport.swing = currentMood.swing;
    Tone.Transport.swingSubdivision = "16n";
    nodes.filter.frequency.value = currentMood.filterFreq;
    nodes.reverb.wet.value = currentMood.reverbWet;
    nodes.chordVoice.set({detune: currentMood.detune});

    nodes.barLoop = new Tone.Loop((time) => {
      const offset = CHORD_PROG[barIndex % CHORD_PROG.length];
      const chordRoot = track.root + 12 + offset;
      const chordNotes = [0,3,7,10].map(iv => Tone.Frequency(chordRoot+iv, "midi").toNote());
      nodes.chordVoice.triggerAttackRelease(chordNotes, currentMood.chordDur, time, 0.55);
      const bassNote = Tone.Frequency(track.root + offset, "midi").toNote();
      nodes.bass.triggerAttackRelease(bassNote, "4n", time, 0.85);
      nodes.bass.triggerAttackRelease(bassNote, "8n", time + Tone.Time("2n").toSeconds(), 0.45);
      barIndex++;
    }, "1m").start(0);

    nodes.drumSeq = new Tone.Sequence((time, step) => {
      if (step===0 || step===10) nodes.kick.triggerAttackRelease("C1","8n", time, 0.95);
      if (step===4 || step===12) nodes.snare.triggerAttackRelease("8n", time, 0.6);
      if (Math.random() < currentMood.hatDensity) nodes.hat.triggerAttackRelease("16n", time, Math.random()*0.3+0.15);
    }, [...Array(16).keys()], "16n").start(0);
  }

  function play(){ Tone.Transport.start(); nodes.vinylNoise.start(); }
  function pause(){ Tone.Transport.pause(); try{ nodes.vinylNoise.stop(); }catch(e){} }
  function setVolume(v){ if (nodes) nodes.master.gain.rampTo(v/100, 0.08); }

  return { loadTrack, play, pause, setVolume };
})();

/* ============================= QUEUE LOGIC ============================= */
function buildQueueFromMixtape(mixtapeId, startTrackId){
  const m = MIXTAPES.find(x=>x.id===mixtapeId);
  let ids = m.tracks.map(t=>t.id);
  if (state.shuffle){
    ids = shuffleArray(ids.slice());
  }
  state.queue = ids;
  state.queuePos = state.queue.indexOf(startTrackId);
  if (state.queuePos === -1) state.queuePos = 0;
}
function shuffleArray(arr){
  for (let i=arr.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [arr[i],arr[j]]=[arr[j],arr[i]]; }
  return arr;
}

let progressTimer = null;
let lastTick = null;

async function playTrackId(id, mixtapeIdForQueue){
  const entry = trackIndex[id];
  if (!entry) return;
  if (mixtapeIdForQueue) buildQueueFromMixtape(mixtapeIdForQueue, id);
  else {
    if (!state.queue.includes(id)){ state.queue = [id]; state.queuePos = 0; }
    else state.queuePos = state.queue.indexOf(id);
  }
  state.elapsed = 0;
  await Engine.loadTrack(entry.track);
  Engine.play();
  state.playing = true;
  startTimer();
  render();
}

function startTimer(){
  stopTimer();
  lastTick = performance.now();
  progressTimer = setInterval(() => {
    const now = performance.now();
    const dt = (now-lastTick)/1000;
    lastTick = now;
    if (!state.playing) return;
    state.elapsed += dt;
    const cur = currentTrack();
    if (cur && state.elapsed >= cur.duration){
      handleTrackEnd();
    } else {
      updateProgressUI();
    }
  }, 200);
}
function stopTimer(){ if (progressTimer) clearInterval(progressTimer); progressTimer=null; }

function currentTrack(){
  if (state.queuePos<0 || state.queuePos>=state.queue.length) return null;
  return trackIndex[state.queue[state.queuePos]].track;
}
function currentMixtape(){
  if (state.queuePos<0) return null;
  return trackIndex[state.queue[state.queuePos]].mixtape;
}

function handleTrackEnd(){
  if (state.repeat === "one"){
    state.elapsed = 0;
    Engine.loadTrack(currentTrack()).then(()=>Engine.play());
    return;
  }
  goNext(true);
}

function togglePlay(){
  if (!state.queue.length){
    const first = MIXTAPES[0].tracks[0];
    playTrackId(first.id, MIXTAPES[0].id);
    return;
  }
  if (state.playing){
    Engine.pause(); state.playing=false; stopTimer();
  } else {
    Engine.play(); state.playing=true; startTimer();
  }
  render();
}

function goNext(auto){
  if (!state.queue.length) return;
  let next = state.queuePos+1;
  if (next >= state.queue.length){
    if (state.repeat==="all") next = 0;
    else { state.playing=false; stopTimer(); Engine.pause(); render(); return; }
  }
  state.queuePos = next;
  state.elapsed = 0;
  Engine.loadTrack(currentTrack()).then(()=>{ if(state.playing || auto){ Engine.play(); state.playing=true; startTimer(); } render(); });
}
function goPrev(){
  if (!state.queue.length) return;
  if (state.elapsed > 4){ state.elapsed = 0; updateProgressUI(); return; }
  let prev = state.queuePos-1;
  if (prev < 0) prev = state.repeat==="all" ? state.queue.length-1 : 0;
  state.queuePos = prev;
  state.elapsed = 0;
  Engine.loadTrack(currentTrack()).then(()=>{ if(state.playing){ Engine.play(); startTimer(); } render(); });
}

function toggleLike(id, e){ e && e.stopPropagation();
  if (state.liked.has(id)) state.liked.delete(id); else state.liked.add(id);
  render();
}
function toggleShuffle(){ state.shuffle = !state.shuffle; render(); }
function cycleRepeat(){
  state.repeat = state.repeat==="off" ? "all" : state.repeat==="all" ? "one" : "off";
  render();
}

/* ============================= RENDER ============================= */
function coverStyle(m){ return `background: linear-gradient(${m.gradient[2]}, ${m.gradient[0]}, ${m.gradient[1]});`; }

function renderCoverInner(m, size){
  return `
    <div class="cover-top"><span>${m.cat}</span><span>PETE-TIDE</span></div>
    <div class="cover-title" style="${size==='sm'?'font-size:15px':''}">${m.title}</div>
  `;
}

function renderSidebar(){
  const v = state.view;
  const isHome = v.type==="home";
  const isLiked = v.type==="liked";
  return `
    <div class="sidebar">
      <div class="brand">${ICN.cassette}<div class="brand-name">Pete<span>-tide</span></div></div>
      <div class="nav-group">
        <div class="nav-item ${isHome?'active':''}" data-action="nav-home">${ICN.home}<span>Home</span></div>
        <div class="nav-item ${isLiked?'active':''}" data-action="nav-liked">${ICN.heartOutline}<span>Liked Tracks</span><span class="count">${state.liked.size}</span></div>
      </div>
      <div class="nav-group" style="flex:1; display:flex; flex-direction:column; min-height:0;">
        <div class="nav-label">Your Mixtapes</div>
        <div class="library-list">
          ${MIXTAPES.map(m => `
            <div class="lib-item ${v.type==='mixtape' && v.id===m.id ? 'active':''}" data-action="open-mixtape" data-id="${m.id}">
              <div class="lib-swatch" style="${coverStyle(m)}"></div>
              <div class="lib-meta">
                <div class="t">${m.title}</div>
                <div class="s">${m.artist}</div>
              </div>
            </div>
          `).join("")}
        </div>
      </div>
    </div>
  `;
}

function renderTrackRow(t, mixtape, opts){
  const isCurrent = currentTrack() && currentTrack().id===t.id;
  const liked = state.liked.has(t.id);
  return `
    <tr class="${isCurrent?'playing':''}" data-action="play-track" data-id="${t.id}" data-mixtape="${mixtape.id}">
      <td class="t-idx">${isCurrent && state.playing ? `<div class="eq"><span></span><span></span><span></span></div>` : (opts && opts.idx!==undefined ? opts.idx+1 : "")}</td>
      <td>
        <div class="t-title">${t.title}</div>
        <div class="t-artist">${mixtape.artist}</div>
      </td>
      <td class="t-like"><span data-action="like" data-id="${t.id}">${liked ? ICN.heartFilled : ICN.heartOutline}</span></td>
      <td class="t-dur">${fmtTime(t.duration)}</td>
    </tr>
  `;
}

function renderHome(){
  return `
    <div class="section-title">Good evening.</div>
    <div class="section-sub">Four tapes, low tempo, nowhere to be.</div>
    <div class="grid">
      ${MIXTAPES.map(m => `
        <div class="card" data-action="open-mixtape" data-id="${m.id}">
          <div class="cover" style="${coverStyle(m)}">
            ${renderCoverInner(m,'sm')}
            <button class="play-fab" data-action="play-mixtape" data-id="${m.id}">${ICN.play}</button>
          </div>
          <div class="card-meta">
            <div class="t">${m.title}</div>
            <div class="s">${m.artist} · ${m.tracks.length} tracks</div>
          </div>
        </div>
      `).join("")}
    </div>
  `;
}

function renderMixtape(id){
  const m = MIXTAPES.find(x=>x.id===id);
  if (!m) return renderHome();
  return `
    <div class="detail-header">
      <div class="detail-cover" style="${coverStyle(m)}">${renderCoverInner(m)}</div>
      <div>
        <div class="detail-eyebrow">Mixtape · ${m.cat}</div>
        <div class="detail-title">${m.title}</div>
        <div class="detail-sub">${m.blurb}</div>
        <div class="detail-actions">
          <button class="btn-play-all" data-action="play-mixtape" data-id="${m.id}">${ICN.play}<span>Play</span></button>
        </div>
      </div>
    </div>
    <table class="tracklist">
      <thead><tr><th>#</th><th>Title</th><th></th><th style="text-align:right;">Time</th></tr></thead>
      <tbody>${m.tracks.map((t,idx)=>renderTrackRow(t,m,{idx})).join("")}</tbody>
    </table>
  `;
}

function renderLiked(){
  const ids = Array.from(state.liked);
  if (!ids.length){
    return `
      <div class="section-title">Liked Tracks</div>
      <div class="empty">${ICN.heartOutline}<div>Nothing liked yet — tap the heart on any track.</div></div>
    `;
  }
  return `
    <div class="section-title">Liked Tracks</div>
    <div class="section-sub">${ids.length} track${ids.length>1?'s':''}</div>
    <table class="tracklist">
      <thead><tr><th>#</th><th>Title</th><th></th><th style="text-align:right;">Time</th></tr></thead>
      <tbody>${ids.map((id,idx)=>{ const e=trackIndex[id]; return renderTrackRow(e.track, e.mixtape, {idx}); }).join("")}</tbody>
    </table>
  `;
}

function renderSearch(q){
  const query = q.toLowerCase();
  const mixMatches = MIXTAPES.filter(m => m.title.toLowerCase().includes(query) || m.artist.toLowerCase().includes(query));
  const trackMatches = [];
  MIXTAPES.forEach(m => m.tracks.forEach(t => { if (t.title.toLowerCase().includes(query)) trackMatches.push({t,m}); }));
  if (!mixMatches.length && !trackMatches.length){
    return `<div class="section-title">No results for "${q}"</div><div class="empty">${ICN.search}<div>Try a different search.</div></div>`;
  }
  return `
    <div class="section-title">Results for "${q}"</div>
    ${mixMatches.length ? `
      <div class="section-sub" style="margin-top:18px;">Mixtapes</div>
      <div class="grid">${mixMatches.map(m => `
        <div class="card" data-action="open-mixtape" data-id="${m.id}">
          <div class="cover" style="${coverStyle(m)}">${renderCoverInner(m,'sm')}
            <button class="play-fab" data-action="play-mixtape" data-id="${m.id}">${ICN.play}</button>
          </div>
          <div class="card-meta"><div class="t">${m.title}</div><div class="s">${m.artist}</div></div>
        </div>`).join("")}</div>` : ""}
    ${trackMatches.length ? `
      <div class="section-sub" style="margin-top:24px;">Tracks</div>
      <table class="tracklist"><tbody>${trackMatches.map(x=>renderTrackRow(x.t,x.m,{})).join("")}</tbody></table>` : ""}
  `;
}

function renderMain(){
  const v = state.view;
  let body;
  if (v.type==="home") body = renderHome();
  else if (v.type==="mixtape") body = renderMixtape(v.id);
  else if (v.type==="liked") body = renderLiked();
  else if (v.type==="search") body = renderSearch(v.q);
  else body = renderHome();

  return `
    <div class="main">
      <div class="topbar">
        <div class="search-wrap">
          ${ICN.search}
          <input type="text" id="search-input" placeholder="Search mixtapes or tracks" value="${state.searchQuery.replace(/"/g,'&quot;')}"/>
        </div>
        <div class="topbar-tag">90s // chill rap // lofi</div>
      </div>
      <div class="content">${body}</div>
    </div>
  `;
}

function renderNowBar(){
  const t = currentTrack();
  const m = currentMixtape();
  const liked = t && state.liked.has(t.id);
  const pct = t ? Math.min(100, (state.elapsed/t.duration)*100) : 0;
  return `
    <div class="now-bar">
      <div class="np-track">
        <div class="np-cover ${state.playing?'playing':''}" style="${m?coverStyle(m):'background:var(--surface-hi);'}">${ICN.reelspin}</div>
        <div class="np-meta">
          <div class="t">${t ? t.title : "Nothing playing"}</div>
          <div class="s">${m ? m.artist : "Pick a mixtape to start"}</div>
        </div>
        ${t ? `<span class="np-like" data-action="like" data-id="${t.id}">${liked?ICN.heartFilled:ICN.heartOutline}</span>` : ""}
      </div>
      <div class="np-center">
        <div class="transport">
          <button class="${state.shuffle?'active':''}" data-action="shuffle" title="Shuffle">${ICN.shuffle}</button>
          <button data-action="prev" title="Previous">${ICN.prev}</button>
          <button class="play-toggle" data-action="toggle-play" title="Play/Pause">${state.playing?ICN.pause:ICN.play}</button>
          <button data-action="next" title="Next">${ICN.next}</button>
          <button class="${state.repeat!=='off'?'active':''}" data-action="repeat" title="Repeat (${state.repeat})">${ICN.repeat}</button>
        </div>
        <div class="progress-row">
          <span class="tape-counter">${fmtTime(state.elapsed)}</span>
          <div class="progress-track" id="progress-track"><div class="progress-fill" id="progress-fill" style="width:${pct}%;"></div></div>
          <span class="tape-counter">${t?fmtTime(t.duration):'0:00'}</span>
        </div>
      </div>
      <div class="np-right">
        <button data-action="toggle-queue" title="Queue">${ICN.queue}</button>
        <div class="volume-row">${ICN.volume}<input type="range" id="volume-slider" min="0" max="100" value="${state.volume}"/></div>
      </div>
    </div>
  `;
}

function renderQueuePanel(){
  const upcoming = state.queue.slice(state.queuePos+1);
  return `
    <div class="queue-panel ${state.queueOpen?'open':''}">
      <div class="queue-head"><h3>Up Next</h3><button data-action="toggle-queue">${ICN.x}</button></div>
      <div class="queue-list">
        ${upcoming.length ? upcoming.map(id => {
          const e = trackIndex[id];
          return `<div class="queue-item" data-action="play-track" data-id="${id}">
            <div class="qi-cover" style="${coverStyle(e.mixtape)}"></div>
            <div><div class="t">${e.track.title}</div><div class="s">${e.mixtape.artist}</div></div>
          </div>`;
        }).join("") : `<div class="queue-empty">Queue's empty. Play a mixtape to fill it up.</div>`}
      </div>
    </div>
  `;
}

function updateProgressUI(){
  const t = currentTrack();
  if (!t) return;
  const pct = Math.min(100, (state.elapsed/t.duration)*100);
  const fill = document.getElementById("progress-fill");
  if (fill) fill.style.width = pct+"%";
  const counters = document.querySelectorAll(".tape-counter");
  if (counters[0]) counters[0].textContent = fmtTime(state.elapsed);
}

function render(){
  document.getElementById("app").innerHTML = `
    <div class="shell">
      ${renderSidebar()}
      ${renderMain()}
    </div>
    ${renderNowBar()}
    ${renderQueuePanel()}
  `;
  const input = document.getElementById("search-input");
  if (input){
    input.addEventListener("input", (e) => {
      state.searchQuery = e.target.value;
      state.view = state.searchQuery.trim() ? {type:"search", q:state.searchQuery.trim()} : {type:"home"};
      const caretPos = e.target.selectionStart;
      render();
      const ni = document.getElementById("search-input");
      if (ni){ ni.focus(); ni.setSelectionRange(caretPos, caretPos); }
    });
  }
  const vol = document.getElementById("volume-slider");
  if (vol) vol.addEventListener("input", (e) => { state.volume = +e.target.value; Engine.setVolume(state.volume); });

  const ptrack = document.getElementById("progress-track");
  if (ptrack) ptrack.addEventListener("click", (e) => {
    const t = currentTrack(); if (!t) return;
    const rect = ptrack.getBoundingClientRect();
    const ratio = (e.clientX-rect.left)/rect.width;
    state.elapsed = Math.max(0, Math.min(t.duration, ratio*t.duration));
    updateProgressUI();
  });
}

/* ============================= EVENTS ============================= */
document.getElementById("app").addEventListener("click", (e) => {
  const el = e.target.closest("[data-action]");
  if (!el) return;
  const action = el.dataset.action;
  const id = el.dataset.id;

  switch(action){
    case "nav-home": state.view={type:"home"}; state.searchQuery=""; render(); break;
    case "nav-liked": state.view={type:"liked"}; state.searchQuery=""; render(); break;
    case "open-mixtape": state.view={type:"mixtape", id}; state.searchQuery=""; render(); break;
    case "play-mixtape": {
      const m = MIXTAPES.find(x=>x.id===id);
      playTrackId(m.tracks[0].id, m.id);
      break;
    }
    case "play-track": {
      const mixId = el.dataset.mixtape;
      playTrackId(id, mixId || undefined);
      break;
    }
    case "like": toggleLike(id, e); break;
    case "toggle-play": togglePlay(); break;
    case "prev": goPrev(); break;
    case "next": goNext(false); break;
    case "shuffle": toggleShuffle(); break;
    case "repeat": cycleRepeat(); break;
    case "toggle-queue": state.queueOpen = !state.queueOpen; render(); break;
  }
});

render();
</script>
</body>
</html>
