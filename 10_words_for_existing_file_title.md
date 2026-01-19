<%*
// Шаблон переименовывает заметку по первым 10 словам (50 символов) или по выделенному фрагменту текста. 
// Также применяет шаблон ямл (проверка и добавление шапки). Если в название попадает URL, поправишь вручную.
// https://lmarena.ai/c/141f99b7-e9d8-4067-9e59-84c70b28380c
// https://www.perplexity.ai/search/etot-shablon-pri-primenenii-in-vr6gxJvZS_GOILUrmKwlqw

tR = "";

async function smartRename() {
  try {
    const file = app.workspace.getActiveFile();
    if (!file) {
      new Notice("❌ Файл не найден", 3000);
      return;
    }

    const forbiddenFolders = ["Templates"];
    const filePath = file.path;
    const isForbidden = forbiddenFolders.some(folder => filePath.startsWith(folder + "/"));
    if (isForbidden) {
      new Notice("⏹️ Переименование шаблонов запрещено", 2000);
      return;
    }

    // 1️⃣ СНАЧАЛА сохраняем выделенный текст (пока редактор не тронут)
    // Это предотвращает потерю выделения при применении YAML
    const editor = app.workspace.activeLeaf?.view?.editor;
    let selectedText = "";
    if (editor && editor.somethingSelected()) {
      selectedText = editor.getSelection();
    }

    // 2️⃣ Применяем шаблон yaml.md
    await tp.file.include("[[yaml]]");

    // 3️⃣ Читаем обновленное содержимое файла
    let content = await app.vault.read(file);

    // --- Очистка контента для генерации имени ---
    
    // Удаляем frontmatter
    if (content.startsWith("---")) {
      const fmEnd = content.indexOf("\n---", 3);
      if (fmEnd !== -1) {
        content = content.slice(fmEnd + 4);
      }
    }

    // Удаляем выноски
    const lines = content.split('\n');
    const filteredLines = lines.filter(line => !line.trim().startsWith('> [!'));
    content = filteredLines.join('\n').trim();

    // Удаляем теги
    content = content.replace(/#[а-яА-ЯёЁa-zA-Z0-9_\-]+/g, '');
    content = content.replace(/\s+/g, ' ').trim();

    // Базовое имя из контента
    let name = "";
    if (selectedText) {
      name = selectedText;
    } else {
      const words = content.split(/\s+/).filter(w => w.trim());
      const maxLen = 50;
      for (let i = 0; i < words.length; i++) {
        let testName = name ? name + " " + words[i] : words[i];
        if (testName.length > maxLen) break;
        name = testName;
      }
    }

    // --- Логика переименования ---
    const currentName = file.basename;
    
    // Условие: Если выделения НЕТ И текущее имя "осмысленное" (не дата, не untitled, не одни цифры/символы)
    if (!selectedText && currentName && 
        !currentName.startsWith("Untitled") && 
        !currentName.startsWith("Без названия") &&
        !currentName.startsWith("Clipboard") &&
        !/^\d{4}-\d{2}-\d{2}/.test(currentName) &&
        !/^[\d\s_\-]+$/.test(currentName) && // Твоё условие: если только цифры/пробелы/тире - считаем имя плохим
        currentName.length > 3) {
      
      // Берем текущее имя, сокращаем до 10 слов
      const words = currentName.split(/\s+/);
      const limitedWords = words.slice(0, 10);
      name = limitedWords.join(' ').replace(/\s+/g, '_');
      
      // Чистим
      name = name.replace(/[^а-яА-ЯёЁa-zA-Z0-9\-\s_]/g, '');
      name = name.replace(/_+/g, '_');
      name = name.trim().replace(/^[\-_]+|[\-_]+$/g, '').trim();
      
    } else {
      // ИНАЧЕ (есть выделение ИЛИ имя плохое) -> формируем новое имя
      
      if (!selectedText) {
        // Если выделения нет, чистим имя, полученное из контента
        name = name.replace(/https?:\/\/(?:www\.)?([^\.\/\s]+)\.([^\.\/\s]+)(?:[\/\?\#][^\s]*)?/g, "$1_$2");
        name = name.replace(/\?/g, "7");
        // Удаляем буллеты (экранировано)
        name = name.replace(/^[\s\-\*•‣▪●○⁃∙‧]+\s*/g, "");
      }

      // Общая чистка для имен из контента или выделения
      name = name.replace(/[^а-яА-ЯёЁa-zA-Z0-9\-\s_]/g, '');
      name = name.replace(/\s+/g, ' ');
      name = name.replace(/ /g, '_');
      name = name.replace(/\-+/g, '-');
      name = name.replace(/_+/g, '_');
      name = name.trim().replace(/^[\-_]+|[\-_]+$/g, '').trim();

      // Первая буква заглавная
      if (name.length > 0) {
        name = name.charAt(0).toUpperCase() + name.slice(1);
      }

      // Удаляем короткие хвостики
      let nameParts = name.split('_');
      while (nameParts.length > 0 && nameParts[nameParts.length - 1].length <= 2 && !/^\d+$/.test(nameParts[nameParts.length - 1])) {
        nameParts.pop();
      }
      name = nameParts.join('_');
    }

    if (name.length < 2) {
      new Notice("❌ Слишком короткое имя", 3000);
      return;
    }

    if (file.basename === name) {
      new Notice("ℹ️ Имя уже актуально", 2000);
      return;
    }

    const newPath = file.parent ? `${file.parent.path}/${name}.md` : `${name}.md`;
    const exists = app.vault.getAbstractFileByPath(newPath);
    if (exists) {
      new Notice(`⚠️ Другой файл с именем "${name}.md" уже существует!`, 3000);
      return;
    }

    await app.fileManager.renameFile(file, newPath);
    new Notice(`✅ Переименовано в: "${name}.md"`, 3000);

  } catch (error) {
    console.error("Ошибка:", error);
    new Notice("❌ Ошибка переименования", 3000);
  }
}
await smartRename();
-%>