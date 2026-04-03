<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>JSON可视化工具2</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Consolas, "Courier New", monospace;
            font-size: 17px;
            color: #222;
        }

        .layout {
            display: flex;
            height: 100vh;
            overflow: hidden;
        }

        .panel-input {
            flex: 0 0 35%;
            border-right: 1px solid #ccc;
            padding: 8px;
            display: flex;
            flex-direction: column;
            gap: 8px;
            min-width: 0;
        }

        .panel-view {
            flex: 0 0 65%;
            display: flex;
            flex-direction: column;
            min-width: 0;
            min-height: 0;
        }

        .toolbar {
            display: flex;
            flex-wrap: wrap;
            gap: 6px;
        }

        button {
            border: 1px solid #999;
            background: #fff;
            padding: 4px 10px;
            font: inherit;
            cursor: pointer;
        }

        button.primary {
            font-weight: bold;
        }

        #jsonInput {
            flex: 1;
            width: 100%;
            min-height: 120px;
            padding: 8px;
            border: 1px solid #ccc;
            resize: none;
            font-family: inherit;
            font-size: 15px;
            line-height: 1.4;
        }

        .error-msg {
            color: #c00;
            font-size: 12px;
        }

        .hint {
            margin: 0;
            font-size: 12px;
            color: #666;
        }

        .viewer-toolbar {
            padding: 8px 27px;
            border-bottom: 0px solid #ccc;
        }

        .viewer-wrapper {
            flex: 1;
            display: flex;
            overflow: auto;
            min-height: 0;
            border-top: 1px solid #ccc;
        }

        .line-numbers {
            flex-shrink: 0;
            text-align: right;
            padding: 8px 8px 8px 4px;
            border-right: 1px solid #ddd;
            color: #888;
            user-select: none;
            white-space: pre;
        }

        .line-numbers div {
            line-height: 22px;
            height: 22px;
        }

        .tree-content {
            flex: 1;
            padding: 8px;
            overflow: auto;
            min-width: 0;
        }

        .tree-row {
            display: flex;
            align-items: flex-start;
            min-height: 22px;
            line-height: 22px;
            white-space: nowrap;
        }

        .indent-spacer {
            display: inline-block;
            flex-shrink: 0;
            width: 18px;
        }

        .indent-level {
            display: inline-flex;
            flex-shrink: 0;
        }

        .toggle-icon {
            display: inline-flex;
            width: 18px;
            margin-right: 2px;
            cursor: pointer;
            flex-shrink: 0;
            user-select: none;
            color: #3b82f6;
        }

        .empty-icon {
            display: inline-block;
            width: 20px;
            flex-shrink: 0;
        }

        .node-content {
            word-break: break-all;
            white-space: pre-wrap;
            flex: 1;
        }

        .json-key {
            color: darkorchid;
            font-weight: bold;
        }

        /* 仅可折叠/展开的容器节点字段名显示手形指针 */
        .json-key-foldable {
            cursor: pointer;
        }

        .json-string {
            color: #3ab54a;
            font-weight: bold;
        }

        .json-number {
            color: #a60;
            font-weight: bold;
        }

        .json-boolean {
            color: #05a;
            font-weight: bold;
        }

        .json-null {
            color: #888;
            font-weight: bold;
        }

        .json-bracket {
            color: #444;
            font-weight: bold;
        }

        .json-count {
            color: darkgrey;
            font-size: 0.9em;
            margin-left: 4px;
            font-weight: bold;
        }

        .json-count-value {
            color: #3b82f6;
            font-size: 23px;
            vertical-align: top;
            font-weight: bold;
        }

        .json-comma {
            color: #444;
            margin-left: 2px;
        }

        .collapse-placeholder {
            color: #666;
            cursor: pointer;
        }
    </style>
</head>

