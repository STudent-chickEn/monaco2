<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monaco Editor - Tabs</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.40.0/min/vs/loader.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.0/jszip.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            height: 100vh;
            overflow: hidden; /* Prevent scrollbars */
            background-color: black; /* Set background color to black */
        }
        #sidebar {
            width: 100px; /* Width of the sidebar */
            background-color: #1e1e1e; /* Set background color to match the editor theme */
            display: flex;
            flex-direction: column;
            border-right: 1px solid white; /* White border for better visibility */
        }
        #tabs {
            display: flex;
            flex-direction: column;
            border-bottom: none;
            flex: 1; /* Allow tabs to take up remaining space */
        }
        #tabs button {
            padding: 5px; /* Padding for tab buttons */
            cursor: pointer;
            background-color: #1e1e1e; /* Set background color to match the editor theme */
            border: none;
            border-bottom: 1px solid white; /* White border for better visibility */
            text-align: left;
            font-size: 12px; /* Font size for tab buttons */
            color: white; /* Set text color to white */
        }
        #tabs button.active {
            background-color: #3e3e3e; /* Set active tab background color */
            font-weight: bold;
        }
        #controls {
            display: flex;
            flex-direction: column;
            padding: 5px;
            border-top: 1px solid white; /* White border for better visibility */
        }
        #controls button {
            padding: 5px 10px;
            cursor: pointer;
            margin-bottom: 5px; /* Space between buttons */
            background-color: #1e1e1e; /* Set button background color */
            color: white; /* Set button text color */
            border: 1px solid white; /* Set button border color */
        }
        #main {
            flex: 1;
            display: flex;
            flex-direction: column;
            height: 144vh;
        }
        .editor-container {
            flex: 1;
            display: none;
            height: 100%; /* Full height */
            width: 100%;
        }
        .editor-container.active {
            display: block;
        }
        #preview {
            width: 100%;
            height: 30%; /* Height of the preview panel */
            border-top: 1px solid white; /* White border for better visibility */
        }
    </style>
</head>
<body>
    <div id="sidebar">
        <div id="tabs">
            <button id="html-tab" class="active" data-editor="html-editor">HTML</button>
            <button id="js-tab" data-editor="js-editor">JavaScript</button>
            <button id="css-tab" data-editor="css-editor">CSS</button>
        </div>
        <div id="controls">
            <button id="run-button">Run</button>
            <button id="clear-button">Clear Editors</button>
            <button id="save-button">Save</button>
        </div>
    </div>

    <div id="main">
        <div id="html-editor" class="editor-container active"></div>
        <div id="js-editor" class="editor-container"></div>
        <div id="css-editor" class="editor-container"></div>
        <iframe id="preview"></iframe>
    </div>

    <script>
        require.config({ paths: { 'vs': 'https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.40.0/min/vs' } });

        require(['vs/editor/editor.main'], function () {
            // Define custom theme
            monaco.editor.defineTheme('custom-dark', {
                base: 'vs-dark',
                inherit: true,
                rules: [
                    { token: '', foreground: 'd4d4d4' }, // Default text color
                    { token: 'tag', foreground: '569cd6' }, // HTML tags blue
                    { token: 'attribute.name', foreground: '9cdcfe' }, // Attribute names light blue
                    { token: 'attribute.value', foreground: 'ce9178' }, // Attribute values orange
                    { token: 'comment', foreground: '6a9955' }, // Comments green
                    { token: 'string', foreground: 'ce9178' }, // Strings orange
                    { token: 'keyword', foreground: 'c586c0' }, // Keywords violet
                    { token: 'number', foreground: 'b5cea8' } // Numbers green
                ],
                colors: {
                    'editor.background': '#000000', // Editor background
                    'editor.foreground': '#FFFFFF', // Editor foreground
                    'editorLineNumber.foreground': '#FFFFFF', // Line numbers
                    'editor.lineHighlightBackground': '#333333', // Line highlight
                    'editorCursor.foreground': '#FFFFFF', // Cursor color
                    'editorIndentGuide.background': '#FFFFFF', // Indent guide
                    'editorIndentGuide.activeBackground': '#FFFFFF', // Active indent guide
                    'editorSuggestWidget.background': '#2d2d2d', // Suggest widget background
                    'editorSuggestWidget.foreground': '#d4d4d4', // Suggest widget foreground
                    'editorSuggestWidget.selectedBackground': '#555555', // Suggest widget selected background
                    'editorSuggestWidget.highlightForeground': '#569cd6' // Suggest widget highlight foreground (e.g., for HTML tags)
                }
            });

            // Monaco Editor instances
            const editors = {
                html: monaco.editor.create(document.getElementById('html-editor'), {
                    value: '<h1>Hello, world!</h1>',
                    language: 'html',
                    theme: 'custom-dark'
                }),
                css: monaco.editor.create(document.getElementById('css-editor'), {
                    value: 'body { background-color: lightblue; }',
                    language: 'css',
                    theme: 'custom-dark'
                }),
                js: monaco.editor.create(document.getElementById('js-editor'), {
                    value: 'console.log("Hello, world!");',
                    language: 'javascript',
                    theme: 'custom-dark'
                })
            };

            const tabs = document.querySelectorAll('#tabs button');
            tabs.forEach(tab => {
                tab.addEventListener('click', () => {
                    // Remove active class from all tabs and editors
                    tabs.forEach(t => t.classList.remove('active'));
                    document.querySelectorAll('.editor-container').forEach(editor => editor.classList.remove('active'));

                    // Activate the selected tab and editor
                    tab.classList.add('active');
                    const editorId = tab.dataset.editor;
                    document.getElementById(editorId).classList.add('active');

                    editors[editorId.split('-')[0]].layout();
                });
            });

    
            document.getElementById('run-button').addEventListener('click', () => {
                const html = editors.html.getValue();
                const css = editors.css.getValue();
                const js = editors.js.getValue();

                const newWindow = window.open("", "_blank");
                newWindow.document.open();
                newWindow.document.write(`
                    <style>${css}</style>
                    ${html}
                    <script>
                        ${js}
                    <\/script>
                `);
                newWindow.document.close();
            });

            // Clear Editors button functionality
            document.getElementById('clear-button').addEventListener('click', () => {
                editors.html.setValue('');
                editors.css.setValue('');
                editors.js.setValue('');
            });
            document.getElementById('save-button').addEventListener('click', () => {
                const html = editors.html.getValue();
                const css = editors.css.getValue();
                const js = editors.js.getValue();

                const zip = new JSZip();
                zip.file("index.html", html);
                zip.file("styles.css", css);
                zip.file("script.js", js);

                zip.generateAsync({ type: "blob" }).then(function(content) {
                    saveAs(content, "project.zip");
                });
            });
            window.addEventListener('resize', () => {
                Object.values(editors).forEach(editor => editor.layout());
            });
        });
    </script>
</body>
</html>
