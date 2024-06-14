<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Syllabus Tracker</title>
    <style>
        body {
            background-color: #121212;
            color: #E0E0E0;
            font-family: 'Arial', sans-serif;
            margin: 0;
            padding: 20px;
            font-size: 18px;
            font-weight: bold;
        }
        .container {
            max-width: 800px;
            margin: auto;
            padding: 20px;
            background-color: #1E1E1E;
            border-radius: 15px;
        }
        h1, h2 {
            text-align: center;
            font-size: 24px;
            font-weight: bold;
        }
        .subject, .add-subject, .progress {
            margin-bottom: 20px;
        }
        .input-group {
            display: flex;
            margin-bottom: 10px;
        }
        .input-group input, .input-group button {
            padding: 10px;
            border: none;
            border-radius: 10px;
            margin-right: 10px;
            font-size: 18px;
        }
        .input-group button {
            background-color: #6200EE;
            color: #fff;
            cursor: pointer;
        }
        .progress-bar {
            background-color: #6200EE;
            border-radius: 10px;
            height: 20px;
            width: 0%;
            transition: width 0.5s;
        }
        .progress-container {
            background-color: #333;
            border-radius: 10px;
            overflow: hidden;
        }
        .chapter-list {
            margin-top: 10px;
        }
        .chapter-item {
            display: flex;
            align-items: center;
            margin-bottom: 5px;
            gap: 10px;
        }
        .chapter-item input[type="checkbox"] {
            margin-right: 10px;
            transform: scale(1.5);
        }
        .chapter-item button {
            background-color: red;
            color: white;
            border: none;
            border-radius: 5px;
            padding: 5px 10px;
            cursor: pointer;
        }
        .subject button {
            background-color: red;
            color: white;
            border: none;
            border-radius: 5px;
            padding: 5px 10px;
            cursor: pointer;
            margin-top: 10px;
        }
        .reddit-link {
            text-align: center;
            margin-top: 20px;
        }
        .reddit-link a {
            color: #BB86FC;
            text-decoration: none;
            font-size: 18px;
        }
        .login-container {
            display: none;
            max-width: 400px;
            margin: auto;
            padding: 20px;
            background-color: #333;
            border-radius: 15px;
            text-align: center;
        }
        .login-container input {
            width: 80%;
            padding: 10px;
            margin: 10px 0;
            border-radius: 10px;
            border: none;
            font-size: 18px;
        }
        .login-container button {
            padding: 10px 20px;
            background-color: #6200EE;
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 18px;
            cursor: pointer;
        }
        .logout-button {
            background-color: #6200EE;
            color: white;
            border: none;
            border-radius: 5px;
            padding: 5px 10px;
            cursor: pointer;
            margin-top: 10px;
            display: block;
            text-align: center;
        }
        .undo-container {
            margin-top: 10px;
            text-align: center;
            display: none;
        }
        .undo-container button {
            background-color: #FFCA28;
            color: black;
            border: none;
            border-radius: 5px;
            padding: 5px 10px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="login-container" id="loginContainer">
        <h2>Login</h2>
        <input type="text" id="username" placeholder="Username" required>
        <button onclick="login()">Login</button>
    </div>
    <div class="container" id="appContainer">
        <h1>Syllabus Tracker</h1>
        <div class="add-subject">
            <h2>Add a Subject</h2>
            <div class="input-group">
                <input type="text" id="subjectName" placeholder="Subject Name" spellcheck="true">
                <button onclick="addSubject()">Add Subject</button>
            </div>
        </div>
        <div id="subjects"></div>
        <div class="reddit-link">
            <p>Check out my <a href="https://www.reddit.com/user/Lav_Srivastava/" target="_blank">Reddit Profile</a></p>
        </div>
        <button class="logout-button" onclick="logout()">Logout</button>
        <div class="undo-container" id="undoContainer">
            <span>Item deleted.</span>
            <button onclick="undoDelete()">Undo</button>
        </div>
    </div>

    <script>
        let undoTimeout;
        let lastDeletedItem;

        document.addEventListener('DOMContentLoaded', (event) => {
            if (localStorage.getItem('loggedIn') === 'true') {
                showApp();
            } else {
                showLogin();
            }
        });

        function login() {
            const username = document.getElementById('username').value;
            if (username) {
                localStorage.setItem('loggedIn', 'true');
                localStorage.setItem('username', username);
                showApp();
            }
        }

        function logout() {
            localStorage.removeItem('loggedIn');
            localStorage.removeItem('username');
            showLogin();
        }

        function showLogin() {
            document.getElementById('loginContainer').style.display = 'block';
            document.getElementById('appContainer').style.display = 'none';
        }

        function showApp() {
            document.getElementById('loginContainer').style.display = 'none';
            document.getElementById('appContainer').style.display = 'block';
            document.getElementById('username').value = '';
        }

        function addSubject() {
            const subjectName = document.getElementById('subjectName').value;
            if (subjectName) {
                const subjectsDiv = document.getElementById('subjects');
                const subjectDiv = document.createElement('div');
                subjectDiv.classList.add('subject');
                subjectDiv.innerHTML = `
                    <h2>${subjectName}</h2>
                    <div class="input-group">
                        <input type="text" placeholder="Chapter Name" spellcheck="true">
                        <button onclick="addChapter(this)">Add Chapter</button>
                    </div>
                    <div class="chapter-list"></div>
                    <div class="progress-container">
                        <div class="progress-bar"></div>
                    </div>
                    <div class="progress">
                        <span>0%</span> completed
                    </div>
                    <button onclick="deleteItem(this.parentElement)">Delete Subject</button>
                `;
                subjectsDiv.appendChild(subjectDiv);
                document.getElementById('subjectName').value = '';
            }
        }

        function addChapter(button) {
            const chapterInput = button.previousElementSibling;
            if (chapterInput.value) {
                const chapterList = button.parentElement.nextElementSibling;
                const chapterItem = document.createElement('div');
                chapterItem.classList.add('chapter-item');
                chapterItem.innerHTML = `
                    <input type="checkbox" onchange="updateProgress(this)">
                    <span>${chapterInput.value}</span>
                    <input type="checkbox" class="revision-checkbox" onchange="updateProgress(this)">
                    <label for="revision-checkbox">Revision</label>
                    <button onclick="deleteItem(this.parentElement)">Delete</button>
                `;
                chapterList.appendChild(chapterItem);
                chapterInput.value = '';
                updateProgress(button);
            }
        }

        function deleteItem(item) {
            lastDeletedItem = item;
            item.style.display = 'none';
            const undoContainer = document.getElementById('undoContainer');
            undoContainer.style.display = 'block';

            undoTimeout = setTimeout(() => {
                undoContainer.style.display = 'none';
                item.remove();
                lastDeletedItem = null;
            }, 3000);
        }

        function undoDelete() {
            clearTimeout(undoTimeout);
            if (lastDeletedItem) {
                lastDeletedItem.style.display = '';
                lastDeletedItem = null;
            }
            document.getElementById('undoContainer').style.display = 'none';
        }

        function updateProgress(checkbox) {
            const subjectDiv = checkbox.closest('.subject');
            const progressBar = subjectDiv.querySelector('.progress-bar');
            const progressText = subjectDiv.querySelector('.progress span');
            const checkboxes = subjectDiv.querySelectorAll('.chapter-item input[type="checkbox"]:not(.revision-checkbox)');
            const checkedCheckboxes = Array.from(checkboxes).filter(chk => chk.checked);

            const totalChapters = checkboxes.length;
            const completedChapters = checkedCheckboxes.length;

            const progressPercentage = totalChapters ? Math.round((completedChapters / totalChapters) * 100) : 0;

            progressBar.style.width = progressPercentage + '%';
            progressText.textContent = progressPercentage + '% completed';
        }
    </script>
</body>
</html>