<body>
    <div class="layout">
        <aside class="panel-input">
            <div class="toolbar">
                <button type="button" id="btnClearJson">清空内容</button>
                <button type="button" id="btnSampleJson">示例Json</button>
                <button type="button" id="btnMinifyJson">压缩json</button>
                <button type="button" id="btnFormatJson">格式化json</button>
                <button type="button" id="btnRender" style="display:none;" class="primary">手动渲染</button>
            </div>
            <textarea id="jsonInput" rows="8" spellcheck="false"></textarea>
            <div id="errorMsg" class="error-msg" style="display: none;"></div>
            <p class="hint">输入后自动刷新右侧可视化界面（约 300ms 防抖）</p>
        </aside>
        <main class="panel-view">
            <div class="toolbar viewer-toolbar">
                <button type="button" id="btnToggleLines">隐藏行号</button>
                <button type="button" id="btnExpandAll">全部展开</button>
                <button type="button" id="btnCollapseAll">全部折叠</button>
            </div>
            <div class="viewer-wrapper">
                <div id="lineNumbersArea" class="line-numbers"></div>
                <div id="treeContainer" class="tree-content"></div>
            </div>
        </main>
    </div>

    <script>
        (function () {
            let currentJsonData = null;
            let foldState = new Map();
            let showLineNumbers = true;

            const jsonInput = document.getElementById('jsonInput');
            const btnClearJson = document.getElementById('btnClearJson');
            const btnSampleJson = document.getElementById('btnSampleJson');
            const btnMinifyJson = document.getElementById('btnMinifyJson');
            const btnFormatJson = document.getElementById('btnFormatJson');
            const btnRender = document.getElementById('btnRender');
            const btnToggleLines = document.getElementById('btnToggleLines');
            const btnExpandAll = document.getElementById('btnExpandAll');
            const btnCollapseAll = document.getElementById('btnCollapseAll');
            const treeContainer = document.getElementById('treeContainer');
            const lineNumbersArea = document.getElementById('lineNumbersArea');
            const errorMsgDiv = document.getElementById('errorMsg');

            function escapeHtml(str) {
                if (typeof str !== 'string') return String(str);
                return str.replace(/[&<>]/g, function (m) {
                    if (m === '&') return '&amp;';
                    if (m === '<') return '&lt;';
                    if (m === '>') return '&gt;';
                    return m;
                });
            }

            function getTypeCount(value) {
                if (value === null) return { type: 'null', count: 0 };
                if (Array.isArray(value)) return { type: 'array', count: value.length };
                if (typeof value === 'object') return { type: 'object', count: Object.keys(value).length };
                return { type: 'primitive', count: 0 };
            }

            /** 将非负整数逐位转为 Unicode 圆圈数字（⓪①…⑨），便于任意位数展示 */
            function toCircledDigits(n) {
                const num = Math.floor(Number(n));
                if (!Number.isFinite(num) || num < 0) return escapeHtml(String(n));
                let out = '';
                for (const ch of String(num)) {
                    const d = ch.charCodeAt(0) - 48;
                    if (d < 0 || d > 9) {
                        out += escapeHtml(ch);
                        continue;
                    }
                    out += d === 0 ? '\u24EA' : String.fromCharCode(0x245F + d);
                }
                return out;
            }

            function itemCountHtml(itemCount) {
                return `<span class="json-count-value">${toCircledDigits(itemCount)}</span>`;
            }

            function buildPath(parentPath, key) {
                if (parentPath === 'root') {
                    return key !== undefined ? `root.${key}` : 'root';
                }
                return key !== undefined ? `${parentPath}.${key}` : parentPath;
            }

            function isFolded(path) {
                return foldState.get(path) === true;
            }

            function setFoldState(path, folded) {
                foldState.set(path, folded);
                renderTree();
            }

            function toggleFold(path) {
                setFoldState(path, !isFolded(path));
            }

            function buildRowsFromNode(node, key, path, indentLevel, isRoot = false, isLastChild = true) {
                const rows = [];
                const nodeType = Array.isArray(node) ? 'array' : (node === null ? 'null' : typeof node);
                const isContainer = (nodeType === 'object' && node !== null) || Array.isArray(node);

                const showKey = !isRoot && key !== null && key !== undefined;
                let keyHtml = '';
                if (showKey) {
                    const keyClass = isContainer ? 'json-key json-key-foldable' : 'json-key';
                    keyHtml = `<span class="${keyClass}">"${escapeHtml(String(key))}"</span><span class="json-bracket">: </span>`;
                }

                if (isContainer) {
                    const typeInfo = getTypeCount(node);
                    const itemCount = typeInfo.count;
                    const isArray = Array.isArray(node);
                    const bracketOpen = isArray ? '[' : '{';
                    const bracketClose = isArray ? ']' : '}';
                    const folded = isFolded(path);

                    const countLabel = isArray
                        ? `${itemCountHtml(itemCount)} Array`
                        : `${itemCountHtml(itemCount)} Object`;
                    let startContent = '';
                    if (folded) {
                        startContent = `${keyHtml}<span class="json-bracket">${bracketOpen}</span><span class="collapse-placeholder"> … </span><span class="json-bracket">${bracketClose}</span><span class="json-count">${countLabel}</span>`;
                    } else {
                        startContent = `${keyHtml}<span class="json-bracket">${bracketOpen}</span><span class="json-count">${countLabel}</span>`;
                    }

                    rows.push({
                        indentLevel,
                        contentHtml: startContent,
                        isFoldable: true,
                        nodePath: path,
                        foldStatus: folded,
                        rawNode: node,
                        isStartRow: true,
                        bracketClose
                    });

                    if (!folded) {
                        const childIndent = indentLevel + 1;
                        let entries = [];

                        if (isArray) {
                            entries = node.map((val, idx) => ({
                                key: null,
                                value: val,
                                childPath: buildPath(path, idx),
                                isLast: idx === node.length - 1
                            }));
                        } else {
                            const keys = Object.keys(node);
                            entries = keys.map((k, idx) => ({
                                key: k,
                                value: node[k],
                                childPath: buildPath(path, k),
                                isLast: idx === keys.length - 1
                            }));
                        }

                        for (let i = 0; i < entries.length; i++) {
                            const { key: childKey, value: childVal, childPath, isLast } = entries[i];
                            const childRows = buildRowsFromNode(childVal, childKey, childPath, childIndent, false, isLast);
                            rows.push(...childRows);
                        }

                        const endRowHtml = `<span class="json-bracket">${bracketClose}</span>`;
                        rows.push({
                            indentLevel,
                            contentHtml: endRowHtml,
                            isFoldable: false,
                            nodePath: path + "_end",
                            foldStatus: false,
                            isEndRow: true
                        });
                    }
                } else {
                    let valueHtml = '';
                    if (nodeType === 'string') {
                        valueHtml = `<span class="json-string">"${escapeHtml(node)}"</span>`;
                    } else if (nodeType === 'number') {
                        valueHtml = `<span class="json-number">${escapeHtml(String(node))}</span>`;
                    } else if (nodeType === 'boolean') {
                        valueHtml = `<span class="json-boolean">${node}</span>`;
                    } else if (node === null) {
                        valueHtml = `<span class="json-null">null</span>`;
                    } else {
                        valueHtml = escapeHtml(String(node));
                    }

                    const content = `${keyHtml}${valueHtml}`;
                    rows.push({
                        indentLevel,
                        contentHtml: content,
                        isFoldable: false,
                        nodePath: path,
                        rawNode: node
                    });
                }

                if (!isLastChild && rows.length > 0) {
                    const lastRow = rows[rows.length - 1];
                    if (!lastRow.contentHtml.includes('<span class="json-comma">,</span>')) {
                        lastRow.contentHtml += '<span class="json-comma">,</span>';
                    }
                }

                return rows;
            }

            function renderTree() {
                if (!currentJsonData) {
                    treeContainer.innerHTML = '<div style="color:#888;">暂无有效数据，请输入合法的 JSON</div>';
                    lineNumbersArea.innerHTML = '';
                    return;
                }

                let rows = [];
                const rootIsContainer = (typeof currentJsonData === 'object' && currentJsonData !== null);
                if (rootIsContainer) {
                    rows = buildRowsFromNode(currentJsonData, null, 'root', 0, true, true);
                } else {
                    let valueHtml = '';
                    const t = typeof currentJsonData;
                    if (t === 'string') valueHtml = `<span class="json-string">"${escapeHtml(currentJsonData)}"</span>`;
                    else if (t === 'number') valueHtml = `<span class="json-number">${currentJsonData}</span>`;
                    else if (t === 'boolean') valueHtml = `<span class="json-boolean">${currentJsonData}</span>`;
                    else if (currentJsonData === null) valueHtml = `<span class="json-null">null</span>`;
                    else valueHtml = escapeHtml(String(currentJsonData));
                    rows.push({
                        indentLevel: 0,
                        contentHtml: valueHtml,
                        isFoldable: false,
                        nodePath: 'root'
                    });
                }

                const fragment = document.createDocumentFragment();
                for (let i = 0; i < rows.length; i++) {
                    const row = rows[i];
                    const div = document.createElement('div');
                    div.className = 'tree-row';
                    div.setAttribute('data-row-idx', i);
                    if (row.isFoldable) {
                        div.setAttribute('data-foldable', 'true');
                        div.setAttribute('data-path', row.nodePath);
                    } else {
                        div.setAttribute('data-foldable', 'false');
                    }

                    let indentHtml = '';
                    for (let lvl = 0; lvl < row.indentLevel; lvl++) {
                        indentHtml += '<span class="indent-spacer"></span>';
                    }
                    indentHtml = `<span class="indent-level">${indentHtml}</span>`;

                    let toggleHtml = '';
                    if (row.isFoldable) {
                        const icon = row.foldStatus ? '<span style="font-family: monospace;">▶</span>' : '<span style="font-family: monospace;">▼</span>';
                        toggleHtml = `<span class="toggle-icon" data-path="${row.nodePath}">${icon}</span>`;
                    } else {
                        toggleHtml = '<span class="empty-icon"></span>';
                    }

                    const contentHtml = `<div class="node-content">${row.contentHtml}</div>`;
                    div.innerHTML = indentHtml + toggleHtml + contentHtml;
                    fragment.appendChild(div);
                }

                treeContainer.innerHTML = '';
                treeContainer.appendChild(fragment);

                if (window._treeClickHandler) {
                    treeContainer.removeEventListener('click', window._treeClickHandler);
                }
                const clickHandler = function (e) {
                    let target = e.target;
                    let targetRow = target.closest('.tree-row');
                    if (!targetRow) return;
                    const isFoldable = targetRow.getAttribute('data-foldable');
                    if (isFoldable !== 'true') return;

                    let path = targetRow.getAttribute('data-path');
                    if (!path) {
                        const iconSpan = target.closest('.toggle-icon');
                        if (iconSpan && iconSpan.getAttribute('data-path')) {
                            path = iconSpan.getAttribute('data-path');
                        }
                    }
                    if (path) {
                        e.stopPropagation();
                        toggleFold(path);
                    }
                };
                treeContainer.addEventListener('click', clickHandler);
                window._treeClickHandler = clickHandler;

                updateLineNumbers(rows.length);
            }

            function updateLineNumbers(rowCount) {
                if (!showLineNumbers) {
                    lineNumbersArea.style.display = 'none';
                    return;
                }
                lineNumbersArea.style.display = 'block';
                const lines = [];
                for (let i = 1; i <= rowCount; i++) {
                    lines.push(`<div>${i}</div>`);
                }
                lineNumbersArea.innerHTML = lines.join('');
            }

            function toggleLineNumbers() {
                showLineNumbers = !showLineNumbers;
                if (showLineNumbers) {
                    btnToggleLines.textContent = '隐藏行号';
                    const rowsCount = treeContainer.querySelectorAll('.tree-row').length;
                    updateLineNumbers(rowsCount);
                } else {
                    btnToggleLines.textContent = '显示行号';
                    lineNumbersArea.style.display = 'none';
                }
            }

            function expandAll() {
                if (!currentJsonData) return;
                const allPaths = new Set();
                function collect(node, path) {
                    const isContainer = (node !== null && typeof node === 'object');
                    if (isContainer) {
                        allPaths.add(path);
                        if (Array.isArray(node)) {
                            for (let i = 0; i < node.length; i++) {
                                collect(node[i], buildPath(path, i));
                            }
                        } else {
                            for (const [k, v] of Object.entries(node)) {
                                collect(v, buildPath(path, k));
                            }
                        }
                    }
                }
                collect(currentJsonData, 'root');
                for (let p of allPaths) foldState.set(p, false);
                renderTree();
            }

            function collapseAll() {
                if (!currentJsonData) return;
                const allPaths = new Set();
                function collect(node, path) {
                    const isContainer = (node !== null && typeof node === 'object');
                    if (isContainer) {
                        allPaths.add(path);
                        if (Array.isArray(node)) {
                            for (let i = 0; i < node.length; i++) {
                                collect(node[i], buildPath(path, i));
                            }
                        } else {
                            for (const [k, v] of Object.entries(node)) {
                                collect(v, buildPath(path, k));
                            }
                        }
                    }
                }
                collect(currentJsonData, 'root');
                for (let p of allPaths) foldState.set(p, true);
                renderTree();
            }

            function renderJsonFromSource() {
                const raw = jsonInput.value.trim();
                if (!raw) {
                    errorMsgDiv.style.display = 'block';
                    errorMsgDiv.textContent = 'JSON 内容为空';
                    currentJsonData = null;
                    treeContainer.innerHTML = '<div style="color:#c00;">请输入 JSON 数据</div>';
                    lineNumbersArea.innerHTML = '';
                    return;
                }
                try {
                    const parsed = JSON.parse(raw);
                    currentJsonData = parsed;
                    errorMsgDiv.style.display = 'none';
                    foldState.clear();
                    renderTree();
                } catch (err) {
                    errorMsgDiv.style.display = 'block';
                    errorMsgDiv.textContent = '解析错误: ' + err.message;
                    currentJsonData = null;
                    treeContainer.innerHTML = '<div style="color:#c00;">JSON 格式无效: ' + escapeHtml(err.message) + '</div>';
                    lineNumbersArea.innerHTML = '';
                }
            }

            let debounceTimerAuto = null;
            function autoRender() {
                if (debounceTimerAuto) clearTimeout(debounceTimerAuto);
                debounceTimerAuto = setTimeout(() => {
                    renderJsonFromSource();
                }, 300);
            }

            function applyJsonToInput(stringifyFn) {
                const raw = jsonInput.value.trim();
                if (!raw) {
                    errorMsgDiv.style.display = 'block';
                    errorMsgDiv.textContent = 'JSON 内容为空';
                    return;
                }
                try {
                    const parsed = JSON.parse(raw);
                    jsonInput.value = stringifyFn(parsed);
                    if (debounceTimerAuto) clearTimeout(debounceTimerAuto);
                    renderJsonFromSource();
                } catch (err) {
                    errorMsgDiv.style.display = 'block';
                    errorMsgDiv.textContent = '解析错误: ' + err.message;
                }
            }

            function loadDefaultJSON() {
                if (debounceTimerAuto) clearTimeout(debounceTimerAuto);
                const defaultData = {
                    "🎧name": "JSON可视化工具",
                    "🌐url": "ktcto.com/json.htm",
                    "features": {
                        "1.点击节点折叠": true,
                        "2.显示元素数量": true,
                        "3.基于AI生成代码": true,
                        "4.Vibe Coding": "🎵",
                        "5.键名双引号": "💡",
                        "6.数组索引": "💻",
                        "7.自动渲染": "💎修改后实时更新"
                    },
                    "sampleArray": ["🤖元素1", "📚元素2", { "inner": "✅多层级嵌套", "values": [108, 308, 508] }],
                    "gitCode": {
                        "💖stars": 128,
                        "⌚forks": 32
                    }
                };
                jsonInput.value = JSON.stringify(defaultData, null, 2);
                renderJsonFromSource();
            }

            btnClearJson.addEventListener('click', () => {
                jsonInput.value = '';
                if (debounceTimerAuto) clearTimeout(debounceTimerAuto);
                renderJsonFromSource();
            });
            btnSampleJson.addEventListener('click', () => {
                loadDefaultJSON();
            });
            btnMinifyJson.addEventListener('click', () => {
                applyJsonToInput(function (parsed) {
                    return JSON.stringify(parsed);
                });
            });
            btnFormatJson.addEventListener('click', () => {
                applyJsonToInput(function (parsed) {
                    return JSON.stringify(parsed, null, 2);
                });
            });
            btnRender.addEventListener('click', () => {
                if (debounceTimerAuto) clearTimeout(debounceTimerAuto);
                renderJsonFromSource();
            });

            jsonInput.addEventListener('input', () => {
                autoRender();
            });

            btnToggleLines.addEventListener('click', () => {
                toggleLineNumbers();
                if (showLineNumbers) {
                    const rows = treeContainer.querySelectorAll('.tree-row').length;
                    updateLineNumbers(rows);
                }
            });
            btnExpandAll.addEventListener('click', () => {
                if (!currentJsonData) {
                    renderJsonFromSource();
                }
                if (currentJsonData) expandAll();
            });
            btnCollapseAll.addEventListener('click', () => {
                if (!currentJsonData) {
                    renderJsonFromSource();
                }
                if (currentJsonData) collapseAll();
            });

            loadDefaultJSON();
        })();
    </script>
</body>

</html>