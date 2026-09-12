// ==UserScript==
// @name         kinito executor
// @namespace    http://tampermonkey.net/
// @version      1
// @description  All-in-one Poki toolkit featuring ad bypass, storage manager, GUI scaling, and full EN/VN translation.
// @author       OpenSource
// @match        https://*.poki.com/*
// @match        https://*.poki-gdn.com/*
// @match        https://*.games.poki.com/*
// @grant        none
// @run-at       document-start
// @allFrames    true
// @downloadURL https://update.greasyfork.org/scripts/593975/Poki%20Hub.user.js
// @updateURL https://update.greasyfork.org/scripts/593975/Poki%20Hub.meta.js
// ==/UserScript==

(function () {
    'use strict';

    if (window.__pokiHubLoaded) return;
    window.__pokiHubLoaded = true;

    // --- PokiSDK Ad Bypass Function ---
    function injectBypass() {
        if (window.PokiSDK) {
            console.log("PokiSDK detected. Applying ad bypass...");
            try {
                Object.defineProperty(window.PokiSDK, 'rewardedBreak', {
                    get: function() {
                        return function() {
                            console.log("Rewarded ad requested -> Auto-rewarding.");
                            return Promise.resolve(true);
                        };
                    },
                    set: function() {},
                    configurable: true
                });

                Object.defineProperty(window.PokiSDK, 'commercialBreak', {
                    get: function() {
                        return function() {
                            console.log("Commercial ad requested -> Auto-skipping.");
                            return Promise.resolve();
                        };
                    },
                    set: function() {},
                    configurable: true
                });
            } catch (e) {
                window.PokiSDK.rewardedBreak = function() {
                    return Promise.resolve(true);
                };
                window.PokiSDK.commercialBreak = function() {
                    return Promise.resolve();
                };
            }
            return true;
        }
        return false;
    }

    const bypassInterval = setInterval(() => {
        if (injectBypass()) {
            clearInterval(bypassInterval);
        }
    }, 50);
    // -----------------------------------

    let currentLang = 'EN';
    let currentScaleIndex = 0;
    const scales = ['100%', '85%', '70%', '115%'];

    const textDict = {
        EN: {
            title: "Kinito Executor",
            waiting: "Waiting for storage data...",
            warningTitle: "Notice",
            warningText: "If modified stats or values revert back after reloading, the game is using server-side validation which overrides local changes.",
            refresh: "Refresh",
            reload: "Reload Game",
            deleteTitle: "Delete",
            scaleTitle: "Change GUI Size",
            langTitle: "Toggle Language",
            collapseTitle: "Collapse/Expand"
        },
        VN: {
            title: "Kinito Executor",
            waiting: "Đang đợi dữ liệu bộ nhớ...",
            warningTitle: "Chú ý",
            warningText: "Nếu các chỉ số hoặc giá trị đã sửa bị hồi phục sau khi tải lại, trò chơi đang sử dụng xác thực phía máy chủ (server-side).",
            refresh: "Làm Mới",
            reload: "Tải Lại Game",
            deleteTitle: "Xóa",
            scaleTitle: "Thay đổi kích thước giao diện",
            langTitle: "Đổi Ngôn Ngữ",
            collapseTitle: "Thu gọn/Mở rộng"
        }
    };

    function initWhenCanvasReady() {
        if (document.getElementById('poki-hub-host')) return;

        const checkInterval = setInterval(() => {
            const gameCanvas = document.querySelector('canvas');
            if (gameCanvas && (gameCanvas.width > 100 || gameCanvas.height > 100 || gameCanvas.style.display !== 'none')) {
                clearInterval(checkInterval);
                createGUI();
            }
        }, 300);
    }

    function createGUI() {
        if (document.getElementById('poki-hub-host')) return;
        const targetParent = document.body || document.documentElement;
        if (!targetParent) return;

        const host = document.createElement('div');
        host.id = 'poki-hub-host';
        host.style.cssText = 'position: fixed; top: 0; left: 0; z-index: 2147483647; pointer-events: none;';
        const shadow = host.attachShadow({ mode: 'open' });

        const style = document.createElement('style');
        style.innerHTML = `
            #poki-hub-container {
                position: fixed !important;
                top: 15px !important;
                right: 15px !important;
                width: 390px !important;
                background: #141414 !important;
                border: 1px solid #2a2a2a !important;
                border-radius: 8px !important;
                pointer-events: auto !important;
                font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif !important;
                color: #e0e0e0 !important;
                box-shadow: 0 8px 24px rgba(0,0,0,0.6) !important;
                user-select: none !important;
                display: flex !important;
                flex-direction: column !important;
                transform-origin: top right !important;
            }
            #poki-hub-header {
                padding: 10px 12px !important;
                background: #1a1a1a !important;
                border-bottom: 1px solid #262626 !important;
                border-top-left-radius: 8px !important;
                border-top-right-radius: 8px !important;
                display: flex !important;
                justify-content: space-between !important;
                align-items: center !important;
                font-size: 12px !important;
                font-weight: 600 !important;
                color: #fff !important;
                cursor: move !important;
            }
            .header-controls {
                display: flex !important;
                gap: 6px !important;
                align-items: center !important;
                pointer-events: auto !important;
            }
            .control-btn, .lang-toggle-btn, #hub-collapse-btn {
                cursor: pointer !important;
                pointer-events: auto !important;
            }
            .lang-toggle-btn, .scale-toggle-btn {
                background: #222 !important;
                border: 1px solid #333 !important;
                color: #38bdf8 !important;
                border-radius: 4px !important;
                padding: 2px 6px !important;
                font-size: 10px !important;
                font-weight: bold !important;
            }
            .lang-toggle-btn:hover, .scale-toggle-btn:hover {
                background: #2a2a2a !important;
            }
            #hub-body-wrap {
                display: flex !important;
                flex-direction: column !important;
            }
            .poki-hub-body {
                padding: 10px !important;
                display: flex !important;
                flex-direction: column !important;
                gap: 8px !important;
                max-height: 350px !important;
                overflow-y: auto !important;
            }
            .hub-warning-box {
                background: rgba(239, 68, 68, 0.1) !important;
                border: 1px solid rgba(239, 68, 68, 0.3) !important;
                color: #fca5a5 !important;
                padding: 8px !important;
                border-radius: 6px !important;
                font-size: 10px !important;
                line-height: 1.4 !important;
            }
            .hub-card {
                background: #1a1a1a !important;
                padding: 8px !important;
                border-radius: 6px !important;
                border: 1px solid #222 !important;
                display: flex !important;
                flex-direction: column !important;
                gap: 4px !important;
                font-size: 11px !important;
            }
            .hub-card textarea {
                background: #111 !important;
                border: 1px solid #333 !important;
                border-radius: 4px !important;
                color: #38bdf8 !important;
                padding: 6px !important;
                font-size: 11px !important;
                font-family: monospace !important;
                resize: vertical !important;
                min-height: 50px !important;
                outline: none !important;
                pointer-events: auto !important;
                user-select: text !important;
            }
            .hub-btn-row {
                display: flex !important;
                gap: 6px !important;
                padding: 10px !important;
                background: #1a1a1a !important;
                border-top: 1px solid #262626 !important;
                border-bottom-left-radius: 8px !important;
                border-bottom-right-radius: 8px !important;
            }
            .hub-action-btn {
                flex: 1 !important;
                background: #222 !important;
                border: 1px solid #333 !important;
                color: #38bdf8 !important;
                border-radius: 4px !important;
                padding: 6px !important;
                font-size: 11px !important;
                font-weight: bold !important;
                cursor: pointer !important;
                text-align: center !important;
            }
            .hub-action-btn:hover {
                background: #2a2a2a !important;
            }
            .hub-reload-btn {
                background: #166534 !important;
                border-color: #22c55e !important;
                color: #4ade80 !important;
            }
            .hub-reload-btn:hover {
                background: #14532d !important;
            }
        `;
        shadow.appendChild(style);

        const container = document.createElement('div');
        container.id = 'poki-hub-container';
        container.innerHTML = `
            <div id="poki-hub-header">
                <span id="header-title">Poki Hub</span>
                <div class="header-controls">
                    <button class="scale-toggle-btn" id="scale-btn" title="Change GUI Size">100%</button>
                    <button class="lang-toggle-btn" id="lang-btn" title="Toggle Language">VN</button>
                    <span id="hub-collapse-btn" style="color:#777; font-size:16px; font-weight:bold; padding: 0 4px;" title="Collapse/Expand">−</span>
                </div>
            </div>
            <div id="hub-body-wrap">
                <div class="poki-hub-body" id="hub-storage-list">
                    <div class="hub-warning-box" id="warning-box-container"></div>
                    <div id="waiting-text" style="text-align:center;color:#71717a;font-size:11px;padding:6px;"></div>
                </div>
                <div class="hub-btn-row">
                    <button class="hub-action-btn" id="btn-refresh-hub">Refresh</button>
                    <button class="hub-action-btn hub-reload-btn" id="btn-reload-game">Reload Game</button>
                </div>
            </div>
        `;
        shadow.appendChild(container);

        function updateTexts() {
            const t = textDict[currentLang];
            shadow.getElementById('header-title').innerText = t.title;
            shadow.getElementById('lang-btn').innerText = currentLang === 'EN' ? 'VN' : 'EN';
            shadow.getElementById('lang-btn').setAttribute('title', t.langTitle);
            shadow.getElementById('scale-btn').setAttribute('title', t.scaleTitle);
            shadow.getElementById('hub-collapse-btn').setAttribute('title', t.collapseTitle);

            const warningBox = shadow.getElementById('warning-box-container');
            if (warningBox) {
                warningBox.innerHTML = `<strong>⚠️ ${t.warningTitle}:</strong> ${t.warningText}`;
            }

            shadow.getElementById('btn-refresh-hub').innerText = t.refresh;
            shadow.getElementById('btn-reload-game').innerText = t.reload;

            const waitingEl = shadow.getElementById('waiting-text');
            if (waitingEl) waitingEl.innerText = t.waiting;

            shadow.querySelectorAll('.delete-btn').forEach(btn => {
                btn.setAttribute('title', t.deleteTitle);
            });
        }

        shadow.getElementById('scale-btn').onclick = (e) => {
            e.stopPropagation();
            currentScaleIndex = (currentScaleIndex + 1) % scales.length;
            const newScale = scales[currentScaleIndex];
            container.style.transform = `scale(${parseFloat(newScale) / 100})`;
            shadow.getElementById('scale-btn').innerText = newScale;
        };

        shadow.getElementById('lang-btn').onclick = (e) => {
            e.stopPropagation();
            currentLang = currentLang === 'EN' ? 'VN' : 'EN';
            updateTexts();
            loadStorage(true); // Fully rebuilds cards to apply translation changes instantly
        };

        let isCollapsed = false;
        const bodyWrap = shadow.getElementById('hub-body-wrap');
        const collapseBtn = shadow.getElementById('hub-collapse-btn');

        collapseBtn.onclick = (e) => {
            e.stopPropagation();
            isCollapsed = !isCollapsed;
            if (isCollapsed) {
                bodyWrap.style.setProperty('display', 'none', 'important');
                collapseBtn.innerText = '+';
            } else {
                bodyWrap.style.setProperty('display', 'flex', 'important');
                collapseBtn.innerText = '−';
            }
        };

        shadow.getElementById('btn-refresh-hub').onclick = () => loadStorage(true);
        shadow.getElementById('btn-reload-game').onclick = () => {
            window.location.reload();
        };

        window.addEventListener('keydown', (e) => {
            if (e.target.tagName === 'TEXTAREA' || e.target.tagName === 'INPUT') return;
            if (e.key.toLowerCase() === 'h') {
                container.style.display = container.style.display === 'none' ? 'flex' : 'none';
            }
        });

        const header = shadow.getElementById('poki-hub-header');
        let isDragging = false;
        let startX, startY, initialX, initialY;

        header.onmousedown = (e) => {
            if (e.target.id === 'hub-collapse-btn' || e.target.id === 'lang-btn' || e.target.id === 'scale-btn' || e.target.closest('.header-controls')) return;
            isDragging = true;
            startX = e.clientX;
            startY = e.clientY;
            initialX = container.offsetLeft;
            initialY = container.offsetTop;
            document.addEventListener('mousemove', onMouseMove);
            document.addEventListener('mouseup', onMouseUp);
        };

        function onMouseMove(e) {
            if (!isDragging) return;
            container.style.left = (initialX + (e.clientX - startX)) + 'px';
            container.style.top = (initialY + (e.clientY - startY)) + 'px';
            container.style.right = 'auto';
        }

        function onMouseUp() {
            isDragging = false;
            document.removeEventListener('mousemove', onMouseMove);
            document.removeEventListener('mouseup', onMouseUp);
        }

        function loadStorage(forceRebuild = false) {
            const listContainer = shadow.getElementById('hub-storage-list');
            if (!listContainer) return;

            try {
                const t = textDict[currentLang];

                const items = [];
                const processStore = (store, typeLabel) => {
                    if (!store) return;
                    for (let i = 0; i < store.length; i++) {
                        const key = store.key(i);
                        const value = store.getItem(key);
                        items.push({ store, typeLabel, key, value });
                    }
                };

                if (typeof localStorage !== 'undefined') processStore(localStorage, 'LOCAL');
                if (typeof sessionStorage !== 'undefined') processStore(sessionStorage, 'SESSION');

                const existingCards = listContainer.querySelectorAll('.hub-card');
                if (forceRebuild || existingCards.length !== items.length) {
                    let tempHTML = `<div class="hub-warning-box" id="warning-box-container"><strong>⚠️ ${t.warningTitle}:</strong> ${t.warningText}</div>`;

                    if (items.length === 0) {
                        tempHTML += `<div id="waiting-text" style="text-align:center;color:#71717a;font-size:11px;padding:6px;">${t.waiting}</div>`;
                    } else {
                        items.forEach(item => {
                            let prettyVal = item.value;
                            try {
                                prettyVal = JSON.stringify(JSON.parse(item.value), null, 2);
                            } catch (e) {}

                            tempHTML += `
                                <div class="hub-card" data-key="${item.key}" data-type="${item.typeLabel}">
                                    <div style="display:flex; justify-content:space-between; color:#a1a1aa; font-weight:600;">
                                        <span>[${item.typeLabel}] ${item.key}</span>
                                        <span class="delete-btn" style="color:#ef4444; cursor:pointer; font-weight:bold;" title="${t.deleteTitle}">×</span>
                                    </div>
                                    <textarea class="hub-textarea">${prettyVal}</textarea>
                                </div>
                            `;
                        });
                    }
                    listContainer.innerHTML = tempHTML;

                    listContainer.querySelectorAll('.hub-card').forEach(card => {
                        const k = card.getAttribute('data-key');
                        const tp = card.getAttribute('data-type');
                        const targetStore = tp === 'LOCAL' ? localStorage : sessionStorage;

                        card.querySelector('.delete-btn').onclick = () => {
                            targetStore.removeItem(k);
                            loadStorage(true);
                        };

                        const textarea = card.querySelector('.hub-textarea');

                        ['keydown', 'keypress', 'keyup', 'input', 'click', 'mousedown'].forEach(eventType => {
                            textarea.addEventListener(eventType, (e) => {
                                e.stopPropagation();
                            });
                        });

                        textarea.oninput = () => {
                            targetStore.setItem(k, textarea.value);
                        };
                    });
                } else {
                    items.forEach((item, index) => {
                        const card = existingCards[index];
                        if (card) {
                            const textarea = card.querySelector('.hub-textarea');
                            if (shadow.activeElement !== textarea) {
                                let prettyVal = item.value;
                                try {
                                    prettyVal = JSON.stringify(JSON.parse(item.value), null, 2);
                                } catch (e) {}
                                if (textarea.value !== prettyVal) {
                                    textarea.value = prettyVal;
                                }
                            }
                        }
                    });
                }
            } catch (e) {}
        }

        updateTexts();
        setInterval(() => loadStorage(false), 1000);
        targetParent.appendChild(host);
        loadStorage(true);
        showNotification(shadow);
    }

    function showNotification(shadowRoot) {
        const notif = document.createElement('div');
        notif.style.cssText = `
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 2147483647;
            background: #18181b;
            color: #f4f4f5;
            padding: 8px 16px;
            border-radius: 6px;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            font-size: 11px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.5);
            border: 1px solid #27272a;
            transition: opacity 0.5s ease;
            opacity: 1;
            pointer-events: auto;
            letter-spacing: 0.3px;
        `;
        notif.innerHTML = '<span style="color: #38bdf8; font-weight: bold;">Poki Hub Loaded</span> (Made by OpenSource | Press <span style="color:#fff;">H</span> to toggle)';
        shadowRoot.appendChild(notif);

        setTimeout(() => {
            notif.style.opacity = '0';
            setTimeout(() => notif.remove(), 500);
        }, 3000);
    }

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', initWhenCanvasReady);
    } else {
        initWhenCanvasReady();
    }
})();
