<%*
// Шаблон для заметки-задачи со вставкой напоминания
// 1. Ввод задачи
let task = await tp.system.prompt("✏️ Введите задачу") || "";

// Формируем название файла
let filename;
if (!task) {
    const now = new Date();
    const year = now.getFullYear();
    const month = String(now.getMonth() + 1).padStart(2, '0');
    const day = String(now.getDate()).padStart(2, '0');
    const hours = String(now.getHours()).padStart(2, '0');
    const minutes = String(now.getMinutes()).padStart(2, '0');
    filename = `${year} ${month} ${day} ${hours}${minutes}`;
} else {
    filename = task.length > 25 ? task.substring(0, 25) : task;
}

// Переименовываем файл, если он Untitled
if (tp.file.title.startsWith("Untitled")) {
    await tp.file.rename(filename);
}

// 2. Выбор даты напоминания
const datePreset = await tp.system.suggester(
    ["Сегодня", "Завтра", "Через неделю", "Выбрать дату..."],
    [
        tp.date.now("YYYY-MM-DD"),
        tp.date.now("YYYY-MM-DD", 1),
        tp.date.now("YYYY-MM-DD", 7),
        null
    ],
    false,
    "⏰ Дата напоминания"
);

const reminderDate = datePreset || await tp.system.prompt("📅 Дата (ГГГГ-ММ-ДД)", tp.date.now("YYYY-MM-DD"));

// 3. Выбор времени напоминания
const now = new Date();
now.setMinutes(now.getMinutes() + 1);
const defaultTime = `${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;

const timeChoice = await tp.system.suggester(
    [defaultTime + " (через 1 мин)", "08:00", "09:00", "12:00", "15:00", "18:00", "21:00", "Другое..."],
    [defaultTime, "08:00", "09:00", "12:00", "15:00", "18:00", "21:00", null],
    false,
    "🕒 Время напоминания"
);

const reminderTime = timeChoice || await tp.system.prompt("⏱️ Время (ЧЧ:ММ)", defaultTime);

// Формируем содержимое
tR += "---\n";
tR += `создано: ${tp.date.now("YYYY-MM-DD")}\n`;
tR += "---\n";

tR += "- [ ] ";
tp.file.cursor(1); // <-- Курсор фиксируется именно здесь
%><%*
// Здесь продолжается JS-логика
if (task) {
    tR += task;
}
if (reminderDate && reminderTime) {
    tR += ` (@${reminderDate} ${reminderTime})`;
}
tR += "\n";
%>