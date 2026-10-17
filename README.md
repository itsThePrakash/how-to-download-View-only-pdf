# Google Drive PDF Downloader: Save Drive Preview Documents as a High-Quality PDF (JavaScript Console Script)

> A simple, free, browser-console script to **download a Google Drive document preview as a single high-resolution PDF**. No extension, no software install, just two copy-paste snippets.

**Keywords:** google drive pdf downloader, save google drive preview as pdf, google drive document to pdf, download drive file as pdf, drive blob image to pdf, jsPDF script, browser console script, high resolution pdf

---

## Table of Contents

1. [What Is This?](#what-is-this)
2. [Why Use This Script Instead of Ctrl+P?](#why-use-this-script-instead-of-ctrlp)
3. [Features](#features)
4. [Quick Start (TL;DR)](#quick-start-tldr)
5. [Step-by-Step Guide](#step-by-step-guide)
6. [Scripts](#scripts)
7. [Troubleshooting](#troubleshooting)
8. [FAQ](#faq)
9. [Limitations](#limitations)
10. [Responsible Use](#responsible-use)
11. [Sponsor](#sponsor)
12. [Contributing](#contributing)

---

## What Is This?

When you open a document in the Google Drive previewer, each page is shown as a `blob:` image inside your browser. This project contains **two JavaScript snippets** that:

1. Scroll through the whole document so every page loads.
2. Collect all page images and combine them into **one PDF file** that downloads automatically.

It works in Chrome, Edge, Brave and Firefox.

---

## Why Use This Script Instead of Ctrl+P?

The browser's **Print to PDF (`Ctrl+P`)** feature often gives poor results for Drive previews:

| Print to PDF (`Ctrl+P`) | This Script |
|---|---|
| Pages can be cut off or split | One image per page, no cut-offs |
| Layout and margins may change | Original page size and orientation kept |
| Lower image quality | Full-resolution images (JPEG quality 1.0) |
| Manual file naming | PDF is named after the document title |

---

## Features

- ✅ **Auto-scrolls** the document so all pages are loaded
- ✅ **Extracts `blob:` image URLs** from the Google Drive preview
- ✅ **Single PDF output** with correct portrait/landscape orientation per page
- ✅ **Automatic file name** taken from the document title
- ✅ **Progress shown in the console** (page X of Y, percentage)

---

## Quick Start (TL;DR)

1. Open the file in Drive → right-click → **Open in new window**
2. Press `F12` (Windows/Linux) or `Option + ⌘ + I` (Mac) → open the **Console** tab
3. Type `allow pasting` + Enter if the browser asks
4. Paste and run **Script 1 (Auto-Scroll)** → wait for the alert
5. Paste and run **Script 2 (PDF Generation)** → your PDF downloads

---

## Step-by-Step Guide

### Step 1: Open the Document in a New Window

1. Go to the file in your Google Drive.
2. Right-click the file and choose **Open in new window**.

> ⚠️ This step is important. The script needs the correct preview elements to be present on the page.

### Step 2: Open Developer Tools

1. With the preview open in the new window, open Developer Tools:
   - **Windows / Linux:** press `F12`
   - **Mac:** press `Option + ⌘ + I`
2. Click the **Console** tab.
3. If the browser shows a warning, type `allow pasting` and press **Enter** to enable pasting.

### Step 3: Run the Auto-Scroll Script

This scrolls through the full document so all pages are loaded into browser memory.

1. Copy [Script 1](#script-1-auto-scroll).
2. Paste it into the console and press **Enter**.
3. Wait until the page reaches the bottom and an alert says:
   **"Auto-scroll complete. Now run the PDF download script."**

### Step 4: Run the PDF Generation Script

1. Copy [Script 2](#script-2-pdf-generation).
2. Paste it into the console and press **Enter**.
3. The PDF is generated and downloads automatically. 🎉

> Keep the tab **in focus** and do not close it until the download finishes.

---

## Scripts

### Script 1: Auto-Scroll

Forces all pages of the document to load.

<details>
<summary>Click to view the Auto-Scroll script</summary>

```javascript
/*
 * Step 1: Auto-Scroll Script
 * This script forces all pages of the document to load.
 */
(function() {
    let scrollableElement = null;
    // Find the main scrollable container
    document.querySelectorAll('*').forEach(element => {
        if (element.scrollHeight > element.clientHeight) {
            if (!scrollableElement || element.scrollHeight > scrollableElement.scrollHeight) {
                scrollableElement = element;
            }
        }
    });

    if (scrollableElement) {
        console.log("Starting auto-scroll to load all pages...");
        scrollableElement.scrollTo(0, 0); // Go to the top

        let scrollInterval = setInterval(() => {
            if (scrollableElement.scrollTop + scrollableElement.clientHeight >= scrollableElement.scrollHeight) {
                clearInterval(scrollInterval);
                console.log("✅ All pages are loaded.");
                alert("Auto-scroll complete. Now run the PDF download script.");
            } else {
                scrollableElement.scrollBy(0, scrollableElement.clientHeight);
            }
        }, 500);
    } else {
        alert("No scrollable area was found. You can try to save to PDF directly with Ctrl+P.");
    }
})();
```

</details>

### Script 2: PDF Generation

Finds all loaded page images, combines them into a PDF and downloads it.

<details>
<summary>Click to view the PDF Generation script</summary>

```javascript
/*
 * Step 2: PDF Generation Script
 * This script finds all loaded blob images, converts them to a PDF, and downloads it.
 */
(function() {
    console.log("Loading jsPDF library...");
    let script = document.createElement("script");
    script.onload = function() {
        const {
            jsPDF
        } = window.jspdf;
        let pdf = null;
        let imgElements = document.getElementsByTagName("img");
        let validImgs = [];

        console.log("Scanning document for page images...");
        // Filter for the blob images used in the Google Drive preview
        for (let i = 0; i < imgElements.length; i++) {
            let img = imgElements[i];
            if (img.src.startsWith("blob:https://drive.google.com/")) {
                validImgs.push(img);
            }
        }

        if (validImgs.length === 0) {
            console.error("No document images found. Did the auto-scroll script run successfully?");
            alert("Error: No document images found. Please ensure all pages were loaded.");
            return;
        }

        console.log(`✅ Found ${validImgs.length} pages!`);
        console.log("Generating PDF file... Please wait.");

        for (let i = 0; i < validImgs.length; i++) {
            let img = validImgs[i];
            let canvasElement = document.createElement("canvas");
            let ctx = canvasElement.getContext("2d");
            canvasElement.width = img.naturalWidth;
            canvasElement.height = img.naturalHeight;
            ctx.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight);
            let imgData = canvasElement.toDataURL("image/jpeg", 1.0);

            let orientation = img.naturalWidth > img.naturalHeight ? "l" : "p";
            let pageWidth = img.naturalWidth;
            let pageHeight = img.naturalHeight;

            if (i === 0) {
                // Initialize PDF with the dimensions of the first page
                pdf = new jsPDF({
                    orientation,
                    unit: "px",
                    format: [pageWidth, pageHeight]
                });
            } else {
                // For subsequent pages, add a new page with its specific dimensions
                pdf.addPage([pageWidth, pageHeight], orientation);
            }

            // Add the image to the current page
            pdf.addImage(imgData, "JPEG", 0, 0, pageWidth, pageHeight, "", "FAST");

            const percentage = Math.floor(((i + 1) / validImgs.length) * 100);
            console.log(`Processing page ${i + 1} of ${validImgs.length} (${percentage}%)`);
        }

        let title = document.querySelector('meta[itemprop="name"]').content || "document.pdf";
        if (!title.toLowerCase().endsWith('.pdf')) {
            title += ".pdf";
        }

        console.log("Downloading PDF file...");
        pdf.save(title, {
            returnPromise: true
        }).then(() => {
            document.body.removeChild(script);
            console.log("✅ PDF downloaded successfully!");
            alert(`Your file "${title}" has been downloaded.`);
        });
    };

    script.src = "https://unpkg.com/jspdf@latest/dist/jspdf.umd.min.js";
    document.body.appendChild(script);
})();
```

</details>

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Console won't let me paste | Type `allow pasting` and press Enter, then paste again |
| "No scrollable area was found" | Make sure you opened the file via **Open in new window**, then retry |
| "No document images found" | Run Script 1 again and wait for the completion alert before running Script 2 |
| Some pages are missing | Scroll slowly to the end manually, then run Script 2 again |
| Browser freezes or crashes | The document is too large. Close other tabs or try a device with more RAM |
| Nothing downloads | Allow downloads and pop-ups for `drive.google.com` and check that jsPDF loaded (needs internet access to `unpkg.com`) |
| Error about `meta[itemprop="name"]` | The page layout differs. Reload the preview in a new window and try again |

---

## FAQ

**How do I save a Google Drive document as a PDF with all pages?**
Open it in a new window, run the auto-scroll script, then the PDF generation script. All loaded pages are merged into one PDF.

**Why is the PDF text not selectable?**
The script saves each page as an image, so the PDF contains pictures of pages, not text. Use an OCR tool afterwards if you need searchable text.

**Is this better than Print to PDF?**
For Drive previews, usually yes, because it keeps the original image resolution and page size.

**Which browsers work?**
Chrome, Edge, Brave and Firefox with Developer Tools.

**Do I need to install anything?**
No. The script loads the jsPDF library automatically.

---

## Limitations

- Built for the **Google Drive document previewer**. It may not work on other websites.
- The PDF contains **images, not selectable text**, because each page is captured as an image.
- **Very large documents** can make the browser slow or run out of memory, and the PDF can be big.
- Keep the browser tab **in focus** while the scripts run.
- Google may change its page structure at any time, which can break the scripts.

---

## Responsible Use

Use this tool only on documents you own or have permission to copy and save. Respect the owner's sharing settings and applicable copyright laws. You are responsible for how you use it.

---

## Sponsor

If this project helps you, you can support ongoing development with the **Sponsor** button on this repository.

### Setting up sponsor / payment info (for the repo owner)

Sponsor details are configured in `.github/FUNDING.yml`. Add your details there, for example:

- `github`: GitHub Sponsors username
- `patreon`, `ko_fi`, `buy_me_a_coffee`: platform usernames
- `custom`: direct payment links (UPI, Razorpay, PayPal, personal payment page)

After you push the changes, GitHub automatically updates the **Sponsor** button.

---

## Contributing

Contributions, issues and feature requests are welcome! Open an issue or submit a pull request.

---

### 🔎 Suggested GitHub Repo Settings (for better search visibility)

**Description:** Download Google Drive preview documents as a single high-resolution PDF using a simple browser console script.

**Topics:** `google-drive`, `pdf`, `pdf-downloader`, `jspdf`, `javascript`, `browser-console`, `userscript`, `google-drive-downloader`, `blob-to-pdf`, `document-to-pdf`
