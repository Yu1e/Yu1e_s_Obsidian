<%*
/* Шаблон для быстрого приведения в красивый вид внутренней (!) ссылки, выделенной,
   или находящейся под курсором/рядом. Берёт ссылку [[Путь/Имя]], извлекает имя
   заметки и превращает в [[Путь/Имя|Имя]]. Если у ссылки уже есть отображалка-алиас
   через |, ничего не меняется. Работает с кнопкой 🔗➩🔗|🥽 Note Toolbar.
   Чат: https://www.perplexity.ai/search/pri-sozdanii-novoi-zametki-che-_8jkjUpcRP.Lb5HjwS3fhA#35 */

tR = "";

const editor = this.app.workspace.activeLeaf.view.editor;
const cursor = editor.getCursor();
const line = editor.getLine(cursor.line);

// --- если есть выделение, пробуем обработать её как ссылку ---
const selection = editor.getSelection();
if (selection && selection.startsWith("[[") && selection.endsWith("]]")) {
  if (!selection.includes("|")) {
    const innerSel = selection.slice(2, -2);
    const lastSlashIndexSel = innerSel.lastIndexOf("/");
    const fileWithExtSel =
      lastSlashIndexSel === -1 ? innerSel : innerSel.substring(lastSlashIndexSel + 1);
    const aliasBaseSel = fileWithExtSel.endsWith(".md")
      ? fileWithExtSel.slice(0, -3)
      : fileWithExtSel;
    const newLinkSel = `[[${innerSel}|${aliasBaseSel}]]`;
    editor.replaceSelection(newLinkSel);
    // курсор в конец вставленной ссылки
    const newCursor = editor.getCursor();
    editor.setCursor({ line: newCursor.line, ch: newCursor.ch });
  }
  return;
}

// --- если выделения нет, работаем по строке ---
// Ищем последнюю полную ссылку [[...]] на строке
let openIndex = line.lastIndexOf("[[");
if (openIndex === -1) {
  return;
}
const closeIndex = line.indexOf("]]", openIndex + 2);
if (closeIndex === -1) {
  return;
}

// Берём текст ссылки
const linkText = line.slice(openIndex, closeIndex + 2);

// Если уже есть алиас, ничего не делаем
if (linkText.includes("|")) {
  return;
}

// Внутренность ссылки
const inner = linkText.slice(2, -2);
const lastSlashIndex = inner.lastIndexOf("/");
const fileWithExt =
  lastSlashIndex === -1 ? inner : inner.substring(lastSlashIndex + 1);
const aliasBase = fileWithExt.endsWith(".md")
  ? fileWithExt.slice(0, -3)
  : fileWithExt;

// Собираем новую ссылку
const newLink = `[[${inner}|${aliasBase}]]`;
const newLine = line.slice(0, openIndex) + newLink + line.slice(closeIndex + 2);
editor.setLine(cursor.line, newLine);

// Ставим курсор в конец обновлённой ссылки
const newCursorCh = openIndex + newLink.length;
editor.setCursor({ line: cursor.line, ch: newCursorCh });
-%>
