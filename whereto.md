<%*
/* Это шаблон вставки событий в заметку — лену событий [[../../ВСЁ_ОСТАЛЬНОЕ/Накопитель/Whereto_Куда|Whereto_Куда]], которая в свою очередь вставлена на [[Дашборд_холстом]]. */
/* ===== БЛОК 1. ТЕКСТОВАЯ АНКЕТА ===== */

const raw = await tp.system.prompt(
  "SHIFT+ENTER — перенос строки! ENTER закончит ввод! \n" +
  "\n" +
  "1: Название события\n" +
  "2: URL (https://…)\n" +
  "3: Текст ссылки (например: Билеты)\n" +
  "4: Задача\n" +
  "5: Важные примечания",
  "",
  true,
  true
);

if (!raw) {
  tR += "";
  return;
}

const lines = raw.split("\n");

let title     = (lines[0] || "").trim();
let linkUrl   = (lines[1] || "").trim();
let linkText  = (lines[2] || "").trim();
let todo      = (lines[3] || "").trim();
let important = (lines[4] || "").trim();

important = important
  .split("\n")
  .map(l => l.trim())
  .filter(l => l.length > 0)
  .join(" ");

/* ===== БЛОК 2. ДИНАМИЧЕСКИЙ КАЛЕНДАРЬ НА 13 МЕСЯЦЕВ ВПЕРЁД ===== */

// Текущие год и месяц (1–12)
const nowYear  = Number(tp.date.now("YYYY"));
const nowMonth = Number(tp.date.now("M"));

// Полные и короткие названия месяцев
const monthNamesFull = [
  "Январь",
  "Февраль",
  "Март",
  "Апрель",
  "Май",
  "Июнь",
  "Июль",
  "Август",
  "Сентябрь",
  "Октябрь",
  "Ноябрь",
  "Декабрь",
];

const monthNamesShort = [
  "янв",
  "февр",
  "март",
  "апр",
  "май",
  "июн",
  "июл",
  "авг",
  "сент",
  "окт",
  "ноя",
  "дек",
];

// Собираем 13 месяцев от текущего включительно
let monthLabels = [];
let monthMeta   = []; // { year, month }

for (let i = 0; i < 13; i++) {
  const idx   = nowMonth - 1 + i;
  const year  = nowYear + Math.floor(idx / 12);
  const month = (idx % 12) + 1;

  // Подпись: для текущего года только месяц, для будущего — месяц + год
  let label = monthNamesFull[month - 1];
  if (year > nowYear) {
    label = `${label} ${year}`;
  }

  monthLabels.push(label);
  monthMeta.push({ year, month });
}

// Выбор месяца
const pickedMonthInfo = await tp.system.suggester(
  monthLabels,
  monthMeta
);

let dateISO   = "";
let dateShort = "";
let weekday   = "";

// Карта дней недели
const weekdayMap = {
  "Mon": "Пн",
  "Tue": "Вт",
  "Wed": "Ср",
  "Thu": "Чт",
  "Fri": "Пт",
  "Sat": "Сб",
  "Sun": "Вс",
};

if (pickedMonthInfo) {
  const { year: pickedYear, month: pickedMonth } = pickedMonthInfo;

  const daysInMonth = new Date(pickedYear, pickedMonth, 0).getDate();

  const pad2    = (n) => (n < 10 ? "0" + n : "" + n);
  const makeISO = (year, m, d) => `${year}-${pad2(m)}-${pad2(d)}`;

  let dateOptionsLabels = [];
  let dateOptionsValues = [];

  for (let d = 1; d <= daysInMonth; d++) {
    const iso = makeISO(pickedYear, pickedMonth, d);

    const ddd     = tp.date.now("ddd", 0, iso, "YYYY-MM-DD");
    const shortRu = weekdayMap[ddd] || ddd;

    const label = tp.date.now("DD MMMM", 0, iso, "YYYY-MM-DD") + " — " + shortRu;
    dateOptionsLabels.push(label);
    dateOptionsValues.push(d);
  }

  const pickedDay = await tp.system.suggester(dateOptionsLabels, dateOptionsValues);

  if (pickedDay != null) {
    dateISO = makeISO(pickedYear, pickedMonth, pickedDay);

    const ddd = tp.date.now("ddd", 0, dateISO, "YYYY-MM-DD");
    weekday   = weekdayMap[ddd] || ddd;

    const dayNum     = Number(tp.date.now("D", 0, dateISO, "YYYY-MM-DD"));
    const monthIdx   = pickedMonth - 1;
    const shortMonth = monthNamesShort[monthIdx] || tp.date.now("MMM", 0, dateISO, "YYYY-MM-DD");

    // Пример: 12 янв
    dateShort = `${dayNum} ${shortMonth}`;

    // точечно: в выводе в заметке "май" -> "мая"
    if (dateShort.endsWith(" май")) {
      dateShort = dateShort.replace(" май", " мая");
    }
  }
}

