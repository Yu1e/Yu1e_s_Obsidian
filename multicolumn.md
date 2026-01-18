<%*
// Этот шаблон в связке с плагином Horizontal blocks позволяет динамически формировать и затем редактировать (!) многоколонный текст. Чат: https://lmarena.ai/c/019bd325-9f39-77da-aead-d9f8ddd3eb48 ранее https://www.perplexity.ai/search/v-chate-https-www-perplexity-a-dpRR0Za8Q2aFAJT4QdpLcQ)
// Варианты использования: 1. Выделить текст в заметке → ВСТАВИТЬ шаблон → после выбора кол-ва колонок и подтверждения текст преобразован. 2. Заранее вырезать текст, ВСТАВИТЬ шаблон, вставить текст в диалоговое окно, выбрать кол-во колонок. 3. В дальнейшем можно редактировать и уже готовые колонки, чтобы поменять их параметры и скорректировать распределение текста.
const ADD_HEADERS = false;
const BLOCK_TYPE = "hblock";
const LINE_WEIGHT = 30;

// ─── ПОЛУЧЕНИЕ ТЕКСТА ──────────────────────────────────────────
const selection = tp.file.selection();
const hasSelection = selection && selection.trim().length > 0;

let inputText;
let isExistingBlock = false;

if (hasSelection) {
    inputText = selection;
    isExistingBlock = /```(?:hblock|horizontal)/i.test(inputText);
} else {
    inputText = await tp.system.prompt("Вставьте текст для разбивки на колонки:");
}

if (!inputText || !inputText.trim()) {
    tR += selection || "";
    return;
}

// ─── ПОДТВЕРЖДЕНИЕ ─────────────────────────────────────────────
if (hasSelection) {
    const message = isExistingBlock 
        ? "✅ Переформатировать колонки"
        : "✅ Преобразовать в колонки";
    
    const confirm = await tp.system.suggester(
        [message, "❌ Отмена"],
        [true, false],
        false,
        isExistingBlock ? "Изменить существующий блок?" : "Заменить выделенный текст?"
    );
    if (!confirm) {
        tR += selection;
        return;
    }
}

// ─── ВЫБОР КОЛИЧЕСТВА КОЛОНОК ──────────────────────────────────
const cols = await tp.system.suggester(
    ["2 колонки", "3 колонки", "4 колонки", "5 колонок"],
    [2, 3, 4, 5],
    false,
    "Сколько колонок?"
);

if (!cols) {
    tR += selection || inputText;
    return;
}

// ─── ПАРСЕР ────────────────────────────────────────────────────
let contentItems = [];

const hblockMatch = inputText.match(/```(?:hblock|horizontal)\s*([\s\S]*?)```/i);

if (hblockMatch) {
    const inner = hblockMatch[1].trim();
    const existingBlocks = inner.split(/\n---\n/);
    
    for (const block of existingBlocks) {
        const lines = block.split('\n');
        const cleanLines = lines.filter(line => 
            !line.match(/^#{1,3}\s*(Block|Блок|Left|Right|Center|Column)\s*\d*/i)
        );
        const cleanText = cleanLines.join('\n').trim();
        
        if (!cleanText) continue;
        
        const paragraphs = cleanText
            .split(/\n\s*\n/)
            .map(p => p.trim())
            .filter(p => p);
        
        contentItems.push(...paragraphs);
    }
} else {
    const paragraphs = inputText
        .split(/\n\s*\n/)
        .map(p => p.trim())
        .filter(p => p);
    
    contentItems = paragraphs.length >= 2 
        ? paragraphs 
        : inputText.split('\n').map(l => l.trim()).filter(l => l);
}

if (contentItems.length === 0) {
    tR += "⚠️ Не найдено текста для обработки";
    return;
}

// ═══════════════════════════════════════════════════════════════
// 📐 УЛУЧШЕННАЯ БАЛАНСИРОВКА
// ═══════════════════════════════════════════════════════════════

function calcWeight(text) {
    return text.length + (text.split('\n').length - 1) * LINE_WEIGHT;
}

const weights = contentItems.map(calcWeight);
const totalWeight = weights.reduce((a, b) => a + b, 0);
const targetPerCol = totalWeight / cols;

// Инициализация колонок
const columns = Array.from({ length: cols }, () => []);
const colWeights = Array(cols).fill(0);

// ─── АЛГОРИТМ: последовательное распределение с look-ahead ─────
// Сохраняем порядок элементов, но умнее выбираем точки разбиения

// Шаг 1: Вычисляем идеальные точки разбиения по накопленному весу
const cumWeights = [];
let cumSum = 0;
for (const w of weights) {
    cumSum += w;
    cumWeights.push(cumSum);
}

// Шаг 2: Находим оптимальные индексы разбиения
const breakpoints = [0]; // начало первой колонки

for (let c = 1; c < cols; c++) {
    const idealCumWeight = targetPerCol * c;
    
    // Ищем индекс, где накопленный вес ближе всего к идеальному
    let bestIdx = breakpoints[breakpoints.length - 1] + 1;
    let bestDiff = Infinity;
    
    // Ищем в диапазоне от последней точки разбиения до конца
    // (оставляя минимум по 1 элементу на оставшиеся колонки)
    const minIdx = breakpoints[breakpoints.length - 1] + 1;
    const maxIdx = contentItems.length - (cols - c);
    
    for (let i = minIdx; i <= maxIdx; i++) {
        const diff = Math.abs(cumWeights[i - 1] - idealCumWeight);
        if (diff < bestDiff) {
            bestDiff = diff;
            bestIdx = i;
        }
    }
    
    breakpoints.push(bestIdx);
}
breakpoints.push(contentItems.length); // конец последней колонки

// Шаг 3: Распределяем элементы по найденным точкам
for (let c = 0; c < cols; c++) {
    const start = breakpoints[c];
    const end = breakpoints[c + 1];
    
    for (let i = start; i < end; i++) {
        columns[c].push(contentItems[i]);
        colWeights[c] += weights[i];
    }
}

// ─── ЗАЩИТА: если остались пустые колонки ──────────────────────
const nonEmptyCols = columns.filter(c => c.length > 0);

if (nonEmptyCols.length < cols && contentItems.length >= cols) {
    // Fallback: равномерное распределение по количеству
    const perCol = Math.ceil(contentItems.length / cols);
    for (let i = 0; i < cols; i++) {
        columns[i] = contentItems.slice(i * perCol, (i + 1) * perCol);
    }
}

// ─── ГЕНЕРАЦИЯ РЕЗУЛЬТАТА ──────────────────────────────────────
const columnTexts = [];

for (let i = 0; i < cols; i++) {
    const colItems = columns[i];
    if (colItems.length === 0) continue;
    
    let text = colItems.join('\n\n');
    if (ADD_HEADERS) {
        text = `### Block ${i + 1}\n${text}`;
    }
    columnTexts.push(text);
}

let result = '\n';
result += '```' + BLOCK_TYPE + '\n';
result += columnTexts.join('\n\n---\n\n');
result += '\n```';
result += '\n';

tR += result;
%>