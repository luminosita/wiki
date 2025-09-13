---
title: OCR and Obsidian Workflow
date: 2025-09-06
author: "ChatGPT"
tags:
  - OCR
  - Obsidian
  - Markdown
  - Mobile
summary: This chat explores how to efficiently scan handwritten or printed documents on a mobile device, perform OCR, and generate Markdown files for use in Obsidian. It covers both all-in-one Obsidian plugin workflows and two-step methods using external scanner/OCR apps like Microsoft Lens, Google Lens, and Pen to Print. The discussion includes detailed plugin recommendations (Obsidian OCR, Image to Text OCR, AI Image OCR, Obsidian AI OCR), troubleshooting common errors on mobile such as "Cannot extract text on mobile" and "Not supported content," file format considerations, embedding images correctly in notes, and tips for maximizing OCR accuracy. The summary also highlights the difference between desktop and mobile OCR capabilities, automation possibilities, and best practices for organizing scanned content in Markdown format within an Obsidian vault.
---

# OCR & Obsidian Workflow

## User:
I need the best way to scan written document, OCR it and create markdown file using mobile phone.

## Assistant:
Got it 👍 You want to use your **mobile phone** to:  
1. **Scan a written document** (handwritten or printed).  
2. **Run OCR** (turn it into editable text).  
3. **Save/export as Markdown (.md)**.  

Here are the **best workflows**, depending on whether you prefer an all-in-one app or a two-step process:

### 🔹 All-in-One Apps (Scan + OCR + Export)
- **Obsidian with a scanner plugin**  
- **Notion**  
- **Bear (iOS)**  

### 🔹 Two-Step Method (More Flexible)
1. **Scan & OCR** with Microsoft Lens, Google Keep, Adobe Scan, Scanner Pro.  
2. **Convert to Markdown** with Obsidian, iA Writer, or Markor.  

### 🔹 For Handwriting
- **Google Lens**  
- **Microsoft OneNote OCR**  
- **Pen to Print**  

✅ **Best balance (free + reliable):** Microsoft Lens → OCR → paste into Obsidian.  

---

## User:
What is the best OCR plugin for Obsidian?

## Assistant:
### Top OCR Plugins:
1. **Obsidian OCR** (MohrJonas) – OCRs images/PDFs, searchable in vault.  
2. **Text Extractor** (scambier) – Extracts text from images/PDFs; works on desktop.  
3. **Image to Text OCR** (dario-baumberger) – Inline conversion of images to text.  
4. **AI Image OCR** (rootiest) – Uses AI APIs for OCR.  
5. **Obsidian AI OCR** (reorx) – Converts PDFs/images with Mistral OCR API.  
6. **Obsidian-Marker** (L3-N0X) – Advanced AI for PDFs to Markdown.  
7. **Excalidraw OCR Feature** – OCR within drawings.  

### Recommendation:
- For vault-wide search → **Obsidian OCR + Text Extractor**.  
- For PDFs → **Obsidian-Marker**.  
- For handwriting → **Excalidraw OCR** or **Taskbone OCR**.  
- For quick image-to-text → **Image to Text OCR**.  

---

## User:
Give me step by step instructions to install obsidian plugin, scan my handwritten document and have markdown file created in the directory on my mobile device.

## Assistant:
Two options:  

### Option A — All inside Obsidian (OCR plugin)
1. Open Obsidian → Settings → Community plugins → Browse → search OCR plugin (e.g., **Image to Text OCR**) → Install → Enable.  
2. Create or open a folder where you want the `.md`.  
3. Insert a photo of the handwritten note into the new note.  
4. Run OCR from the Command Palette.  
5. Text is extracted into your note, saved as `.md` in that folder.  

### Option B — Use Scanner App + Obsidian (recommended for handwriting)
1. Install **Microsoft Lens**, **Google Lens**, or **Pen to Print**.  
2. Scan & OCR the handwritten page → Copy text.  
3. Create new note in Obsidian → Paste text → Save.  
4. Attach original image if desired.  

Tips:  
- Good lighting, crop tightly, write neatly.  
- Try Pen to Print for best handwriting accuracy.  

