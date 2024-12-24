# OWASP Copilot Autofix Demo

This repository demonstrates a Node.js application that allows users to:

1. **Upload a ZIP file** containing source code  
2. **Unzip & scan** each file using a simulated integration with GitHub Copilot Autofix for OWASP vulnerabilities  
3. **Review suggested fixes**, choose which to apply, and  
4. **Download** a ZIP of the revised code

> **Note**: In this demo, the “Copilot Autofix” portion is **simulated**. You would replace the placeholder scanning logic with actual calls to Copilot or any other security scanning engine to detect vulnerabilities and suggest fixes.

---

## Features

- **File Upload**: Allows uploading a ZIP archive with source code.  
- **Automated Scanning**: Performs a mock scan of JavaScript files and identifies potential OWASP vulnerabilities.  
- **Patch Review**: Displays the original code next to the suggested fix, letting the user select which patches to apply.  
- **Zip & Download**: After selecting fixes, the revised project is compressed back into a ZIP for download.

---

## Project Structure

. ├── Dockerfile ├── package.json ├── package-lock.json ├── /app │ ├── index.js // Main Express server │ ├── /routes │ │ └── index.js // Routing definitions │ ├── /controllers │ │ └── owaspScanner.js // Placeholder scanning logic │ ├── /views │ │ ├── upload.ejs // Page 1: Upload form │ │ ├── scan.ejs // Page 2: Display scan results & fixes │ │ └── download.ejs // Page 3: Final ZIP download │ └── /public │ └── css │ └── styles.css └── /uploads // Generated uploads & revised ZIPs (created on runtime)

---

## Prerequisites

- [Docker](https://www.docker.com/)  
- [Node.js](https://nodejs.org/en/) (if you plan to run locally without Docker)

---

## Building & Running via Docker

1. **Clone this repository**:
   ```bash
   git clone https://github.com/your-username/owasp-copilot-autofix-demo.git
   cd owasp-copilot-autofix-demo

2. Build the Docker image:

bash
複製程式碼
docker build -t owasp-copilot-autofix-demo .

Run the Docker container:

bash
複製程式碼
docker run -p 3000:3000 owasp-copilot-autofix-demo
Access the application: Open http://localhost:3000 in your browser.

Running Locally (Without Docker)
Install dependencies:
