<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Редактор прототипов</title>
    <style>
        :root {
            --primary-color: #0d66ff;
            --secondary-color: #f5f5f5;
            --text-color: #222;
            --light-gray: #e5e5e5;
            --dark-gray: #888;
            --danger-color: #ff3b30;
            --sidebar-bg: #1e1e1e;
            --sidebar-text: #ffffff;
            --toolbar-bg: #2d2d2d;
            --canvas-bg: #ffffff;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: #fafafa;
            color: var(--text-color);
        }

        .app-container {
            display: flex;
            height: 100vh;
            overflow: hidden;
        }

        /* Боковая панель */
        .sidebar {
            width: 250px;
            background-color: var(--sidebar-bg);
            color: var(--sidebar-text);
            border-right: 1px solid #333;
            padding: 15px;
            display: flex;
            flex-direction: column;
        }

        .sidebar-header {
            padding: 10px 0;
            border-bottom: 1px solid #333;
            margin-bottom: 15px;
        }

        .sidebar-section {
            margin-bottom: 20px;
        }

        .sidebar-title {
            font-size: 12px;
            font-weight: 600;
            color: #999;
            margin-bottom: 10px;
            text-transform: uppercase;
        }

        .layer-item {
            padding: 8px 10px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
            margin-bottom: 2px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: var(--sidebar-text);
        }

        .layer-item:hover {
            background-color: #333;
        }

        .layer-item.active {
            background-color: var(--primary-color);
            color: white;
        }

        .delete-layer {
            color: var(--danger-color);
            background: none;
            border: none;
            cursor: pointer;
            font-size: 14px;
            padding: 2px;
            display: none;
        }

        .layer-item:hover .delete-layer {
            display: block;
        }

        .layer-item.active .delete-layer {
            display: block;
            color: white;
        }

        /* Основное содержимое */
        .main-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            background-color: var(--canvas-bg);
        }

        .toolbar {
            padding: 10px 20px;
            background-color: var(--toolbar-bg);
            border-bottom: 1px solid #333;
            display: flex;
            align-items: center;
            gap: 15px;
        }

        .tool-button {
            width: 32px;
            height: 32px;
            border-radius: 4px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            background: none;
            border: none;
            color: var(--sidebar-text);
        }

        .tool-button:hover {
            background-color: #333;
        }

        .tool-button.active {
            background-color: var(--primary-color);
            color: white;
        }

        .canvas-container {
            flex: 1;
            background-color: var(--canvas-bg);
            overflow: auto;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
            position: relative;
        }

        .canvas {
            background-color: white;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            position: relative;
            transform-origin: 0 0;
        }

        .canvas-element {
            position: absolute;
            border: 1px dashed transparent;
            cursor: move;
            user-select: none;
            transform-origin: center;
        }

        .canvas-element.selected {
            border-color: var(--primary-color);
        }

        .canvas-element.text-element {
            background-color: transparent;
            padding: 5px;
        }

        .canvas-element.shape-element {
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: transparent !important;
        }

        .canvas-element img {
            width: 100%;
            height: 100%;
            object-fit: contain;
            pointer-events: none;
        }

        .resize-handle {
            width: 8px;
            height: 8px;
            background-color: white;
            border: 1px solid var(--primary-color);
            position: absolute;
            z-index: 10;
        }

        .resize-handle.nw {
            top: -4px;
            left: -4px;
            cursor: nwse-resize;
        }

        .resize-handle.ne {
            top: -4px;
            right: -4px;
            cursor: nesw-resize;
        }

        .resize-handle.sw {
            bottom: -4px;
            left: -4px;
            cursor: nesw-resize;
        }

        .resize-handle.se {
            bottom: -4px;
            right: -4px;
            cursor: nwse-resize;
        }

        .rotate-handle {
            width: 16px;
            height: 16px;
            background-color: var(--primary-color);
            border-radius: 50%;
            position: absolute;
            top: -25px;
            left: 50%;
            transform: translateX(-50%);
            cursor: grab;
            z-index: 10;
        }

        /* Направляющие и сетка */
        .canvas-grid {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-image: linear-gradient(#e5e5e5 1px, transparent 1px),
                              linear-gradient(90deg, #e5e5e5 1px, transparent 1px);
            background-size: 20px 20px;
            pointer-events: none;
            display: none;
        }

        .guide-line {
            position: absolute;
            background-color: var(--primary-color);
            z-index: 1000;
            display: none;
        }

        .guide-line.horizontal {
            height: 1px;
            width: 100%;
        }

        .guide-line.vertical {
            width: 1px;
            height: 100%;
        }

        .snap-point {
            position: absolute;
            width: 6px;
            height: 6px;
            background-color: var(--primary-color);
            border-radius: 50%;
            z-index: 1001;
            display: none;
        }

        /* Панель свойств */
        .properties-panel {
            width: 250px;
            background-color: var(--sidebar-bg);
            color: var(--sidebar-text);
            border-left: 1px solid #333;
            padding: 15px;
            overflow-y: auto;
        }

        .property-group {
            margin-bottom: 15px;
        }

        .property-label {
            font-size: 12px;
            margin-bottom: 5px;
            display: block;
            color: #999;
        }

        .property-input {
            width: 100%;
            padding: 8px;
            border: 1px solid #333;
            border-radius: 4px;
            background-color: #2d2d2d;
            color: var(--sidebar-text);
        }

        .shape-options {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 8px;
            margin-top: 10px;
        }

        .shape-option {
            width: 100%;
            aspect-ratio: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            border: 1px solid #333;
            border-radius: 4px;
            cursor: pointer;
            font-size: 20px;
            background-color: #2d2d2d;
            color: var(--sidebar-text);
        }

        .shape-option:hover {
            border-color: var(--primary-color);
        }

        /* SVG фигуры */
        .shape-svg {
            width: 100%;
            height: 100%;
            pointer-events: none;
        }

        /* Модальные окна */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 100;
        }

        .modal {
            background-color: var(--sidebar-bg);
            color: var(--sidebar-text);
            border-radius: 8px;
            width: 600px;
            max-width: 90%;
            max-height: 90%;
            overflow: auto;
            padding: 20px;
            border: 1px solid #333;
        }

        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .modal-title {
            font-size: 18px;
            font-weight: 600;
        }

        .close-button {
            background: none;
            border: none;
            font-size: 20px;
            cursor: pointer;
            color: var(--sidebar-text);
        }

        .template-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
        }

        .template-item {
            border: 1px solid #333;
            border-radius: 4px;
            overflow: hidden;
            cursor: pointer;
            background-color: #2d2d2d;
        }

        .template-item:hover {
            border-color: var(--primary-color);
        }

        .template-preview {
            height: 120px;
            background-color: var(--canvas-bg);
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
        }

        .template-preview-content {
            position: absolute;
            width: 90%;
            height: 90%;
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        .template-preview-element {
            background-color: #ddd;
            border-radius: 2px;
        }

        .template-name {
            padding: 10px;
            text-align: center;
            font-size: 14px;
        }

        .button {
            padding: 10px 15px;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: 500;
        }

        .button:hover {
            opacity: 0.9;
        }

        .button.secondary {
            background-color: #2d2d2d;
            color: var(--sidebar-text);
            border: 1px solid #333;
        }

        .button.danger {
            background-color: var(--danger-color);
        }

        .modal-footer {
            display: flex;
            justify-content: flex-end;
            gap: 10px;
            margin-top: 20px;
        }

        /* Загрузка файлов */
        .file-upload {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 30px;
            border: 2px dashed #333;
            border-radius: 8px;
            margin-bottom: 20px;
            cursor: pointer;
            background-color: #2d2d2d;
        }

        .file-upload:hover {
            border-color: var(--primary-color);
        }

        .file-upload-icon {
            font-size: 40px;
            margin-bottom: 10px;
            color: #999;
        }

        .file-input {
            display: none;
        }

        /* Коллаборация */
        .collab-section {
            margin-top: 20px;
            padding-top: 20px;
            border-top: 1px solid #333;
        }

        .collab-link {
            display: flex;
            margin-top: 10px;
        }

        .collab-input {
            flex: 1;
            padding: 8px;
            border: 1px solid #333;
            border-radius: 4px 0 0 4px;
            background-color: #2d2d2d;
            color: var(--sidebar-text);
        }

        .collab-copy {
            padding: 8px 12px;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 0 4px 4px 0;
            cursor: pointer;
        }

        /* Редактирование текста */
        .text-edit-input {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            padding: 5px;
            border: 1px solid var(--primary-color);
            background-color: white;
            font-family: inherit;
            font-size: inherit;
            color: inherit;
            resize: none;
            outline: none;
            box-sizing: border-box;
        }

        /* Обрезка изображений */
        .crop-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.8);
            z-index: 200;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        .crop-container {
            width: 80%;
            height: 80%;
            position: relative;
            background-color: white;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }

        .crop-image {
            max-width: 100%;
            max-height: 100%;
            display: block;
        }

        .crop-controls {
            margin-top: 20px;
            display: flex;
            gap: 10px;
        }

        .crop-selection {
            position: absolute;
            border: 2px dashed var(--primary-color);
            background-color: rgba(0, 0, 0, 0.3);
            cursor: move;
        }

        .crop-handle {
            width: 10px;
            height: 10px;
            background-color: var(--primary-color);
            position: absolute;
            cursor: nwse-resize;
        }

        .crop-handle.nw {
            top: -5px;
            left: -5px;
            cursor: nwse-resize;
        }

        .crop-handle.ne {
            top: -5px;
            right: -5px;
            cursor: nesw-resize;
        }

        .crop-handle.sw {
            bottom: -5px;
            left: -5px;
            cursor: nesw-resize;
        }

        .crop-handle.se {
            bottom: -5px;
            right: -5px;
            cursor: nwse-resize;
        }

        /* Цвет фона */
        .bg-color-picker {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-top: 10px;
        }

        .bg-color-label {
            font-size: 12px;
            color: #999;
        }

        .bg-color-input {
            width: 30px;
            height: 30px;
            border: 1px solid #333;
            border-radius: 4px;
            cursor: pointer;
        }

        /* Улучшенные иконки */
        .icon-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
            margin-top: 15px;
        }

        .icon-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 10px;
            border-radius: 4px;
            cursor: pointer;
            background-color: #2d2d2d;
        }

        .icon-item:hover {
            background-color: #333;
        }

        .icon-item i {
            font-size: 24px;
            margin-bottom: 5px;
        }

        .icon-item span {
            font-size: 10px;
            text-align: center;
        }
    </style>
    <!-- Font Awesome для иконок -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
