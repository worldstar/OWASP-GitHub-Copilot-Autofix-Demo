const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const { v4: uuidv4 } = require('uuid');
const AdmZip = require('adm-zip');
const { scanWithCopilot } = require('../controllers/owaspScanner');

const router = express.Router();

// Multer setup for file uploads
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    const uploadPath = path.join(__dirname, '../../uploads');
    // Create the uploads folder if it doesn't exist
    if (!fs.existsSync(uploadPath)) {
      fs.mkdirSync(uploadPath);
    }
    cb(null, uploadPath);
  },
  filename: function (req, file, cb) {
    // Generate a unique filename
    const uniqueName = `${uuidv4()}-${file.originalname}`;
    cb(null, uniqueName);
  },
});
const upload = multer({ storage });

// 1. GET - Display upload page
router.get('/', (req, res) => {
  res.render('upload');
});

// 2. POST - Handle file upload, unzip, and run scan
router.post('/upload', upload.single('codeZip'), async (req, res) => {
  try {
    const zipFilePath = req.file.path;
    const outputDir = path.join(__dirname, '../../uploads', uuidv4());
    fs.mkdirSync(outputDir);

    // Unzip the uploaded file
    const zip = new AdmZip(zipFilePath);
    zip.extractAllTo(outputDir, true);

    // Gather all the code files from the extracted folder
    const fileList = getAllFiles(outputDir);

    // Run a scan (simulate GitHub Copilot Autofix) on each file
    // The scanner returns a data structure like:
    // [
    //   {
    //     filePath: '...',
    //     originalCode: '...',
    //     issues: [
    //       {
    //         issueDescription: '...',
    //         suggestedFix: '...'
    //       }
    //     ]
    //   }
    // ]
    const scanResults = await scanWithCopilot(fileList);

    // Store these in session or a global object to reference in the next step.
    // For this demo, store in a simple variable or pass in query (not secure, but for demonstration).
    // Better approach is to store a reference in a DB or in memory store like Redis.
    const scanId = uuidv4();
    global.scanData = global.scanData || {};
    global.scanData[scanId] = {
      outputDir,
      scanResults,
      zipFilePath,
    };

    res.render('scan', { scanId, scanResults });
  } catch (err) {
    console.error(err);
    res.status(500).send('An error occurred while processing the file.');
  }
});

// 3. POST - After user selects revisions, finalize and offer download
router.post('/finalize/:scanId', (req, res) => {
  const scanId = req.params.scanId;
  const { outputDir, scanResults, zipFilePath } = global.scanData[scanId] || {};

  if (!outputDir || !scanResults) {
    return res.status(400).send('No scan results found. Please start over.');
  }

  // The user’s selection for each file revision would be stored in the request body
  // Example: { "file0_issue0": "include" or "exclude", ... }
  const selections = req.body;

  // For each file, we apply or skip the suggested fix
  scanResults.forEach((fileResult, fileIndex) => {
    fileResult.issues.forEach((issue, issueIndex) => {
      const key = `file${fileIndex}_issue${issueIndex}`;
      const decision = selections[key];
      if (decision === 'include') {
        // Overwrite the original code with the suggested fix
        fs.writeFileSync(fileResult.filePath, issue.suggestedFix, 'utf8');
      }
    });
  });

  // Now re-zip the updated folder
  const newZipPath = path.join(__dirname, '../../uploads', `${uuidv4()}-revised.zip`);
  const newZip = new AdmZip();
  newZip.addLocalFolder(outputDir);
  newZip.writeZip(newZipPath);

  // Provide a download link or push the file to the user
  res.render('download', { newZipPath: path.basename(newZipPath) });
});

// 4. GET - Download the final ZIP
router.get('/download/:zipFile', (req, res) => {
  const zipFile = req.params.zipFile;
  const zipPath = path.join(__dirname, '../../uploads', zipFile);
  if (!fs.existsSync(zipPath)) {
    return res.status(404).send('File not found.');
  }
  res.download(zipPath, (err) => {
    if (err) {
      console.error(err);
    } else {
      // Optionally, cleanup the file after download
      fs.unlinkSync(zipPath);
    }
  });
});

/** Recursively gather all file paths in a directory */
function getAllFiles(dirPath, arrayOfFiles = []) {
  const files = fs.readdirSync(dirPath);

  files.forEach((file) => {
    const fileStatPath = path.join(dirPath, file);
    if (fs.statSync(fileStatPath).isDirectory()) {
      arrayOfFiles = getAllFiles(fileStatPath, arrayOfFiles);
    } else {
      arrayOfFiles.push(fileStatPath);
    }
  });

  return arrayOfFiles;
}

module.exports = router;
