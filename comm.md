<%*
/*
 * Шаблон для применения к существующей заметке. Переключает отображение форматирования за счёт html-комментария <!-- -->. Полезен при проблемах с редактированием переусложнённо оформленной заметки. Учитывает наличие frontmatter. Применение к закомментированному файлу — раскомментирует. После короткой паузы отлистывает файл обратно к курсору. В режиме просмотра закомментированный текст не будет виден!
 * Использование: назначьте шаблону горячую клавишу в настройках Obsidian 
 * (Settings > Hotkeys > Templater: Insert [имя_шаблона]). Например Ctrl+Shift+/
 */

const editor = app.workspace.activeLeaf.view.editor;
const cursor = editor.getCursor();
const fullText = editor.getValue();

let contentStartPos = 0;
let frontmatter = "";

if (fullText.startsWith("---")) {
    const endFrontmatter = fullText.indexOf("---", 3);
    if (endFrontmatter !== -1) {
        const endWithNewline = fullText.indexOf("\n", endFrontmatter);
        if (endWithNewline !== -1) {
            contentStartPos = endWithNewline + 1;
            frontmatter = fullText.substring(0, contentStartPos);
        } else {
            contentStartPos = endFrontmatter + 3;
            frontmatter = fullText.substring(0, contentStartPos) + "\n";
        }
    }
}

const content = fullText.substring(contentStartPos);

const commentPattern = /^\s*<!--\s*([\s\S]*?)\s*-->\s*$/;
const isCommented = commentPattern.test(content);

if (isCommented) {
    const match = content.match(commentPattern);
    if (match && match[1]) {
        editor.setValue(frontmatter + match[1]);
    }
} else {
    editor.setValue(frontmatter + `<!--\n${content}\n-->`);
}

setTimeout(() => {
    editor.setCursor(cursor);
}, 15);
%>