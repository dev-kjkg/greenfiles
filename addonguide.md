Green Files Addon Development Guide(guide by Gemini) 🌿Welcome to the comprehensive development guide for Green Files addons. Green Files is a web-based, IndexedDB-powered file manager that supports modular extensions via JavaScript.This guide will teach you how to create tools, viewers, editors, and themes to extend the functionality of Green Files.📚 Table of ContentsArchitecture OverviewGetting StartedAPI ReferenceThe files ObjectThe db Object (Dexie)UI DevelopmentCode Snippets & PatternsBasic TemplateLoading External LibrariesCreating a ModalBest PracticesArchitecture OverviewGreen Files addons are essentially JavaScript snippets that are executed within the application's context. When a user runs an addon, the application executes your code as an anonymous function, passing two critical arguments:files: An array of the currently selected file nodes (or all files in the current directory, depending on the context).db: The instance of the IndexedDB database (powered by Dexie.js).Execution Flow:(async function(files, db) {
    // Your code runs here
})(selectedFiles, databaseInstance);
Getting StartedTo create and test an addon:Open Green Files.Click the Menu (⋮) button -> Settings / Addons.Enter a Name for your addon.Paste your JavaScript code into the Code textarea.Click Add.To run it:Select file(s) (optional).Right-click (or long-press) to open the Context Menu.Select "Use Extension".Choose your addon from the list.API ReferenceThe files ObjectThe files argument is an Array of File Node Objects.If the user selects files before running the addon, this array contains the selected items.Structure of a File Node:{
  id: number,          // Unique ID (Primary Key)
  parentId: number,    // ID of the parent folder (0 for Root)
  name: string,        // Filename (e.g., "image.png")
  type: string,        // "file" or "folder"
  mimeType: string,    // MIME type (e.g., "image/png") or undefined for folders
  size: number,        // Size in bytes
  content: Blob,       // The actual file data (Blob object)
  lastModified: number // Timestamp (ms)
}
The db ObjectThe db object is a Dexie.js instance. You have full access to the database.Table Schema:db.nodes: Stores all files and folders.Common Operations:1. CRUD Operations:// Get a file by ID
const node = await db.nodes.get(id);

// Add a new file
await db.nodes.add({
    parentId: 0, // Root
    name: "new_file.txt",
    type: "file",
    content: new Blob(["Hello"], {type: "text/plain"}),
    lastModified: Date.now()
});

// Update a file
await db.nodes.update(id, { name: "renamed.txt" });

// Delete a file
await db.nodes.delete(id);
2. Querying:// Get all children of a specific folder
const children = await db.nodes.where({ parentId: currentFolderId }).toArray();
UI DevelopmentSince Green Files is a web app, you can use standard DOM manipulation APIs.1. StylingYou can inject CSS directly into the head using JavaScript.const style = document.createElement('style');
style.id = 'my-addon-style'; // ID prevents duplication
style.innerHTML = `
    .my-modal { background: white; padding: 20px; }
`;
document.head.appendChild(style);
2. OverlaysTo create custom windows or dialogs, create a div with position: fixed and append it to document.body.Important: Always handle cleanup (closing the modal) yourself.Code Snippets & PatternsBasic TemplateUse this template for every new addon./**
 * Addon Name
 * Description of what it does.
 */
(async function(files, db) {
    // 1. Validation
    if (!files || files.length === 0) {
        alert("Please select a file first.");
        return;
    }

    // 2. Main Logic
    try {
        const file = files[0];
        console.log("Processing:", file.name);
        
        // Your logic here...
        alert("Selected: " + file.name);

    } catch (error) {
        console.error(error);
        alert("Error: " + error.message);
    }
})(files, db);
Loading External LibrariesGreen Files does not bundle every library. You can load CDN scripts dynamically.async function loadScript(url) {
    return new Promise((resolve, reject) => {
        if (document.querySelector(`script[src="${url}"]`)) return resolve();
        const script = document.createElement('script');
        script.src = url;
        script.onload = resolve;
        script.onerror = reject;
        document.head.appendChild(script);
    });
}

// Usage inside the main function
await loadScript('[https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js](https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js)');
// Now you can use JSZip
Creating a Modal (Standard UI Pattern)This pattern ensures a consistent look and feel.function showModal(title, contentHTML) {
    const overlay = document.createElement('div');
    overlay.style.cssText = "position:fixed; inset:0; background:rgba(0,0,0,0.5); z-index:10000; display:flex; justify-content:center; align-items:center;";
    
    const dialog = document.createElement('div');
    dialog.style.cssText = "background:white; width:90%; max-width:400px; border-radius:8px; overflow:hidden; box-shadow:0 10px 25px rgba(0,0,0,0.2);";
    
    // Header
    const header = document.createElement('div');
    header.style.cssText = "padding:16px; background:#4CAF50; color:white; font-weight:bold;";
    header.innerText = title;
    
    // Content
    const content = document.createElement('div');
    content.style.cssText = "padding:20px;";
    content.innerHTML = contentHTML;
    
    // Footer / Actions
    const footer = document.createElement('div');
    footer.style.cssText = "padding:10px 20px; text-align:right; border-top:1px solid #eee;";
    
    const closeBtn = document.createElement('button');
    closeBtn.innerText = "Close";
    closeBtn.style.cssText = "padding:8px 16px; background:#ddd; border:none; border-radius:4px; cursor:pointer;";
    closeBtn.onclick = () => document.body.removeChild(overlay);
    
    footer.appendChild(closeBtn);
    dialog.append(header, content, footer);
    overlay.appendChild(dialog);
    document.body.appendChild(overlay);
}

// Usage
showModal("My Addon", "<p>Hello World!</p>");
Best PracticesCheck for Duplicates:If your addon creates a UI (overlay/styles), check if it already exists before creating it again to prevent multiple windows opening.if (document.getElementById('my-addon-id')) return;
Blob URL Management:If you create URLs using URL.createObjectURL(), remember to revoke them (URL.revokeObjectURL()) when the addon closes to prevent memory leaks.Data Integrity:When modifying db.nodes, ensure you handle the parentId correctly so files don't get "lost" (orphaned) in the file system.Async/Await:Database operations are asynchronous. Always use await when reading or writing to db.Error Handling:Wrap your main logic in try...catch blocks to prevent the main application from crashing or hanging silently.Theme Customization (CSS Variables)You can create "Theming Addons" by modifying the root CSS variables.VariableDescriptionDefault--primaryMain App Color#4CAF50--bgBackground Color#FAFAFA--surfaceCard/Panel Color#FFFFFF--text-mainPrimary Text Color#212121--text-secSecondary Text Color#757575Example Theme Script:document.documentElement.style.setProperty('--primary', '#FF5722');
document.documentElement.style.setProperty('--bg', '#222222');
document.documentElement.style.setProperty('--text-main', '#FFFFFF');
Happy Coding! 🚀
