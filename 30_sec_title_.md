<%*
// 30-СЕКУНДНОЕ ПЕРЕИМЕНОВАНИЕ после создания заметки, для заметок, которые открываешь и сразу начинаешь в них печатать. Использует шаблон "10_words..." 

// --- ШАГ 1: НАЙТИ ВСЕ НЕОБХОДИМЫЕ ФАЙЛЫ НЕМЕДЛЕННО ---

// 1. Находим файл, который мы только что создали ("Untitled")
const targetFile = tp.file.find_tfile(tp.file.path(true));
if (!targetFile) {
    new Notice("❌ Не удалось определить целевой файл.", 5000);
    return;
}

// 2. Находим файл ВТОРОГО шаблона ("10_words...")
const templateNameToApply = "10_words_for_existing_file_title";
const templateFile = tp.file.find_tfile(templateNameToApply);

// 3. Проверяем, что второй шаблон найден. Если нет - останавливаемся.
if (!templateFile) {
    new Notice(`❌ Шаблон для переименования "${templateNameToApply}" НЕ НАЙДЕН!`, 6000);
    return;
}


// --- ШАГ 2: ПЛАНИРОВАНИЕ ОТЛОЖЕННОЙ ЗАДАЧИ ---

new Notice(`Через 30 сек. применится шаблон "${templateNameToApply}"`, 4000);

// Запускаем отложенную задачу.
setTimeout(async () => {
    try {
        // --- КЛЮЧЕВОЕ РЕШЕНИЕ: ДЕЛАЕМ ФАЙЛ АКТИВНЫМ ПЕРЕД ПРИМЕНЕНИЕМ ---
        
        // Проверяем, существует ли еще целевой файл
        const fileExists = app.vault.getAbstractFileByPath(targetFile.path);
        if (!fileExists) {
            console.log("Задача отменена: целевой файл был удален.");
            return;
        }

        // Получаем доступ к плагину Templater напрямую через `app`
        const templater = app.plugins.plugins['templater-obsidian'];
        if (!templater) {
            new Notice("❌ Плагин Templater не найден.", 5000);
            return;
        }

        // Открываем наш целевой файл в новой вкладке.
        // Это делает его активным и гарантирует, что шаблон применится куда нужно.
        const leaf = app.workspace.getLeaf(true);
        await leaf.openFile(targetFile);

        // Теперь, когда файл гарантированно активен, применяем к нему второй шаблон.
        // Мы передаем объект файла шаблона, который нашли на Шаге 1.
        await templater.templater.append_template_to_active_file(templateFile);
        
        // Даем небольшую задержку, чтобы переименование успело произойти,
        // а затем можем закрыть новую вкладку, если она не нужна.
        setTimeout(() => {
            // leaf.detach(); // Раскомментируйте эту строку, если хотите, чтобы вкладка закрывалась сама
        }, 500);
        
    } catch (error) {
        console.error("Ошибка при выполнении отложенного шаблона:", error);
        new Notice("❌ Ошибка при применении второго шаблона.", 5000);
    }
}, 30000); // 30 секунд


// --- ШАГ 3: МГНОВЕННАЯ ВСТАВКА КОНТЕНТА ---
// Это выполняется сразу и не дает заметке закрыться.
tR = `---
создано: ${tp.date.now("YYYY-MM-DD")}
tags: 
---

`;
%>
<% tp.file.cursor(1) %>