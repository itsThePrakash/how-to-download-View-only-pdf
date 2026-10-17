
# 📄 Google Drive Document to PDF Saver

This repository contains two JavaScript snippets that let you download Google Drive documents (previewed as `blob:` images) into a **single high-resolution PDF file**.

-----

## 🤔 Why Use This Script?

 "Print to PDF" (`Ctrl+P`) feature,  This script downloads the **original, full-resolution images** 


-----

## 🚀 Features

  - ✅ Auto-scrolls through the document to **load all pages**.
  - ✅ Extracts `blob:` image URLs from the Google Drive preview.
  - ✅ Stitches images into a **single PDF** with correct page orientation (portrait/landscape).
  - ✅ Automatically names the PDF using the document’s title.

-----

## 🛠️ How to Use

Follow these steps carefully in your browser (like Chrome or Firefox) to generate the PDF.

### **Step 1: Open the Document in a New Window**

1.  Navigate to the file in your Google Drive.
2.  Right-click on the file and select **Open in new window**. This is crucial for the script to access the correct preview elements.

### **Step 2: Open Developer Tools**

1.  With the document preview open in the new window, open your browser's Developer Tools.
      - **Windows/Linux**: Press `F12`
      - **Mac**: Press `Option + ⌘ + I`
2.  Click on the **"Console"** tab.
3.  If you see a warning, you might need to type `allow pasting` and press **Enter** to enable pasting code.

### **Step 3: Run the Auto-Scroll Script**

This script scrolls through the entire document to ensure all pages are loaded into the browser's memory.

1.  Copy the code below.
2.  Paste it into the console and press **Enter**.
3.  Wait for the page to scroll to the bottom and for an alert box that says: **"Auto-scroll complete. Now run the PDF download script."**

\<details\>
\<summary\>Click to view Auto-Scroll Script\</summary\>

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

\</details\>

### **Step 4: Run the PDF Generation Script**

After all pages are loaded, this script will find all the page images, combine them into a PDF, and download it.

1.  Copy the code below.
2.  Paste it into the console and press **Enter**.
3.  The PDF will be generated and should start downloading automatically. 🎉

\<details\>
\<summary\>Click to view PDF Generation Script\</summary\>

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

\</details\>

-----

## ⚠️ Limitations & Important Notes

  - This script is designed for the **Google Drive document previewer** and may not work on other websites.
  - The final PDF will contain **images**, not selectable text. This is because the script is essentially taking screenshots of each page.
  - For **very large documents**, the browser may become slow or run out of memory. The final PDF file size can also be quite large.
  - You must keep the browser tab **in focus** while the scripts are running.

-----

## ❤️ Sponsor

If this project helps you, you can support ongoing development from the **Sponsor** button on this repository.

### 📌 Payment info kahan add karein?

All sponsor/payment details are configured in:

- `.github/FUNDING.yml`

You can add your payment/support info there, for example:

- `github`: GitHub Sponsors username
- `patreon`, `ko_fi`, `buy_me_a_coffee`, etc. for platform usernames
- `custom`: direct payment links (UPI, Razorpay, PayPal, personal payment page)

After updating `FUNDING.yml` and pushing changes, GitHub automatically updates the **Sponsor** button.

-----

## 🤝 Contributing

Contributions, issues, and feature requests are welcome\!
