<%*
// Этот шаблон применяет логику скрипта yaml.js к текущему файлу
// https://lmarena.ai/c/141f99b7-e9d8-4067-9e59-84c70b28380c 
// https://www.perplexity.ai/search/etot-shablon-pri-primenenii-in-vr6gxJvZS_GOILUrmKwlqw
// https://www.perplexity.ai/search/etot-shablon-pri-primenenii-in-vr6gxJvZS_GOILUrmKwlqw

// Ждём, чтобы файл точно был записан
await new Promise(resolve => setTimeout(resolve, 1000));

const file = app.workspace.getActiveFile();
if (file) {
  let content = await app.vault.read(file);
  // Проверяем, что файл не пустой
  if (content && content.length > 0) {
    await tp.user.yaml.run(tp, file);
  } else {
    console.log("Файл пустой, YAML не применяем");
  }
}
%>