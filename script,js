// ── Config ────────────────────────────────────────────────────────────────────
const SUPABASE_URL     = 'https://jsvgschahauhaghimxru.supabase.co';
const SUPABASE_KEY     = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImpzdmdzY2hhaGF1aGFnaGlteHJ1Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzgzOTExNTIsImV4cCI6MjA5Mzk2NzE1Mn0.ullYQgYarYfnHp_ByETjNZ0BIkI15kxn5DZ9U0-qyjs';
const DISCORD_WEBHOOK  = 'https://discordapp.com/api/webhooks/1502935423150723183/Ri6dKg-JgjjCBcewx7Z0bXaqTGKW0ITlR_u1zhECip6cv-8dFeby8isZuDxm3qjYMgSd';

const db = supabase.createClient(SUPABASE_URL, SUPABASE_KEY);

// ── Priority helpers ──────────────────────────────────────────────────────────
const PRIORITY_COLOR = { Low: 0x23a55a, Medium: 0xf0b132, High: 0xf23f42 };
const PRIORITY_DOT   = { Low: '🟢', Medium: '🟡', High: '🔴' };
const PRIORITY_CLASS  = { Low: 'low', Medium: 'med', High: 'high' };

// ── Pre-select priority from URL param (?priority=High etc.) ──────────────────
const urlPriority = new URLSearchParams(location.search).get('priority');
if (urlPriority) {
  const radio = document.querySelector(`input[name="priority"][value="${urlPriority}"]`);
  if (radio) radio.checked = true;
}

// ── DOM refs ──────────────────────────────────────────────────────────────────
const form          = document.getElementById('bugForm');
const submitBtn     = document.getElementById('submitBtn');
const successScreen = document.getElementById('successScreen');

// ── Inline validation ─────────────────────────────────────────────────────────
function validate() {
  let ok = true;
  [
    ['username',    'username-error',    v => v.trim().length > 0],
    ['category',    'category-error',    v => v !== ''],
    ['description', 'description-error', v => v.trim().length >= 20],
  ].forEach(([id, errId, test]) => {
    const el  = document.getElementById(id);
    const err = document.getElementById(errId);
    if (!test(el.value)) {
      el.classList.add('error');
      err.classList.add('show');
      ok = false;
    } else {
      el.classList.remove('error');
      err.classList.remove('show');
    }
  });
  return ok;
}

['username', 'category', 'description'].forEach(id => {
  document.getElementById(id).addEventListener('input', () => {
    document.getElementById(id).classList.remove('error');
    document.getElementById(id + '-error').classList.remove('show');
  });
});

// ── Submit ────────────────────────────────────────────────────────────────────
form.addEventListener('submit', async e => {
  e.preventDefault();
  if (!validate()) return;

  const user  = document.getElementById('username').value.trim();
  const cat   = document.getElementById('category').value;
  const pri   = document.querySelector('input[name="priority"]:checked').value;
  const desc  = document.getElementById('description').value.trim();
  const steps = document.getElementById('steps').value.trim();

  submitBtn.classList.add('loading');
  submitBtn.disabled = true;

  try {
    // 1. Save to Supabase ─────────────────────────────────────────────────────
    const { error: dbErr } = await db.from('bug_reports').insert([{
      discord_username: user,
      category:         cat,
      priority:         pri,
      description:      desc,
      steps:            steps || null,
    }]);

    if (dbErr) throw new Error(dbErr.message);

    // 2. Fire Discord webhook (fire-and-forget) ────────────────────────────────
    const embed = {
      embeds: [{
        title:  `${PRIORITY_DOT[pri]} New Bug Report — ${cat}`,
        color:  PRIORITY_COLOR[pri],
        fields: [
          { name: 'Reported by', value: user, inline: true },
          { name: 'Priority',    value: pri,  inline: true },
          { name: 'Category',    value: cat,  inline: true },
          { name: 'Description', value: desc.slice(0, 1024) },
          ...(steps ? [{ name: 'Steps to Reproduce', value: steps.slice(0, 1024) }] : []),
        ],
        footer:    { text: 'Bug Reporting Portal' },
        timestamp: new Date().toISOString(),
      }],
    };

    fetch(DISCORD_WEBHOOK, {
      method:  'POST',
      headers: { 'Content-Type': 'application/json' },
      body:    JSON.stringify(embed),
    }).catch(() => {/* Don't block UI if Discord fails */});

    // 3. Show success screen ───────────────────────────────────────────────────
    const now  = new Date();
    const time = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }) +
                 ' · ' + now.toLocaleDateString([], { month: 'short', day: 'numeric' });

    document.getElementById('s-user').textContent = user;
    document.getElementById('s-cat').textContent  = cat;
    document.getElementById('s-time').textContent = time;

    const pClass = PRIORITY_CLASS[pri];
    const dot    = PRIORITY_DOT[pri];
    document.getElementById('s-pri').innerHTML = `<span class="p-tag ${pClass}">${dot} ${pri}</span>`;

    form.style.display = 'none';
    successScreen.classList.add('show');

  } catch (err) {
    console.error('Submission error:', err);
    alert('Something went wrong. Please try again.\n\n' + err.message);
  } finally {
    submitBtn.classList.remove('loading');
    submitBtn.disabled = false;
  }
});

// ── Reset ─────────────────────────────────────────────────────────────────────
document.getElementById('resetBtn').addEventListener('click', () => {
  form.reset();
  form.style.display = 'block';
  successScreen.classList.remove('show');
  document.getElementById('p-med').checked = true;
});
  
