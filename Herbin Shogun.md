---
ink-image: https://krasniykarandash.ru/upload/resize_cache/iblock/1d4/758_506_1/1d4f0a6500e4baeff4837a2db3c4d81a.png
ink-name: Shogun
ink-brand: Herbin
ink-line: "1670"
ink-colour:
  - СЕ
ink-shimmer: ОР
ink-sheen:
ink-ml: 50
ink-container: флакон
ink-status: в коллекции
ink-buying-date: 2025 10 19
ink-shop: Красный Карандаш
ink-price: 3350
ink-ml-price: 67
ink-pen-now: "[[Moonman M2 розовая]]"
ink-fill-date: 2026 01 14
ink-rating:
---

```dataviewjs
const c = dv.current();
const safeStr = (val, def = "—") => val === undefined || val === null || val === "" ? def : String(val).trim();

// 🎨 Чтение справочника цветов
let colours = {};
try {
    const coloursFile = app.vault.getAbstractFileByPath("Чайник_с_чернилками/Serv/ink-colours-ref.md");
    if (coloursFile) {
        const coloursContent = await app.vault.read(coloursFile);
        const coloursMatch = coloursContent.match(/ink-colours-ref:\s*\n([\s\S]*?)(?=\n---|\n\.\.\.|\$)/);
        if (coloursMatch) {
            const lines = coloursMatch[1].split('\n');
            let currentKey = null;
            lines.forEach(line => {
                const keyMatch = line.match(/^\s{2}(\S+):\s*$/);
                if (keyMatch) { currentKey = keyMatch[1]; return; }
                const hexMatch = line.match(/^\s{4}hex:\s*"([^"]+)"/);
                if (hexMatch && currentKey) { colours[currentKey] = hexMatch[1]; }
            });
        }
    }
} catch (e) { console.error("Colours ref error:", e); }

// 🎨 Цвета чернил
const colourKey = safeStr(c["ink-colour"], null);
const colourHex = (colourKey && colours[colourKey]) ? colours[colourKey] : "#888";

// ✨ Шиммер: если указан код цвета — рисуем
const shimmerKey = safeStr(c["ink-shimmer"], null);
const hasShimmer = shimmerKey && colours[shimmerKey];
const shimmerHex = hasShimmer ? colours[shimmerKey] : "#FFD700";

// 🌟 Градиент фона
const gradientBg = hasShimmer 
    ? `linear-gradient(135deg, #FFFFFF 0%, #FFF9E6 15%, ${colourHex} 30%, ${colourHex} 45%, ${shimmerHex}60 55%, ${colourHex} 65%, ${colourHex} 80%, ${shimmerHex}40 90%, ${colourHex} 100%)`
    : `linear-gradient(135deg, #FFFFFF 0%, #f0f4f8 25%, ${colourHex} 100%)`;

// ✨ Шиммер (звёздочки)
const shimmerHTML = hasShimmer 
    ? `<span style="position:absolute;bottom:12px;right:12px;color:${shimmerHex};font-size:24px;line-height:1;text-shadow:0 0 4px rgba(0,0,0,0.4);z-index:10">✸</span>
       <span style="position:absolute;bottom:2px;right:2px;color:${shimmerHex};font-size:22px;line-height:1;text-shadow:0 0 4px rgba(0,0,0,0.4);z-index:10">✸</span>
       <span style="position:absolute;bottom:-2px;right:20px;color:${shimmerHex};font-size:22px;line-height:1;text-shadow:0 0 4px rgba(0,0,0,0.4);z-index:10">✸</span>`
    : '';

// 📦 Значения полей
const brand = safeStr(c["ink-brand"], "");
const lineVal = safeStr(c["ink-line"]);
const mlVal = safeStr(c["ink-ml"]) !== "—" ? safeStr(c["ink-ml"]) + " мл" : "—";
const contVal = safeStr(c["ink-container"]);
const inkImage = safeStr(c["ink-image"], null);

// 🖼️ HTML карточки
const html = `
<div style="display:grid;grid-template-columns:auto 1fr auto auto auto;gap:20px;padding:24px 28px;background:${gradientBg};border-radius:24px 12px 24px 12px;font-size:0.95em;border:1px solid rgba(0,0,0,0.08);align-items:center">
    
    <!-- Кружок с цветом чернил + шиммер -->
    <div style="position:relative;width:56px;height:56px;border-radius:50%;background:${colourHex};flex-shrink:0;box-shadow:inset 0 2px 4px rgba(255,255,255,0.3), inset 0 -2px 4px rgba(0,0,0,0.1)">
        ${shimmerHTML}
    </div>
    
    <!-- Название и бренд -->
    <div style="min-width:0">
        <div style="font-weight:700;font-size:1.6em;color:#2d3436;line-height:1.2;letter-spacing:-0.5px">${safeStr(c["ink-name"])}</div>
        <div style="color:#636e72;font-size:1.15em;font-weight:400;margin-top:3px">${brand}</div>
    </div>
    
    <!-- Блок: Линейка -->
    <div style="text-align:center;padding:14px 18px;background:#fff;border-radius:12px 8px 12px 8px;border-left:4px solid ${colourHex};border:1px solid rgba(0,0,0,0.08)">
        <div style="color:#adb5bd;font-size:0.75em;font-weight:600;text-transform:uppercase;margin-bottom:4px;letter-spacing:0.5px">Линейка</div>
        <div style="color:#2d3436;font-weight:600;font-size:1.1em">${lineVal}</div>
    </div>
    
    <!-- Блок: Объём -->
    <div style="text-align:center;padding:14px 18px;background:#fff;border-radius:12px 8px 12px 8px;border-left:4px solid ${colourHex};border:1px solid rgba(0,0,0,0.06)">
        <div style="color:#adb5bd;font-size:0.75em;font-weight:600;text-transform:uppercase;margin-bottom:4px;letter-spacing:0.5px">Объём</div>
        <div style="color:#495057;font-weight:500;font-size:1em">${mlVal}</div>
    </div>
    
    <!-- Блок: Ёмкость -->
    <div style="text-align:center;padding:14px 18px;background:#fff;border-radius:12px 8px 12px 8px;border-left:4px solid ${colourHex};border:1px solid rgba(0,0,0,0.06)">
        <div style="color:#adb5bd;font-size:0.75em;font-weight:600;text-transform:uppercase;margin-bottom:4px;letter-spacing:0.5px">Ёмкость</div>
        <div style="color:#495057;font-weight:500;font-size:1em">${contVal}</div>
    </div>
</div>`;

dv.paragraph(html);

// 🖼️ Вывод изображения (если есть)
if (inkImage && inkImage !== "—") {
    dv.container.innerHTML += `<div style="margin:16px 0;text-align:center"><img src="${inkImage.trim()}" alt="${safeStr(c["ink-name"])}" style="width:800px;max-width:100%;height:auto;object-fit:contain"></div>`;
}
```

> [!quote|no-icon]- 🛒 Покупка
> ```dataviewjs
> const ruMonths = ["янв","февр","март","апр","мая","июня","июля","авг","сент","окт","ноя","дек"];
> const formatDate = (val) => {
>     if (!val || val === "—" || val === "") return "—";
>     const parts = String(val).trim().split(/\s+/);
>     if (parts.length < 3) return String(val).trim();
>     const [year, month, day] = parts;
>     const monthIdx = parseInt(month, 10) - 1;
>     const monthName = ruMonths[monthIdx] || month;
>     return `${day} ${monthName} ${year}`;
> };
>
> const c=dv.current();
> let shopLink=c["ink-shop"]||"";
>
> try{
>     const shopsFile=app.vault.getAbstractFileByPath("Serv/ink-shops-ref.md");
>     if(shopsFile){
>         const shopsContent=await app.vault.read(shopsFile);
>         const shopsMatch=shopsContent.match(/ink-shops-ref:\s*\n([\s\S]*?)(?=\n\n|\n[a-z])/);
>         if(shopsMatch){
>             const shops={};
>             const lines=shopsMatch[1].split('\n');
>             lines.forEach(line=>{
>                 const match=line.match(/\s{2}"?([^"]+)"?:\s*"([^"]*)"/);
>                 if(match)shops[match[1].trim()]=match[2]
>             });
>             const shopName=c["ink-shop"]?.trim();
>             if(shops[shopName])shopLink=`<a href="${shops[shopName]}" target="_blank" style="color:#4169E1;text-decoration:none">${shopName}</a>`
>         }
>     }
> }catch(e){}
>
> const rows=[];
>
> if(c["ink-status"]) rows.push(`<tr><td style="text-align:right;padding-right:12px;color:#7f8c8d;width:40%"><strong>Статус:</strong></td><td style="color:#2c3e50">${c["ink-status"]}</td></tr>`);
> if(c["ink-buying-date"]) rows.push(`<tr><td style="text-align:right;padding-right:12px;color:#7f8c8d;width:40%"><strong>Дата покупки:</strong></td><td style="color:#2c3e50">${formatDate(c["ink-buying-date"])}</td></tr>`);
> if(c["ink-shop"]) rows.push(`<tr><td style="text-align:right;padding-right:12px;color:#7f8c8d;width:40%"><strong>Магазин:</strong></td><td style="color:#2c3e50">${shopLink||c["ink-shop"]}</td></tr>`);
>
> if(c["ink-price"]) {
>     const mlPrice = c["ink-ml-price"];
>     const mlPricePart = (mlPrice !== undefined && mlPrice !== null && mlPrice !== "") ? ` (${mlPrice} ₽/мл)` : "";
>     rows.push(`<tr><td style="text-align:right;padding-right:12px;color:#7f8c8d;width:40%"><strong>Цена:</strong></td><td style="color:#2c3e50">${c["ink-price"]} ₽${mlPricePart}</td></tr>`);
> }
>
> if(rows.length) dv.paragraph(`<table style="width:100%;border-collapse:collapse;font-size:0.95em">${rows.join("")}</table>`);
>
> // 🔘 Кнопка — показываем только если ink-ml-price ещё не заполнено
> const existingMlPrice = c["ink-ml-price"];
> const alreadyCalced = existingMlPrice !== undefined && existingMlPrice !== null && existingMlPrice !== "";
>
> if (!alreadyCalced) {
>     const price = parseFloat(c["ink-price"]);
>     const ml = parseFloat(c["ink-ml"]);
>     const canCalc = !isNaN(price) && !isNaN(ml) && ml > 0;
>
>     const btnContainer = dv.container.createEl("div", { attr: { style: "margin:10px 0 0 0" } });
>
>     const btn = btnContainer.createEl("span", {
>         text: "⟳ Посчитать цену за мл",
>         attr: { style: "display:inline-block;padding:6px 14px;background:#4169E1;color:white;border-radius:6px;font-weight:600;font-size:0.95em;cursor:pointer;user-select:none" }
>     });
>
>     btn.addEventListener("click", async () => {
>         if (canCalc) {
>             const perMl = Math.round(price / ml);
>             const file = app.workspace.getActiveFile();
>             if (file) {
>                 await app.fileManager.processFrontMatter(file, (fm) => {
>                     fm["ink-ml-price"] = perMl;
>                 });
>             }
>         } else {
>             btn.textContent = "⚠ Заполните ink-price и ink-ml";
>             btn.style.background = "#e74c3c";
>             setTimeout(() => {
>                 btn.textContent = "⟳ Посчитать цену за мл";
>                 btn.style.background = "#4169E1";
>             }, 2000);
>             return;
>         }
>         btnContainer.remove();
>     });
> }
> ```
```dataviewjs
// --- ВСТАВКА: Функция форматирования ---
const ruMonths = ["янв","февр","март","апр","мая","июня","июля","авг","сент","окт","ноя","дек"];
const formatDate = (val) => {
    if (!val || val === "—" || val === "") return "—";
    const parts = String(val).trim().split(/\s+/);
    if (parts.length < 3) return String(val).trim();
    const [year, month, day] = parts;
    const monthIdx = parseInt(month, 10) - 1;
    const monthName = ruMonths[monthIdx] || month;
    return `${day} ${monthName} ${year}`;
};
// ----------------------------------------

const c=dv.current();
const rows=[];

if(c["ink-pen-now"]){
    const penVal=String(c["ink-pen-now"]).trim();
    const wikiMatch=penVal.match(/\[\[([^\]|]+)(?:\|([^\]]+))?\]\]/);
    const displayName=wikiMatch?(wikiMatch[2]||wikiMatch[1]):penVal;
    rows.push(`<tr><td style="text-align:right;padding-right:12px;color:#7f8c8d;width:40%"><strong>Сейчас в ручке:</strong></td><td style="color:#2c3e50">${displayName}</td></tr>`)
}