/* ===== БЛОК 3. ВВОД ВРЕМЕНИ + ЭМОДЗИ ЦИФЕРБЛАТ ===== */

let timeRaw = await tp.system.prompt(
  "Время события.\n" +
  "Примеры: 12 → 12:00, 945 → 09:45, 1245 → 12:45, 12:30 → 12:30.\n" +
  "Можно оставить пустым.",
  ""
);

let time = "";

if (timeRaw) {
  let t = timeRaw.trim();

  if (t.includes(":")) {
    time = t;
  } else {
    const digits = t.replace(/\D/g, "");

    if (digits.length === 1 || digits.length === 2) {
      const h = digits.padStart(2, "0");
      time = `${h}:00`;
    } else if (digits.length === 3) {
      const h = digits.slice(0, 1).padStart(2, "0");
      const m = digits.slice(1, 3);
      time = `${h}:${m}`;
    } else if (digits.length === 4) {
      const h = digits.slice(0, 2);
      const m = digits.slice(2, 4);
      time = `${h}:${m}`;
    } else {
      time = t;
    }
  }
}

// Массив эмодзи циферблатов по 30 минут
const clockEmojis = ["🕛","🕧","🕐","🕜","🕑","🕝","🕒","🕞","🕓","🕟","🕔","🕠","🕕","🕡","🕖","🕢","🕗","🕣","🕘","🕤","🕙","🕥","🕚","🕦"];

let clockEmoji = "";

if (time) {
  const match = time.match(/^(\d{1,2}):(\d{2})$/);
  if (match) {
    const h = Number(match[1]);
    const m = Number(match[2]);

    const idx = Number((2 * (h + m / 60)).toFixed()) % 24;
    clockEmoji = clockEmojis[idx];
  }
}

/* ===== БЛОК 4. СБОРКА ВЫВОДА ===== */

let outLines = [];

// Заголовок события (H5)
if (title) {
  outLines.push("##### " + title);
}

// Дата / день недели / время (+ эмодзи, если есть)
let dateLine = [];
if (dateShort) dateLine.push("**`__" + dateShort + "`**");
if (weekday)   dateLine.push(weekday);
if (time) {
  if (clockEmoji) {
    dateLine.push(clockEmoji + " " + time);
  } else {
    dateLine.push(time);
  }
}
if (dateLine.length > 0) {
  outLines.push(dateLine.join(" "));
}

// Задача
if (todo) {
  if (!todo.trim().startsWith("- [ ]")) {
    todo = "- [ ] " + todo;
  }
  outLines.push(todo);
}

// Ссылка:
// если есть текст ссылки → [Текст: обрезанный_URL](полный_URL)
// если текста нет → [обрезанный_URL](полный_URL)
if (linkUrl) {
  const snippet = linkUrl.slice(0, 15);

  let visible = snippet;
  if (linkText) {
    visible = `${linkText}: ${snippet}`;
  }

  outLines.push(`[${visible}](${linkUrl})`);
}

// Важные примечания
if (important) {
  outLines.push("Важно: " + important);
}

tR += outLines.join("\n");
-%>
