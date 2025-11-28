# door-schedule-spec-upload
Scan PDFs to pull door information from schedules and specs.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Door Schedule Extraction</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        
        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            max-width: 600px;
            width: 100%;
            padding: 40px;
        }
        
        h1 {
            color: #1a202c;
            font-size: 32px;
            margin-bottom: 10px;
            text-align: center;
        }
        
        .subtitle {
            color: #718096;
            text-align: center;
            margin-bottom: 40px;
            font-size: 16px;
        }
        
        .upload-box {
            border: 3px dashed #cbd5e0;
            border-radius: 12px;
            padding: 40px;
            text-align: center;
            transition: all 0.3s;
            cursor: pointer;
            margin-bottom: 20px;
        }
        
        .upload-box:hover {
            border-color: #667eea;
            background: #f7fafc;
        }
        
        .upload-box.dragover {
            border-color: #667eea;
            background: #edf2f7;
        }
        
        .upload-icon {
            font-size: 48px;
            margin-bottom: 15px;
        }
        
        input[type="file"] {
            display: none;
        }
        
        input[type="email"] {
            width: 100%;
            padding: 15px;
            border: 2px solid #e2e8f0;
            border-radius: 8px;
            font-size: 16px;
            margin-bottom: 20px;
            transition: border-color 0.3s;
        }
        
        input[type="email"]:focus {
            outline: none;
            border-color: #667eea;
        }
        
        .btn {
            width: 100%;
            padding: 15px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 18px;
            font-weight: 600;
            cursor: pointer;
            transition: transform 0.2s;
        }
        
        .btn:hover {
            transform: translateY(-2px);
        }
        
        .btn:active {
            transform: translateY(0);
        }
        
        .btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }
        
        .result {
            margin-top: 20px;
            padding: 20px;
            border-radius: 8px;
            display: none;
        }
        
        .result.success {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .result.error {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
        .result.processing {
            background: #fff3cd;
            color: #856404;
            border: 1px solid #ffeaa7;
            display: block;
        }
        
        .file-name {
            margin-top: 10px;
            color: #4a5568;
            font-size: 14px;
        }
        
        .spinner {
            border: 3px solid #f3f3f3;
            border-top: 3px solid #667eea;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 10px auto;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🏗️ AI Door Schedule Extraction</h1>
        <p class="subtitle">Upload your blueprint PDF and get door schedules instantly</p>
        
        <div class="upload-box" id="uploadBox" onclick="document.getElementById('fileInput').click()">
            <div class="upload-icon">📄</div>
            <h3>Click to upload or drag and drop</h3>
            <p style="color: #718096; margin-top: 10px;">PDF files only</p>
            <p class="file-name" id="fileName"></p>
        </div>
        
        <input type="file" id="fileInput" accept=".pdf" onchange="handleFileSelect(event)">
        
        <input type="email" id="emailInput" placeholder="Your email for results" required>
        
        <button class="btn" id="submitBtn" onclick="uploadFile()">Extract Door Schedule</button>
        
        <div class="result" id="result"></div>
    </div>
    
    <script>
        const uploadBox = document.getElementById('uploadBox');
        const fileInput = document.getElementById('fileInput');
        const fileName = document.getElementById('fileName');
        const resultDiv = document.getElementById('result');
        
        // Drag and drop
        uploadBox.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadBox.classList.add('dragover');
        });
        
        uploadBox.addEventListener('dragleave', () => {
            uploadBox.classList.remove('dragover');
        });
        
        uploadBox.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadBox.classList.remove('dragover');
            
            if (e.dataTransfer.files.length) {
                fileInput.files = e.dataTransfer.files;
                handleFileSelect({ target: fileInput });
            }
        });
        
        function handleFileSelect(event) {
            const file = event.target.files[0];
            if (file) {
                fileName.textContent = `Selected: ${file.name}`;
            }
        }
        
        async function uploadFile() {
            const file = fileInput.files[0];
            const email = document.getElementById('emailInput').value;
            const submitBtn = document.getElementById('submitBtn');
            
            // Validation
            if (!file) {
                alert('⚠️ Please select a PDF file');
                return;
            }
            
            if (!email || !email.includes('@')) {
                alert('⚠️ Please enter a valid email address');
                return;
            }
            
            // Show processing state
            resultDiv.className = 'result processing';
            resultDiv.style.display = 'block';
            resultDiv.innerHTML = `
                <div class="spinner"></div>
                <p><strong>⏳ Processing your blueprint...</strong></p>
                <p>This may take 30-60 seconds</p>
            `;
            
            submitBtn.disabled = true;
            submitBtn.textContent = 'Processing...';
            
            // Prepare form data
            const formData = new FormData();
            formData.append('data', file);
            formData.append('email', email);
            
            try {
                // REPLACE THIS URL WITH YOUR ACTUAL WEBHOOK URL
                const response = await fetch('https://automation.aitradebuilder.com/webhook/upload-blueprint', {
                    method: 'POST',
                    body: formData
                });
                
                const result = await response.json();
                
                if (result.success) {
                    resultDiv.className = 'result success';
                    resultDiv.innerHTML = `
                        <h3>✅ Success!</h3>
                        <p>Door schedule has been extracted and sent to:</p>
                        <p><strong>${email}</strong></p>
                        <p style="margin-top: 10px; font-size: 14px;">Check your email in a few moments.</p>
                    `;
                } else {
                    resultDiv.className = 'result error';
                    resultDiv.innerHTML = `
                        <h3>⚠️ ${result.message}</h3>
                        <p>File: ${result.fileName}</p>
                        <p style="margin-top: 10px; font-size: 14px;">An email has been sent with more details.</p>
                    `;
                }
            } catch (error) {
                resultDiv.className = 'result error';
                resultDiv.innerHTML = `
                    <h3>❌ Upload Failed</h3>
                    <p>${error.message}</p>
                    <p style="margin-top: 10px; font-size: 14px;">Please try again or contact support.</p>
                `;
            } finally {
                submitBtn.disabled = false;
                submitBtn.textContent = 'Extract Door Schedule';
            }
        }
    </script>
</body>
</html>
