<%*
// Шаблон вставляет спойлер с заголовком, свёрнутый по умолчанию, или превращает выделенное в спойлер. Повешен на кнопку. Если нужен открытый по умолчанию блок — замени <details></details> на <details open></details>.
const sel = tp.file.selection();

const esc = (s) => String(s ?? "")
  .replace(/&/g, "&amp;")
  .replace(/</g, "&lt;")
  .replace(/>/g, "&gt;");

if (sel && sel.trim().length) {
  const normalized = sel.replace(/\r\n/g, "\n");
  const lines = normalized.split("\n");
  const title = esc((lines.shift() ?? "").trim());   // 1-я строка → заголовок
  const body  = lines.join("\n").replace(/\s+$/,"");  // остальное → тело

  tR += `<details><summary>${title}</summary>

${body}
</details>`;
} else {
  tR += `<details><summary></summary>

</details>`;
}
%>