<%*
// Отсюда https://github.com/SilentVoid13/Templater/discussions/922
const callouts = {
//  Callout name   |  Prompt Name     |  UI Icon Description

    // White - Quotes
    "quote":          "⬜️ ⍘ Quote",       // Quotation Mark icon

// Blue - Information group
    "example":        "🟦 📑 Например",    // Outline icon (ish so a folder?)
    "tldr":           "🟦 📋 Вкрации",     // Clipboard icon  
   "note":           "🟦 ✏️ И заметим",       // Pencil icon
    "info":           "🟦 ⓘ Инфо",       // 'i' in Circle icon
    "tip":            "🟦 🔥 #лайвхинт",        // Flame icon
    
    
    // Red - Critical/Error group
    "bug":            "🟥 🪳 Бага-а-не-фича",        // Bug icon
    "danger":         "🟥 🐍 Опасное",     // Lightning Bolt icon
    "error":          "🟥 ⚡️ Ашипка",      // Lightning Bolt icon
    "fail":           "🟥 ❌ Упс",       // 'X' mark icon
    
    // Orange - Warning group
    "warning":        "🟧 ⚠️ Ахтунг!",    // Exclamation Sign icon
    "question":       "🟧 ❓ Чокак?",   // Question Mark in Circle icon
    
    // Green - Success group
        // "done":           "🟩 ✅ Done",       // Green Checkmark icon
    
    
    // Custom types (via Callout Manager)
};

const typeNames = [];
const typeLabels = [];

Object.keys(callouts)
	// Uncomment the line below to sort the callouts order alphabetically
	//.sort()
	.forEach((key, index) => {
	    typeNames.push(key);
	    // Add number prefix to each option for keyboard selection
	    typeLabels.push(`${index+1}. ${callouts[key]}`);
	});

let calloutType = await tp.system.suggester(
    typeLabels,
    typeNames,
    false,
    "Select callout type (use numbers 1-" + typeLabels.length + " to select)"
);

// Stop here when the prompt was cancelled (ESC).
if (!calloutType) {
    return;
}

// Extract the main name from the label to pre-fill the header
let defaultTitle = callouts[calloutType].split(' ').pop();

let title = await tp.system.prompt("Callout Header:", defaultTitle);

let foldState = await tp.system.suggester(
    ["1. Static", "2. Expanded", "3. Collapsed"],
    ["", "+", "-"],
    false,
    "Select callout folding option (use numbers 1-3 to select)"
);

let content = await tp.file.selection();

// Format each line of content to be part of the callout
const formattedContent = content.split('\n').map(line => `> ${line}`).join('\n');
_%>

> [!<% calloutType %>]<% foldState %> <% title %>
<% formattedContent %> <%* tp.file.cursor() %>