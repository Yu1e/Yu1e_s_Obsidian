<%*

// Шаблон переименовывает заметку по первым 10 словам (50 символов) или по выделенному фрагменту текста. Также применяет шаблон ямл (проверка и добавление шапки). Если в название попадает URL, поправишь вручную. https://lmarena.ai/c/141f99b7-e9d8-4067-9e59-84c70b28380c
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

    // Читаем содержимое файла
    let content = await app.vault.read(file);
    
    // Применяем шаблон yaml.md для работы с frontmatter
    await tp.file.include("[[yaml]]");
    
    // Читаем содержимое файла (уже с обновлённым frontmatter)
    content = await app.vault.read(file);

    // Проверяем выделенный текст
    const editor = app.workspace.activeLeaf?.view?.editor;
    let selectedText = "";
    if (editor && editor.somethingSelected()) {
      selectedText = editor.getSelection();
      const cursor = editor.getCursor();
      editor.setCursor(cursor);
    }

    if (content.startsWith("---")) {
      const fmEnd = content.indexOf("\n---", 3);
      if (fmEnd !== -1) {
        content = content.slice(fmEnd + 4);
      }
    }

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

    // Регулярные выражения
    // Заменяем URL на домен, но сохраняем остальной текст
name = name.replace(/https?:\/\/(?:www\.)?([^\.\/\s]+)\.([^\.\/\s]+)(?:[\/\?\#][^\s]*)?/g, "$1_$2");
    // Удаляем из имени все квадратные скобки с одним любым символом внутри
name = name.replace(/```math[^]{1}/g, "");
// заменяем все ? на 7
name = name.replace(/\?/g, "7");
// Удаляем буллеты и дефисы в начале строки (и пробелы после них)
name = name.replace(/^(\s*[-*•‣▪–—·●○⁃∙‧﹒﹣－‒–—―]+\s*)/, "");
// Удаляем все символы, кроме разрешенных (включая подчеркивания):
name = name.replace(/[^а-яА-ЯёЁa-zA-Z0-9-\s_]/g, '');
// Заменяем несколько пробелов подряд на один
name = name.replace(/\s+/g, ' ');
name = name.replace(/ /g, '_');
name = name.replace(/-+/g, '-');
name = name.replace(/_+/g, '_');
name = name.trim().replace(/^[-_]+|[-_]+$/g, '').trim();

// Первую букву делаем прописной
if (name.length > 0) {
  name = name.charAt(0).toUpperCase() + name.slice(1);
}

// Блок удаления коротких слов в конце имени
let nameParts = name.split('_');
while (nameParts.length > 1 && nameParts[nameParts.length - 1].length <= 2 && !/^\d+$/.test(nameParts[nameParts.length - 1])) {
  nameParts.pop();
}
name = nameParts.join('_');
    {
      nameParts.pop();
    }
    name = nameParts.join('_');

    if (name.length < 3) {
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
      new Notice(`⚠️ Файл "${name}.md" уже существует!`, 3000);
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
%>