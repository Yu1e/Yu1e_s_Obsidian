<%*
// Шаблон открывает список выносок с иконками и подсказкой цвета выноски, вставляет пустую или форматирует выделение. Выноски создаются без заголовка, в формате [!note]+. Курсор должен стоять в начале выделения, следовательно выделять текст от конца к началу.
const callouts = [
    {name: "Цитата", type: "quote", icon: "“", colorDot: "🌫️", desc: " (серый)"},
    {name: "Примечание", type: "note", icon: "📝", colorDot: "🔵"},
    {name: "Вопрос", type: "question", icon: "❓", colorDot: "🟠"},
    {name: "Галочка", type: "success", icon: "✓", colorDot: "🟢"},
    {name: "Список", type: "example", icon: "📜", colorDot: "🟣"},
    {name: "Осторожно", type: "warning", icon: "⚠️", colorDot: "🟠"},
    {name: "Опасно", type: "danger", icon: "⚡", colorDot: "🔴"},
    {name: "Отмена", type: "failure", icon: "✗", colorDot: "🔴"},
    {name: "Инфо", type: "info", icon: "ℹ️", colorDot: "🔵"},
    {name: "Подсказка", type: "tip", icon: "🔥", colorDot: "🔵🟢", desc: " (бирюзовый)"},
    {name: "Жучок", type: "bug", icon: "🐞", colorDot: "🔴"}
];

// Формируем строки для отображения
const items = callouts.map(item => ({
    display: `${item.icon} ${item.name}${item.colorDot}${item.desc || ""}`,
    type: item.type
}));

// Показываем диалог выбора
const choice = await tp.system.suggester(
    item => item.display,
    items,
    false,
    "Выберите тип выноски"
);

if (choice) {
    const editor = app.workspace.activeLeaf.view.editor;
    const cursor = editor.getCursor();
    const lineText = editor.getLine(cursor.line).trim();

    // Регулярное выражение для определения выноски
    const calloutRegex = /^>\s*\[!([^\]]+)\][+\-]?\s*(.*)/;
    const isCallout = lineText.match(calloutRegex);

    // 4-е условие: если курсор на существующей выноске
    if (isCallout) {
        const currentType = isCallout[1];
        const content = isCallout[2] || "";
        editor.setLine(cursor.line, `> [!${choice.type}]+ ${content}`.trim());
        editor.setCursor(cursor); // Сохраняем позицию курсора
    }
    // Оригинальные условия
    else if (lineText) {
        editor.setLine(cursor.line, `> [!${choice.type}]+ ${lineText}`);
        editor.setCursor({line: cursor.line, ch: `> [!${choice.type}]+ ${lineText}`.length});
    } 
    else if (tp.file.selection()) {
        let selection = tp.file.selection();
        let lines = selection.split('\n');
        tR += `> [!${choice.type}]+ ${lines[0].trim()}`;
        if (lines.length > 1) tR += `\n> ${lines.slice(1).join('\n> ')}`;
    } 
    else {
        tR += `> [!${choice.type}]+ `;
        await tp.file.cursor();
    }
}
%>