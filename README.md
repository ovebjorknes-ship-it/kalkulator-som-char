<!doctype html>
<html lang="nb">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>easyReq – Chatkalkulator</title>

<style>
  :root{
    --er-primary: #ea5c0c;
    --er-accent:  #ea5c0c;
    --er-page-bg: #000000;

    --er-surface:   #111418;
    --er-surface-2: #161b21;
    --er-border:    #233044;
    --er-text:      #E5E7EB;
    --er-text-inv:  #FFFFFF;
    --er-muted:     #9AA6B2;

    --er-btn-radius: 10px;
    --er-bubble-radius: 16px;
    --er-chat-maxw: 760px;   /* sentrert samtale-bredde */
    --er-side-pad:  6%;      /* "litt" offset fra kant */
  }

  html, body { background:#000; }
  body {
    color:var(--er-text);
    margin:0;
    font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,Noto Sans,sans-serif;
  }

  .container{max-width:1024px;margin:0 auto;padding:24px}

  /* Hero */
  .tpb-hero{background:#000;color:var(--er-text);padding:48px 0}
  .tpb-hero h1{margin:0 0 10px 0}
  .tpb-hero p{margin:0}

  /* Chat wrapper – sentrert */
  .chat-wrap{
    margin:24px auto 0 auto;
    border:1px solid var(--er-border);
    background:var(--er-surface);
    border-radius:14px;
    padding:12px;
    min-height:420px;
    display:flex; flex-direction:column;
    max-width:var(--er-chat-maxw);
  }
  .chat{
    display:flex; flex-direction:column; gap:12px;
    padding:8px;
    overflow-y:auto;
    max-height:70vh;
  }

  /* Rader (avatar + boble) */
  .row{ display:flex; gap:10px; align-items:flex-end; }
  .row.bot{ justify-content:flex-start; margin-left: var(--er-side-pad); }
  .row.user{ justify-content:flex-end; margin-right: var(--er-side-pad); }

  /* Avatarer – responsiv skalering via clamp() */
  .avatar{
    width:clamp(28px, 3.8vw, 44px);
    height:clamp(28px, 3.8vw, 44px);
    object-fit:cover; display:block; border:1px solid var(--er-border);
    background:#fff;
  }
  .avatar.bot{ border-radius:10px; }   /* logo: litt kvadratisk */
  .avatar.user{ border-radius:50%; }   /* besøkende: rund */

  /* Bobler */
  .msg{
    max-width: 78%;
    padding:12px 14px;
    border:1px solid var(--er-border);
    border-radius: var(--er-bubble-radius);
    line-height: 1.5;
    position: relative;
    word-wrap: break-word;
    white-space: pre-wrap;
  }
  .msg.bot{
    background:var(--er-surface-2);
    color:var(--er-text);
  }
  .msg.user{
    background:var(--er-primary);
    color:var(--er-text-inv);
    border-color: transparent;
  }

  /* Handlingsknapper (valgchips etc.) – litt til høyre */
  .actions{
  	display:flex;
  	gap:8px;
  	flex-wrap:wrap;
  	margin-top:8px;

  /* venstrejuster under siste boble */
  align-self:flex-start;
  margin-left: var(--er-side-pad);
  margin-right: 0;
}

  .chip, .btn{
    cursor:pointer; text-decoration:none; display:inline-flex; align-items:center; justify-content:center;
    border-radius: var(--er-btn-radius);
    transition: background-color .15s ease, color .15s ease, border-color .15s ease, box-shadow .15s ease, filter .15s ease;
    user-select:none;
  }
  .chip{
    background:transparent; color:var(--er-accent);
    border:1.5px solid var(--er-accent);
    padding:8px 12px;
  }
  .chip:hover{ background:rgba(234,92,12,0.12); border-color:#ff6b1e; box-shadow:0 0 0 3px rgba(234,92,12,0.18); }
  .chip.selected{
    background:rgba(234,92,12,0.18);
    color:#ffd9c6;
    border-color:#ff6b1e;
    box-shadow:0 0 0 3px rgba(234,92,12,0.18);
  }

  .btn.primary{
    background:var(--er-primary); color:#fff; border:0;
    padding:12px 16px;
  }
  .btn.primary:hover{ filter:brightness(0.92); box-shadow:0 0 0 3px rgba(234,92,12,0.18); }

  /* Runde små knapper for +/− */
  .btn.round{
    width:clamp(36px, 4.2vw, 42px);
    height:clamp(36px, 4.2vw, 42px);
    padding:0;
    border-radius:999px;
    background:transparent;
    color:var(--er-text);
    border:1.5px solid var(--er-border);
    font-weight:700;
    font-size:18px;
  }
  .btn.round:hover{
    background:rgba(234,92,12,0.12);
    border-color:#ff6b1e;
  }

  .inline-form{ display:flex; gap:8px; align-items:center; margin-top:8px; flex-wrap:wrap }
  .inline-form input, .inline-form select{
    padding:10px 12px;
    border:1px solid var(--er-border);
    border-radius:10px;
    background:var(--er-surface-2);
    color:var(--er-text);
    min-width:140px;
    text-align:center;
  }
  .num-field{ display:flex; align-items:center; gap:8px; }

  .help{ font-size:13px; color:var(--er-muted); margin-top:6px }
  .meta{ font-size:12px; color:var(--er-muted) }
  
    /* Resultat inni vanlig chat-boble */
  .result-lines{ display:grid; gap:8px; margin-top:8px; }
  .result-line{
    display:flex;
    justify-content:space-between;
    gap:12px;
    padding-top:6px;
    border-top:1px solid var(--er-border);
  }
  .result-line:first-child{
    border-top:0;
    padding-top:0;
  }
  .result-line .k{ color:var(--er-muted); font-size:13px; }
  .result-line .v{ font-weight:700; }

  /* Resultat-kort i chat (bot-innhold) */
  .result-card{
    background:var(--er-surface-2);
    border:1px solid var(--er-border);
    border-radius:12px;
    padding:12px;
    display:grid; gap:10px;
  }
  .result-grid{
    display:grid; grid-template-columns:repeat(2,1fr); gap:10px;
  }
  @media (max-width:720px){
    .result-grid{ grid-template-columns:1fr; }
  }
  .res-item{
    border:1px solid var(--er-border);
    border-radius:10px;
    padding:12px;
  }
  .res-item h4{ margin:0 0 4px 0 }

  .neg { color:#ff6b6b; }

  /* Typing-indikator */
  .typing{ display:inline-flex; gap:6px; }
  .dot{ width:8px; height:8px; border-radius:50%; background:#9AA6B2; opacity:.6; animation: blink 1s infinite; }
  .dot:nth-child(2){ animation-delay: .15s }
  .dot:nth-child(3){ animation-delay: .3s }
  @keyframes blink { 0%,80%,100%{ opacity:.2 } 40%{ opacity:1 } }

  /* Fokus synlighet */
  .chip:focus-visible, .btn:focus-visible, input:focus-visible, select:focus-visible, a:focus-visible {
    outline: 3px solid rgba(234,92,12,0.6);
    outline-offset: 2px;
  }

  /* Redusert bevegelse */
  @media (prefers-reduced-motion: reduce){
    .btn, .chip{ transition:none; }
    html{ scroll-behavior:auto; }
  }
</style>
</head>
<body>

<section class="tpb-hero">
  <div class="container" style="display:flex; flex-direction:column; align-items:center; text-align:center;">
    <h1>Hva koster en tur på butikken?</h1>
    <p>Små turer blir store kostnader. La oss regne ut potensialet deres i en kort chat.</p>
  </div>
</section>

<section class="container" style="display:flex; justify-content:center;">
  <div class="chat-wrap" aria-live="polite" aria-label="Kalkulator som chat">
    <div id="chat" class="chat"></div>
    <div class="meta" id="progress" style="padding:8px 12px 4px 12px;">
      <span id="progressText"></span>
    </div>
  </div>
</section>

<script>
  /* =========================
     Avatar-URLer (sett disse!)
     ========================= */
  const AVATAR_BOT_URL  = 'https://cdn.prod.website-files.com/664c5827289f14e928596bdf/665095ee36fc75b882d809ff_easyReq%20icon%2032x32.png'
  const AVATAR_USER_URL = ''; // eks: 'https://uploads-ssl.webflow.com/.../visitor.png'

  // Fallbacks (enkle innebygde SVG-er om URL ikke er satt)
  const FALLBACK_BOT = 'data:image/svg+xml;utf8,' + encodeURIComponent(`
    <svg xmlns="http://www.w3.org/2000/svg" width="44" height="44" viewBox="0 0 44 44">
      <rect width="44" height="44" rx="10" fill="white"/>
      <g font-family="Segoe UI, Arial, sans-serif" font-weight="900">
        <text x="9" y="26" font-size="18" fill="#1f2937">e</text>
        <text x="20" y="31" font-size="22" fill="#ea5c0c">R</text>
      </g>
    </svg>
  `);
  const FALLBACK_USER = 'data:image/svg+xml;utf8,' + encodeURIComponent(`
    <svg xmlns="http://www.w3.org/2000/svg" width="44" height="44" viewBox="0 0 44 44">
      <circle cx="22" cy="22" r="22" fill="#232832"/>
      <circle cx="22" cy="15" r="7" fill="#9AA6B2"/>
      <path d="M8,36c3-7,10.5-7,14-7s11,0,14,7" fill="#9AA6B2"/>
    </svg>
  `);

  // Valuta
  const NOK = new Intl.NumberFormat('nb-NO', { style:'currency', currency:'NOK', maximumFractionDigits:0 });

  // Lisensmatrise
  function licenseFromTurnover(turnoverBand){
    if (turnoverBand === '0-5')     return  4000;
    if (turnoverBand === '5-10')    return  8500;
    if (turnoverBand === '10-20')   return 12500;
    if (turnoverBand === '20-40')   return 16000;
    if (turnoverBand === '40-60')   return 20000;
    if (turnoverBand === '60-100')  return 28000;
    if (turnoverBand === '100-200') return 35000;
    if (turnoverBand === '200-400') return 59000;
    if (turnoverBand === '400-800') return 65000;
    if (turnoverBand === 'over-800') return null; // POA
    return 0;
  }

  // Chat state
  const chat = document.getElementById('chat');
  const progressText = document.getElementById('progressText');
  const answers = {};
  let currentStep = -1;

  const steps = [
    {
      id: 'turnover',
      label: 'Årsomsetning',
      type: 'choice',
      question: 'Hva er deres årsomsetning?',
      choices: [
        ['0-5','0–5 MNOK'],['5-10','5–10 MNOK'],['10-20','10–20 MNOK'],['20-40','20–40 MNOK'],
        ['40-60','40–60 MNOK'],['60-100','60–100 MNOK'],['100-200','100–200 MNOK'],
        ['200-400','200–400 MNOK'],['400-800','400–800 MNOK'],['over-800','Over 800 MNOK']
      ],
      toText: v => (steps[0].choices.find(c=>c[0]===v)?.[1] || v)
    },
    { id: 'vans',           label: 'Antall biler',               type: 'number', question: 'Hvor mange biler har dere?',               min:1,  step:1,  value:10 },
    { id: 'personsPerCar',  label: 'Antall personer i bilen',    type: 'number', question: 'Hvor mange personer er vanligvis i bilen?',min:1,  step:1,  value:2,
      help:'Vi ganger tidskostnaden per tur med antall personer (flere i bilen = flere lønnstimer).' },
    { id: 'tripsPerWeek',   label: 'Butikkturer pr. uke per bil',type: 'number', question: 'Hvor mange butikkturer per uke per bil?',  min:0,  step:1,  value:2 },
    { id: 'minutesPerTrip', label: 'Gj.sn. tid per tur (min)',   type: 'number', question: 'Hvor mange minutter tar en tur i snitt?',   min:15, step:15, value:60 }, // ← 15-min hopp
    { id: 'reductionPct',   label: 'Andel turer som kan unngås (%)', type: 'chips',
      question: 'Hvor stor andel av handleturene trur du det er mulig å redusere?',
      help:	'Typisk ser vi 25-60 % færre handleturer.',
      min:0, max:80, step:5, value:50, quick: [0,25,50,60,70,80] }, // ← kun chips, 50% valgt
    { id: 'hourlyCost',     label: 'Timekost arbeid (kr/t)',     type: 'number', question: 'Hva er timekost arbeid (kr/t)?',            min:300,step:50, value:500 }
  ];

  // Utils
  const sleep = (ms)=> new Promise(r=>setTimeout(r, ms));
  function scrollToBottom(){
    const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    chat.scrollTo({ top: chat.scrollHeight, behavior: reduce ? 'auto' : 'smooth' });
  }
  function updateProgress() {
    const done = Math.max(0, currentStep);
    progressText.textContent = (done > 0) ? `Fremdrift: ${done}/${steps.length} spørsmål besvart` : '';
  }

  function addBotRow(content){
    const row = document.createElement('div');
    row.className = 'row bot';

    const img = document.createElement('img');
    img.className = 'avatar bot';
    img.alt = 'easyReq';
    img.src = AVATAR_BOT_URL || FALLBACK_BOT;
    row.appendChild(img);

    const bubble = document.createElement('div');
    bubble.className = 'msg bot';
    if (typeof content === 'string') bubble.innerHTML = content;
    else bubble.appendChild(content);
    row.appendChild(bubble);

    chat.appendChild(row);
    scrollToBottom();
    return bubble;
  }
  function addUserRow(text){
    const row = document.createElement('div');
    row.className = 'row user';

    const bubble = document.createElement('div');
    bubble.className = 'msg user';
    bubble.textContent = text;
    row.appendChild(bubble);

    const img = document.createElement('img');
    img.className = 'avatar user';
    img.alt = 'Besøkende';
    img.src = AVATAR_USER_URL || FALLBACK_USER;
    row.appendChild(img);

    chat.appendChild(row);
    scrollToBottom();
  }

  function typingBubble(){
    const row = document.createElement('div');
    row.className = 'row bot';

    const img = document.createElement('img');
    img.className = 'avatar bot';
    img.alt = 'easyReq';
    img.src = AVATAR_BOT_URL || FALLBACK_BOT;
    row.appendChild(img);

    const bubble = document.createElement('div');
    bubble.className = 'msg bot';
    const t = document.createElement('div');
    t.className = 'typing';
    t.innerHTML = '<span class="dot"></span><span class="dot"></span><span class="dot"></span>';
    bubble.appendChild(t);
    row.appendChild(bubble);

    chat.appendChild(row);
    scrollToBottom();
    return ()=> row.remove();
  }

  async function botSay(html, delayMs = 450){
    const stop = typingBubble();
    await sleep(delayMs);
    stop();
    addBotRow(html);
  }

  function validateNumber(v, cfg){
    if (!Number.isFinite(v)) return false;
    if (typeof cfg.min === 'number' && v < cfg.min) return false;
    if (typeof cfg.max === 'number' && v > cfg.max) return false;
    return true;
  }

  // Flyt
  async function showStart(){
    await botSay('Hei! 👋 La oss sjekke deres potensiale for å spare tid og kostnader.');
    const actions = document.createElement('div');
    actions.className = 'actions';
    const btn = document.createElement('button');
    btn.className = 'btn primary';
    btn.textContent = 'La oss sjekke';
    btn.addEventListener('click', () => askNext());
    actions.appendChild(btn);
    addBotRow('Når du er klar, trykker du under.');
    chat.appendChild(actions);
    scrollToBottom();
  }

  async function askNext(){
    currentStep++;
    updateProgress();
    if (currentStep >= steps.length){
      await computeAndShow();
      return;
    }
    const s = steps[currentStep];

    if (currentStep > 0){
      const acks = ['Supert!', 'Takk! 👍', 'Notert.', 'Flott!', 'Kjempebra!'];
      const ack = acks[Math.floor(Math.random()*acks.length)];
      await botSay(ack, 250);
      await sleep(80);
    }

    if (s.type === 'choice'){
      await botSay(s.question);
      const actions = document.createElement('div');
      actions.className = 'actions';
      s.choices.forEach(([value,label])=>{
        const b = document.createElement('button');
        b.className = 'chip';
        b.textContent = label;
        b.addEventListener('click', async () => {
          answers[s.id] = value;
          addUserRow(label);
          await sleep(150);
          askNext();
        });
        actions.appendChild(b);
      });
      chat.appendChild(actions);
      scrollToBottom();
      return;
    }

    if (s.type === 'number'){
      await botSay(s.question);
      const wrap = document.createElement('div');
      wrap.className = 'inline-form';
      wrap.style.alignSelf = 'flex-end';
      wrap.style.marginRight = 'var(--er-side-pad)';

      // num-field med +/−
      const numField = document.createElement('div');
      numField.className = 'num-field';

      const minus = document.createElement('button');
      minus.className = 'btn round';
      minus.type = 'button';
      minus.setAttribute('aria-label', 'Mindre');
      minus.textContent = '−';

      const input = document.createElement('input');
      input.type = 'number';
      input.inputMode = 'numeric';
      if (s.min !== undefined)  input.min  = String(s.min);
      if (s.max !== undefined)  input.max  = String(s.max);
      if (s.step !== undefined) input.step = String(s.step); // 15 for minutter
      input.value = (answers[s.id] ?? s.value ?? '');
      input.setAttribute('aria-label', s.label);

      const plus = document.createElement('button');
      plus.className = 'btn round';
      plus.type = 'button';
      plus.setAttribute('aria-label', 'Mer');
      plus.textContent = '+';

      function nudge(dir){
        const step = Number(input.step) || 1;
        const min  = Number(input.min || '-Infinity');
        const max  = Number(input.max || 'Infinity');
        let v = Number(input.value);
        if (!Number.isFinite(v)) v = s.value ?? 0;
        v = v + dir * step;
        if (Number.isFinite(min)) v = Math.max(v, min);
        if (Number.isFinite(max)) v = Math.min(v, max);
        input.value = Math.round(v / step) * step;
      }
      minus.addEventListener('click', ()=> nudge(-1));
      plus .addEventListener('click', ()=> nudge(+1));

      const send = document.createElement('button');
      send.className = 'btn primary';
      send.textContent = 'OK';

      async function submit(){
        const v = Number(input.value);
        if (!validateNumber(v, s)){
          input.focus(); input.select?.(); return;
        }
        answers[s.id] = v;
        addUserRow(`${s.label}: ${v}`);
        await sleep(120);
        askNext();
      }
      input.addEventListener('keydown', (e)=>{ if(e.key==='Enter') submit(); });
      send.addEventListener('click', submit);

      numField.appendChild(minus);
      numField.appendChild(input);
      numField.appendChild(plus);

      wrap.appendChild(numField);
      wrap.appendChild(send);

      if (s.help) await botSay(`<span class="meta">${s.help}</span>`, 200);

      chat.appendChild(wrap);
      scrollToBottom();
      return;
    }

    if (s.type === 'chips'){
      await botSay(s.question);
      if (s.help) await botSay(`<span class="meta">${s.help}</span>`, 200);

      const actions = document.createElement('div');
      actions.className = 'actions';

      let selectedVal = s.value; // 50 %
      answers[s.id] = selectedVal;

      function renderChips(){
        actions.querySelectorAll('.chip').forEach(ch => {
          const v = Number(ch.dataset.val);
          ch.classList.toggle('selected', v === selectedVal);
        });
      }

      (s.quick || []).forEach(val=>{
        const b = document.createElement('button');
        b.className = 'chip';
        b.dataset.val = String(val);
        b.textContent = `${val}%`;
        if (val === selectedVal) b.classList.add('selected');
        b.addEventListener('click', async ()=>{
          selectedVal = val;
          answers[s.id] = val;
          renderChips();
          addUserRow(`${s.label}: ${val}%`);
          await sleep(140);
          askNext();
        });
        actions.appendChild(b);
      });

      chat.appendChild(actions);
      renderChips(); // marker 50 % ved start
      scrollToBottom();
      return;
    }
  }

  async function computeAndShow(){
    // Hent svar
    const band = answers['turnover'];

    const B  = Number(answers['vans'] ?? 1);
    const P  = Number(answers['personsPerCar'] ?? 1);
    const T  = Number(answers['tripsPerWeek'] ?? 0);
    const M  = Number(answers['minutesPerTrip'] ?? 60);
    const R  = Number(answers['reductionPct'] ?? 0) / 100;
    const K  = Number(answers['hourlyCost'] ?? 500);

    const WEEKS_PER_YEAR = 46;
    const tripsPerYear   = B * T * WEEKS_PER_YEAR;
    const hoursPerYear   = tripsPerYear * (M / 60) * P;
    const timeCost       = hoursPerYear * K;

    const totalCost = timeCost;
    const savings   = totalCost * R;

    const license = licenseFromTurnover(band);
    const netGain = (license === null) ? null : (savings - license);

    await botSay('Flott – da regner jeg ut basert på svarene dine …', 400);

       // Resultat som vanlig bot-melding (samme stil som chatten)
    const licenseText = (license === null)
      ? 'Abonnement på forespørsel'
      : NOK.format(license);

    const netText = (license === null)
      ? ''
      : (netGain < 0 ? `−${NOK.format(Math.abs(netGain))}` : NOK.format(netGain));

    const netClass = (license !== null && netGain < 0) ? 'neg' : '';

    const html = `
      <div style="font-weight:800; font-size:16px;">Ditt resultat</div>

      <div class="result-lines">
        <div class="result-line">
          <div class="k">Total «tur‑kost» (per år)</div>
          <div class="v">${NOK.format(totalCost)}</div>
        </div>

        <div class="result-line">
          <div class="k">Potensiell besparelse (per år)</div>
          <div class="v">${NOK.format(savings)}</div>
        </div>

        <div class="result-line">
          <div class="k">Typisk lisens (per år)</div>
          <div class="v">${licenseText}</div>
        </div>

        ${license === null ? '' : `
        <div class="result-line">
          <div class="k">Netto gevinst (per år)</div>
          <div class="v ${netClass}">${netText}</div>
        </div>`}
      </div>
    `;

    addBotRow(html);


    // Ekstra bot-linje med kontakttekst
    await botSay(
      'Vil du høre mer? Avtal et møte, send oss en epost eller ring :)',
      300
    );

    if (license === null){
      await botSay('For deres omsetning skreddersyr vi pris (abonnement på forespørsel). Ta kontakt, så finner vi riktig løsning. 🙌', 300);
    }
    
        // CTA – møtebooking + kontaktlinje
    const actions = document.createElement('div');
    actions.className = 'actions';
    const book = document.createElement('a');
    book.className = 'btn primary';
    book.href = 'https://outlook.office.com/book/easyReq1@easyreq.no/?ismsaljsauthenabled';
    book.target = '_blank';
    book.rel = 'noopener noreferrer';
    book.textContent = 'Avtal et møte';

    const email = document.createElement('a');
    email.className = 'chip';
    email.href = 'mailto:ove@easyreq.no';
    email.textContent = 'Send e‑post';

		const phone = document.createElement('a');
		phone.className = 'chip';
		phone.href = 'tel:+4748272245';
		phone.textContent = 'Ring oss: 482 72 245';

		const PHONE_LABEL_DEFAULT = 'Ring oss';
		const PHONE_LABEL_SHOW    = 'Ring oss: 482 72 245';

		phone.addEventListener('click', () => {
  		phone.textContent = (phone.textContent === PHONE_LABEL_SHOW)
    		? PHONE_LABEL_DEFAULT
    		: PHONE_LABEL_SHOW;
		}); // <-- VIKTIG: lukk eventlisteneren her

		const restart = document.createElement('button');
		restart.className = 'chip';
		restart.textContent = 'Start på nytt';
		restart.addEventListener('click', ()=> {
  		chat.innerHTML = '';
  		Object.keys(answers).forEach(k=>delete answers[k]);
  		currentStep = -1;
  		updateProgress();
  		showStart();
		});

    actions.appendChild(book);
    actions.appendChild(email);
    actions.appendChild(phone);
    actions.appendChild(restart);
    chat.appendChild(actions);
    updateProgress();
    
    
  }

  // Start
  (async function init(){ await showStart(); })();
</script>

</body>
</html>