// 🗓️ ИСПРАВЛЕНО: применяем formatDate
if(c["ink-fill-date"]) rows.push(`<tr><td style="text-align:right;padding-right:12px;color:#7f8c8d;width:40%"><strong>Дата заправки:</strong></td><td style="color:#2c3e50">${formatDate(c["ink-fill-date"])}</td></tr>`);

if(c["ink-rating"]){
    const rating=Number(c["ink-rating"]);
    if(!isNaN(rating)) rows.push(`<tr><td style="text-align:right;padding-right:12px;color:#7f8c8d;width:40%"><strong>Оценка:</strong></td><td style="color:#2c3e50">${"⭐".repeat(rating)}</td></tr>`)
}

if(rows.length) dv.paragraph(`<table style="width:100%;border-collapse:collapse;font-size:0.95em">${rows.join("")}</table>`)
```
> [!attention]- Остальные свойства <sup>прочерк означает «ещё нет данных»</sup>
> - [ink-sheen:: ]
> - [ink-halo:: ]
> - [ink-bleeding:: ]
> - [ink-shading:: ]
> - [ink-ghosting:: ]
> - [ink-feathering:: ]

> [!attention] Критично, важно

> [!note] Мои наблюдения и замечания
> Чуть ли не самые дорогие у меня? Долго облизывалась очень. Последний флакон в Москве? — Нифига не последний, и потом увидела дешевле, так что никогда не надо торопиться с дорогими.

> [!abstract] Описание от ИИ
> Глубокий чёрный цвет с тонкими золотистыми блёстками, создающими эффектное мерцание. 

> [!tip] #Этимология от ИИ
> «Сёгун». Японский военный титул, фактический правитель страны в эпоху сёгуната (1192–1868). Символ власти, силы, стратегического мастерства. В искусстве и литературе — образ лидера, полководца, часто ассоциируется с чёрным лаком, золотом, парадной бронёй. Цвет — глубокий чёрный с золотистым мерцанием, напоминает парадные доспехи сёгунов, лаковые шлемы с золотыми украшениями.

##### Ссылки
%% [Ссылка на заказ]()
[Карточка товара]()
[Обзор 1]()
[Обзор 2]() %%
##### Больше фото
%% [1]()
[2]()
[3]() %%