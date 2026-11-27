/* Sokol Plzeň-Radčice: rezervace a kroužky. MOCK-UP: data se ukládají jen v tomto prohlížeči. */
(function () {
  'use strict';

  // ---------- konfigurace (k ověření se Sokolem) ----------
  var OPEN = 8, CLOSE = 21, PRICE = 100;
  var MOCK_IBAN = 'CZ6508000000192000145399'; // smyšlené číslo účtu pro ukázku QR platby
  var FACILITIES = [
    { id: 'kurt1', name: 'Tenisový kurt 1', short: 'Kurt 1', season: [4, 12] },
    { id: 'kurt2', name: 'Tenisový kurt 2', short: 'Kurt 2', season: [4, 12] },
    { id: 'zed', name: 'Kurt se zdí', short: 'Zeď', season: [4, 12] },
    { id: 'telocvicna', name: 'Tělocvična', short: 'Tělocvična', season: [1, 12] }
  ];
  // Pravidelné kroužky = opakované rezervace typu "Kroužek". Z nich se generuje rozvrh i obsazenost.
  var CLUBS = [
    { slug: 'stolni-tenis', name: 'Stolní tenis', fac: 'telocvicna', sessions: [[3, 18, 20, 'Stolní tenis'], [5, 17, 19, 'Stolní tenis']] },
    { slug: 'tenis', name: 'Tenis', fac: 'kurt1', sessions: [[2, 16, 18, 'Tenis: mládež'], [4, 16, 18, 'Tenis: dospělí']] },
    { slug: 'sport-pro-vsechny', name: 'Sport pro všechny', fac: 'telocvicna', sessions: [[1, 18, 20, 'Sport pro všechny']] },
    { slug: 'cviceni-pro-zeny', name: 'Cvičení pro ženy', fac: 'telocvicna', sessions: [[2, 19, 20, 'Cvičení pro ženy'], [4, 19, 20, 'Cvičení pro ženy']] },
    { slug: 'cviceni-pro-deti', name: 'Cvičení pro děti', fac: 'telocvicna', sessions: [[1, 16, 17, 'Předškoláci 4–6 let'], [1, 17, 18, 'Mladší žáci 6–10 let'], [3, 16, 17, 'Rodiče a děti 2–4 roky'], [3, 17, 18, 'Starší žáci 10–15 let']] },
    { slug: 'aerobik', name: 'Aerobik', fac: 'telocvicna', sessions: [[2, 18, 19, 'Aerobik'], [4, 18, 19, 'Aerobik']] }
  ];
  var DOW = ['Ne', 'Po', 'Út', 'St', 'Čt', 'Pá', 'So'];
  var DOW_LONG = ['neděle', 'pondělí', 'úterý', 'středa', 'čtvrtek', 'pátek', 'sobota'];
  var DOW_EVERY = ['každou neděli', 'každé pondělí', 'každé úterý', 'každou středu', 'každý čtvrtek', 'každý pátek', 'každou sobotu'];

  // ---------- pomocné ----------
  function $(s, r) { return (r || document).querySelector(s); }
  function $$(s, r) { return Array.prototype.slice.call((r || document).querySelectorAll(s)); }
  function pad(n) { return (n < 10 ? '0' : '') + n; }
  function key(d) { return d.getFullYear() + '-' + pad(d.getMonth() + 1) + '-' + pad(d.getDate()); }
  function parse(k) { var p = k.split('-'); return new Date(+p[0], +p[1] - 1, +p[2]); }
  function addDays(d, n) { var x = new Date(d); x.setDate(x.getDate() + n); return x; }
  function monday(d) { var x = new Date(d.getFullYear(), d.getMonth(), d.getDate()); var w = (x.getDay() + 6) % 7; return addDays(x, -w); }
  function human(d) { return d.getDate() + '. ' + (d.getMonth() + 1) + '. ' + d.getFullYear(); }
  function humanShort(d) { return d.getDate() + '. ' + (d.getMonth() + 1) + '.'; }
  function kc(n) { return n.toLocaleString('cs-CZ') + ' Kč'; }
  function esc(s) { return String(s == null ? '' : s).replace(/[&<>"']/g, function (c) { return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]; }); }
  function fac(id) { for (var i = 0; i < FACILITIES.length; i++) if (FACILITIES[i].id === id) return FACILITIES[i]; }
  function inSeason(f, d) { var m = d.getMonth() + 1; return m >= f.season[0] && m <= f.season[1]; }
  function hours(a, b) { return pad(a) + ':00–' + pad(b) + ':00'; }

  function toast(msg) {
    var t = $('#toast'); if (!t) { t = document.createElement('div'); t.id = 'toast'; t.className = 'toast'; t.setAttribute('role', 'status'); document.body.appendChild(t); }
    t.textContent = msg; t.classList.add('show'); clearTimeout(t._h); t._h = setTimeout(function () { t.classList.remove('show'); }, 3600);
  }

  // ---------- úložiště (mock) ----------
  var STORE_KEY = 'sokolRadciceMock.v2';
  var mem = null;
  function load() {
    if (mem) return mem;
    try { mem = JSON.parse(localStorage.getItem(STORE_KEY) || 'null'); } catch (e) { mem = null; }
    if (!mem || !mem.res) mem = seed();
    return mem;
  }
  function save() { try { localStorage.setItem(STORE_KEY, JSON.stringify(mem)); } catch (e) { /* bez úložiště běží demo jen v paměti */ } }
  function nextNo() { mem.counter = (mem.counter || 0) + 1; return mem.counter; }
  function resNo(n) { return 'R-' + new Date().getFullYear() + '-' + ('000' + n).slice(-4); }

  function seed() {
    mem = { res: [], zajemci: [], counter: 0, series: 0 };
    var names = [['Jan Novák', 'jan.novak@example.cz', '+420 601 111 222'], ['Petra Svobodová', 'petra.s@example.cz', '+420 602 333 444'], ['Martin Dvořák', 'm.dvorak@example.cz', '+420 603 555 666'], ['Lucie Černá', 'lucie.cerna@example.cz', '+420 604 777 888'], ['Tomáš Procházka', 't.prochazka@example.cz', '+420 605 999 000'], ['Eva Kučerová', 'eva.k@example.cz', '+420 606 121 314']];
    var t0 = new Date(), mon = new Date(t0.getFullYear(), t0.getMonth(), t0.getDate());
    var plan = [ // [den od pondělí tohoto týdne, hodina, sportoviště, stav, zaplaceno]
      [0, 19, 'kurt1', 'ok', true], [0, 20, 'kurt2', 'ok', true], [1, 10, 'kurt2', 'ok', false], [1, 18, 'zed', 'ok', true],
      [2, 8, 'kurt1', 'ok', true], [2, 15, 'telocvicna', 'ok', false], [3, 19, 'kurt2', 'wait', false], [4, 9, 'kurt1', 'ok', true],
      [4, 18, 'kurt2', 'wait', false], [5, 10, 'kurt1', 'ok', true], [5, 11, 'kurt1', 'ok', true], [5, 14, 'telocvicna', 'wait', false],
      [6, 9, 'kurt2', 'ok', false], [6, 16, 'zed', 'wait', false], [7, 18, 'kurt1', 'ok', false], [8, 10, 'telocvicna', 'wait', false],
      [9, 17, 'kurt2', 'wait', false], [11, 9, 'kurt1', 'ok', false], [12, 15, 'zed', 'wait', false]
    ];
    plan.forEach(function (p, i) {
      var who = names[i % names.length], n = nextNo();
      mem.res.push({ id: resNo(n), vs: n, type: 'res', fac: p[2], date: key(addDays(mon, p[0])), h: p[1], name: who[0], email: who[1], phone: who[2], note: '', status: p[3], paid: p[4], pay: p[4] ? (i % 2 ? 'hotově' : 'převodem') : '', created: key(addDays(mon, p[0] - 3)) });
    });
    // opakovaná série čekající na schválení (pravidelný hráč)
    mem.series = 1;
    for (var w = 0; w < 6; w++) {
      var n2 = nextNo();
      mem.res.push({ id: resNo(n2), vs: n2, series: 'S-' + new Date().getFullYear() + '-001', type: 'res', fac: 'kurt2', date: key(addDays(mon, 2 + 7 * w)), h: 18, name: 'Ondřej Veselý', email: 'ondrej.v@example.cz', phone: '+420 607 454 545', note: 'Pravidelná středeční hra', status: 'wait', paid: false, pay: '', created: key(new Date()) });
    }
    // výluka: údržba kurtu 2
    for (var h = 8; h < 12; h++) mem.res.push({ id: 'V-údržba', type: 'out', fac: 'kurt2', date: key(addDays(mon, 5)), h: h, note: 'Údržba antuky', status: 'ok' });
    mem.zajemci.push({ club: 'cviceni-pro-deti', name: 'Kateřina Malá', email: 'katerina.m@example.cz', phone: '', age: '5 let', note: '', created: key(new Date()) });
    mem.zajemci.push({ club: 'tenis', name: 'Pavel Horák', email: 'p.horak@example.cz', phone: '+420 608 100 200', age: '', note: 'Začátečník', created: key(new Date()) });
    save();
    return mem;
  }

  function clubAt(facId, d, h) {
    var dow = d.getDay();
    for (var i = 0; i < CLUBS.length; i++) {
      var c = CLUBS[i]; if (c.fac !== facId) continue;
      for (var j = 0; j < c.sessions.length; j++) { var s = c.sessions[j]; if (s[0] === dow && h >= s[1] && h < s[2]) return { club: c, label: s[3] }; }
    }
    return null;
  }

  // stav slotu: free | wait | busy | club | off | past | season
  function slotState(facId, d, h) {
    var f = fac(facId), now = new Date();
    if (!inSeason(f, d)) return { s: 'season' };
    var start = new Date(d.getFullYear(), d.getMonth(), d.getDate(), h);
    if (start < now) return { s: 'past' };
    var c = clubAt(facId, d, h); if (c) return { s: 'club', label: c.label, club: c.club };
    var k = key(d), list = load().res;
    for (var i = 0; i < list.length; i++) {
      var r = list[i]; if (r.fac !== facId || r.date !== k || r.h !== h || r.status === 'no') continue;
      if (r.type === 'out') return { s: 'off', label: r.note || 'Uzavřeno' };
      return { s: r.status === 'ok' ? 'busy' : 'wait' };
    }
    return { s: 'free' };
  }

  // ---------- kalendář obsazenosti ----------
  function initBooking() {
    var root = $('#rezervace-app'); if (!root) return;
    load();
    var state = { fac: 'kurt1', week: 0 };
    var tabs = $('.tabs', root);
    tabs.innerHTML = FACILITIES.map(function (f) { return '<button class="tab" role="tab" data-fac="' + f.id + '" aria-selected="' + (f.id === state.fac) + '">' + f.name + '</button>'; }).join('');
    tabs.addEventListener('click', function (e) { var b = e.target.closest('.tab'); if (!b) return; state.fac = b.dataset.fac; $$('.tab', tabs).forEach(function (t) { t.setAttribute('aria-selected', t === b); }); render(); });
    $('#prev', root).addEventListener('click', function () { if (state.week > 0) { state.week--; render(); } });
    $('#next', root).addEventListener('click', function () { if (state.week < 10) { state.week++; render(); } });
    $('#today', root).addEventListener('click', function () { state.week = 0; render(); });
    $$('[data-book]').forEach(function (a) { a.addEventListener('click', function () { var id = a.dataset.book; var t = $('.tab[data-fac="' + id + '"]', tabs); if (t) t.click(); }); });
    window.addEventListener('sokol:changed', render);

    function render() {
      var t0 = new Date(), mon = addDays(new Date(t0.getFullYear(), t0.getMonth(), t0.getDate()), state.week * 7), sun = addDays(mon, 6), f = fac(state.fac);
      $('#weeklabel', root).textContent = humanShort(mon) + ' – ' + human(sun);
      $('#prev', root).disabled = state.week === 0;
      var anyIn = false; for (var i = 0; i < 7; i++) if (inSeason(f, addDays(mon, i))) anyIn = true;
      var box = $('.grid-scroll', root);
      if (!anyIn) { box.innerHTML = '<div class="off-season"><h3>Mimo sezónu</h3><p>' + f.name + ' je v provozu od dubna do prosince. Rezervace na novou sezónu otevřeme v březnu.</p></div>'; return; }
      var today = key(new Date());
      var html = '<table class="cal"><thead><tr><th><span class="sr">Čas</span></th>';
      for (i = 0; i < 7; i++) { var d = addDays(mon, i); html += '<th class="' + (key(d) === today ? 'today' : '') + '">' + DOW[d.getDay()] + '<small>' + humanShort(d) + '</small></th>'; }
      html += '</tr></thead><tbody>';
      for (var h = OPEN; h < CLOSE; h++) {
        html += '<tr><td class="hr">' + pad(h) + ':00</td>';
        for (i = 0; i < 7; i++) {
          d = addDays(mon, i); var st = slotState(state.fac, d, h), lab = pad(h) + ':00 ' + DOW_LONG[d.getDay()] + ' ' + humanShort(d);
          if (st.s === 'free') html += '<td><button class="slot s-free" data-d="' + key(d) + '" data-h="' + h + '" aria-label="Volno ' + lab + ', rezervovat">Volno</button></td>';
          else if (st.s === 'wait') html += '<td><span class="slot s-wait" style="display:grid;place-items:center" title="Čeká na schválení správcem">Předběžně</span></td>';
          else if (st.s === 'busy') html += '<td><span class="slot s-busy" style="display:grid;place-items:center">Obsazeno</span></td>';
          else if (st.s === 'club') html += '<td><a class="slot s-club" style="display:grid;place-items:center;text-decoration:none" href="krouzky/' + st.club.slug + '/" title="' + esc(st.label) + '">' + esc(st.label) + '</a></td>';
          else if (st.s === 'off') html += '<td><span class="slot s-off" style="display:grid;place-items:center" title="' + esc(st.label) + '">' + esc(st.label) + '</span></td>';
          else if (st.s === 'season') html += '<td><span class="slot s-off" style="display:grid;place-items:center">Mimo sezónu</span></td>';
          else html += '<td><span class="slot s-past" style="display:grid;place-items:center">·</span></td>';
        }
        html += '</tr>';
      }
      box.innerHTML = html + '</tbody></table>';
    }
    root.addEventListener('click', function (e) { var b = e.target.closest('.s-free'); if (b) openForm(state.fac, parse(b.dataset.d), +b.dataset.h); });
    render();
  }

  // ---------- rezervační formulář ----------
  var form = null;
  function openForm(facId, d, h) {
    var m = $('#modal'), f = fac(facId);
    form = { fac: facId, d: d, h: h };
    $('.dialog', m).innerHTML =
      '<button class="x" aria-label="Zavřít" data-close>&times;</button>' +
      '<div class="eyebrow">Žádost o rezervaci</div><h3>' + f.name + '</h3>' +
      '<div class="summary"><b>' + DOW_LONG[d.getDay()] + ' ' + human(d) + '</b>, od ' + pad(h) + ':00</div>' +
      '<form id="bookform" novalidate>' +
      '<label>Délka<select name="len" id="len"></select></label>' +
      '<div class="row"><label>Jméno a příjmení<input name="name" required autocomplete="name"></label><label>Telefon<input name="phone" required type="tel" autocomplete="tel" placeholder="+420"></label></div>' +
      '<label>E-mail<input name="email" required type="email" autocomplete="email"></label>' +
      '<div class="repeat" id="rep"><label class="check" style="margin:0"><input type="checkbox" id="repon"> <span>Opakovat ' + DOW_EVERY[d.getDay()] + ' ve stejný čas<br><span class="note">Pro tréninky a pravidelnou hru. Správce schvaluje celou sérii najednou.</span></span></label>' +
      '<div class="body"><label>Opakovat do<input type="date" id="until"></label><div class="note" id="repinfo"></div><ul class="dates" id="replist"></ul></div></div>' +
      '<label>Poznámka (nepovinné)<textarea name="note" rows="2"></textarea></label>' +
      '<input class="hp" name="web" tabindex="-1" autocomplete="off" aria-hidden="true">' +
      '<label class="check"><input type="checkbox" name="rules" required> <span>Souhlasím s <a href="#pravidla" data-close>provozním řádem</a> a beru na vědomí, že kurt před hrou pokropím a po hře zametu.</span></label>' +
      '<p class="note">Osobní údaje zpracovává Sokol Plzeň-Radčice jen pro vyřízení rezervace a platby. Ve veřejném kalendáři se jméno nezobrazuje.</p>' +
      '<div class="price"><span>Cena celkem<br><span class="note" id="pricenote"></span></span><strong id="price"></strong></div>' +
      '<p class="note" id="err" style="color:var(--red-dark)"></p>' +
      '<button class="btn btn-red" type="submit" style="width:100%;justify-content:center;margin-top:6px">Odeslat ke schválení</button>' +
      '</form>';
    var len = $('#len', m), maxLen = 0;
    for (var x = h; x < CLOSE && x < h + 3; x++) { if (slotState(facId, d, x).s !== 'free') break; maxLen++; }
    for (var i = 1; i <= maxLen; i++) len.insertAdjacentHTML('beforeend', '<option value="' + i + '">' + i + (i === 1 ? ' hodina' : ' hodiny') + ' (' + hours(h, h + i) + ')</option>');
    var until = $('#until', m), defU = addDays(d, 7 * 8);
    until.min = key(addDays(d, 7)); until.max = key(new Date(d.getFullYear(), fac(facId).season[1], 0)); until.value = key(defU > parse(until.max) ? parse(until.max) : defU);
    if (facId === 'telocvicna') until.max = key(addDays(d, 7 * 40));
    $('#repon', m).addEventListener('change', function () { $('#rep', m).classList.toggle('on', this.checked); calc(); });
    until.addEventListener('change', calc); len.addEventListener('change', calc);
    $('#bookform', m).addEventListener('submit', submit);
    m.classList.add('open'); document.body.style.overflow = 'hidden';
    setTimeout(function () { $('input[name=name]', m).focus(); }, 50);
    calc();
  }
  function occurrences() {
    var m = $('#modal'), L = +$('#len', m).value, rep = $('#repon', m).checked, out = [];
    var last = rep ? parse($('#until', m).value || key(form.d)) : form.d;
    for (var d = new Date(form.d); d <= last; d = addDays(d, 7)) {
      var ok = true; for (var x = form.h; x < form.h + L; x++) { var s = slotState(form.fac, d, x).s; if (s !== 'free') { ok = false; break; } }
      out.push({ d: new Date(d), ok: ok });
    }
    return out;
  }
  function calc() {
    var m = $('#modal'), L = +$('#len', m).value, occ = occurrences(), good = occ.filter(function (o) { return o.ok; });
    if ($('#repon', m).checked) {
      $('#repinfo', m).textContent = occ.length + ' termínů, z toho ' + (occ.length - good.length) + ' koliduje (budou vynechány).';
      $('#replist', m).innerHTML = occ.map(function (o) { return '<li class="' + (o.ok ? '' : 'bad') + '">' + DOW[o.d.getDay()] + ' ' + humanShort(o.d) + (o.ok ? '' : ' obsazeno') + '</li>'; }).join('');
    }
    $('#price', m).textContent = kc(good.length * L * PRICE);
    $('#pricenote', m).textContent = good.length + ' × ' + L + ' h × ' + PRICE + ' Kč';
  }
  function submit(e) {
    e.preventDefault();
    var m = $('#modal'), fm = e.target, err = $('#err', m), data = {};
    ['name', 'phone', 'email', 'note', 'web'].forEach(function (n) { data[n] = (fm.elements[n].value || '').trim(); });
    if (data.web) return; // honeypot
    if (!data.name || !data.phone || !/^\S+@\S+\.\S+$/.test(data.email)) { err.textContent = 'Vyplňte prosím jméno, telefon a platný e-mail.'; return; }
    if (!fm.elements.rules.checked) { err.textContent = 'Potvrďte prosím souhlas s provozním řádem.'; return; }
    var L = +$('#len', m).value, occ = occurrences().filter(function (o) { return o.ok; });
    if (!occ.length) { err.textContent = 'Žádný z vybraných termínů není volný.'; return; }
    var series = occ.length > 1 ? 'S-' + new Date().getFullYear() + '-' + ('00' + (++mem.series)).slice(-3) : null;
    var first = null, vs = nextNo();
    occ.forEach(function (o) {
      for (var x = form.h; x < form.h + L; x++) {
        var n = first ? nextNo() : vs, r = { id: resNo(n), vs: vs, series: series, type: 'res', fac: form.fac, date: key(o.d), h: x, name: data.name, email: data.email, phone: data.phone, note: data.note, status: 'wait', paid: false, pay: '', created: key(new Date()) };
        if (!first) first = r; mem.res.push(r);
      }
    });
    save(); window.dispatchEvent(new Event('sokol:changed'));
    var total = occ.length * L * PRICE, ref = series || first.id, f = fac(form.fac);
    var spd = 'SPD*1.0*ACC:' + MOCK_IBAN + '*AM:' + total.toFixed(2) + '*CC:CZK*X-VS:' + vs + '*MSG:' + ref.replace(/[^A-Z0-9-]/gi, '') + ' Sokol Radcice';
    $('.dialog', m).innerHTML =
      '<button class="x" aria-label="Zavřít" data-close>&times;</button><div class="done">' +
      '<div class="eyebrow">Žádost odeslána</div><h3>Děkujeme, ' + esc(data.name.split(' ')[0]) + '</h3>' +
      '<div class="badge">Čeká na schválení správcem</div>' +
      '<p>' + esc(f.name) + ', ' + (series ? occ.length + ' termínů, ' + DOW_EVERY[form.d.getDay()] : DOW_LONG[form.d.getDay()] + ' ' + human(form.d)) + ', ' + hours(form.h, form.h + L) + '.<br>Číslo ' + (series ? 'série' : 'rezervace') + ': <b>' + ref + '</b></p>' +
      '<p class="note">Termín je v kalendáři označen jako předběžný. Jakmile ho správce schválí, přijde vám e-mail s potvrzením a platebními údaji.</p></div>' +
      '<div class="mail"><div class="eyebrow" style="margin-bottom:8px">Ukázka e-mailu po schválení</div>' +
      '<div class="qr"><div id="qr"></div><div><b>K úhradě ' + kc(total) + '</b><br>Účet: ' + MOCK_IBAN + ' <span class="note">(smyšlený)</span><br>Variabilní symbol: <b>' + vs + '</b><br><span class="note">Zaplatit lze i v hotovosti pokladníkovi. Při dešti nebo nehratelném kurtu rezervaci zrušíte zdarma.</span></div></div></div>' +
      '<button class="btn btn-ghost" data-close style="margin-top:16px;width:100%;justify-content:center">Zavřít</button>';
    drawQR($('#qr', m), spd, 3);
  }
  function drawQR(el, text, cell) {
    if (!el) return;
    if (typeof qrcode !== 'function') { el.innerHTML = '<span class="note">QR kód</span>'; return; }
    try { var q = qrcode(0, 'M'); q.addData(text); q.make(); el.innerHTML = q.createSvgTag(cell || 3, 0); el.setAttribute('aria-label', 'QR platba'); } catch (e) { el.textContent = 'QR'; }
  }
  function initModal() {
    var m = $('#modal'); if (!m) return;
    function close() { m.classList.remove('open'); document.body.style.overflow = ''; }
    m.addEventListener('click', function (e) { if (e.target === m || e.target.closest('[data-close]')) close(); });
    document.addEventListener('keydown', function (e) { if (e.key === 'Escape' && m.classList.contains('open')) close(); });
  }

  // ---------- kroužky ----------
  function sessionsText(c) {
    return c.sessions.map(function (s) { return DOW[s[0]] + ' ' + s[1] + '–' + s[2]; }).join(', ');
  }
  function initClubs() {
    $$('[data-club-when]').forEach(function (el) { var c = CLUBS.filter(function (x) { return x.slug === el.dataset.clubWhen; })[0]; if (c) el.textContent = sessionsText(c) + ' h · ' + fac(c.fac).name; });
    var tt = $('#rozvrh'); if (!tt) return;
    var order = [1, 2, 3, 4, 5], html = '<table><thead><tr><th>Den</th><th>Tělocvična</th><th>Tenisový kurt 1</th></tr></thead><tbody>';
    order.forEach(function (dow) {
      var cell = { telocvicna: [], kurt1: [] };
      CLUBS.forEach(function (c) { c.sessions.forEach(function (s) { if (s[0] === dow) cell[c.fac].push({ from: s[1], html: '<a class="pill" href="krouzky/' + c.slug + '/">' + s[1] + '–' + s[2] + ' ' + esc(s[3]) + '</a>' }); }); });
      function out(a) { return a.length ? a.sort(function (x, y) { return x.from - y.from; }).map(function (x) { return x.html; }).join('') : '<span class="note">–</span>'; }
      html += '<tr><td>' + DOW_LONG[dow].charAt(0).toUpperCase() + DOW_LONG[dow].slice(1) + '</td><td>' + out(cell.telocvicna) + '</td><td>' + out(cell.kurt1) + '</td></tr>';
    });
    tt.innerHTML = html + '</tbody></table>';
  }
  function initClubDetail() {
    var el = $('[data-club-detail]'); if (!el) return;
    var c = CLUBS.filter(function (x) { return x.slug === el.dataset.clubDetail; })[0]; if (!c) return;
    el.innerHTML = c.sessions.map(function (s) { return '<li><span><b>' + DOW_LONG[s[0]] + '</b><br><span class="note">' + esc(s[3]) + '</span></span><span style="text-align:right">' + hours(s[1], s[2]) + '<br><span class="note">' + fac(c.fac).name + '</span></span></li>'; }).join('');
  }
  function initInterest() {
    $$('form[data-interest]').forEach(function (fm) {
      fm.addEventListener('submit', function (e) {
        e.preventDefault(); load();
        var get = function (n) { return fm.elements[n] ? fm.elements[n].value.trim() : ''; };
        if (get('web')) return;
        if (!get('name') || !/^\S+@\S+\.\S+$/.test(get('email'))) { toast('Vyplňte prosím jméno a platný e-mail.'); return; }
        if (fm.dataset.interest !== 'kontakt') { mem.zajemci.push({ club: fm.dataset.interest, name: get('name'), email: get('email'), phone: get('phone'), age: get('age'), note: get('note'), created: key(new Date()) }); save(); }
        fm.reset(); var ok = fm.parentNode.querySelector('.okmsg'); if (ok) ok.classList.add('show');
      });
    });
  }

  // ---------- Facebook až po kliknutí ----------
  function initFB() {
    var b = $('#fbload'); if (!b) return;
    b.addEventListener('click', function () {
      var box = $('#fbbox');
      box.innerHTML = '<iframe title="Příspěvky Hospůdky Golf na Facebooku" loading="lazy" src="https://www.facebook.com/plugins/page.php?href=' + encodeURIComponent('https://www.facebook.com/HospudkaGolfPlzenRadcice/') + '&tabs=timeline&width=500&height=560&small_header=true&adapt_container_width=true&hide_cover=false&show_facepile=false&locale=cs_CZ" allow="encrypted-media"></iframe>';
    });
  }

  // ---------- správa (ukázka) ----------
  function initAdmin() {
    var root = $('#admin'); if (!root) return;
    load();
    var filter = 'wait';
    $$('[data-filter]', root).forEach(function (b) { b.addEventListener('click', function () { filter = b.dataset.filter; $$('[data-filter]', root).forEach(function (x) { x.classList.toggle('btn-red', x === b); x.classList.toggle('btn-ghost', x !== b); }); render(); }); });
    var vf = $('#vyluka');
    vf.elements.fac.innerHTML = FACILITIES.map(function (f) { return '<option value="' + f.id + '">' + f.name + '</option>'; }).join('');
    vf.elements.date.value = key(addDays(new Date(), 1));
    vf.addEventListener('submit', function (e) {
      e.preventDefault();
      var f = vf.elements.fac.value, d = vf.elements.date.value, a = +vf.elements.from.value, b = +vf.elements.to.value, why = vf.elements.why.value.trim() || 'Uzavřeno', cancelled = 0;
      if (!d || b <= a) { toast('Zkontrolujte datum a čas výluky.'); return; }
      for (var h = a; h < b; h++) {
        mem.res.forEach(function (r) { if (r.type === 'res' && r.fac === f && r.date === d && r.h === h && r.status !== 'no') { r.status = 'no'; r.reason = why; cancelled++; } });
        mem.res.push({ id: 'V-' + d, type: 'out', fac: f, date: d, h: h, note: why, status: 'ok' });
      }
      save(); render(); toast('Výluka uložena.' + (cancelled ? ' Zrušeno rezervací: ' + cancelled + ', hráčům odešel e-mail.' : ''));
    });
    $('#reset', root).addEventListener('click', function () { try { localStorage.removeItem(STORE_KEY); } catch (e) {} mem = null; load(); render(); toast('Ukázková data obnovena.'); });
    root.addEventListener('click', function (e) {
      var b = e.target.closest('[data-act]'); if (!b) return;
      var id = b.dataset.id, ser = b.dataset.series, act = b.dataset.act;
      var group = mem.res.filter(function (r) { return r.type === 'res' && (ser ? r.series === ser : r.vs === +id); });
      if (act === 'ok' || act === 'no') { group.forEach(function (r) { r.status = act; }); toast(act === 'ok' ? 'Schváleno. ' + group[0].name + ' dostal(a) e-mail s QR platbou.' : 'Zamítnuto. Termín se uvolnil, hráč dostal e-mail.'); }
      if (act === 'paid') { group.forEach(function (r) { r.paid = !r.paid; r.pay = r.paid ? 'převodem' : ''; }); }
      save(); render();
    });
    function groups() {
      var map = {}, out = [];
      mem.res.forEach(function (r) {
        if (r.type !== 'res') return;
        var g = r.series || ('v' + r.vs);
        if (!map[g]) { map[g] = { key: g, series: r.series, vs: r.vs, id: r.series || r.id, fac: r.fac, name: r.name, email: r.email, phone: r.phone, note: r.note, status: r.status, paid: r.paid, items: [] }; out.push(map[g]); }
        map[g].items.push(r);
      });
      return out;
    }
    function render() {
      var gs = groups(), now = key(new Date());
      var waiting = gs.filter(function (g) { return g.status === 'wait'; });
      var unpaid = gs.filter(function (g) { return g.status === 'ok' && !g.paid; });
      $('#k-wait').textContent = waiting.length;
      $('#k-ok').textContent = gs.filter(function (g) { return g.status === 'ok'; }).length;
      $('#k-unpaid').textContent = kc(unpaid.reduce(function (s, g) { return s + g.items.length * PRICE; }, 0));
      $('#k-int').textContent = mem.zajemci.length;
      var list = gs.filter(function (g) { return filter === 'all' ? true : filter === 'unpaid' ? (g.status === 'ok' && !g.paid) : g.status === filter; });
      list.sort(function (a, b) { return (a.items[0].date + pad(a.items[0].h)).localeCompare(b.items[0].date + pad(b.items[0].h)); });
      var rows = list.map(function (g) {
        var it = g.items, d0 = parse(it[0].date), dates = {}; it.forEach(function (r) { dates[r.date] = 1; });
        var hrs = it.filter(function (r) { return r.date === it[0].date; }).map(function (r) { return r.h; }), from = Math.min.apply(null, hrs), to = Math.max.apply(null, hrs) + 1, nd = Object.keys(dates).length;
        var when = g.series ? DOW_EVERY[d0.getDay()] + ', ' + nd + '× od ' + humanShort(d0) : DOW[d0.getDay()] + ' ' + humanShort(d0);
        var st = g.status === 'wait' ? '<span class="st st-wait">Čeká</span>' : g.status === 'ok' ? '<span class="st st-ok">Schváleno</span>' : '<span class="st st-no">Zamítnuto</span>';
        var attr = g.series ? 'data-series="' + g.series + '"' : 'data-id="' + g.vs + '"';
        var actions = g.status === 'wait' ? '<button class="btn btn-sm btn-red" data-act="ok" ' + attr + '>Schválit</button> <button class="btn btn-sm btn-ghost" data-act="no" ' + attr + '>Zamítnout</button>'
          : g.status === 'ok' ? '<label class="check" style="margin:0;font-size:.85rem"><input type="checkbox" data-act="paid" ' + attr + (g.paid ? ' checked' : '') + '> zaplaceno</label>' : '';
        return '<tr><td><b>' + g.id + '</b>' + (g.series ? '<br><span class="note">série</span>' : '') + '</td><td>' + fac(g.fac).short + '</td><td>' + when + '<br><span class="note">' + hours(from, to) + '</span></td><td>' + esc(g.name) + '<br><span class="note">' + esc(g.phone) + '</span></td><td>' + kc(it.length * PRICE) + '<br><span class="note">VS ' + g.vs + '</span></td><td>' + st + '</td><td style="white-space:nowrap">' + actions + '</td></tr>';
      }).join('');
      $('#admin-rows').innerHTML = rows || '<tr><td colspan="7" class="note" style="padding:18px">Nic k zobrazení.</td></tr>';
      var names = {}; CLUBS.forEach(function (c) { names[c.slug] = c.name; });
      $('#int-rows').innerHTML = mem.zajemci.map(function (z) { return '<tr><td>' + esc(names[z.club] || z.club) + '</td><td>' + esc(z.name) + '</td><td>' + esc(z.email) + '<br><span class="note">' + esc(z.phone) + '</span></td><td>' + esc(z.age || '–') + '</td><td>' + esc(z.note || '') + '</td><td>' + humanShort(parse(z.created)) + '</td></tr>'; }).join('');
    }
    render();
  }

  // ---------- menu ----------
  function initNav() {
    var b = $('.burger'), n = $('.nav'); if (!b || !n) return;
    b.addEventListener('click', function () { var o = n.classList.toggle('open'); b.setAttribute('aria-expanded', o); });
    n.addEventListener('click', function (e) { if (e.target.tagName === 'A') n.classList.remove('open'); });
  }

  document.addEventListener('DOMContentLoaded', function () {
    initNav(); initModal(); initBooking(); initClubs(); initClubDetail(); initInterest(); initFB(); initAdmin();
    var y = $('#year'); if (y) y.textContent = new Date().getFullYear();
  });
})();
