<%*
// Этот шаблон вставляет ссылку на предыдущую ежедневную заметку в нечётком формате в конец текущей. Его можно применять двояко: в момент создания нового файла (у меня он применяется для новых ежедневных заметок), а также к существующей заметке. См. также: [[структура_чужих_ежедневок]]
const currentTitle = tp.file.title;
const currentFile = tp.file.find_tfile(currentTitle);

// 1. Ищем дату в названии: YYYY(любой символ)MM(любой символ)DD
const currentRe = /^(\d{4}).(\d{2}).(\d{2})/;
const currentMatch = currentTitle.match(currentRe);

if (currentMatch) {
    const currentDate = moment(
        currentMatch[1] + "-" + currentMatch[2] + "-" + currentMatch[3],
        "YYYY-MM-DD"
    );

    const candidateRe = /^(\d{4}).(\d{2}).(\d{2})(?:.(\d{2})(\d{2})(\d{2}))?/;

    // Получаем ТОЛЬКО markdown-файлы и фильтруем корзину
    const allFiles = app.vault.getMarkdownFiles().filter(f => {
        const p = f.path.toLowerCase();
        // Исключаем все варианты корзины
        return !p.includes('.trash') && 
               !p.includes('/trash/') && 
               !p.startsWith('trash/');
    });

    let candidates = [];

    for (const f of allFiles) {
        const base = f.basename;
        if (base === currentTitle) continue;

        const m = base.match(candidateRe);
        if (!m) continue;

        let fileMoment;
        if (m[4] && m[5] && m[6]) {
            fileMoment = moment(
                `${m[1]}-${m[2]}-${m[3]} ${m[4]}:${m[5]}:${m[6]}`,
                "YYYY-MM-DD HH:mm:ss"
            );
        } else {
            fileMoment = moment(`${m[1]}-${m[2]}-${m[3]}`, "YYYY-MM-DD");
        }

        if (fileMoment.isValid() && fileMoment.isBefore(currentDate)) {
            candidates.push({ file: f, date: fileMoment });
        }
    }

    // 2. Добавляем ссылку В КОНЕЦ файла
    if (candidates.length > 0) {
        candidates.sort((a, b) => a.date.valueOf() - b.date.valueOf());
        const prev = candidates[candidates.length - 1].file;

        // append() добавляет текст именно в конец файла
        await app.vault.append(
            currentFile, 
            "\n꧁இ꧂ \n[[" + prev.basename + "|Предыдущая ежедневка: " + prev.basename + "]]"
        );
    }
}
%>