<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Quiz Game</title>
  <style>
    :root{--bg:#0f172a;--card:#111827;--muted:#9ca3af;--accent:#10b981}
    html,body{height:100%;margin:0;font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial}
    body{background:linear-gradient(180deg,#071033 0%, #0b1220 100%);display:flex;align-items:center;justify-content:center;padding:24px;color:#e6eef8}
    .card{background:rgba(255,255,255,0.03);backdrop-filter:blur(6px);width:760px;max-width:100%;border-radius:14px;padding:26px;box-shadow:0 8px 30px rgba(2,6,23,0.7)}
    h1{margin:0 0 6px;font-size:22px}
    p.lead{margin:0 0 18px;color:var(--muted)}
    .question{font-size:18px;margin:16px 0}
    .options{display:grid;grid-template-columns:1fr 1fr;gap:12px}
    button.opt{background:#0b1220;border:1px solid rgba(255,255,255,0.04);padding:14px;border-radius:10px;text-align:left;cursor:pointer;font-size:15px}
    button.opt:hover{transform:translateY(-2px);box-shadow:0 6px 18px rgba(2,6,23,0.6)}
    .letter{display:inline-block;width:28px;height:28px;border-radius:6px;background:rgba(255,255,255,0.04);text-align:center;line-height:28px;margin-right:10px;font-weight:700}
    .meta{display:flex;justify-content:space-between;align-items:center;margin-top:18px}
    .feedback{margin-top:14px;font-weight:600}
    .score{font-weight:700}
    .controls{display:flex;gap:10px}
    .btn{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:10px 14px;border-radius:8px;cursor:pointer}
    .btn.primary{background:linear-gradient(90deg,var(--accent),#06b6d4);color:#021018;border:1px solid rgba(255,255,255,0.06)}
    footer{margin-top:16px;color:var(--muted);font-size:13px}
    .hidden{display:none}
  </style>
</head>
<body>
  <div class="card" role="application">
    <h1>🎉 Quiz Game</h1>
    <p class="lead">Answer the multiple choice questions. Click an option to submit.</p>

    <div id="quiz">
      <div id="qmeta" class="meta">
        <div class="progress"><span id="qindex">1</span>/<span id="qtotal">4</span></div>
        <div class="score">Score: <span id="score">0</span></div>
      </div>

      <div id="questionArea">
        <div id="question" class="question">Loading...</div>
        <div class="options" id="options"></div>
        <div id="feedback" class="feedback"></div>
      </div>

      <div class="controls" style="margin-top:18px">
        <button id="nextBtn" class="btn" disabled>Next</button>
        <button id="restartBtn" class="btn" style="display:none">Restart</button>
      </div>

      <footer id="final" class="hidden">🏆 You scored <span id="finalScore"></span>/<span id="finalTotal"></span></footer>
    </div>
  </div>

<script>
// Questions translated from the original Python version
const questions = [
  {
    question: "What is the capital of France?",
    options: ["A) Berlin", "B) Madrid", "C) Paris", "D) Rome"],
    answer: "C"
  },
  {
    question: "Which language is used for web apps?",
    options: ["A) Python", "B) JavaScript", "C) HTML", "D) All of the above"],
    answer: "D"
  },
  {
    question: "Who developed Python?",
    options: ["A) Dennis Ritchie", "B) James Gosling", "C) Guido van Rossum", "D) Bjarne Stroustrup"],
    answer: "C"
  },
  {
    question: "Which planet is known as the Red Planet?",
    options: ["A) Earth", "B) Venus", "C) Mars", "D) Jupiter"],
    answer: "C"
  }
];

let index = 0;
let score = 0;
const total = questions.length;

const qindexEl = document.getElementById('qindex');
const qtotalEl = document.getElementById('qtotal');
const scoreEl = document.getElementById('score');
const questionEl = document.getElementById('question');
const optionsEl = document.getElementById('options');
const feedbackEl = document.getElementById('feedback');
const nextBtn = document.getElementById('nextBtn');
const restartBtn = document.getElementById('restartBtn');
const finalEl = document.getElementById('final');
const finalScoreEl = document.getElementById('finalScore');
const finalTotalEl = document.getElementById('finalTotal');

qtotalEl.textContent = total;

function renderQuestion() {
  feedbackEl.textContent = '';
  nextBtn.disabled = true;
  const q = questions[index];
  qindexEl.textContent = index + 1;
  questionEl.textContent = q.question;
  optionsEl.innerHTML = '';

  q.options.forEach(optText => {
    const letter = optText.trim().slice(0,1);
    const text = optText.slice(3).trim();

    const btn = document.createElement('button');
    btn.className = 'opt';
    btn.setAttribute('data-letter', letter);
    btn.innerHTML = `<span class="letter">${letter}</span><span class="opt-text">${text}</span>`;

    btn.addEventListener('click', () => selectOption(btn, q.answer));
    optionsEl.appendChild(btn);
  });
}

function selectOption(btn, correct) {
  if (nextBtn.disabled === false) return; // prevent double-submits

  const selected = btn.getAttribute('data-letter');
  const allButtons = optionsEl.querySelectorAll('button.opt');

  // disable all buttons
  allButtons.forEach(b => b.disabled = true);

  if (selected === correct) {
    btn.style.border = '1px solid rgba(16,185,129,0.8)';
    feedbackEl.textContent = '✅ Correct!';
    score += 1;
    scoreEl.textContent = score;
  } else {
    btn.style.border = '1px solid rgba(239,68,68,0.8)';
    feedbackEl.textContent = `❌ Wrong! The correct answer was ${correct}.`;

    // highlight the correct one
    const correctBtn = Array.from(allButtons).find(b => b.getAttribute('data-letter') === correct);
    if (correctBtn) correctBtn.style.border = '1px solid rgba(16,185,129,0.8)';
  }

  nextBtn.disabled = false;
}

nextBtn.addEventListener('click', () => {
  index += 1;
  if (index >= total) {
    finishQuiz();
    return;
  }
  renderQuestion();
});

restartBtn.addEventListener('click', () => {
  index = 0; score = 0; scoreEl.textContent = score; finalEl.classList.add('hidden'); restartBtn.style.display = 'none'; nextBtn.style.display = '';
  renderQuestion();
});

function finishQuiz() {
  questionEl.textContent = '';
  optionsEl.innerHTML = '';
  feedbackEl.textContent = '';
  finalScoreEl.textContent = score;
  finalTotalEl.textContent = total;
  finalEl.classList.remove('hidden');
  nextBtn.style.display = 'none';
  restartBtn.style.display = '';
}

// keyboard shortcuts: A/B/C/D
window.addEventListener('keydown', (e) => {
  const key = e.key.toUpperCase();
  if (!['A','B','C','D'].includes(key)) return;
  const btn = Array.from(optionsEl.querySelectorAll('button.opt')).find(b => b.getAttribute('data-letter') === key);
  if (btn) btn.click();
});

// start
renderQuestion();
</script>
</body>
</html>
# Task
