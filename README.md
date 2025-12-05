<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>buksan mo, subukan mo kong dedmahin</title>
  <style>
    /* simple, mobile-first, cute aesthetic (all text intentionally lowercase) */
    :root{--bg:#fff6fb;--card:#fff;--accent:#ff78b6;--muted:#6b6b6b;font-family: 'system-ui', -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;}
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#fff);color:#222}
    .wrap{min-height:100%;display:flex;align-items:center;justify-content:center;padding:24px}
    .card{width:100%;max-width:420px;background:var(--card);border-radius:16px;box-shadow:0 8px 30px rgba(0,0,0,0.06);padding:22px}
    h1{margin:0 0 8px;font-size:20px;color:var(--accent)}
    p.lead{margin:0 0 18px;color:var(--muted);font-size:14px}
    .question{font-weight:600;margin:12px 0}
    .options{display:grid;grid-template-columns:1fr;gap:10px;margin-top:8px}
    button.opt{background:linear-gradient(180deg,#fff,#fff);border:1px solid #f2dbe6;padding:12px;border-radius:10px;font-size:14px;cursor:pointer}
    button.opt:hover{transform:translateY(-2px)}
    .progress{height:8px;background:#f3e8ef;border-radius:8px;overflow:hidden;margin-top:14px}
    .bar{height:100%;background:linear-gradient(90deg,var(--accent),#ffb3d9);width:0%}
    .center{display:flex;gap:8px;align-items:center;justify-content:center;margin-top:14px}
    .small{font-size:12px;color:var(--muted)}
    .reveal{display:none;flex-direction:column;align-items:center;text-align:center;padding:6px}
    .reveal.show{display:flex}
    .name{font-size:28px;margin:8px 0;color:var(--accent);font-weight:700}
    .note{font-size:15px;color:#333;margin-top:6px}
    .play{margin-top:12px}
    .footer{margin-top:12px;font-size:12px;color:var(--muted);text-align:center}
    .btn-reset{margin-top:12px;background:transparent;border:1px dashed var(--accent);padding:8px 12px;border-radius:10px;cursor:pointer}
    @media(min-width:700px){.card{padding:34px}}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>buksan mo, subukan mo kong dedmahin</h1>
      <p class="lead">tangina mo gawin mo nalng di kita i hahack promise</p>

      <div id="quiz">
        <div id="qbox">
          <div class="question" id="question">what's your favorite way to spend a saturday?</div>
          <div class="options" id="options"></div>
          <div class="progress" aria-hidden="true"><div class="bar" id="bar"></div></div>
          <div class="center small">question <span id="step">1</span> of <span id="total">3</span></div>
        </div>

        <div id="reveal" class="reveal" aria-hidden="true">
          <div class="emoji">💌</div>
          <div class="name" id="reveal-name">you</div>
          <div class="note" id="reveal-note">you've reached the secret ending.</div>
          <audio id="reveal-audio" class="play" controls style="width:100%;display:none"></audio>
          <button class="btn-reset" id="restart">take again</button>
        </div>
      </div>

      <div class="footer small">made with <span style="color:var(--accent)">♥</span> — customize the code to insert your name + message</div>
    </div>
  </div>

  <script>
    // ---- configuration: change these to personalize ----
    const finalName = "(pretot)"; // the name that will be revealed as the confessor
    const finalMessage = "“uhmm i just want to be honest with you… i like you. matagal ko na gusto sabihin pero hindi ko alam pano sisimulan. i don’t wanna make things weird between us, and i’m not expecting anything back, pero i just want you to know how i really feel. you make me happy in small ways, and every time we talk, i feel comfortable and safe, and that’s rare for me,di kita nagustuhan dhil sa pogi ka kanon ganyan ksi for me i like u for u ur perfect the way u are and always remember that im always proud of u jm, i know i won't stand a chance on the girl u truly want butt its okyyy, ksi alm ko nmn na di moko gusto e i just want to be honest with u..

i’m not rushing you or asking you for anything… i just don’t wanna keep this to myself anymore. whatever your answer is, okay lang sakin, i just wanted to be honest with you.”
  "; // the confession text
    // optional audio url (set to an mp3 url or leave empty). if provided, controls will appear.
    const audioUrl = "";
    // number of quiz steps (3 recommended). don't set lower than 1.
    const STEPS = 3;
    // ---------------------------------------------------

    const questions = [
      {
        q: "what's your favorite colour?",
        opts: ["blue","black","red","yellow"]
      },
      {
        q: "pick a snack:",
        opts: ["ice cream","fries","chips","fruit"]
      },
      {
        q: "favorite game?",
        opts: ["minecraft","call of duty","roblox","valorant"]
      }
    ];


    // if STEPS differs from questions length, slice/extend questions safely
    const qs = questions.slice(0, Math.max(1, STEPS));
    document.getElementById('total').textContent = qs.length;

    let step = 0;
    const optsEl = document.getElementById('options');
    const qEl = document.getElementById('question');
    const stepEl = document.getElementById('step');
    const bar = document.getElementById('bar');
    const reveal = document.getElementById('reveal');
    const qbox = document.getElementById('qbox');
    const revealName = document.getElementById('reveal-name');
    const revealNote = document.getElementById('reveal-note');
    const audio = document.getElementById('reveal-audio');
    const restart = document.getElementById('restart');

    function renderStep(){
      if(step >= qs.length){
        showReveal();
        return;
      }
      const cur = qs[step];
      qEl.textContent = cur.q;
      optsEl.innerHTML = '';
      cur.opts.forEach(o => {
        const btn = document.createElement('button');
        btn.className = 'opt';
        btn.type = 'button';
        btn.textContent = o;
        btn.addEventListener('click', onChoose);
        optsEl.appendChild(btn);
      });
      stepEl.textContent = Math.min(step+1, qs.length);
      const pct = Math.round((step / qs.length) * 100);
      bar.style.width = pct + '%';
    }

    function onChoose(){
      // you can add small animation or store answers here if you want
      step++;
      renderStep();
    }

    function showReveal(){
      qbox.style.display = 'none';
      reveal.classList.add('show');
      revealName.textContent = finalName;
      revealNote.textContent = finalMessage;
      if(audioUrl){
        audio.src = audioUrl;
        audio.style.display = 'block';
      }
    }

    restart.addEventListener('click', ()=>{
      // reset
      step = 0;
      qbox.style.display = '';
      reveal.classList.remove('show');
      renderStep();
    });

    // initial render
    renderStep();
  </script>
</body>
</html>