---

## User:
In Obsidian I am receiving an error message "Cannot extract text on mobile".

## Assistant:
That error happens because some plugins (like **Text Extractor**) only run OCR on desktop. On mobile, they can only read cached results.  

### Fixes:
- Use plugins that work on mobile: **Image to Text OCR**, **AI Image OCR**, **Obsidian AI OCR**.  
- Or use a dedicated OCR scanner app (Lens, Google Lens, Pen to Print).  
- Hybrid: Run OCR on desktop → cached results searchable on mobile.  

---

## User:
Another error, "Not supported content".

## Assistant:
This means:  
1. File type not supported (plugin only works with `.jpg`, `.png`, not HEIC/PDF).  
2. File not embedded in note.  
3. Plugin limitation.  

### Fixes:
- Convert image to **JPEG**.  
- Make sure it’s embedded with `![[file.jpg]]`.  
- Run OCR again.  

If still failing → use external OCR app and paste into Obsidian.  

## Chat

You asked:
----------

I need the best way to scan written document, OCR it and create markdown file using mobile phone

---

ChatGPT Replied:
----------------

Got it 👍 You want to use your **mobile phone** to:

1.  **Scan a written document** (handwritten or printed).
    
2.  **Run OCR** (turn it into editable text).
    
3.  **Save/export as Markdown (.md)**.
    

Here are the **best workflows**, depending on whether you prefer an all-in-one app or a two-step process:

* * *

🔹 All-in-One Apps (Scan + OCR + Export)
----------------------------------------

These are easiest:

*   **Obsidian with a scanner plugin**
    
    *   Obsidian is a markdown-based note app.
        
    *   Pair it with **Obsidian Mobile + OCR/Scanner plugin** → directly scan → OCR → save into your vault as `.md`.
        
*   **Notion**
    
    *   Has built-in OCR when you scan images with the app.
        
    *   You can export the text as Markdown, but it's not fully automatic.
        
*   **Bear (iOS)**
    
    *   Supports Markdown natively.
        
    *   Use the in-app scan feature with OCR (iOS Notes OCR works too) → paste into Bear → Markdown file.
        

* * *

🔹 Two-Step Method (More Flexible)
----------------------------------

1.  **Scan & OCR with a dedicated app**
    
    *   **Microsoft Lens** (free, iOS & Android) → OCR → copy text.
        
    *   **Google Keep** → scan image → "Grab text from image" → copy.
        
    *   **Adobe Scan** → OCR → export to text.
        
    *   **Scanner Pro (iOS)** → excellent OCR accuracy.
        
2.  **Convert to Markdown**
    
    *   Paste OCR’d text into a Markdown editor:
        
        *   **Obsidian (iOS/Android)** – saves directly as `.md`.
            
        *   **iA Writer** – simple Markdown editor.
            
        *   **Markor (Android)** – lightweight Markdown editor.
            
    *   If you want automation:
        
        *   Use **Shortcuts (iOS)** or **Tasker (Android)** to create a workflow:
            
            *   Scan with OCR app → auto-save `.md` file in your Markdown folder (Obsidian vault, Git repo, etc.).
                

* * *

🔹 For Handwriting
------------------

*   **Google Lens** → surprisingly good with neat handwriting.
    
*   **Microsoft OneNote OCR** → handles handwriting fairly well.
    
*   **Pen to Print (iOS/Android)** → specialized in handwritten OCR → export to text.
    

* * *

✅ **Best balance (free + reliable):**

*   Use **Microsoft Lens** to scan → OCR → copy text → paste into **Obsidian Mobile** → save `.md`.
    

⚡ If you want **fully automated scans to Markdown** without copy/paste, Obsidian with a scanner plugin is probably the smoothest route.

* * *

Do you want me to suggest a **step-by-step setup** (e.g., Microsoft Lens → Obsidian Mobile vault with auto .md saving), or do you prefer an **all-in-one app** recommendation?

---

You asked:
----------

What is the best OCR plugin for Obsidian

---

ChatGPT Replied:
----------------

Here’s a plan for finding the **best OCR plugin for Obsidian**, based on community feedback and current plugin capabilities:

