# Build

How to build PDF documents:
* Open the repo in VSCode/VSCodium.
* Install the [Markdown PDF](https://open-vsx.org/vscode/item?itemName=yzane.markdown-pdf) extension.
* Apply the following settings to `settings.json`. The workspace version of it is recommended:
  ```json
  "markdown-pdf.displayHeaderFooter": false,
  "markdown-pdf.sanitize": "gfm-allow-style"
  ```
* Press Ctrl+Shift+P and select "Markdown PDF: Export (pdf)".
