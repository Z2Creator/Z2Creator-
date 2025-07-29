<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <title>ChatPack TXTZP</title>
  <style>
    html, body {
      margin: 0; padding: 0; height: 100%;
      font-family: Arial, sans-serif;
      background: #f9fbfc;
      display: flex;
      flex-direction: column;
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px 20px;
      background: #1e90ff;
      color: white;
      flex-shrink: 0;
    }
    #importLabel {
      background: white;
      color: #1e90ff;
      padding: 8px 12px;
      border-radius: 6px;
      cursor: pointer;
      font-weight: bold;
      user-select: none;
    }
    #importLabel:hover {
      background: #e0f0ff;
    }
    #fileInput {
      display: none;
    }
    #search {
      padding: 10px;
      font-size: 14px;
      border-radius: 8px;
      border: 1px solid #ccc;
      margin: 10px 20px 5px 20px;
      flex-shrink: 0;
    }
    #fileList {
      margin: 0 20px;
      padding: 0;
      list-style: none;
      overflow-y: auto;
      flex-grow: 1;
      max-height: calc(100vh - 350px);
      font-size: 13px;
    }
    #fileList li {
      padding: 8px 10px;
      border-bottom: 1px solid #eee;
      cursor: pointer;
      user-select: none;
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
    }
    #fileList li:hover {
      background: #e0f0ff;
    }
    #answerContainer {
      position: sticky;
      bottom: 0;
      background: white;
      box-shadow: 0 -4px 12px rgba(0,0,0,0.15);
      max-height: 70vh;
      overflow: hidden;
      border-top-left-radius: 10px;
      border-top-right-radius: 10px;
      flex-shrink: 0;
      display: none;
      flex-direction: column;
    }
    #answerHeader {
      display: flex;
      justify-content: flex-end;
      background: #f0f0f0;
      padding: 8px 12px;
      border-bottom: 1px solid #ddd;
    }
    #closeBtn {
      background: transparent;
      border: none;
      font-size: 18px;
      cursor: pointer;
      color: #555;
      font-weight: bold;
    }
    #closeBtn:hover {
      color: red;
    }
    #answer {
      padding: 20px;
      overflow-y: auto;
      font-size: 15px;
      line-height: 1.5;
      user-select: text;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>

<header>
  <h1>ChatPack TXTZP</h1>
  <label id="importLabel" for="fileInput">📂 Importa .TXT o .TXTZP</label>
  <input type="file" id="fileInput" accept=".txt,.txtzp" multiple />
</header>

<input type="text" id="search" placeholder="Cerca un file..." />
<ul id="fileList"></ul>

<div id="answerContainer">
  <div id="answerHeader">
    <button id="closeBtn">✖</button>
  </div>
  <div id="answer">Seleziona un file per visualizzarne il contenuto.</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script>
  const fileInput = document.getElementById('fileInput');
  const searchInput = document.getElementById('search');
  const fileList = document.getElementById('fileList');
  const answer = document.getElementById('answer');
  const answerContainer = document.getElementById('answerContainer');
  const closeBtn = document.getElementById('closeBtn');

  let txtFiles = [];

  async function readZipFile(file) {
    try {
      const arrayBuffer = await file.arrayBuffer();
      const zip = await JSZip.loadAsync(arrayBuffer);
      const extractedTxtFiles = [];

      for (const filename of Object.keys(zip.files)) {
        if (filename.toLowerCase().endsWith('.txt')) {
          const content = await zip.file(filename).async("string");
          extractedTxtFiles.push({ name: filename, content });
        }
      }
      return extractedTxtFiles;
    } catch (error) {
      alert("Errore nella lettura del file ZIP: " + file.name);
      return [];
    }
  }

  function displayFiles(filter = '') {
    fileList.innerHTML = '';
    txtFiles
      .filter(f => f.name.toLowerCase().includes(filter.toLowerCase()))
      .forEach(f => {
        const li = document.createElement('li');
        li.textContent = f.name;
        li.onclick = () => {
          answer.textContent = f.content;
          answer.scrollTop = 0;
          answerContainer.style.display = 'flex';
        };
        fileList.appendChild(li);
      });
  }

  fileInput.addEventListener('change', async () => {
    txtFiles = [];
    fileList.innerHTML = '';
    answer.textContent = "Seleziona un file per visualizzarne il contenuto.";
    answerContainer.style.display = 'none';

    const files = Array.from(fileInput.files);
    for (const file of files) {
      const name = file.name.toLowerCase();

      if (name.endsWith('.txtzp')) {
        const extracted = await readZipFile(file);
        txtFiles = txtFiles.concat(extracted);
      } else if (name.endsWith('.txt')) {
        const text = await file.text();
        txtFiles.push({ name: file.name, content: text });
      } else {
        alert(`Il file "${file.name}" non è supportato.`);
      }
    }

    displayFiles(searchInput.value);
  });

  searchInput.addEventListener('input', () => {
    displayFiles(searchInput.value);
    answer.textContent = "Seleziona un file per visualizzarne il contenuto.";
    answerContainer.style.display = 'none';
  });

  closeBtn.addEventListener('click', () => {
    answerContainer.style.display = 'none';
  });
</script>

</body>
</html