* * *

Top Obsidian OCR Plugins & Tools
--------------------------------

### 1\. **Obsidian OCR** _(MohrJonas)_

*   **What it does**: Automatically OCRs images and PDFs in your vault using Tesseract.js and ImageMagick; indexed so content becomes searchable in Obsidian's global search [GitHub](https://github.com/MohrJonas/obsidian-ocr?utm_source=chatgpt.com)[Obsidian Stats](https://www.obsidianstats.com/plugins/obsidian-ocr?utm_source=chatgpt.com).
    
*   **What users say**:
    
    > “Obsidian OCR allows you to search for text in the images and PDFs in your vault.” [Reddit](https://www.reddit.com/r/ObsidianMD/comments/vvqfqj?utm_source=chatgpt.com)
    
    *   Pros: Convenient automation, mostly local.
        
    *   Cons: Accuracy dependent on Tesseract and image quality.
        

* * *

### 2\. **Text Extractor** _(scambier)_

*   **What it does**: Acts as a companion plugin that extracts text from images, PDFs, and office documents (e.g., DOCX, XLSX) using Tesseract.js and pdf-extract. Can be paired with search tools like Omnisearch [GitHub](https://github.com/scambier/obsidian-text-extractor?utm_source=chatgpt.com).
    
*   **Notes**:
    
    *   Extraction happens on demand and stores results as small cached `.json` files for quick reuse [GitHub](https://github.com/scambier/obsidian-text-extractor?utm_source=chatgpt.com).
        
    *   **Limitation**: Doesn’t natively work on mobile—will fallback to cached results only [GitHub](https://github.com/scambier/obsidian-text-extractor?utm_source=chatgpt.com).
        

* * *

### 3\. **Image to Text OCR** _(dario-baumberger)_

*   **What it does**: Converts embedded images to text directly in your note. Offers options to append or replace the image with recognized text. Uses Tesseract.js [GitHub](https://github.com/dario-baumberger/obsidian-image-to-text-ocr?utm_source=chatgpt.com).
    
*   **Advantages**: Simple and fast for manual conversion of individual images.
    

* * *

### 4\. **AI Image OCR** _(rootiest)_

*   **What it does**: Uses AI (e.g., OpenAI/Gemini or local LLMs) to extract text from images. Supports batch processing and embedding directly into your notes [GitHub](https://github.com/rootiest/obsidian-ai-image-ocr?utm_source=chatgpt.com).
    
*   **Considerations**: Requires internet or local model; depends on LLM APIs and token limits; privacy policies apply [GitHub](https://github.com/rootiest/obsidian-ai-image-ocr?utm_source=chatgpt.com).
    

* * *

### 5\. **Obsidian AI OCR** _(reorx)_

*   **What it does**: Converts images or PDFs into Markdown with embedded images via Mistral AI's Document OCR API. Gives options to store images as attachments or inline base64, and automatically generates new `.md` notes [GitHub](https://github.com/reorx/obsidian-ai-ocr?utm_source=chatgpt.com).
    

* * *

### 6\. **Obsidian-Marker** _(L3-N0X)_

*   **What it does**: Uses advanced AI (Marker or MistralAI) to convert PDFs—including text, tables, LaTeX formulas, images—into Markdown. Works on mobile if API is accessible [GitHub](https://github.com/L3-N0X/obsidian-marker?utm_source=chatgpt.com).
    
*   **Setup options**:
    
    *   Use free MistralAI key
        
    *   Paid service (datalab.to)
        
    *   Self-hosted with Docker or Python API [GitHub](https://github.com/L3-N0X/obsidian-marker?utm_source=chatgpt.com)
        

* * *

### 7\. **Excalidraw OCR Feature**

*   **What it does**: With Excalidraw plugin version 1.8+, you can OCR handwritten text within drawings using a third-party service [YouTube](https://www.youtube.com/watch?v=7gu4ETx7zro&utm_source=chatgpt.com)[Reddit](https://www.reddit.com/r/ObsidianMD/comments/z0cit8?utm_source=chatgpt.com).
    
*   **Context**: Great for embedded hand-drawn notes; works inside Excalidraw canvases.
    

* * *

Community Tips & Use Cases
--------------------------

*   **Taskbone OCR (schlundd)**: Users find it more effective for handwriting, especially when converting GoodNotes PDFs to images first [Reddit+1](https://www.reddit.com/r/ObsidianMD/comments/15vj0g4?utm_source=chatgpt.com)[GitHub](https://github.com/schlundd/obsidian-ocr-plugin?utm_source=chatgpt.com).
    
*   **Combining plugins**: A popular setup is using **Text Extractor + Omnisearch** for powerful search across OCR content [Reddit](https://www.reddit.com/r/ObsidianMD/comments/1fdfod6?utm_source=chatgpt.com).
    

* * *

Recommendation Based on Your Needs
----------------------------------

| Your need | Best Plugin(s) | Why it fits |
| --- | --- | --- |
| Enable full vault search of OCR text | **Obsidian OCR** + **Text Extractor** | Automated indexing + searchable text |
| Convert PDFs or complex docs | **Obsidian-Marker** or **AI OCR** | Accurate extraction of syntax, layouts, formulas |
| Extract handwritten text/images | **Excalidraw OCR** or **Taskbone OCR** | Tailored for drawn or handwritten notes |
| Manual image → text conversion | **Image to Text OCR** | Quick, on-demand OCR for single images |

* * *

Next Steps for You
------------------

1.  **Enable Obsidian OCR**, then optionally **add Text Extractor** and **Omnisearch** for powerful searchable results.
    
2.  If you're working often with PDFs (especially with formatting or formulas), try **Obsidian-Marker** (MistralAI).
    
3.  For handwritten notes, test **Excalidraw’s built-in OCR** or **Taskbone OCR**, based on your workflow.
    
4.  Make sure to configure settings like OCR language, image density, and attachments depending on your content.
    

* * *

Let me know what type of documents you're working with—PDFs, handwritten, printed, annotated—and I can suggest the perfect plugin combo and setup tailored for you!

---

You asked:
----------

Give me step by step instructions to install obsidian plugin, scan my handwritten document and have markdown file created in the directory on my mobile device

---

ChatGPT Replied:
----------------

Thought for 31s

Nice — here are two **reliable, mobile-friendly** workflows (one using an Obsidian plugin end-to-end, and one more dependable two-step method that uses a scanner app + Obsidian). Follow the one that fits you best.

Option A — (One-app) Install OCR plugin in Obsidian Mobile → scan inside Obsidian → get a .md
=============================================================================================

> Good if you want everything inside Obsidian. NOTE: some OCR plugins use Tesseract/JS and on some phones they can be slow or unavailable — if that happens, use Option B below.

### 1) Install the plugin in Obsidian (mobile)

1.  Open **Obsidian** on your phone.
    
2.  Tap the **Settings (cog)** icon.
    
3.  Tap **Community plugins**.
    
4.  Turn **off** _Safe mode_ when prompted (this allows community plugins).
    
5.  Tap **Browse**. In the search box type the plugin name (examples: **Image to Text OCR** or **Obsidian OCR**).
    
6.  Tap the plugin → **Install** → then **Enable**.
    
7.  Exit settings.
    

### 2) Create/open the folder (directory) where you want the `.md`

1.  In the Vault view, navigate to (or create) the folder you want (e.g., `Handwritten/`).
    
2.  Tap the **+** (New note) and give it a name like `2025-09-06 - notes.md` (you can change later).
    

### 3) Capture your handwritten page into that note

1.  Open the note you just created (so you’re editing inside it and the note is located in the right folder).
    
2.  Insert the photo of the handwriting:
    
    *   Option A: In the editor, tap the **\+ / attachment** icon → **Camera** → take the picture → confirm. The photo will be saved into the vault (usually attachments folder) and inserted into the note.
        
    *   Option B: Take the photo with your phone camera first, then in Obsidian use **\+ / Attach file** → **Choose from photos** and pick it.
        

### 4) Run the plugin to OCR the image

1.  With the note open and the image inserted, open the **Command palette**: tap the three-dot menu → **Command palette** (or swipe down on the editor).
    
2.  Type `OCR` or the plugin name and choose the plugin command such as **Extract text from image** (or similar).
    
3.  The plugin will process the image and either:
    
    *   Insert the recognized text below the image, or
        
    *   Create a new note with the recognized text (depends on plugin settings).
        
4.  Edit/clean the text (fix OCR errors, apply Markdown headings, bullets, code blocks, etc.).
    

### 5) Confirm the file is a `.md` in the directory

*   Obsidian stores notes as Markdown files in your vault. The note you edited/saved is already a `.md` file in the folder you picked. You can verify in the Vault file browser or your phone’s Files app.
    

* * *

Option B — (Most reliable) Scan with a dedicated app → OCR → paste into Obsidian (recommended for handwriting)
==============================================================================================================

This is normally the most accurate for handwriting and avoids plugin/compatibility issues.

### 1) Install a scanner/OCR app

*   **iOS**: Microsoft Lens (free), Apple Notes (Live Text), or **Pen to Print** (handwriting-focused).
    
*   **Android**: Google Lens (or Google Photos’ “Grab text”), Microsoft Lens, or Pen to Print.
    

### 2) Scan and OCR

1.  Open your chosen app (Microsoft Lens or Google Lens).
    
2.  Use the app’s _document scan_ mode — align, crop, then apply enhancement to make handwriting clear (increase contrast). Save/confirm.
    
3.  Use the app’s OCR/extract-text action:
    
    *   **Microsoft Lens**: after capture choose **Actions → Extract text / Copy text**.
        
    *   **Google Keep / Photos / Lens**: choose **Grab text from image** or **Copy text**.
        
4.  Select and **Copy** the recognized text.
    

### 3) Create the Markdown file in Obsidian

1.  Open **Obsidian mobile**. Navigate to the folder where you want the file.
    
2.  Tap **\+ → New note**. Name it (e.g., `Handwritten - 2025-09-06.md`).
    
3.  Tap into the editor and **Paste** the text.
    
4.  Clean the text and add Markdown formatting (headings `#`, lists `-`, code fences, etc.).
    
5.  Save — Obsidian automatically saves it as a `.md` file in that folder.
    

### 4) Attach original image (optional)

*   From the scanner app, share the image to Obsidian (If Obsidian appears in the share sheet) or save to Photos/Files then in Obsidian use **\+ → Attach file** to add the original image into the same note. That keeps the source image next to the OCR text.
    

* * *

Quick tips to improve handwritten OCR accuracy
==============================================

*   Use **good lighting** and a **high contrast** background.
    
*   Crop tightly to the page; remove shadows.
    
*   Write as neatly as possible; uppercase letters OCR better.
    
*   If you have many pages, scan them as a multi-page document (Lens can multi-scan), then OCR each page in turn.
    
*   Try multiple OCR apps — handwriting performance varies (Pen to Print is specialized for handwriting).
    

Troubleshooting & notes
=======================

*   If the plugin **doesn’t show** in mobile browse, update Obsidian to the latest version — some plugins require newer app versions.
    
*   If plugin OCR is slow or fails, use Option B (scanner app + paste) — it’s almost always more reliable on phones.
    
*   If you need automation (scan → automatic .md created without manual paste), tell me your OS (iOS/Android) and I’ll give a ready Shortcut (iOS) or Tasker/intent script (Android) you can install.
    

* * *

If you tell me **iOS or Android** I’ll give a compact, device-specific mini-guide (or an iOS Shortcut you can copy) to fully automate scan → OCR → create `.md` in the folder you want. Which phone are you using?

---

You asked:
----------

In Obsidian I am receiving an error message "Cannot extract text on mobile"

---

ChatGPT Replied:
----------------

That error explains it 👍

Some Obsidian OCR plugins (for example **Text Extractor**) only run the actual OCR engine on **desktop**, not on iOS/Android. On mobile, they can only **read cached results** (if OCR was already done on desktop). That’s why you see:

> **"Cannot extract text on mobile"**

* * *

✅ What you can do
-----------------

### 1\. Use a plugin that works **directly on mobile**

*   **Image to Text OCR** (by dario-baumberger) → runs Tesseract.js inside Obsidian on your phone. Works for single images, although slower.
    
*   **AI Image OCR** (by rootiest) → uses an online AI service (OpenAI, Gemini, etc.) to process OCR, so it works on mobile too.
    
*   **Obsidian AI OCR** (by reorx) → sends images/PDFs to Mistral’s OCR API → returns Markdown.
    

👉 These don’t rely on cached extraction, so they’ll actually work from mobile.

* * *

### 2\. Use a dedicated OCR app → paste into Obsidian

Much more reliable (especially for handwriting):

*   **Microsoft Lens** (iOS/Android) → Scan → “Copy text.”
    
*   **Google Lens / Google Keep** → Grab text → Copy.
    
*   **Pen to Print** → Best for handwritten OCR.
    

Then just **paste** into your Obsidian note → saved as `.md`.  
_(This avoids plugin limitations altogether.)_

* * *

### 3\. Hybrid workflow

*   If you want vault-wide search of images/PDFs, keep **Text Extractor** installed.
    
*   Run OCR once on desktop (so the cache is built).
    
*   Then you can **search OCR’d content on mobile** (though you can’t run new OCR jobs).
    

* * *

⚡ So in short:

*   The plugin you installed doesn’t support OCR on mobile.
    
*   Either install a **different OCR plugin** (Image to Text OCR, AI OCR) or use a **scanner/OCR app** → paste results into Obsidian.
    

* * *

Do you want me to walk you through **installing Image to Text OCR on mobile** (so you can scan handwritten pages directly in Obsidian), or would you prefer I give you a **scanner app + Shortcut/Tasker workflow** that automatically drops OCR text into a Markdown file in your vault?

---

You asked:
----------

Another error, "Not supported content\*

---

ChatGPT Replied:
----------------

Got it — thanks for sharing that new error 🙏

When you see **“Not supported content”** in Obsidian OCR plugins, it usually means one of these:

* * *

🔎 Common causes
----------------

1.  **File type isn’t supported**
    
    *   Some plugins only work on **embedded images (`.png`, `.jpg`, `.jpeg`)** — not PDFs, HEIC (iPhone default), or webp.
        
    *   If you took the photo with your phone, it might be in **HEIC** format → the plugin can’t read it.
        
2.  **File wasn’t embedded in the note**
    
    *   The plugin expects the image to be inside your note, not just linked or stored outside the vault.
        
3.  **Plugin limitation**
    
    *   Example: _Image to Text OCR_ works only on inline images inserted into notes. If you try to OCR an attachment from the File Explorer, it throws “Not supported content.”
        

* * *

✅ Fixes
-------

### Step 1: Convert your image to a supported format

*   On iPhone: when you share/save the photo, choose **JPEG** instead of HEIC.
    
*   On Android: most scanners save as JPEG by default (Microsoft Lens, Google Lens, Adobe Scan).
    

### Step 2: Insert it properly into a note

1.  Open the note where you want OCR text.
    
2.  Tap **\+ → Attach file** (or paste from gallery).
    
3.  Ensure the note shows something like:
    
        ![[scanned_note.jpg]]
        
    
    (That means the image is embedded, not just a link.)
    

### Step 3: Run OCR again

*   Open the **Command Palette** → search for the plugin command (e.g. `Image to Text OCR: Extract text`).
    
*   Now it should process the embedded image.
    

* * *

🚀 Alternative (more reliable)
------------------------------

If you still hit “Not supported content,” it might be faster to use an **external scanner/OCR app** (Microsoft Lens, Google Lens, or Pen to Print for handwriting) → copy text → paste into Obsidian.  
This avoids plugin limitations and gives better accuracy, especially for handwriting.

* * *

👉 Question: are you trying to OCR a **handwritten photo (HEIC)** or a **PDF scan**? That will tell me whether you should switch plugins or use a scanner app + automation. Would you like me to give you a **step-by-step iOS/Android automation** so that when you scan with Lens, it **auto-creates a Markdown note in your Obsidian vault**?

