// ==UserScript==
// @name         Kinito Web Executor & Poki Hub
// @namespace    http://tampermonkey.net/
// @version      2.2
// @description  Kinito executor with integrated Poki Hub, state memory, collapsible icon state, and smooth UI animations.
// @match        *://*/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    if (window.self !== window.top) return;
    if (window.__customExecutorInjected) return;
    window.__customExecutorInjected = true;

    // Load saved scale and collapse state from memory if available
    let currentScale = localStorage.getItem('kinito_ui_scale') || '100%';
    let isCollapsed = localStorage.getItem('kinito_ui_collapsed') === 'true';

    const kinitoIconUri = "data:image/svg+xml;utf8," + encodeURIComponent(`
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
            <circle cx="50" cy="50" r="46" fill="#5c2d91"/>
            <circle cx="50" cy="50" r="32" fill="#1e1e2f" stroke="#c5a059" stroke-width="5"/>
            <polygon points="50,24 54,50 50,76 46,50" fill="#3b82f6"/>
            <polygon points="24,50 50,54 76,50 50,46" fill="#c5a059"/>
            <circle cx="50" cy="50" r="8" fill="#ffffff"/>
        </svg>
    `);

    const css = `
        #ce-container {
            position: fixed;
            top: 20px;
            right: 20px;
            width: 360px;
            background: #161618;
            color: #ffffff;
            border: 1px solid #2d2d30;
            border-radius: 10px;
            box-shadow: 0 12px 35px rgba(0,0,0,0.7);
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            font-size: 13px;
            z-index: 999999;
            user-select: none;
            transition: width 0.3s ease, border-radius 0.3s ease;
            transform-origin: top right;
        }
        #ce-header {
            background: #1f1f23;
            padding: 10px 14px;
            font-weight: 600;
            cursor: move;
            border-bottom: 1px solid #2d2d30;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-top-left-radius: 10px;
            border-top-right-radius: 10px;
        }
        .ce-title-area {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .ce-icon {
            width: 22px;
            height: 22px;
            object-fit: contain;
            border-radius: 50%;
            transition: transform 0.3s ease;
        }
        .ce-header-controls {
            display: flex;
            align-items: center;
            gap: 6px;
        }
        .ce-ctrl-btn {
            background: none;
            border: none;
            color: #888;
            cursor: pointer;
            font-size: 14px;
            padding: 2px 6px;
            border-radius: 4px;
            transition: all 0.2s ease;
        }
        .ce-ctrl-btn:hover {
            background: #2a2a2e;
            color: #fff;
        }
        #ce-settings-menu {
            display: none;
            position: absolute;
            top: 36px;
            right: 40px;
            width: 170px;
            background: #1f1f23;
            border: 1px solid #3f3f46;
            border-radius: 6px;
            box-shadow: 0 6px 18px rgba(0,0,0,0.6);
            z-index: 1000001;
            padding: 6px;
            animation: ce-fadein 0.15s ease-out;
        }
        #ce-settings-menu.active {
            display: block;
        }
        .ce-menu-title {
            font-size: 10px;
            color: #777;
            padding: 4px 8px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        .ce-menu-item {
            padding: 6px 8px;
            font-size: 11px;
            color: #ccc;
            cursor: pointer;
            border-radius: 4px;
            display: flex;
            justify-content: space-between;
            transition: background 0.15s;
        }
        .ce-menu-item:hover {
            background: #2a2d2e;
            color: #fff;
        }
        .ce-menu-item.selected {
            color: #4ec9b0;
            font-weight: 600;
        }
        #ce-body {
            padding: 12px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            transition: max-height 0.3s ease, opacity 0.2s ease, padding 0.3s ease;
            max-height: 400px;
            opacity: 1;
            overflow: hidden;
        }
        /* COLLAPSED STATE STYLES */
        #ce-container.collapsed #ce-body {
            max-height: 0 !important;
            opacity: 0 !important;
            padding-top: 0 !important;
            padding-bottom: 0 !important;
        }
        #ce-container.collapsed .ce-icon {
            transform: scale(1.3);
        }
        #ce-editor {
            width: 100%;
            height: 140px;
            background: #0d0d0f;
            color: #d4d4d4;
            border: 1px solid #2d2d30;
            border-radius: 6px;
            padding: 8px;
            font-family: 'Courier New', Courier, monospace;
            font-size: 12px;
            resize: vertical;
            box-sizing: border-box;
            outline: none;
            transition: border-color 0.2s;
        }
        #ce-editor:focus {
            border-color: #007acc;
        }
        #ce-footer {
            display: flex;
            gap: 8px;
        }
        .ce-btn {
            flex: 1;
            padding: 8px 0;
            border: none;
            border-radius: 6px;
            font-weight: 600;
            cursor: pointer;
            color: #fff;
            font-size: 12px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.3);
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .ce-btn:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 6px 14px rgba(0,0,0,0.5);
            filter: brightness(1.15);
        }
        .ce-btn:active:not(:disabled) {
            transform: translateY(1px) scale(0.97);
            box-shadow: 0 1px 3px rgba(0,0,0,0.2);
        }
        #ce-inject { background: #0e639c; }
        #ce-execute { background: #28a745; }
        #ce-reset { background: #d9534f; }
        .ce-btn:disabled {
            opacity: 0.4;
            cursor: not-allowed;
            transform: none !important;
            box-shadow: none !important;
        }
        @keyframes ce-fadein {
            from { opacity: 0; transform: translateY(-5px); }
            to { opacity: 1; transform: translateY(0); }
        }
    `;

    const styleTag = document.createElement('style');
    styleTag.innerHTML = css;
    document.head.appendChild(styleTag);

    const container = document.createElement('div');
    container.id = 'ce-container';
    if (isCollapsed) container.classList.add('collapsed');
    container.style.transform = `scale(${parseFloat(currentScale) / 100})`;

    container.innerHTML = `
        <div id="ce-header">
            <div class="ce-title-area">
                <img src="${kinitoIconUri}" class="ce-icon" alt="Kinito">
                <span id="ce-title-text">Kinito Executor</span>
            </div>
            <div class="ce-header-controls">
                <span id="ce-status" style="font-size:11px; color:#888; margin-right:4px;">Idle</span>
                <button class="ce-ctrl-btn" id="ce-scale-btn" title="UI Scale">${currentScale}</button>
                <div id="ce-settings-menu">
                    <div class="ce-menu-title">UI Scale (Memory)</div>
                    <div class="ce-menu-item ${currentScale === '100%' ? 'selected' : ''}" data-scale="100%">100%</div>
                    <div class="ce-menu-item ${currentScale === '85%' ? 'selected' : ''}" data-scale="85%">85%</div>
                    <div class="ce-menu-item ${currentScale === '70%' ? 'selected' : ''}" data-scale="70%">70%</div>
                    <div class="ce-menu-item ${currentScale === '115%' ? 'selected' : ''}" data-scale="115%">115%</div>
                </div>
                <button class="ce-ctrl-btn" id="ce-collapse-btn" title="Collapse/Expand">${isCollapsed ? '+' : '−'}</button>
                <button class="ce-ctrl-btn" id="ce-close-btn" title="Close Panel">×</button>
            </div>
        </div>
        <div id="ce-body">
            <textarea id="ce-editor" placeholder="// Write your JavaScript code here...&#10;console.log('Kinito ready!');"></textarea>
            <div id="ce-footer">
                <button id="ce-inject" class="ce-btn">Inject</button>
                <button id="ce-execute" class="ce-btn" disabled>Execute</button>
                <button id="ce-reset" class="ce-btn">Reset</button>
            </div>
        </div>
    `;
    document.body.appendChild(container);

    const editor = document.getElementById('ce-editor');
    const btnInject = document.getElementById('ce-inject');
    const btnExecute = document.getElementById('ce-execute');
    const btnReset = document.getElementById('ce-reset');
    const statusLabel = document.getElementById('ce-status');
    const scaleBtn = document.getElementById('ce-scale-btn');
    const settingsMenu = document.getElementById('ce-settings-menu');
    const collapseBtn = document.getElementById('ce-collapse-btn');
    const closeBtn = document.getElementById('ce-close-btn');

    let isInjected = false;

    // Scale Menu Toggle
    scaleBtn.addEventListener('click', (e) => {
        e.stopPropagation();
        settingsMenu.classList.toggle('active');
    });

    document.addEventListener('click', () => {
        settingsMenu.classList.remove('active');
    });

    settingsMenu.addEventListener('click', (e) => {
        e.stopPropagation();
    });

    // Handle Scale Selection & Memory
    settingsMenu.querySelectorAll('.ce-menu-item').forEach(item => {
        item.addEventListener('click', () => {
            settingsMenu.querySelectorAll('.ce-menu-item').forEach(i => i.classList.remove('selected'));
            item.classList.add('selected');
            currentScale = item.getAttribute('data-scale');
            scaleBtn.textContent = currentScale;
            container.style.transform = `scale(${parseFloat(currentScale) / 100})`;
            localStorage.setItem('kinito_ui_scale', currentScale);
            settingsMenu.classList.remove('active');
        });
    });

    // Collapse Toggle & Memory
    collapseBtn.addEventListener('click', () => {
        isCollapsed = !isCollapsed;
        container.classList.toggle('collapsed', isCollapsed);
        collapseBtn.textContent = isCollapsed ? '+' : '−';
        localStorage.setItem('kinito_ui_collapsed', isCollapsed);
    });

    // Close Panel Button
    closeBtn.addEventListener('click', () => {
        container.remove();
    });

    // Inject Logic
    btnInject.addEventListener('click', () => {
        if (isInjected) return;
        isInjected = true;
        btnInject.disabled = true;
        btnExecute.disabled = false;
        statusLabel.textContent = 'Injected';
        statusLabel.style.color = '#4ec9b0';
    });

    // Execute Logic
    btnExecute.addEventListener('click', () => {
        if (!isInjected) return;
        const code = editor.value;
        if (!code.trim()) return;

        try {
            const executeFn = new Function(code);
            executeFn();
            statusLabel.textContent = 'Executed ✓';
            setTimeout(() => { if(isInjected) statusLabel.textContent = 'Injected'; }, 2000);
        } catch (err) {
            console.error('Executor Error:', err);
            statusLabel.textContent = 'Error ❌';
            alert('Script error: ' + err.message);
        }
    });

    // Reset Logic
    btnReset.addEventListener('click', () => {
        editor.value = '';
        isInjected = false;
        btnInject.disabled = false;
        btnExecute.disabled = true;
        statusLabel.textContent = 'Idle';
        statusLabel.style.color = '#888';
    });

    // Draggable Window Feature
    const header = document.getElementById('ce-header');
    let isDragging = false, startX, startY, initialX, initialY;

    header.addEventListener('mousedown', (e) => {
        if (e.target.closest('#ce-settings-menu') || e.target.closest('.ce-header-controls')) return;
        isDragging = true;
        startX = e.clientX;
        startY = e.clientY;
        const rect = container.getBoundingClientRect();
        initialX = rect.left;
        initialY = rect.top;
        container.style.right = 'auto';
        container.style.left = initialX + 'px';
        container.style.top = initialY + 'px';
    });

    document.addEventListener('mousemove', (e) => {
        if (!isDragging) return;
        const dx = e.clientX - startX;
        const dy = e.clientY - startY;
        container.style.left = (initialX + dx) + 'px';
        container.style.top = (initialY + dy) + 'px';
    });

    document.addEventListener('mouseup', () => {
        isDragging = false;
    });

})();