</head>
<body>
    <div class="app-container">
        <!-- Боковая панель -->
        <div class="sidebar">
            <div class="sidebar-header">
                <h2>Мой проект</h2>
            </div>
            <div class="sidebar-section">
                <div class="sidebar-title">Слои</div>
                <div id="layers-list" class="layers-list">
                    <!-- Слои будут добавляться динамически -->
                </div>
            </div>
            <div class="sidebar-section">
                <button id="add-text-btn" class="button">Добавить текст</button>
                <button id="add-image-btn" class="button" style="margin-top: 10px;">Добавить изображение</button>
                <button id="add-shape-btn" class="button" style="margin-top: 10px;">Добавить фигуру</button>
                <button id="add-icon-btn" class="button" style="margin-top: 10px;">Добавить иконку</button>
            </div>
            <div class="sidebar-section">
                <div class="sidebar-title">Фон холста</div>
                <div class="bg-color-picker">
                    <span class="bg-color-label">Цвет:</span>
                    <input type="color" id="canvas-bg-color" class="bg-color-input" value="#ffffff">
                </div>
            </div>
        </div>

        <!-- Основное содержимое -->
        <div class="main-content">
            <div class="toolbar">
                <button class="tool-button active" data-tool="select" title="Выделение">
                    <i class="fas fa-mouse-pointer"></i>
                </button>
                <button class="tool-button" data-tool="rectangle" title="Прямоугольник">
                    <i class="far fa-square"></i>
                </button>
                <button class="tool-button" data-tool="text" title="Текст">
                    <i class="fas fa-font"></i>
                </button>
                <div style="flex: 1;"></div>
                <button id="crop-btn" class="button secondary" style="display: none;" title="Обрезать изображение">
                    <i class="fas fa-crop"></i>
                </button>
                <button id="delete-btn" class="button danger" style="display: none;" title="Удалить выделенное">
                    <i class="fas fa-trash"></i>
                </button>
                <button id="share-btn" class="button" title="Поделиться">
                    <i class="fas fa-share-alt"></i>
                </button>
                <button id="save-btn" class="button secondary" title="Сохранить">
                    <i class="fas fa-save"></i>
                </button>
                <button id="export-png-btn" class="button" title="Экспорт в PNG">
                    <i class="fas fa-file-export"></i>
                </button>
            </div>
            <div class="canvas-container">
                <div id="canvas" class="canvas" style="width: 800px; height: 600px;">
                    <div class="canvas-grid" id="canvas-grid"></div>
                    <!-- Элементы будут добавляться динамически -->
                </div>
                <div class="guide-line horizontal" id="horizontal-guide"></div>
                <div class="guide-line vertical" id="vertical-guide"></div>
                <div class="snap-point" id="snap-point"></div>
            </div>
        </div>

        <!-- Панель свойств -->
        <div class="properties-panel">
            <div class="sidebar-title">Свойства</div>
            <div id="properties-content">
                <div class="property-group">
                    <label class="property-label">Элемент не выбран</label>
                </div>
            </div>
            <div class="collab-section">
                <div class="sidebar-title">Совместная работа</div>
                <div class="collab-link">
                    <input type="text" id="collab-link-input" class="collab-input" readonly value="Подключитесь, чтобы увидеть ссылку">
                    <button id="copy-link-btn" class="collab-copy">Копировать</button>
                </div>
            </div>
        </div>

        <!-- Модальное окно выбора шаблона -->
        <div id="template-modal" class="modal-overlay" style="display: flex;">
            <div class="modal">
                <div class="modal-header">
                    <div class="modal-title">Выберите шаблон</div>
                    <button class="close-button" id="close-template-modal">&times;</button>
                </div>
                <div class="template-grid">
                    <div class="template-item" data-template="landing">
                        <div class="template-preview">
                            <div class="template-preview-content">
                                <div class="template-preview-element" style="height: 15%; background-color: #0d66ff;"></div>
                                <div class="template-preview-element" style="height: 10%; width: 80%; margin: 5px auto;"></div>
                                <div class="template-preview-element" style="height: 8%; width: 90%; margin: 0 auto;"></div>
                                <div class="template-preview-element" style="height: 8%; width: 40%; margin: 10px auto; background-color: #0d66ff;"></div>
                            </div>
                        </div>
                        <div class="template-name">Лендинг</div>
                    </div>
                    <div class="template-item" data-template="portfolio">
                        <div class="template-preview">
                            <div class="template-preview-content">
                                <div class="template-preview-element" style="height: 15%; width: 80%; margin: 0 auto;"></div>
                                <div style="display: flex; justify-content: space-between; height: 40%;">
                                    <div class="template-preview-element" style="width: 45%;"></div>
                                    <div class="template-preview-element" style="width: 45%;"></div>
                                </div>
                            </div>
                        </div>
                        <div class="template-name">Портфолио</div>
                    </div>
                    <div class="template-item" data-template="ecommerce">
                        <div class="template-preview">
                            <div class="template-preview-content">
                                <div class="template-preview-element" style="height: 15%; background-color: #333;"></div>
                                <div class="template-preview-element" style="height: 10%; width: 60%; margin: 5px auto;"></div>
                                <div style="display: flex; flex-wrap: wrap; gap: 5px; justify-content: center; height: 60%;">
                                    <div class="template-preview-element" style="width: 45%; height: 45%;"></div>
                                    <div class="template-preview-element" style="width: 45%; height: 45%;"></div>
                                    <div class="template-preview-element" style="width: 45%; height: 45%;"></div>
                                    <div class="template-preview-element" style="width: 45%; height: 45%;"></div>
                                </div>
                            </div>
                        </div>
                        <div class="template-name">Интернет-магазин</div>
                    </div>
                    <div class="template-item" data-template="blog">
                        <div class="template-preview">
                            <div class="template-preview-content">
                                <div class="template-preview-element" style="height: 15%;"></div>
                                <div style="display: flex; height: 70%;">
                                    <div class="template-preview-element" style="width: 70%;"></div>
                                    <div class="template-preview-element" style="width: 25%; margin-left: 5%;"></div>
                                </div>
                            </div>
                        </div>
                        <div class="template-name">Блог</div>
                    </div>
                    <div class="template-item" data-template="dashboard">
                        <div class="template-preview">
                            <div class="template-preview-content">
                                <div class="template-preview-element" style="height: 15%;"></div>
                                <div style="display: flex; height: 70%;">
                                    <div class="template-preview-element" style="width: 20%;"></div>
                                    <div style="width: 75%; margin-left: 5%; display: grid; grid-template-columns: repeat(2, 1fr); grid-template-rows: repeat(2, 1fr); gap: 5px;">
                                        <div class="template-preview-element"></div>
                                        <div class="template-preview-element"></div>
                                        <div class="template-preview-element"></div>
                                        <div class="template-preview-element"></div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="template-name">Панель управления</div>
                    </div>
                    <div class="template-item" data-template="mobile-app">
                        <div class="template-preview">
                            <div class="template-preview-content">
                                <div class="template-preview-element" style="height: 10%; width: 50%; margin: 0 auto; border-radius: 20px;"></div>
                                <div class="template-preview-element" style="height: 50%; border-radius: 20px;"></div>
                                <div class="template-preview-element" style="height: 10%; width: 80%; margin: 5px auto; border-radius: 10px;"></div>
                                <div class="template-preview-element" style="height: 10%; width: 60%; margin: 0 auto; border-radius: 10px; background-color: #0d66ff;"></div>
                            </div>
                        </div>
                        <div class="template-name">Мобильное приложение</div>
                    </div>
                    <div class="template-item" data-template="blank">
                        <div class="template-preview">
                            <i class="fas fa-file" style="font-size: 40px; color: #999;"></i>
                        </div>
                        <div class="template-name">Чистый холст</div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button id="cancel-template" class="button secondary">Отмена</button>
                    <button id="confirm-template" class="button">Создать</button>
                </div>
            </div>
        </div>

        <!-- Модальное окно добавления изображения -->
        <div id="image-modal" class="modal-overlay" style="display: none;">
            <div class="modal">
                <div class="modal-header">
                    <div class="modal-title">Добавить изображение</div>
                    <button class="close-button" id="close-image-modal">&times;</button>
                </div>
                <label class="file-upload" for="image-upload">
                    <div class="file-upload-icon"><i class="fas fa-cloud-upload-alt"></i></div>
                    <div>Нажмите для загрузки или перетащите файл</div>
                    <div style="font-size: 12px; color: #999; margin-top: 5px;">PNG, JPG до 5MB</div>
                </label>
                <input type="file" id="image-upload" class="file-input" accept="image/*">
                <div class="modal-footer">
                    <button id="cancel-image" class="button secondary">Отмена</button>
                    <button id="confirm-image" class="button" disabled>Добавить</button>
                </div>
            </div>
        </div>

        <!-- Модальное окно выбора фигуры -->
        <div id="shape-modal" class="modal-overlay" style="display: none;">
            <div class="modal">
                <div class="modal-header">
                    <div class="modal-title">Выберите фигуру</div>
                    <button class="close-button" id="close-shape-modal">&times;</button>
                </div>
                <div class="shape-options">
                    <div class="shape-option" data-shape="rectangle">
                        <i class="far fa-square"></i>
                    </div>
                    <div class="shape-option" data-shape="circle">
                        <i class="far fa-circle"></i>
                    </div>
                    <div class="shape-option" data-shape="triangle">
                        <i class="fas fa-play fa-rotate-270"></i>
                    </div>
                    <div class="shape-option" data-shape="diamond">
                        <i class="far fa-gem"></i>
                    </div>
                    <div class="shape-option" data-shape="star">
                        <i class="fas fa-star"></i>
                    </div>
                    <div class="shape-option" data-shape="arrow">
                        <i class="fas fa-arrow-right"></i>
                    </div>
                    <div class="shape-option" data-shape="heart">
                        <i class="fas fa-heart"></i>
                    </div>
                    <div class="shape-option" data-shape="hexagon">
                        <svg class="shape-svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                            <path d="M12 2l8 6v8l-8 6-8-6V8l8-6z"></path>
                        </svg>
                    </div>
                    <div class="shape-option" data-shape="pentagon">
                        <svg class="shape-svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                            <path d="M12 2l5.5 4.5v7L12 22l-5.5-8.5v-7L12 2z"></path>
                        </svg>
                    </div>
                </div>
                <div class="modal-footer">
                    <button id="cancel-shape" class="button secondary">Отмена</button>
                    <button id="confirm-shape" class="button">Добавить</button>
                </div>
            </div>
        </div>

        <!-- Модальное окно выбора иконки -->
        <div id="icon-modal" class="modal-overlay" style="display: none;">
            <div class="modal">
                <div class="modal-header">
                    <div class="modal-title">Выберите иконку</div>
                    <button class="close-button" id="close-icon-modal">&times;</button>
                </div>
                <div class="icon-grid">
                    <div class="icon-item" data-icon="user">
                        <i class="fas fa-user"></i>
                        <span>Пользователь</span>
                    </div>
                    <div class="icon-item" data-icon="home">
                        <i class="fas fa-home"></i>
                        <span>Дом</span>
                    </div>
                    <div class="icon-item" data-icon="search">
                        <i class="fas fa-search"></i>
                        <span>Поиск</span>
                    </div>
                    <div class="icon-item" data-icon="cart">
                        <i class="fas fa-shopping-cart"></i>
                        <span>Корзина</span>
                    </div>
                    <div class="icon-item" data-icon="settings">
                        <i class="fas fa-cog"></i>
                        <span>Настройки</span>
                    </div>
                    <div class="icon-item" data-icon="message">
                        <i class="fas fa-comment"></i>
                        <span>Сообщение</span>
                    </div>
                    <div class="icon-item" data-icon="phone">
                        <i class="fas fa-phone"></i>
                        <span>Телефон</span>
                    </div>
                    <div class="icon-item" data-icon="email">
                        <i class="fas fa-envelope"></i>
                        <span>Почта</span>
                    </div>
                    <div class="icon-item" data-icon="calendar">
                        <i class="fas fa-calendar"></i>
                        <span>Календарь</span>
                    </div>
                    <div class="icon-item" data-icon="map">
                        <i class="fas fa-map-marker-alt"></i>
                        <span>Карта</span>
                    </div>
                    <div class="icon-item" data-icon="camera">
                        <i class="fas fa-camera"></i>
                        <span>Камера</span>
                    </div>
                    <div class="icon-item" data-icon="music">
                        <i class="fas fa-music"></i>
                        <span>Музыка</span>
                    </div>
                    <div class="icon-item" data-icon="video">
                        <i class="fas fa-video"></i>
                        <span>Видео</span>
                    </div>
                    <div class="icon-item" data-icon="book">
                        <i class="fas fa-book"></i>
                        <span>Книга</span>
                    </div>
                    <div class="icon-item" data-icon="heart">
                        <i class="fas fa-heart"></i>
                        <span>Сердце</span>
                    </div>
                    <div class="icon-item" data-icon="star">
                        <i class="fas fa-star"></i>
                        <span>Звезда</span>
                    </div>
                </div>
                <div class="modal-footer">
                    <button id="cancel-icon" class="button secondary">Отмена</button>
                    <button id="confirm-icon" class="button">Добавить</button>
                </div>
            </div>
        </div>

        <!-- Модальное окно совместного доступа -->
        <div id="share-modal" class="modal-overlay" style="display: none;">
            <div class="modal">
                <div class="modal-header">
                    <div class="modal-title">Поделиться проектом</div>
                    <button class="close-button" id="close-share-modal">&times;</button>
                </div>
                <div class="property-group">
                    <label class="property-label">Ссылка на проект</label>
                    <div class="collab-link">
                        <input type="text" id="project-link-input" class="collab-input" readonly>
                        <button id="copy-project-link" class="collab-copy">Копировать</button>
                    </div>
                </div>
                <div class="property-group">
                    <label class="property-label">Участники</label>
                    <input type="text" class="property-input" placeholder="Введите email для приглашения">
                </div>
                <div class="modal-footer">
                    <button id="close-share" class="button">Готово</button>
                </div>
            </div>
        </div>

        <!-- Окно обрезки изображения -->
        <div id="crop-modal" class="crop-overlay" style="display: none;">
            <div class="crop-container">
                <img id="crop-image" class="crop-image" src="">
                <div id="crop-selection" class="crop-selection" style="display: none;">
                    <div class="crop-handle nw"></div>
                    <div class="crop-handle ne"></div>
                    <div class="crop-handle sw"></div>
                    <div class="crop-handle se"></div>
                </div>
            </div>
            <div class="crop-controls">
                <button id="cancel-crop" class="button secondary">Отмена</button>
                <button id="apply-crop" class="button">Применить</button>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // Состояние приложения
            const state = {
                currentTool: 'select',
                selectedElement: null,
                elements: [],
                nextId: 1,
                projectId: generateProjectId(),
                template: null,
                isDragging: false,
                isResizing: false,
                isRotating: false,
                dragStartX: 0,
                dragStartY: 0,
                elementStartX: 0,
                elementStartY: 0,
                elementStartWidth: 0,
                elementStartHeight: 0,
                elementStartRotation: 0,
                resizeHandle: null,
                selectedShape: null,
                selectedIcon: null,
                isEditingText: false,
                isCropping: false,
                cropSelection: {
                    x: 0,
                    y: 0,
                    width: 0,
                    height: 0,
                    startX: 0,
                    startY: 0,
                    isDragging: false,
                    isResizing: false,
                    resizeHandle: null
                },
                showGrid: false,
                snapThreshold: 5,
                guideElements: {
                    horizontal: document.getElementById('horizontal-guide'),
                    vertical: document.getElementById('vertical-guide'),
                    snapPoint: document.getElementById('snap-point'),
                    grid: document.getElementById('canvas-grid')
                }
            };

            // DOM элементы
            const canvas = document.getElementById('canvas');
            const layersList = document.getElementById('layers-list');
            const propertiesContent = document.getElementById('properties-content');
            const templateModal = document.getElementById('template-modal');
            const imageModal = document.getElementById('image-modal');
            const shapeModal = document.getElementById('shape-modal');
            const iconModal = document.getElementById('icon-modal');
            const shareModal = document.getElementById('share-modal');
            const cropModal = document.getElementById('crop-modal');
            const cropImage = document.getElementById('crop-image');
            const cropSelection = document.getElementById('crop-selection');
            const cropContainer = document.querySelector('.crop-container');
            const imageUpload = document.getElementById('image-upload');
            const projectLinkInput = document.getElementById('project-link-input');
            const deleteBtn = document.getElementById('delete-btn');
            const cropBtn = document.getElementById('crop-btn');
            const addTextBtn = document.getElementById('add-text-btn');
            const addIconBtn = document.getElementById('add-icon-btn');
            const exportPngBtn = document.getElementById('export-png-btn');
            const canvasBgColor = document.getElementById('canvas-bg-color');
            
            // Инициализация приложения
            init();

            function init() {
                setupEventListeners();
                showTemplateModal();
                setupCollaboration();
            }

            function setupEventListeners() {
                // Кнопки инструментов
                document.querySelectorAll('.tool-button').forEach(button => {
                    button.addEventListener('click', function() {
                        document.querySelectorAll('.tool-button').forEach(btn => btn.classList.remove('active'));
                        this.classList.add('active');
                        state.currentTool = this.dataset.tool;
                    });
                });

                // Модальное окно шаблонов
                document.querySelectorAll('.template-item').forEach(item => {
                    item.addEventListener('click', function() {
                        document.querySelectorAll('.template-item').forEach(i => i.style.borderColor = '#333');
                        this.style.borderColor = 'var(--primary-color)';
                        state.template = this.dataset.template;
                    });
                });

                document.getElementById('confirm-template').addEventListener('click', function() {
                    if (state.template) {
                        loadTemplate(state.template);
                        templateModal.style.display = 'none';
                    } else {
                        alert('Пожалуйста, выберите шаблон');
                    }
                });

                document.getElementById('close-template-modal').addEventListener('click', hideTemplateModal);
                document.getElementById('cancel-template').addEventListener('click', hideTemplateModal);

                // Модальное окно изображений
                document.getElementById('add-image-btn').addEventListener('click', showImageModal);
                document.getElementById('close-image-modal').addEventListener('click', hideImageModal);
                document.getElementById('cancel-image').addEventListener('click', hideImageModal);
                document.getElementById('confirm-image').addEventListener('click', addImageToCanvas);

                imageUpload.addEventListener('change', function() {
                    if (this.files.length > 0) {
                        document.getElementById('confirm-image').disabled = false;
                    }
                });

                // Модальное окно фигур
                document.getElementById('add-shape-btn').addEventListener('click', showShapeModal);
                document.getElementById('close-shape-modal').addEventListener('click', hideShapeModal);
                document.getElementById('cancel-shape').addEventListener('click', hideShapeModal);
                document.getElementById('confirm-shape').addEventListener('click', addShapeToCanvas);

                document.querySelectorAll('.shape-option').forEach(option => {
                    option.addEventListener('click', function() {
                        document.querySelectorAll('.shape-option').forEach(opt => opt.style.borderColor = '#333');
                        this.style.borderColor = 'var(--primary-color)';
                        state.selectedShape = this.dataset.shape;
                    });
                });

                // Модальное окно иконок
                document.getElementById('add-icon-btn').addEventListener('click', showIconModal);
                document.getElementById('close-icon-modal').addEventListener('click', hideIconModal);
                document.getElementById('cancel-icon').addEventListener('click', hideIconModal);
                document.getElementById('confirm-icon').addEventListener('click', addIconToCanvas);

                document.querySelectorAll('.icon-item').forEach(option => {
                    option.addEventListener('click', function() {
                        document.querySelectorAll('.icon-item').forEach(opt => opt.style.backgroundColor = '#2d2d2d');
                        this.style.backgroundColor = '#333';
                        state.selectedIcon = this.dataset.icon;
                    });
                });

                // Кнопка добавления текста
                addTextBtn.addEventListener('click', addTextElement);

                // Модальное окно совместного доступа
                document.getElementById('share-btn').addEventListener('click', showShareModal);
                document.getElementById('close-share-modal').addEventListener('click', hideShareModal);
                document.getElementById('close-share').addEventListener('click', hideShareModal);
                document.getElementById('copy-project-link').addEventListener('click', copyProjectLink);
                document.getElementById('copy-link-btn').addEventListener('click', copyCollabLink);

                // Кнопка удаления
                deleteBtn.addEventListener('click', deleteSelectedElement);

                // Кнопка обрезки изображения
                cropBtn.addEventListener('click', startImageCropping);

                // Кнопка экспорта в PNG
                exportPngBtn.addEventListener('click', exportToPNG);

                // События холста для перемещения и изменения размера
                canvas.addEventListener('mousedown', handleCanvasMouseDown);
                document.addEventListener('mousemove', handleMouseMove);
                document.addEventListener('mouseup', handleMouseUp);

                // События для редактирования текста
                canvas.addEventListener('dblclick', handleCanvasDoubleClick);

                // События для обрезки изображения
                cropImage.addEventListener('mousedown', handleCropImageMouseDown);
                cropSelection.addEventListener('mousedown', handleCropSelectionMouseDown);
                document.addEventListener('mousemove', handleCropMouseMove);
                document.addEventListener('mouseup', handleCropMouseUp);
                document.getElementById('cancel-crop').addEventListener('click', cancelCropping);
                document.getElementById('apply-crop').addEventListener('click', applyCropping);

                // Изменение цвета фона холста
                canvasBgColor.addEventListener('change', function() {
                    document.querySelector('.canvas-container').style.backgroundColor = this.value;
                });
            }

            function handleCanvasMouseDown(e) {
                if (state.isEditingText) return;
                
                // Показываем сетку при начале перемещения
                state.guideElements.grid.style.display = 'block';
                
                if (e.target === canvas) {
                    // Клик по пустому холсту - снимаем выделение
                    if (state.selectedElement) {
                        document.querySelector(`.canvas-element[data-id="${state.selectedElement.id}"]`).classList.remove('selected');
                        state.selectedElement = null;
                        updatePropertiesPanel();
                        updateLayersList();
                        deleteBtn.style.display = 'none';
                        cropBtn.style.display = 'none';
                    }
                    return;
                }

                // Проверяем, кликнули ли мы на маркер изменения размера
                if (e.target.classList.contains('resize-handle')) {
                    state.isResizing = true;
                    state.resizeHandle = e.target;
                    state.dragStartX = e.clientX;
                    state.dragStartY = e.clientY;
                    state.elementStartX = state.selectedElement.x;
                    state.elementStartY = state.selectedElement.y;
                    state.elementStartWidth = state.selectedElement.width;
                    state.elementStartHeight = state.selectedElement.height;
                    e.preventDefault();
                    return;
                }

                // Проверяем, кликнули ли мы на маркер вращения
                if (e.target.classList.contains('rotate-handle')) {
                    state.isRotating = true;
                    state.dragStartX = e.clientX;
                    state.dragStartY = e.clientY;
                    state.elementStartRotation = state.selectedElement.rotation || 0;
                    e.preventDefault();
                    return;
                }

                // Находим кликнутый элемент
                let target = e.target;
                while (target && !target.classList.contains('canvas-element')) {
                    target = target.parentElement;
                }

                if (target && target.classList.contains('canvas-element')) {
                    const id = parseInt(target.dataset.id);
                    selectElement(id);
                    
                    // Начинаем перемещение
                    if (state.currentTool === 'select') {
                        state.isDragging = true;
                        state.dragStartX = e.clientX;
                        state.dragStartY = e.clientY;
                        state.elementStartX = state.selectedElement.x;
                        state.elementStartY = state.selectedElement.y;
                        e.preventDefault(); // Предотвращаем выделение текста
                    }
                }
            }

            function handleCanvasDoubleClick(e) {
                if (state.currentTool !== 'select') return;
                
                let target = e.target;
                while (target && !target.classList.contains('canvas-element')) {
                    target = target.parentElement;
                }

                if (target && target.classList.contains('canvas-element')) {
                    const id = parseInt(target.dataset.id);
                    const element = state.elements.find(el => el.id === id);
                    
                    if (element && element.type === 'text') {
                        startTextEditing(element, target);
                    }
                }
            }

            function startTextEditing(element, elementDiv) {
                state.isEditingText = true;
                state.selectedElement = element;
                
                // Создаем textarea для редактирования
                const textarea = document.createElement('textarea');
                textarea.className = 'text-edit-input';
                textarea.value = element.content;
                textarea.style.fontSize = element.fontSize;
                textarea.style.fontWeight = element.fontWeight;
                textarea.style.color = element.color;
                textarea.style.textAlign = element.textAlign;
                textarea.style.lineHeight = element.lineHeight;
                
                // Заменяем содержимое элемента на textarea
                elementDiv.innerHTML = '';
                elementDiv.appendChild(textarea);
                textarea.focus();
                
                // Обработчик завершения редактирования
                function finishEditing() {
                    element.content = textarea.value;
                    elementDiv.innerHTML = element.content;
                    elementDiv.style.fontSize = element.fontSize;
                    elementDiv.style.fontWeight = element.fontWeight;
                    elementDiv.style.color = element.color;
                    elementDiv.style.textAlign = element.textAlign;
                    elementDiv.style.lineHeight = element.lineHeight;
                    
                    state.isEditingText = false;
                    textarea.removeEventListener('blur', finishEditing);
                    document.removeEventListener('keydown', handleTextEditKeyDown);
                }
                
                // Обработчик нажатия клавиш
                function handleTextEditKeyDown(e) {
                    if (e.key === 'Enter' && !e.shiftKey) {
                        e.preventDefault();
                        finishEditing();
                    } else if (e.key === 'Escape') {
                        finishEditing();
                    }
                }
                
                textarea.addEventListener('blur', finishEditing);
                document.addEventListener('keydown', handleTextEditKeyDown);
            }

            function handleMouseMove(e) {
                if (state.isDragging && state.selectedElement) {
                    const dx = e.clientX - state.dragStartX;
                    const dy = e.clientY - state.dragStartY;
                    
                    let newX = state.elementStartX + dx;
                    let newY = state.elementStartY + dy;
                    
                    // Проверяем привязку к другим элементам и границам
                    const snapResult = checkSnapping(newX, newY, state.selectedElement.width, state.selectedElement.height);
                    
                    if (snapResult.snapped) {
                        newX = snapResult.x;
                        newY = snapResult.y;
                        
                        // Показываем направляющие
                        if (snapResult.guide === 'horizontal') {
                            state.guideElements.horizontal.style.display = 'block';
                            state.guideElements.horizontal.style.top = `${newY}px`;
                            state.guideElements.vertical.style.display = 'none';
                        } else if (snapResult.guide === 'vertical') {
                            state.guideElements.vertical.style.display = 'block';
                            state.guideElements.vertical.style.left = `${newX}px`;
                            state.guideElements.horizontal.style.display = 'none';
                        }
                        
                        // Показываем точку привязки
                        state.guideElements.snapPoint.style.display = 'block';
                        state.guideElements.snapPoint.style.left = `${snapResult.snapX}px`;
                        state.guideElements.snapPoint.style.top = `${snapResult.snapY}px`;
                    } else {
                        // Скрываем направляющие, если нет привязки
                        state.guideElements.horizontal.style.display = 'none';
                        state.guideElements.vertical.style.display = 'none';
                        state.guideElements.snapPoint.style.display = 'none';
                    }
                    
                    // Обновляем позицию элемента
                    state.selectedElement.x = newX;
                    state.selectedElement.y = newY;
                    
                    // Обновляем отображение
                    const elementDiv = document.querySelector(`.canvas-element[data-id="${state.selectedElement.id}"]`);
                    if (elementDiv) {
                        elementDiv.style.left = `${newX}px`;
                        elementDiv.style.top = `${newY}px`;
                    }
                    
                    // Обновляем свойства
                    updatePropertiesPanel();
                } else if (state.isResizing && state.selectedElement) {
                    const dx = e.clientX - state.dragStartX;
                    const dy = e.clientY - state.dragStartY;
                    
                    let newWidth = state.elementStartWidth;
                    let newHeight = state.elementStartHeight;
                    let newX = state.elementStartX;
                    let newY = state.elementStartY;

                    // Определяем, какой маркер используется для изменения размера
                    if (state.resizeHandle.classList.contains('nw')) {
                        // Левый верхний угол
                        newWidth = Math.max(20, state.elementStartWidth - dx);
                        newHeight = Math.max(20, state.elementStartHeight - dy);
                        newX = state.elementStartX + dx;
                        newY = state.elementStartY + dy;
                    } else if (state.resizeHandle.classList.contains('ne')) {
                        // Правый верхний угол
                        newWidth = Math.max(20, state.elementStartWidth + dx);
                        newHeight = Math.max(20, state.elementStartHeight - dy);
                        newY = state.elementStartY + dy;
                    } else if (state.resizeHandle.classList.contains('sw')) {
                        // Левый нижний угол
                        newWidth = Math.max(20, state.elementStartWidth - dx);
                        newHeight = Math.max(20, state.elementStartHeight + dy);
                        newX = state.elementStartX + dx;
                    } else if (state.resizeHandle.classList.contains('se')) {
                        // Правый нижний угол
                        newWidth = Math.max(20, state.elementStartWidth + dx);
                        newHeight = Math.max(20, state.elementStartHeight + dy);
                    }

                    // Обновляем размер и позицию элемента
                    state.selectedElement.width = newWidth;
                    state.selectedElement.height = newHeight;
                    state.selectedElement.x = newX;
                    state.selectedElement.y = newY;
                    
                    // Обновляем отображение
                    const elementDiv = document.querySelector(`.canvas-element[data-id="${state.selectedElement.id}"]`);
                    if (elementDiv) {
                        elementDiv.style.width = `${newWidth}px`;
                        elementDiv.style.height = `${newHeight}px`;
                        elementDiv.style.left = `${newX}px`;
                        elementDiv.style.top = `${newY}px`;
                    }
                    
                    // Обновляем свойства
                    updatePropertiesPanel();
                } else if (state.isRotating && state.selectedElement) {
                    const elementDiv = document.querySelector(`.canvas-element[data-id="${state.selectedElement.id}"]`);
                    if (elementDiv) {
                        const rect = elementDiv.getBoundingClientRect();
                        const centerX = rect.left + rect.width / 2;
                        const centerY = rect.top + rect.height / 2;
                        
                        const angle = Math.atan2(e.clientY - centerY, e.clientX - centerX) * 180 / Math.PI + 90;
                        const rotation = Math.round(angle);
                        
                        state.selectedElement.rotation = rotation;
                        elementDiv.style.transform = `rotate(${rotation}deg)`;
                        
                        updatePropertiesPanel();
                    }
                }
            }

            function handleMouseUp() {
                state.isDragging = false;
                state.isResizing = false;
                state.isRotating = false;
                state.resizeHandle = null;
                
                // Скрываем сетку и направляющие после перемещения
                state.guideElements.grid.style.display = 'none';
                state.guideElements.horizontal.style.display = 'none';
                state.guideElements.vertical.style.display = 'none';
                state.guideElements.snapPoint.style.display = 'none';
            }

            function checkSnapping(x, y, width, height) {
                const result = {
                    snapped: false,
                    x: x,
                    y: y,
                    guide: null,
                    snapX: 0,
                    snapY: 0
                };
                
                // Проверяем привязку к границам холста
                const canvasRect = canvas.getBoundingClientRect();
                
                // Левый край
                if (Math.abs(x) < state.snapThreshold) {
                    result.snapped = true;
                    result.x = 0;
                    result.guide = 'vertical';
                    result.snapX = 0;
                    result.snapY = y + height / 2;
                }
                // Правый край
                else if (Math.abs(x + width - canvasRect.width) < state.snapThreshold) {
                    result.snapped = true;
                    result.x = canvasRect.width - width;
                    result.guide = 'vertical';
                    result.snapX = canvasRect.width;
                    result.snapY = y + height / 2;
                }
                // Верхний край
                else if (Math.abs(y) < state.snapThreshold) {
                    result.snapped = true;
                    result.y = 0;
                    result.guide = 'horizontal';
                    result.snapX = x + width / 2;
                    result.snapY = 0;
                }
                // Нижний край
                else if (Math.abs(y + height - canvasRect.height) < state.snapThreshold) {
                    result.snapped = true;
                    result.y = canvasRect.height - height;
                    result.guide = 'horizontal';
                    result.snapX = x + width / 2;
                    result.snapY = canvasRect.height;
                }
                
                // Проверяем привязку к другим элементам
                if (!result.snapped) {
                    const elements = canvas.querySelectorAll('.canvas-element');
                    elements.forEach(el => {
                        if (el.dataset.id && parseInt(el.dataset.id) !== state.selectedElement.id) {
                            const elRect = el.getBoundingClientRect();
                            const elX = elRect.left - canvasRect.left;
                            const elY = elRect.top - canvasRect.top;
                            const elWidth = elRect.width;
                            const elHeight = elRect.height;
                            
                            // Проверяем совпадение по горизонтали (левый край)
                            if (Math.abs(x - elX) < state.snapThreshold) {
                                result.snapped = true;
                                result.x = elX;
                                result.guide = 'vertical';
                                result.snapX = elX;
                                result.snapY = y + height / 2;
                            }
                            // Проверяем совпадение по горизонтали (правый край)
                            else if (Math.abs((x + width) - (elX + elWidth)) < state.snapThreshold) {
                                result.snapped = true;
                                result.x = elX + elWidth - width;
                                result.guide = 'vertical';
                                result.snapX = elX + elWidth;
                                result.snapY = y + height / 2;
                            }
                            // Проверяем совпадение по горизонтали (центр)
                            else if (Math.abs((x + width/2) - (elX + elWidth/2)) < state.snapThreshold) {
                                result.snapped = true;
                                result.x = elX + elWidth/2 - width/2;
                                result.guide = 'vertical';
                                result.snapX = elX + elWidth/2;
                                result.snapY = y + height / 2;
                            }
                            
                            // Проверяем совпадение по вертикали (верхний край)
                            if (Math.abs(y - elY) < state.snapThreshold) {
                                result.snapped = true;
                                result.y = elY;
                                result.guide = 'horizontal';
                                result.snapX = x + width / 2;
                                result.snapY = elY;
                            }
                            // Проверяем совпадение по вертикали (нижний край)
                            else if (Math.abs((y + height) - (elY + elHeight)) < state.snapThreshold) {
                                result.snapped = true;
                                result.y = elY + elHeight - height;
                                result.guide = 'horizontal';
                                result.snapX = x + width / 2;
                                result.snapY = elY + elHeight;
                            }
                            // Проверяем совпадение по вертикали (центр)
                            else if (Math.abs((y + height/2) - (elY + elHeight/2)) < state.snapThreshold) {
                                result.snapped = true;
                                result.y = elY + elHeight/2 - height/2;
                                result.guide = 'horizontal';
                                result.snapX = x + width / 2;
                                result.snapY = elY + elHeight/2;
                            }
                        }
                    });
                }
                
                return result;
            }

            function setupCollaboration() {
                // В реальном приложении здесь было бы подключение к WebSocket
                const collabLink = `${window.location.origin}${window.location.pathname}?project=${state.projectId}`;
                document.getElementById('collab-link-input').value = collabLink;
                projectLinkInput.value = collabLink;
            }

            function copyProjectLink() {
                projectLinkInput.select();
                document.execCommand('copy');
                alert('Ссылка скопирована в буфер обмена!');
            }

            function copyCollabLink() {
                const input = document.getElementById('collab-link-input');
                input.select();
                document.execCommand('copy');
                alert('Ссылка для совместной работы скопирована!');
            }

            function showTemplateModal() {
                templateModal.style.display = 'flex';
            }

            function hideTemplateModal() {
                templateModal.style.display = 'none';
                if (!state.template) {
                    state.template = 'blank';
                    loadTemplate(state.template);
                }
            }

            function showImageModal() {
                imageModal.style.display = 'flex';
            }

            function hideImageModal() {
                imageModal.style.display = 'none';
                imageUpload.value = '';
                document.getElementById('confirm-image').disabled = true;
            }

            function showShapeModal() {
                shapeModal.style.display = 'flex';
            }

            function hideShapeModal() {
                shapeModal.style.display = 'none';
                state.selectedShape = null;
                document.querySelectorAll('.shape-option').forEach(opt => opt.style.borderColor = '#333');
            }

            function showIconModal() {
                iconModal.style.display = 'flex';
            }

            function hideIconModal() {
                iconModal.style.display = 'none';
                state.selectedIcon = null;
                document.querySelectorAll('.icon-item').forEach(opt => opt.style.backgroundColor = '#2d2d2d');
            }

            function showShareModal() {
                shareModal.style.display = 'flex';
            }

            function hideShareModal() {
                shareModal.style.display = 'none';
            }

            function loadTemplate(templateName) {
                // Очищаем существующие элементы
                state.elements = [];
                canvas.innerHTML = '';
                layersList.innerHTML = '';
                state.nextId = 1;

                // Загружаем элементы шаблона
                switch (templateName) {
                    case 'landing':
                        addElement({
                            type: 'rectangle',
                            x: 0,
                            y: 0,
                            width: 800,
                            height: 80,
                            backgroundColor: '#0d66ff',
                            content: ''
                        });
                        addElement({
                            type: 'text',
                            x: 50,
                            y: 120,
                            width: 700,
                            height: 60,
                            content: 'Добро пожаловать на наш сайт',
                            fontSize: '36px',
                            fontWeight: 'bold',
                            color: '#222'
                        });
                        addElement({
                            type: 'text',
                            x: 50,
                            y: 200,
                            width: 700,
                            height: 40,
                            content: 'Это шаблон лендинга. Загрузите свои изображения и настройте его!',
                            fontSize: '18px',
                            color: '#555'
                        });
                        addElement({
                            type: 'rectangle',
                            x: 50,
                            y: 280,
                            width: 200,
                            height: 50,
                            backgroundColor: '#0d66ff',
                            content: 'Узнать больше',
                            color: 'white',
                            textAlign: 'center',
                            lineHeight: '50px'
                        });
                        break;
                    case 'portfolio':
                        addElement({
                            type: 'rectangle',
                            x: 0,
                            y: 0,
                            width: 800,
                            height: 600,
                            backgroundColor: '#f5f5f5',
                            content: ''
                        });
                        addElement({
                            type: 'text',
                            x: 50,
                            y: 50,
                            width: 700,
                            height: 60,
                            content: 'Мое портфолио',
                            fontSize: '36px',
                            fontWeight: 'bold',
                            color: '#222'
                        });
                        addElement({
                            type: 'rectangle',
                            x: 50,
                            y: 150,
                            width: 300,
                            height: 200,
                            backgroundColor: '#ddd',
                            content: 'Проект 1'
                        });
                        addElement({
                            type: 'rectangle',
                            x: 400,
                            y: 150,
                            width: 300,
                            height: 200,
                            backgroundColor: '#ddd',
                            content: 'Проект 2'
                        });
                        break;
                    case 'ecommerce':
                        addElement({
                            type: 'rectangle',
                            x: 0,
                            y: 0,
                            width: 800,
                            height: 80,
                            backgroundColor: '#333',
                            content: ''
                        });
                        addElement({
                            type: 'text',
                            x: 50,
                            y: 120,
                            width: 700,
                            height: 60,
                            content: 'Наши товары',
                            fontSize: '36px',
                            fontWeight: 'bold',
                            color: '#222'
                        });
                        for (let i = 0; i < 4; i++) {
                            addElement({
                                type: 'rectangle',
                                x: 50 + (i % 2) * 375,
                                y: 200 + Math.floor(i / 2) * 250,
                                width: 350,
                                height: 200,
                                backgroundColor: '#fff',
                                border: '1px solid #ddd',
                                content: `Товар ${i + 1}`
                            });
                        }
                        break;
                    case 'blog':
                        addElement({
                            type: 'rectangle',
                            x: 0,
                            y: 0,
                            width: 800,
                            height: 80,
                            backgroundColor: '#f5f5f5',
                            content: ''
                        });
                        addElement({
                            type: 'text',
                            x: 50,
                            y: 100,
                            width: 700,
                            height: 60,
                            content: 'Мой блог',
                            fontSize: '36px',
                            fontWeight: 'bold',
                            color: '#222'
                        });
                        addElement({
                            type: 'rectangle',
                            x: 50,
                            y: 180,
                            width: 500,
                            height: 300,
                            backgroundColor: '#fff',
                            border: '1px solid #ddd',
                            content: ''
                        });
                        addElement({
                            type: 'rectangle',
                            x: 580,
                            y: 180,
                            width: 170,
                            height: 300,
                            backgroundColor: '#f5f5f5',
                            border: '1px solid #ddd',
                            content: ''
                        });
                        break;
                    case 'dashboard':
                        addElement({
                            type: 'rectangle',
                            x: 0,
                            y: 0,
                            width: 800,
                            height: 60,
                            backgroundColor: '#333',
                            content: ''
                        });
                        addElement({
                            type: 'rectangle',
                            x: 0,
                            y: 60,
                            width: 200,
                            height: 540,
                            backgroundColor: '#f5f5f5',
                            content: ''
                        });
                        for (let i = 0; i < 4; i++) {
                            addElement({
                                type: 'rectangle',
                                x: 220 + (i % 2) * 280,
                                y: 80 + Math.floor(i / 2) * 250,
                                width: 250,
                                height: 200,
                                backgroundColor: '#fff',
                                border: '1px solid #ddd',
                                content: `Виджет ${i + 1}`
                            });
                        }
                        break;
                    case 'mobile-app':
                        addElement({
                            type: 'rectangle',
                            x: 100,
                            y: 50,
                            width: 300,
                            height: 500,
                            backgroundColor: '#f5f5f5',
                            borderRadius: '40px',
                            content: ''
                        });
                        addElement({
                            type: 'rectangle',
                            x: 130,
                            y: 80,
                            width: 240,
                            height: 40,
                            backgroundColor: '#333',
                            borderRadius: '20px',
                            content: ''
                        });
                        addElement({
                            type: 'rectangle',
                            x: 130,
                            y: 140,
                            width: 240,
                            height: 350,
                            backgroundColor: '#fff',
                            borderRadius: '20px',
                            content: ''
                        });
                        addElement({
                            type: 'rectangle',
                            x: 180,
                            y: 510,
                            width: 140,
                            height: 5,
                            backgroundColor: '#333',
                            borderRadius: '5px',
                            content: ''
                        });
                        break;
                    case 'blank':
                        // Просто пустой холст
                        break;
                    default:
                        // Шаблон по умолчанию
                        addElement({
                            type: 'text',
                            x: 50,
                            y: 50,
                            width: 700,
                            height: 60,
                            content: 'Ваш контент здесь',
                            fontSize: '24px',
                            color: '#222'
                        });
                }
            }

            function addElement(options) {
                const element = {
                    id: state.nextId++,
                    type: options.type || 'rectangle',
                    x: options.x || 0,
                    y: options.y || 0,
                    width: options.width || 100,
                    height: options.height || 100,
                    rotation: options.rotation || 0,
                    content: options.content || '',
                    backgroundColor: options.backgroundColor || (options.type === 'text' || options.type === 'shape' || options.type === 'icon' ? 'transparent' : '#ffffff'),
                    color: options.color || '#000000',
                    fontSize: options.fontSize || '16px',
                    fontWeight: options.fontWeight || 'normal',
                    textAlign: options.textAlign || 'left',
                    lineHeight: options.lineHeight || '1',
                    border: options.border || 'none',
                    borderRadius: options.borderRadius || '0',
                    imageUrl: options.imageUrl || null,
                    shape: options.shape || null,
                    icon: options.icon || null
                };

                state.elements.push(element);
                renderElement(element);
                updateLayersList();
                return element;
            }

            function renderElement(element) {
                const elementDiv = document.createElement('div');
                elementDiv.className = `canvas-element ${element.type === 'text' ? 'text-element' : ''} ${element.type === 'shape' ? 'shape-element' : ''} ${element.type === 'icon' ? 'icon-element' : ''}`;
                elementDiv.dataset.id = element.id;
                elementDiv.style.left = `${element.x}px`;
                elementDiv.style.top = `${element.y}px`;
                elementDiv.style.width = `${element.width}px`;
                elementDiv.style.height = `${element.height}px`;
                elementDiv.style.backgroundColor = element.backgroundColor;
                elementDiv.style.color = element.color;
                elementDiv.style.fontSize = element.fontSize;
                elementDiv.style.fontWeight = element.fontWeight;
                elementDiv.style.textAlign = element.textAlign;
                elementDiv.style.lineHeight = element.lineHeight;
                elementDiv.style.border = element.border;
                elementDiv.style.borderRadius = element.borderRadius;
                elementDiv.style.transform = `rotate(${element.rotation || 0}deg)`;

                if (element.type === 'text') {
                    elementDiv.textContent = element.content || 'Текст';
                } else if (element.type === 'image' && element.imageUrl) {
                    const img = document.createElement('img');
                    img.src = element.imageUrl;
                    elementDiv.appendChild(img);
                } else if (element.type === 'shape') {
                    // Создаем SVG для фигуры
                    const svg = document.createElementNS("http://www.w3.org/2000/svg", "svg");
                    svg.setAttribute("width", "100%");
                    svg.setAttribute("height", "100%");
                    svg.setAttribute("viewBox", "0 0 24 24");
                    svg.setAttribute("fill", element.color || "#000000");
                    
                    let shapeElement;
                    switch(element.shape) {
                        case 'rectangle':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "rect");
                            shapeElement.setAttribute("x", "3");
                            shapeElement.setAttribute("y", "3");
                            shapeElement.setAttribute("width", "18");
                            shapeElement.setAttribute("height", "18");
                            shapeElement.setAttribute("rx", "2");
                            shapeElement.setAttribute("ry", "2");
                            break;
                        case 'circle':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "circle");
                            shapeElement.setAttribute("cx", "12");
                            shapeElement.setAttribute("cy", "12");
                            shapeElement.setAttribute("r", "10");
                            break;
                        case 'triangle':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "path");
                            shapeElement.setAttribute("d", "M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z");
                            break;
                        case 'diamond':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "rect");
                            shapeElement.setAttribute("x", "12");
                            shapeElement.setAttribute("y", "1");
                            shapeElement.setAttribute("width", "15");
                            shapeElement.setAttribute("height", "15");
                            shapeElement.setAttribute("transform", "rotate(45 12 1)");
                            break;
                        case 'star':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "polygon");
                            shapeElement.setAttribute("points", "12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2");
                            break;
                        case 'arrow':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "g");
                            const line = document.createElementNS("http://www.w3.org/2000/svg", "line");
                            line.setAttribute("x1", "5");
                            line.setAttribute("y1", "12");
                            line.setAttribute("x2", "19");
                            line.setAttribute("y2", "12");
                            const polyline = document.createElementNS("http://www.w3.org/2000/svg", "polyline");
                            polyline.setAttribute("points", "12 5 19 12 12 19");
                            shapeElement.appendChild(line);
                            shapeElement.appendChild(polyline);
                            break;
                        case 'heart':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "path");
                            shapeElement.setAttribute("d", "M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z");
                            break;
                        case 'hexagon':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "path");
                            shapeElement.setAttribute("d", "M12 2l8 6v8l-8 6-8-6V8l8-6z");
                            break;
                        case 'pentagon':
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "path");
                            shapeElement.setAttribute("d", "M12 2l5.5 4.5v7L12 22l-5.5-8.5v-7L12 2z");
                            break;
                        default:
                            shapeElement = document.createElementNS("http://www.w3.org/2000/svg", "rect");
                            shapeElement.setAttribute("x", "3");
                            shapeElement.setAttribute("y", "3");
                            shapeElement.setAttribute("width", "18");
                            shapeElement.setAttribute("height", "18");
                            shapeElement.setAttribute("rx", "2");
                            shapeElement.setAttribute("ry", "2");
                    }
                    
                    svg.appendChild(shapeElement);
                    elementDiv.appendChild(svg);
                } else if (element.type === 'icon' && element.icon) {
                    // Создаем элемент иконки
                    const icon = document.createElement('i');
                    icon.className = `fas fa-${element.icon}`;
                    icon.style.fontSize = '24px';
                    icon.style.color = element.color || '#000000';
                    elementDiv.style.display = 'flex';
                    elementDiv.style.alignItems = 'center';
                    elementDiv.style.justifyContent = 'center';
                    elementDiv.appendChild(icon);
                }

                // Добавляем маркеры изменения размера и вращения
                if (element.type !== 'text') {
                    const nw = document.createElement('div');
                    nw.className = 'resize-handle nw';
                    elementDiv.appendChild(nw);

                    const ne = document.createElement('div');
                    ne.className = 'resize-handle ne';
                    elementDiv.appendChild(ne);

                    const sw = document.createElement('div');
                    sw.className = 'resize-handle sw';
                    elementDiv.appendChild(sw);

                    const se = document.createElement('div');
                    se.className = 'resize-handle se';
                    elementDiv.appendChild(se);

                    const rotate = document.createElement('div');
                    rotate.className = 'rotate-handle';
                    elementDiv.appendChild(rotate);
                }

                canvas.appendChild(elementDiv);
            }

            function updateLayersList() {
                layersList.innerHTML = '';
                state.elements.forEach(element => {
                    const layerItem = document.createElement('div');
                    layerItem.className = 'layer-item';
                    if (state.selectedElement && state.selectedElement.id === element.id) {
                        layerItem.classList.add('active');
                    }
                    const typeName = element.type === 'rectangle' ? 'Прямоугольник' : 
                                   element.type === 'text' ? 'Текст' : 
                                   element.type === 'image' ? 'Изображение' :
                                   element.type === 'shape' ? 'Фигура' :
                                   element.type === 'icon' ? 'Иконка' : 'Элемент';
                    layerItem.textContent = `${typeName} ${element.id}`;
                    layerItem.dataset.id = element.id;
                    
                    // Добавляем кнопку удаления
                    const deleteBtn = document.createElement('button');
                    deleteBtn.className = 'delete-layer';
                    deleteBtn.innerHTML = '×';
                    deleteBtn.addEventListener('click', function(e) {
                        e.stopPropagation();
                        deleteElement(element.id);
                    });
                    
                    layerItem.addEventListener('click', function() {
                        selectElement(element.id);
                    });
                    
                    layerItem.appendChild(deleteBtn);
                    layersList.appendChild(layerItem);
                });
            }

            function selectElement(id) {
                // Снимаем выделение со всех элементов
                document.querySelectorAll('.canvas-element').forEach(el => {
                    el.classList.remove('selected');
                });

                // Находим и выделяем элемент
                const element = state.elements.find(el => el.id === id);
                if (element) {
                    state.selectedElement = element;
                    const elementDiv = document.querySelector(`.canvas-element[data-id="${id}"]`);
                    if (elementDiv) {
                        elementDiv.classList.add('selected');
                    }
                    updatePropertiesPanel();
                    updateLayersList();
                    deleteBtn.style.display = 'block';
                    cropBtn.style.display = element.type === 'image' ? 'block' : 'none';
                }
            }

            function deleteElement(id) {
                // Удаляем элемент из массива
                state.elements = state.elements.filter(el => el.id !== id);
                
                // Удаляем элемент из DOM
                const elementDiv = document.querySelector(`.canvas-element[data-id="${id}"]`);
                if (elementDiv) {
                    elementDiv.remove();
                }
                
                // Если удалили выбранный элемент, сбрасываем выбор
                if (state.selectedElement && state.selectedElement.id === id) {
                    state.selectedElement = null;
                    deleteBtn.style.display = 'none';
                    cropBtn.style.display = 'none';
                }
                
                updatePropertiesPanel();
                updateLayersList();
            }

            function deleteSelectedElement() {
                if (state.selectedElement) {
                    deleteElement(state.selectedElement.id);
                }
            }

            function addTextElement() {
                addElement({
                    type: 'text',
                    x: 100,
                    y: 100,
                    width: 200,
                    height: 50,
                    content: 'Новый текст',
                    fontSize: '16px',
                    color: '#000000',
                    backgroundColor: 'transparent'
                });
            }

            function updatePropertiesPanel() {
                if (!state.selectedElement) {
                    propertiesContent.innerHTML = '<div class="property-group"><label class="property-label">Элемент не выбран</label></div>';
                    return;
                }

                const element = state.selectedElement;
                let html = `
                    <div class="property-group">
                        <label class="property-label">Тип</label>
                        <input type="text" class="property-input" value="${element.type === 'rectangle' ? 'Прямоугольник' : 
                                                                       element.type === 'text' ? 'Текст' : 
                                                                       element.type === 'image' ? 'Изображение' :
                                                                       element.type === 'shape' ? 'Фигура' :
                                                                       element.type === 'icon' ? 'Иконка' : 'Элемент'}" readonly>
                    </div>
                    <div class="property-group">
                        <label class="property-label">Позиция X</label>
                        <input type="number" class="property-input" value="${element.x}" data-property="x">
                    </div>
                    <div class="property-group">
                        <label class="property-label">Позиция Y</label>
                        <input type="number" class="property-input" value="${element.y}" data-property="y">
                    </div>
                    <div class="property-group">
                        <label class="property-label">Ширина</label>
                        <input type="number" class="property-input" value="${element.width}" data-property="width">
                    </div>
                    <div class="property-group">
                        <label class="property-label">Высота</label>
                        <input type="number" class="property-input" value="${element.height}" data-property="height">
                    </div>
                    <div class="property-group">
                        <label class="property-label">Вращение</label>
                        <input type="number" class="property-input" value="${element.rotation || 0}" data-property="rotation" min="0" max="360">
                    </div>
                `;

                if (element.type === 'text') {
                    html += `
                        <div class="property-group">
                            <label class="property-label">Текст</label>
                            <input type="text" class="property-input" value="${element.content}" data-property="content">
                        </div>
                        <div class="property-group">
                            <label class="property-label">Размер шрифта</label>
                            <input type="text" class="property-input" value="${element.fontSize}" data-property="fontSize">
                        </div>
                        <div class="property-group">
                            <label class="property-label">Насыщенность</label>
                            <select class="property-input" data-property="fontWeight">
                                <option value="normal" ${element.fontWeight === 'normal' ? 'selected' : ''}>Обычный</option>
                                <option value="bold" ${element.fontWeight === 'bold' ? 'selected' : ''}>Жирный</option>
                            </select>
                        </div>
                        <div class="property-group">
                            <label class="property-label">Цвет текста</label>
                            <input type="color" class="property-input" value="${element.color}" data-property="color">
                        </div>
                        <div class="property-group">
                            <label class="property-label">Выравнивание</label>
                            <select class="property-input" data-property="textAlign">
                                <option value="left" ${element.textAlign === 'left' ? 'selected' : ''}>По левому краю</option>
                                <option value="center" ${element.textAlign === 'center' ? 'selected' : ''}>По центру</option>
                                <option value="right" ${element.textAlign === 'right' ? 'selected' : ''}>По правому краю</option>
                            </select>
                        </div>
                        <div class="property-group">
                            <label class="property-label">Цвет фона</label>
                            <input type="color" class="property-input" value="transparent" data-property="backgroundColor">
                        </div>
                    `;
                } else if (element.type === 'rectangle' || element.type === 'shape') {
                    html += `
                        <div class="property-group">
                            <label class="property-label">Цвет элемента</label>
                            <input type="color" class="property-input" value="${element.color}" data-property="color">
                        </div>
                        <div class="property-group">
                            <label class="property-label">Граница</label>
                            <input type="text" class="property-input" value="${element.border}" data-property="border" placeholder="Например: 1px solid #000">
                        </div>
                        <div class="property-group">
                            <label class="property-label">Скругление углов</label>
                            <input type="text" class="property-input" value="${element.borderRadius}" data-property="borderRadius" placeholder="Например: 5px">
                        </div>
                        ${element.type === 'shape' ? '' : `
                        <div class="property-group">
                            <label class="property-label">Текст (опционально)</label>
                            <input type="text" class="property-input" value="${element.content}" data-property="content">
                        </div>
                        `}
                    `;
                } else if (element.type === 'image') {
                    html += `
                        <div class="property-group">
                            <label class="property-label">Изображение</label>
                            <button id="change-image" class="button" style="width: 100%; margin-top: 5px;">Заменить</button>
                        </div>
                        <div class="property-group">
                            <label class="property-label">Скругление углов</label>
                            <input type="text" class="property-input" value="${element.borderRadius}" data-property="borderRadius" placeholder="Например: 5px">
                        </div>
                    `;
                } else if (element.type === 'icon') {
                    html += `
                        <div class="property-group">
                            <label class="property-label">Иконка</label>
                            <button id="change-icon" class="button" style="width: 100%; margin-top: 5px;">Изменить</button>
                        </div>
                        <div class="property-group">
                            <label class="property-label">Цвет иконки</label>
                            <input type="color" class="property-input" value="${element.color}" data-property="color">
                        </div>
                        <div class="property-group">
                            <label class="property-label">Размер иконки</label>
                            <input type="text" class="property-input" value="${element.fontSize || '24px'}" data-property="fontSize">
                        </div>
                    `;
                }

                propertiesContent.innerHTML = html;

                // Добавляем обработчики событий для полей ввода
                document.querySelectorAll('.property-input').forEach(input => {
                    input.addEventListener('change', function() {
                        const property = this.dataset.property;
                        let value = this.value;

                        // Преобразуем числовые значения
                        if (['x', 'y', 'width', 'height', 'rotation'].includes(property)) {
                            value = parseInt(value);
                        }

                        // Обновляем элемент
                        state.selectedElement[property] = value;
                        
                        // Обновляем холст
                        const elementDiv = document.querySelector(`.canvas-element[data-id="${state.selectedElement.id}"]`);
                        if (elementDiv) {
                            if (['x', 'y', 'width', 'height'].includes(property)) {
                                elementDiv.style[property] = `${value}px`;
                            } else if (property === 'rotation') {
                                elementDiv.style.transform = `rotate(${value}deg)`;
                            } else if (property === 'content' && state.selectedElement.type !== 'image') {
                                elementDiv.textContent = value;
                            } else if (property === 'backgroundColor') {
                                elementDiv.style.backgroundColor = value;
                                if (state.selectedElement.type === 'text') {
                                    elementDiv.style.backgroundColor = value === '#ffffff' ? 'transparent' : value;
                                }
                            } else if (property === 'color' && (state.selectedElement.type === 'shape' || state.selectedElement.type === 'icon')) {
                                // Для фигур и иконок обновляем цвет
                                if (state.selectedElement.type === 'shape') {
                                    const svg = elementDiv.querySelector('svg');
                                    if (svg) {
                                        const shape = svg.querySelector('rect, circle, path, polygon, g');
                                        if (shape) {
                                            shape.setAttribute('fill', value);
                                        }
                                    }
                                } else if (state.selectedElement.type === 'icon') {
                                    const icon = elementDiv.querySelector('i');
                                    if (icon) {
                                        icon.style.color = value;
                                    }
                                }
                            } else if (property === 'fontSize' && state.selectedElement.type === 'icon') {
                                const icon = elementDiv.querySelector('i');
                                if (icon) {
                                    icon.style.fontSize = value;
                                }
                            } else if (property === 'borderRadius') {
                                elementDiv.style.borderRadius = value;
                            } else {
                                elementDiv.style[property] = value;
                            }
                        }
                    });
                });

                // Обработчик кнопки замены изображения
                if (element.type === 'image') {
                    document.getElementById('change-image').addEventListener('click', function() {
                        showImageModal();
                    });
                }

                // Обработчик кнопки изменения иконки
                if (element.type === 'icon') {
                    document.getElementById('change-icon').addEventListener('click', function() {
                        showIconModal();
                    });
                }
            }

            function addImageToCanvas() {
                const file = imageUpload.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = function(e) {
                        addElement({
                            type: 'image',
                            x: 100,
                            y: 100,
                            width: 200,
                            height: 200,
                            imageUrl: e.target.result
                        });
                        hideImageModal();
                    };
                    reader.readAsDataURL(file);
                }
            }

            function addShapeToCanvas() {
                if (state.selectedShape) {
                    addElement({
                        type: 'shape',
                        shape: state.selectedShape,
                        x: 100,
                        y: 100,
                        width: 100,
                        height: 100,
                        backgroundColor: 'transparent',
                        color: '#000000',
                        border: 'none'
                    });
                    hideShapeModal();
                }
            }

            function addIconToCanvas() {
                if (state.selectedIcon) {
                    addElement({
                        type: 'icon',
                        icon: state.selectedIcon,
                        x: 100,
                        y: 100,
                        width: 50,
                        height: 50,
                        backgroundColor: 'transparent',
                        color: '#000000',
                        fontSize: '24px'
                    });
                    hideIconModal();
                }
            }

            function startImageCropping() {
                if (!state.selectedElement || state.selectedElement.type !== 'image') return;
                
                state.isCropping = true;
                cropImage.src = state.selectedElement.imageUrl;
                cropModal.style.display = 'flex';
                
                // Инициализация обрезки
                initCropSelection();
            }

            function initCropSelection() {
                const img = new Image();
                img.src = cropImage.src;
                
                img.onload = function() {
                    // Рассчитываем размеры для отображения изображения
                    const containerWidth = cropContainer.clientWidth;
                    const containerHeight = cropContainer.clientHeight;
                    
                    const imgRatio = img.width / img.height;
                    const containerRatio = containerWidth / containerHeight;
                    
                    let displayWidth, displayHeight;
                    
                    if (imgRatio > containerRatio) {
                        displayWidth = containerWidth;
                        displayHeight = containerWidth / imgRatio;
                    } else {
                        displayHeight = containerHeight;
                        displayWidth = containerHeight * imgRatio;
                    }
                    
                    // Устанавливаем размеры изображения
                    cropImage.style.width = `${displayWidth}px`;
                    cropImage.style.height = `${displayHeight}px`;
                    
                    // Инициализируем область обрезки
                    const initialSize = Math.min(displayWidth, displayHeight) * 0.6;
                    const initialX = (displayWidth - initialSize) / 2;
                    const initialY = (displayHeight - initialSize) / 2;
                    
                    state.cropSelection = {
                        x: initialX,
                        y: initialY,
                        width: initialSize,
                        height: initialSize,
                        startX: 0,
                        startY: 0,
                        isDragging: false,
                        isResizing: false,
                        resizeHandle: null
                    };
                    
                    updateCropSelection();
                    cropSelection.style.display = 'block';
                };
            }

            function updateCropSelection() {
                cropSelection.style.left = `${state.cropSelection.x}px`;
                cropSelection.style.top = `${state.cropSelection.y}px`;
                cropSelection.style.width = `${state.cropSelection.width}px`;
                cropSelection.style.height = `${state.cropSelection.height}px`;
            }

            function handleCropImageMouseDown(e) {
                if (state.isCropping) {
                    const rect = cropImage.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    
                    // Проверяем, не кликнули ли мы на маркер изменения размера
                    if (e.target.classList.contains('crop-handle')) {
                        return;
                    }
                    
                    // Начинаем создание новой области обрезки
                    state.cropSelection = {
                        x: x,
                        y: y,
                        width: 0,
                        height: 0,
                        startX: x,
                        startY: y,
                        isDragging: true,
                        isResizing: false,
                        resizeHandle: null
                    };
                    
                    e.preventDefault();
                }
            }

            function handleCropSelectionMouseDown(e) {
                if (state.isCropping) {
                    // Проверяем, кликнули ли мы на маркер изменения размера
                    if (e.target.classList.contains('crop-handle')) {
                        state.cropSelection.isResizing = true;
                        state.cropSelection.resizeHandle = e.target;
                        state.cropSelection.startX = e.clientX;
                        state.cropSelection.startY = e.clientY;
                        e.preventDefault();
                        return;
                    }
                    
                    // Начинаем перемещение области обрезки
                    state.cropSelection.isDragging = true;
                    const rect = cropSelection.getBoundingClientRect();
                    state.cropSelection.startX = e.clientX - rect.left;
                    state.cropSelection.startY = e.clientY - rect.top;
                    e.preventDefault();
                }
            }

            function handleCropMouseMove(e) {
                if (state.isCropping) {
                    if (state.cropSelection.isDragging && !state.cropSelection.isResizing) {
                        // Перемещаем область обрезки
                        const rect = cropImage.getBoundingClientRect();
                        const x = e.clientX - rect.left - state.cropSelection.startX;
                        const y = e.clientY - rect.top - state.cropSelection.startY;
                        
                        // Ограничиваем перемещение в пределах изображения
                        const maxX = cropImage.clientWidth - state.cropSelection.width;
                        const maxY = cropImage.clientHeight - state.cropSelection.height;
                        
                        state.cropSelection.x = Math.max(0, Math.min(x, maxX));
                        state.cropSelection.y = Math.max(0, Math.min(y, maxY));
                        
                        updateCropSelection();
                    } else if (state.cropSelection.isResizing) {
                        // Изменяем размер области обрезки
                        const rect = cropImage.getBoundingClientRect();
                        const dx = e.clientX - state.cropSelection.startX;
                        const dy = e.clientY - state.cropSelection.startY;
                        
                        let newWidth = state.cropSelection.width;
                        let newHeight = state.cropSelection.height;
                        let newX = state.cropSelection.x;
                        let newY = state.cropSelection.y;
                        
                        // Определяем, какой маркер используется для изменения размера
                        if (state.cropSelection.resizeHandle.classList.contains('nw')) {
                            // Левый верхний угол
                            newWidth = Math.max(20, state.cropSelection.width - dx);
                            newHeight = Math.max(20, state.cropSelection.height - dy);
                            newX = state.cropSelection.x + dx;
                            newY = state.cropSelection.y + dy;
                        } else if (state.cropSelection.resizeHandle.classList.contains('ne')) {
                            // Правый верхний угол
                            newWidth = Math.max(20, state.cropSelection.width + dx);
                            newHeight = Math.max(20, state.cropSelection.height - dy);
                            newY = state.cropSelection.y + dy;
                        } else if (state.cropSelection.resizeHandle.classList.contains('sw')) {
                            // Левый нижний угол
                            newWidth = Math.max(20, state.cropSelection.width - dx);
                            newHeight = Math.max(20, state.cropSelection.height + dy);
                            newX = state.cropSelection.x + dx;
                        } else if (state.cropSelection.resizeHandle.classList.contains('se')) {
                            // Правый нижний угол
                            newWidth = Math.max(20, state.cropSelection.width + dx);
                            newHeight = Math.max(20, state.cropSelection.height + dy);
                        }
                        
                        // Ограничиваем размеры в пределах изображения
                        const maxWidth = cropImage.clientWidth - newX;
                        const maxHeight = cropImage.clientHeight - newY;
                        
                        newWidth = Math.min(newWidth, maxWidth);
                        newHeight = Math.min(newHeight, maxHeight);
                        
                        state.cropSelection.width = newWidth;
                        state.cropSelection.height = newHeight;
                        state.cropSelection.x = newX;
                        state.cropSelection.y = newY;
                        
                        updateCropSelection();
                    } else if (state.cropSelection.isDragging && state.cropSelection.width === 0) {
                        // Создаем новую область обрезки
                        const rect = cropImage.getBoundingClientRect();
                        const x = Math.min(e.clientX - rect.left, state.cropSelection.startX);
                        const y = Math.min(e.clientY - rect.top, state.cropSelection.startY);
                        const width = Math.abs(e.clientX - rect.left - state.cropSelection.startX);
                        const height = Math.abs(e.clientY - rect.top - state.cropSelection.startY);
                        
                        state.cropSelection.x = x;
                        state.cropSelection.y = y;
                        state.cropSelection.width = width;
                        state.cropSelection.height = height;
                        
                        updateCropSelection();
                    }
                }
            }

            function handleCropMouseUp() {
                if (state.isCropping) {
                    state.cropSelection.isDragging = false;
                    state.cropSelection.isResizing = false;
                    state.cropSelection.resizeHandle = null;
                }
            }

            function cancelCropping() {
                state.isCropping = false;
                cropModal.style.display = 'none';
            }

            function applyCropping() {
                if (!state.selectedElement || !state.isCropping) return;
                
                const img = new Image();
                img.src = cropImage.src;
                
                img.onload = function() {
                    // Рассчитываем соотношение между отображаемым и реальным изображением
                    const displayRatioX = img.width / cropImage.clientWidth;
                    const displayRatioY = img.height / cropImage.clientHeight;
                    
                    // Рассчитываем реальные координаты обрезки
                    const cropX = state.cropSelection.x * displayRatioX;
                    const cropY = state.cropSelection.y * displayRatioY;
                    const cropWidth = state.cropSelection.width * displayRatioX;
                    const cropHeight = state.cropSelection.height * displayRatioY;
                    
                    // Создаем canvas для обрезки
                    const canvas = document.createElement('canvas');
                    canvas.width = cropWidth;
                    canvas.height = cropHeight;
                    const ctx = canvas.getContext('2d');
                    
                    // Обрезаем изображение
                    ctx.drawImage(
                        img,
                        cropX, cropY, cropWidth, cropHeight,
                        0, 0, cropWidth, cropHeight
                    );
                    
                    // Получаем обрезанное изображение в виде Data URL
                    const croppedImageUrl = canvas.toDataURL('image/png');
                    
                    // Обновляем элемент на холсте
                    state.selectedElement.imageUrl = croppedImageUrl;
                    const elementDiv = document.querySelector(`.canvas-element[data-id="${state.selectedElement.id}"]`);
                    if (elementDiv) {
                        elementDiv.querySelector('img').src = croppedImageUrl;
                    }
                    
                    // Закрываем модальное окно
                    state.isCropping = false;
                    cropModal.style.display = 'none';
                };
            }

            function exportToPNG() {
                // Создаем canvas для экспорта
                const exportCanvas = document.createElement('canvas');
                exportCanvas.width = canvas.clientWidth;
                exportCanvas.height = canvas.clientHeight;
                const ctx = exportCanvas.getContext('2d');
                
                // Заливаем фон выбранным цветом
                const bgColor = document.querySelector('.canvas-container').style.backgroundColor || '#ffffff';
                ctx.fillStyle = bgColor;
                ctx.fillRect(0, 0, exportCanvas.width, exportCanvas.height);
                
                // Отрисовываем все элементы на canvas
                const elements = canvas.querySelectorAll('.canvas-element');
                elements.forEach(element => {
                    const style = getComputedStyle(element);
                    const left = parseInt(style.left);
                    const top = parseInt(style.top);
                    const width = parseInt(style.width);
                    const height = parseInt(style.height);
                    const rotation = parseInt(style.transform.replace(/[^0-9\-]/g, '')) || 0;
                    const borderRadius = style.borderRadius;
                    
                    // Сохраняем текущее состояние canvas
                    ctx.save();
                    
                    // Применяем вращение
                    ctx.translate(left + width/2, top + height/2);
                    ctx.rotate(rotation * Math.PI / 180);
                    ctx.translate(-(left + width/2), -(top + height/2));
                    
                    if (element.classList.contains('text-element')) {
                        // Отрисовываем текст
                        ctx.font = `${style.fontWeight} ${style.fontSize} ${style.fontFamily}`;
                        ctx.fillStyle = style.color;
                        ctx.textAlign = style.textAlign;
                        
                        const lines = element.textContent.split('\n');
                        const lineHeight = parseInt(style.fontSize) * 1.2;
                        
                        lines.forEach((line, i) => {
                            let x;
                            if (style.textAlign === 'center') {
                                x = left + width / 2;
                            } else if (style.textAlign === 'right') {
                                x = left + width;
                            } else {
                                x = left;
                            }
                            
                            ctx.fillText(line, x, top + (i * lineHeight) + parseInt(style.fontSize));
                        });
                    } else if (element.classList.contains('shape-element')) {
                        // Отрисовываем фигуру
                        ctx.fillStyle = style.backgroundColor;
                        ctx.strokeStyle = style.border.includes('none') ? 'transparent' : style.border.split(' ')[2];
                        ctx.lineWidth = style.border.includes('none') ? 0 : parseInt(style.border.split(' ')[0]);
                        
                        const elementData = state.elements.find(el => el.id === parseInt(element.dataset.id));
                        if (elementData) {
                            switch(elementData.shape) {
                                case 'rectangle':
                                    ctx.beginPath();
                                    if (borderRadius && borderRadius !== '0px') {
                                        const radius = parseInt(borderRadius);
                                        ctx.roundRect(left, top, width, height, radius);
                                    } else {
                                        ctx.rect(left, top, width, height);
                                    }
                                    ctx.fill();
                                    ctx.stroke();
                                    break;
                                case 'circle':
                                    ctx.beginPath();
                                    ctx.arc(left + width/2, top + height/2, Math.min(width, height)/2, 0, Math.PI * 2);
                                    ctx.fill();
                                    ctx.stroke();
                                    break;
                                case 'triangle':
                                    ctx.beginPath();
                                    ctx.moveTo(left + width/2, top);
                                    ctx.lineTo(left + width, top + height);
                                    ctx.lineTo(left, top + height);
                                    ctx.closePath();
                                    ctx.fill();
                                    ctx.stroke();
                                    break;
                                // Другие фигуры можно добавить по аналогии
                            }
                        }
                    } else if (element.classList.contains('icon-element')) {
                        // Для иконок просто рисуем прямоугольник с цветом фона
                        ctx.fillStyle = style.backgroundColor;
                        ctx.strokeStyle = style.border.includes('none') ? 'transparent' : style.border.split(' ')[2];
                        ctx.lineWidth = style.border.includes('none') ? 0 : parseInt(style.border.split(' ')[0]);
                        
                        if (borderRadius && borderRadius !== '0px') {
                            const radius = parseInt(borderRadius);
                            ctx.roundRect(left, top, width, height, radius);
                        } else {
                            ctx.rect(left, top, width, height);
                        }
                        ctx.fill();
                        ctx.stroke();
                    } else if (element.querySelector('img')) {
                        // Отрисовываем изображение
                        const img = element.querySelector('img');
                        if (borderRadius && borderRadius !== '0px') {
                            const radius = parseInt(borderRadius);
                            ctx.save();
                            ctx.beginPath();
                            ctx.roundRect(left, top, width, height, radius);
                            ctx.clip();
                            ctx.drawImage(img, left, top, width, height);
                            ctx.restore();
                        } else {
                            ctx.drawImage(img, left, top, width, height);
                        }
                    } else {
                        // Отрисовываем прямоугольник
                        ctx.fillStyle = style.backgroundColor;
                        ctx.strokeStyle = style.border.includes('none') ? 'transparent' : style.border.split(' ')[2];
                        ctx.lineWidth = style.border.includes('none') ? 0 : parseInt(style.border.split(' ')[0]);
                        
                        if (borderRadius && borderRadius !== '0px') {
                            const radius = parseInt(borderRadius);
                            ctx.beginPath();
                            ctx.roundRect(left, top, width, height, radius);
                            ctx.fill();
                            ctx.stroke();
                        } else {
                            ctx.beginPath();
                            ctx.rect(left, top, width, height);
                            ctx.fill();
                            ctx.stroke();
                        }
                        
                        // Если есть текст внутри прямоугольника
                        if (element.textContent) {
                            ctx.font = '16px Arial';
                            ctx.fillStyle = '#000000';
                            ctx.textAlign = 'center';
                            ctx.fillText(element.textContent, left + width/2, top + height/2);
                        }
                    }
                    
                    // Восстанавливаем состояние canvas
                    ctx.restore();
                });
                
                // Создаем ссылку для скачивания
                const link = document.createElement('a');
                link.download = 'prototype.png';
                link.href = exportCanvas.toDataURL('image/png');
                link.click();
            }

            function saveProject() {
                const projectData = {
                    elements: state.elements,
                    canvasWidth: canvas.clientWidth,
                    canvasHeight: canvas.clientHeight,
                    projectId: state.projectId,
                    canvasBgColor: document.querySelector('.canvas-container').style.backgroundColor
                };
                
                const dataStr = JSON.stringify(projectData);
                const blob = new Blob([dataStr], {type: 'application/json'});
                const url = URL.createObjectURL(blob);
                
                const link = document.createElement('a');
                link.download = 'project.design';
                link.href = url;
                link.click();
                
                URL.revokeObjectURL(url);
            }

            function loadProject(file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    try {
                        const projectData = JSON.parse(e.target.result);
                        
                        // Очищаем текущий проект
                        state.elements = [];
                        canvas.innerHTML = '';
                        layersList.innerHTML = '';
                        state.nextId = 1;
                        
                        // Загружаем данные проекта
                        state.projectId = projectData.projectId || generateProjectId();
                        canvas.style.width = `${projectData.canvasWidth || 800}px`;
                        canvas.style.height = `${projectData.canvasHeight || 600}px`;
                        
                        // Восстанавливаем цвет фона
                        if (projectData.canvasBgColor) {
                            document.querySelector('.canvas-container').style.backgroundColor = projectData.canvasBgColor;
                            canvasBgColor.value = projectData.canvasBgColor;
                        }
                        
                        // Восстанавливаем элементы
                        projectData.elements.forEach(elementData => {
                            addElement(elementData);
                        });
                        
                        setupCollaboration();
                    } catch (error) {
                        alert('Ошибка загрузки файла проекта');
                        console.error(error);
                    }
                };
                reader.readAsText(file);
            }

            function generateProjectId() {
                return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
                    const r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8);
                    return v.toString(16);
                });
            }

            // Обработчики для кнопки сохранения
            document.getElementById('save-btn').addEventListener('click', saveProject);
            
            // Обработчик для загрузки проекта (можно добавить кнопку в интерфейсе)
            document.addEventListener('dragover', function(e) {
                e.preventDefault();
            });
            
            document.addEventListener('drop', function(e) {
                e.preventDefault();
                if (e.dataTransfer.files.length > 0) {
                    const file = e.dataTransfer.files[0];
                    if (file.name.endsWith('.design')) {
                        loadProject(file);
                    }
                }
            });
        });
    </script>
</body>
</html>
