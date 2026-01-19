<%*
//v1.4: Добавление опции для включения заголовка для каждого дня DNP (страницы ежедневных заметок) для сворачивания. Чтобы создать задачу, начни текст с ";" (точки с запятой!) https://github.com/SilentVoid13/Templater/discussions/164

//'first' добавит в начало файла НЕ СОВМЕСТИМО С МЕТАШАПКОЙ В ФАЙЛЕ БЫСТРОГО ЗАХВАТА. 'last' добавит в конец файла.
let firstOrLastLine = 'last';

// Имя файла быстрого захвата. НЕ добавляйте расширение '.md'.
let qcFileName = 'Quick_сapture_note';

//Оставьте это поле пустым, если вы хотите использовать путь к файлу по умолчанию (задайте «/», чтобы использовать корень хранилища)
let folderOverride = '';

//Добавить заголовок для каждого дня, чтобы вложить под ним заметки быстрого сбора данных (работает только когда firstOrLastLine = 'first')
let bAddHeader = false;

let timeWithClock = await tp.file.include("[[time]]");
let curDateFormat = '**`' + tp.date.now("YYYY MM DD") + ' `** '; 
let finalTimestamp = curDateFormat + ' ' + timeWithClock;

let qcFolderLocation;
if(folderOverride) {
    qcFolderLocation = folderOverride;
} else {
    if(this.app.vault.config.newFileLocation != 'current') {
        qcFolderLocation = this.app.fileManager.getNewFileParent().path;
    } else {
        qcFolderLocation = '/';
    }
}
if(qcFolderLocation != ''){qcFolderLocation = qcFolderLocation + '/'}
qcFolderLocation = qcFolderLocation.replace(/\/\//g,'/');
if(qcFolderLocation == '/'){qcFolderLocation = ''}
if(qcFolderLocation.startsWith('/')){qcFolderLocation = qcFolderLocation.substring(1)}

let qcFilePath = qcFolderLocation + qcFileName + '.md';
let qcFile = this.app.vault.getAbstractFileByPath(qcFilePath);
if(!qcFile) {
    qcFile = await this.app.vault.create(qcFilePath, '');
}

if(qcFile) {
    let qcNote = await tp.system.prompt("Enter a Quick Capture note");
    let isTodo = qcNote.startsWith(';');
    let finalNote = (isTodo ? '- [ ] ' : '') + finalTimestamp + (isTodo ? qcNote.substring(1) : qcNote);
    let curContent = await this.app.vault.read(qcFile);
    let newContents;
    if(firstOrLastLine == 'last'){newContents = curContent + '\n' + finalNote}
    else {
        if(bAddHeader) {
            let curDateHeader = '# ' + curDateFormat;
            curContent = curContent.replace('\n' + curDateHeader + '\n\n', '');
            newContents = '\n' + curDateHeader + '\n\n' + finalNote + '\n' + curContent;
        } else {
            newContents = finalNote + '\n' + curContent
        }
    }
    this.app.vault.modify(qcFile, newContents);
}
%>