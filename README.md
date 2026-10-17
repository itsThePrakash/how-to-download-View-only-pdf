# Google Drive PDF Guide — Shared & View-Only Files

A lightweight, step-by-step guide for saving shared Google Drive PDFs on a laptop or desktop when the file owner allows downloads. It also explains what to do when a file is view-only and downloads are disabled.

**Live page:** <https://itstheprakash.github.io/how-to-download-View-only-pdf/>

## Laptop or desktop: download a shared PDF

1. Open Google Drive and sign in to the account that was given access to the file.
2. Find the shared PDF. If **Download** is available, right-click the file and choose **Download**.
3. For a Google Doc, open it and choose **File → Download → PDF Document (.pdf)**.
4. Open the saved file from your browser's downloads list or your computer's Downloads folder.

A view-only role may still allow downloading if the owner has enabled it. If **Download** is missing or disabled, check that you are using the right account, then choose **Request access** or contact the owner. Ask them to enable downloading or send an approved copy. The available options can depend on the owner, shared-drive rules, account type, and organization administrator. These general steps are for users in India and other countries; follow local laws and your organization's rules.

## About the code in this repository

The website is plain HTML and CSS. It does not sign in to Google Drive, read or upload files, and needs no JavaScript to use the guide.

The existing `auto-scroll.js` and `script.js` are legacy snippets, not an official Drive export feature. The website does not load them. This README does not provide steps for using scripts to turn a restricted preview into a downloadable file. If a download is disabled, request permission or an approved copy from the owner.

## Preview the website locally

No build step or package install is needed:

```sh
python3 -m http.server 8080
```

Then open <http://localhost:8080>. These steps run the website locally; they do not run the legacy snippets.

## GitHub Pages

Pages is configured to publish the repository's `main` branch from the root directory. `index.html` is the page entry point; `styles.css` and `favicon.svg` provide the design, while `robots.txt` and `sitemap.xml` provide basic crawl metadata. The live page updates after changes are merged into `main` and GitHub Pages finishes its build.

## Search metadata

The page uses a concise title and description, clear headings, and natural search phrases such as “download a shared Google Drive PDF,” “view-only PDF download access,” and “save a Google Doc as PDF.” It avoids keyword stuffing and oversized `meta keywords` lists.
