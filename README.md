# testebt<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Esala — Segunda / Quinta / Domingo</title>
  <style>
    :root{
      --bg:#f4f6f8;
      --card:#064a86;
      --accent:#ffcc00;
      --muted:#6b7280;
      --white:#fff;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial;background:var(--bg);color:#111}
    header{background:#0b324f;color:var(--white);padding:.8rem 1rem;display:flex;align-items:center;justify-content:space-between;gap:.5rem}
    header h1{margin:0;font-size:1rem}
    .controls{display:flex;gap:.4rem;align-items:center}
    .controls button{background:var(--white);border:0;padding:.4rem .6rem;border-radius:6px;cursor:pointer}
    main{max-width:980px;margin:1rem auto;padding:0 1rem}
    .week-label{margin:0 0 .8rem 0;color:var(--muted);font-weight:600}
    .grid{display:grid;grid-template-columns:repeat(3,1fr);gap:.6rem}
    .card{background:linear-gradient(180deg,#0b5aaa 0%, #083d6f 100%);color:#fff;border-radius:8px;padding:.7rem;min-height:160px;display:flex;flex-direction:column;justify-content:space-between}
    .day-head{display:flex;align-items:baseline;justify-content:space-between;gap:.5rem}
    .day-name{font-weight:900;letter-spacing:.6px}
    .day-date{font-size:.95rem;opacity:.95}
    .field{background:rgba(255,255,255,0.06);padding:.45rem .5rem;border-radius:6px;margin-top:.5rem;display:flex;flex-direction:column;gap:.25rem}
    .field label{font-size:.75rem;font-weight:700;opacity:.95}
    .field input{border:0;padding:.45rem .5rem;border-radius:6px;background:rgba(255,255,255,0.04);color:#fff;font-weight:700}
    .field input::placeholder{color:rgba(255,255,255,0.6);font-weight:600}
    .actions{display:flex;gap:.5rem;align-items:center}
    .small-btn{background:var(--white);color:#083d6f;border:0;padding:.4rem .55rem;border-radius:6px;cursor:pointer;font-weight:700}
    footer{padding:1rem;text-align:center;color:var(--muted);font-size:.9rem}
    @media (max-width:900px){
      .grid{grid-template-columns:repeat(2,1fr)}
    }
    @media (max-width:520px){
      .grid{grid-template-columns:1fr}
    }
  </style>
</head>
<body>
  <header>
    <h2> TESTE ESCALA SEMANAL </h2> 
    <div class="controls" aria-hidden="false">
      <button id="prevWeek" title="Semana anterior">◀</button>
      <button id="todayWeek" title="Semana atual">Hoje</button>
      <button id="nextWeek" title="Próxima semana">▶</button>
    </div>
  </header>

  <main>
    <div id="weekLabel" class="week-label">Semana</div>

    <div class="grid" id="cards">
      <!-- Cards gerados por JS -->
    </div>

    <div style="margin-top:.75rem;display:flex;gap:.5rem;align-items:center;flex-wrap:wrap">
      <button id="clearWeek" class="small-btn">Limpar essa semana</button>
      <button id="exportJSON" class="small-btn">Exportar (JSON)</button>
      <button id="importJSON" class="small-btn">Importar (cole JSON)</button>
      <span style="margin-left:auto;color:var(--muted);font-size:.9rem">Salvo localmente no navegador</span>
    </div>
  </main>

  <footer>
    Modelo simples — toque nos campos para editar. Alterações salvas automaticamente.
  </footer>

  <script>
    (function(){
      // Dias que queremos: Monday (0), Thursday (3), Sunday (6) relative to Monday
      const PICKS = [
        {key:'Segunda', offset:0},
        {key:'Quinta',  offset:3},
        {key:'Domingo', offset:6}
      ];
      const STORAGE_PREFIX = 'escala_adtc_v1_'; // + date (YYYY-MM-DD)
      const WEEK_KEY_PREFIX = 'escala_weekmeta_v1_'; // for week metadata if needed

      const cardsEl = document.getElementById('cards');
      const weekLabelEl = document.getElementById('weekLabel');
      const prevBtn = document.getElementById('prevWeek');
      const nextBtn = document.getElementById('nextWeek');
      const todayBtn = document.getElementById('todayWeek');
      const clearWeekBtn = document.getElementById('clearWeek');
      const exportBtn = document.getElementById('exportJSON');
      const importBtn = document.getElementById('importJSON');

      // Utilities
      function getMonday(d){
        const date = new Date(d);
        const day = (date.getDay() + 6) % 7; // 0 = Monday
        date.setDate(date.getDate() - day);
        date.setHours(0,0,0,0);
        return date;
      }
      function addDays(d,n){
        const x = new Date(d);
        x.setDate(x.getDate() + n);
        return x;
      }
      function ymd(date){
        const y = date.getFullYear();
        const m = String(date.getMonth()+1).padStart(2,'0');
        const day = String(date.getDate()).padStart(2,'0');
        return `${y}-${m}-${day}`;
      }
      function dmSlash(date){
        return String(date.getDate()).padStart(2,'0') + '/' + String(date.getMonth()+1).padStart(2,'0');
      }

      let currentMonday = getMonday(new Date());

      function storageKeyForDate(dateStr){
        return STORAGE_PREFIX + dateStr;
      }

      function loadDay(dateStr){
        try {
          const raw = localStorage.getItem(storageKeyForDate(dateStr));
          if (!raw) return {};
          return JSON.parse(raw);
        } catch(e){
          return {};
        }
      }
      function saveDay(dateStr, obj){
        // if empty -> remove
        if (!obj || Object.keys(obj).length === 0){
          localStorage.removeItem(storageKeyForDate(dateStr));
          return;
        }
        localStorage.setItem(storageKeyForDate(dateStr), JSON.stringify(obj));
      }

      function clearWeek(monday){
        for (let i=0;i<7;i++){
          const k = ymd(addDays(monday,i));
          localStorage.removeItem(storageKeyForDate(k));
        }
      }

      function buildCards(){
        cardsEl.innerHTML = '';
        const dates = PICKS.map(p => {
          const d = addDays(currentMonday, p.offset);
          return {name: p.key, date: d, dateStr: ymd(d)};
        });

        dates.forEach(item => {
          const card = document.createElement('div');
          card.className = 'card';
          card.dataset.date = item.dateStr;

          const head = document.createElement('div');
          head.className = 'day-head';
          const nameEl = document.createElement('div');
          nameEl.className = 'day-name';
          nameEl.textContent = item.name.toUpperCase();
          const dateEl = document.createElement('div');
          dateEl.className = 'day-date';
          dateEl.textContent = dmSlash(item.date);
          head.appendChild(nameEl);
          head.appendChild(dateEl);

          const body = document.createElement('div');

          // DIRIGENTE field
          const f1 = document.createElement('div');
          f1.className = 'field';
          const lbl1 = document.createElement('label');
          lbl1.textContent = 'DIRIGENTE';
          const input1 = document.createElement('input');
          input1.type = 'text';
          input1.placeholder = 'toque para preencher';
          input1.dataset.role = 'DIRIGENTE';
          f1.appendChild(lbl1);
          f1.appendChild(input1);

          // PORTEIRO field
          const f2 = document.createElement('div');
          f2.className = 'field';
          const lbl2 = document.createElement('label');
          lbl2.textContent = 'PORTEIRO';
          const input2 = document.createElement('input');
          input2.type = 'text';
          input2.placeholder = 'toque para preencher';
          input2.dataset.role = 'PORTEIRO';
          f2.appendChild(lbl2);
          f2.appendChild(input2);

          body.appendChild(f1);
          body.appendChild(f2);

          card.appendChild(head);
          card.appendChild(body);

          // load saved values
          const data = loadDay(item.dateStr);
          input1.value = data.DIRIGENTE || '';
          input2.value = data.PORTEIRO || '';

          // events: save on blur and Enter
          [input1, input2].forEach(inp => {
            inp.addEventListener('blur', () => {
              const dayKey = card.dataset.date;
              const cur = loadDay(dayKey);
              const role = inp.dataset.role;
              const val = inp.value.trim();
              if (val === '') {
                delete cur[role];
              } else {
                cur[role] = val;
              }
              saveDay(dayKey, cur);
            });
            inp.addEventListener('keydown', (ev) => {
              if (ev.key === 'Enter') {
                ev.preventDefault();
                inp.blur();
              }
            });
          });

          cardsEl.appendChild(card);
        });

        // update week label
        const sunday = addDays(currentMonday,6);
        weekLabelEl.textContent = `Semana: ${dmSlash(currentMonday)} — ${dmSlash(sunday)}`;
      }

      // navigation
      prevBtn.addEventListener('click', () => {
        currentMonday = addDays(currentMonday, -7);
        buildCards();
      });
      nextBtn.addEventListener('click', () => {
        currentMonday = addDays(currentMonday, 7);
        buildCards();
      });
      todayBtn.addEventListener('click', () => {
        currentMonday = getMonday(new Date());
        buildCards();
      });

      clearWeekBtn.addEventListener('click', () => {
        if (!confirm('Limpar todos os responsáveis desta semana? Essa ação não pode ser desfeita.')) return;
        clearWeek(currentMonday);
        buildCards();
      });

      exportBtn.addEventListener('click', () => {
        // collect data for the three dates
        const payload = {};
        PICKS.forEach(p => {
          const dstr = ymd(addDays(currentMonday, p.offset));
          const data = loadDay(dstr);
          if (Object.keys(data).length) payload[dstr] = data;
        });
        const text = JSON.stringify(payload, null, 2);
        // copy to clipboard if available, otherwise show prompt
        if (navigator.clipboard && navigator.clipboard.writeText) {
          navigator.clipboard.writeText(text).then(()=> {
            alert('JSON copiado para a área de transferência.');
          }, ()=> {
            prompt('Copie o JSON abaixo:', text);
          });
        } else {
          prompt('Copie o JSON abaixo:', text);
        }
      });

      importBtn.addEventListener('click', () => {
        const raw = prompt('Cole o JSON para importar (substitui apenas as datas incluídas no JSON):');
        if (!raw) return;
        try {
          const obj = JSON.parse(raw);
          // iterate keys (dates)
          Object.keys(obj).forEach(dateStr => {
            const data = obj[dateStr];
            // only keep DIRIGENTE and PORTEIRO if present
            const keep = {};
            if (data.DIRIGENTE) keep.DIRIGENTE = String(data.DIRIGENTE);
            if (data.PORTEIRO) keep.PORTEIRO = String(data.PORTEIRO);
            if (Object.keys(keep).length) {
              saveDay(dateStr, keep);
            } else {
              // if nothing, remove
              localStorage.removeItem(storageKeyForDate(dateStr));
            }
          });
          buildCards();
          alert('Importação concluída.');
        } catch(e){
          alert('JSON inválido.');
        }
      });

      // init
      buildCards();
    })();
  </script>
</body>
</html>
