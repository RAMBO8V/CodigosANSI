<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Explorador Interactivo de Códigos ANSI</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chosen Palette: Slate Gray and Neutral Tones --><!-- Application Structure Plan: A single-page tool with two main interactive sections. The first is an "Escape Code Playground" where users can select styles and colors from controls and see a live preview and the corresponding code, making learning active instead of passive. The second is a "Dynamic ASCII Explorer" grid that shows character details on hover. This structure transforms a static reference document into a hands-on learning tool, which is more effective for the target developer audience. --><!-- Visualization & Content Choices: Report's SGR tables -> Interactive Playground -> Goal: Active learning -> User selects options, JS updates preview text and generates code snippet -> Justification: Replaces manual lookup and memorization with experimentation. Added 256-color input. Replaced manual RGB text fields with an interactive HSV Color Picker (using Canvas for visual selection), drastically improving the user experience for 24-bit color selection. Implemented dual language code generation (Java/Python) and fixed the clipboard function. Replaced the "Default" color box with an "X" icon for clearing color selections. Report's ASCII table -> Dynamic Grid -> Goal: Easy reference -> JS generates a grid, user hovers to see details (decimal, hex) -> Justification: Improves scannability and provides info-on-demand. CONFIRMING NO SVG/Mermaid. --><!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. --><style>
        body {
            font-family: 'Inter', sans-serif;
        }
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&family=JetBrains+Mono:wght[400;700]&display=swap');
        .font-mono {
            font-family: 'JetBrains Mono', monospace;
        }
        .color-box {
            transition: transform 0.1s ease-in-out;
            cursor: pointer;
        }
        .color-box:hover {
            transform: scale(1.1);
        }
        .color-box.selected {
            box-shadow: 0 0 0 3px #3b82f6; /* Ring-blue-500 */
            transform: scale(1.05);
        }
        .control-label {
            @apply block text-sm font-medium text-slate-400 mb-1;
        }
        .ascii-char:hover .ascii-tooltip {
            opacity: 1;
            visibility: visible;
        }
        /* Custom styling for inputs in dark mode */
        input[type="number"] {
            -moz-appearance: textfield;
        }
        input[type="number"]::-webkit-outer-spin-button,
        input[type="number"]::-webkit-inner-spin-button {
            -webkit-appearance: none;
            margin: 0;
        }
        /* Style for the 'X' clear button */
        .clear-color-x {
            background-color: #333; /* Darker background for contrast */
            border: 2px solid #ef4444; /* Red border */
            color: #ef4444; /* Red 'X' */
            font-weight: bold;
            font-size: 1.25rem; /* Larger 'X' */
            display: flex;
            align-items: center;
            justify-content: center;
            line-height: 1; /* Adjust line height for perfect centering */
        }
        .clear-color-x:hover {
            background-color: #444;
            transform: scale(1.1);
        }
        .clear-color-x.selected {
             box-shadow: 0 0 0 3px #3b82f6; /* Ring-blue-500 */
             transform: scale(1.05);
        }
        
        /* Color Picker Styles */
        .color-picker-container {
            display: flex;
            gap: 10px;
            cursor: pointer;
            user-select: none;
            position: relative;
        }
        .color-field {
            width: 150px;
            height: 150px;
            position: relative;
        }
        .hue-slider {
            width: 25px;
            height: 150px;
            position: relative;
        }
        .color-selector {
            position: absolute;
            width: 12px;
            height: 12px;
            border: 2px solid #fff;
            border-radius: 50%;
            box-shadow: 0 0 0 1px #000;
            pointer-events: none;
            transform: translate(-6px, -6px);
        }
        .hue-selector {
            position: absolute;
            width: 100%;
            height: 5px;
            background-color: #fff;
            border: 1px solid #000;
            pointer-events: none;
            transform: translateY(-2px);
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-200">

    <div class="container mx-auto p-4 md:p-8 max-w-7xl">
        <header class="text-center mb-10">
            <h1 class="text-4xl md:text-5xl font-bold text-white">Explorador Interactivo de Códigos ANSI</h1>
            <p class="mt-4 text-lg text-slate-400">Una herramienta para visualizar y construir secuencias de escape ANSI.</p>
        </header>

        <main>
            <!-- Sección de Playground de Códigos de Escape --><section id="playground" class="bg-slate-800 rounded-xl shadow-lg p-6 md:p-8 mb-12">
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <!-- Panel de Controles --><div>
                        <h2 class="text-2xl font-bold text-white mb-6">Playground de Códigos de Escape</h2>
                        <p class="text-slate-400 mb-6">
                            Selecciona diferentes estilos y colores para ver el resultado en tiempo real. El código correspondiente se generará automáticamente para que puedas copiarlo y usarlo en tus proyectos de terminal.
                        </p>
                        
                        <div class="space-y-6">
                            <!-- Estilos de Texto --><div>
                                <label class="control-label">Estilo de Texto</label>
                                <div id="style-controls" class="flex flex-wrap gap-2">
                                    <button data-style="1" class="px-3 py-1.5 text-sm rounded-md bg-slate-700 hover:bg-slate-600 focus:outline-none focus:ring-2 focus:ring-blue-500">Negrita (1)</button>
                                    <button data-style="3" class="px-3 py-1.5 text-sm rounded-md bg-slate-700 hover:bg-slate-600 focus:outline-none focus:ring-2 focus:ring-blue-500">Cursiva (3)</button>
                                    <button data-style="4" class="px-3 py-1.5 text-sm rounded-md bg-slate-700 hover:bg-slate-600 focus:outline-none focus:ring-2 focus:ring-blue-500">Subrayado (4)</button>
                                    <button data-style="9" class="px-3 py-1.5 text-sm rounded-md bg-slate-700 hover:bg-slate-600 focus:outline-none focus:ring-2 focus:ring-blue-500">Tachado (9)</button>
                                </div>
                            </div>
                            
                            <!-- Color de Texto --><div class="space-y-4">
                                <label class="control-label">Color de Texto (Primer Plano)</label>
                                <div class="bg-slate-700 p-3 rounded-lg">
                                    <h4 class="text-sm font-semibold mb-2 text-slate-300">Modo Básico (16 colores)</h4>
                                    <div id="fg-colors-basic" class="grid grid-cols-5 md:grid-cols-9 gap-2"></div>
                                </div>

                                <!-- Modo 256 --><div class="bg-slate-700 p-3 rounded-lg">
                                    <h4 class="text-sm font-semibold mb-2 text-slate-300">Modo 256 Colores (Índice 0-255)</h4>
                                    <div class="flex space-x-2">
                                        <input type="number" id="fg-256-input" min="0" max="255" placeholder="0-255" class="w-full px-3 py-1.5 text-sm rounded-md bg-slate-900 border border-slate-700 focus:ring-blue-500 focus:border-blue-500 text-white">
                                        <button id="fg-256-apply" data-color-type="fg" class="px-3 py-1.5 text-sm rounded-md bg-blue-600 hover:bg-blue-700">Aplicar</button>
                                    </div>
                                </div>

                                <!-- Nuevo Modo RGB (Color Picker) --><div class="bg-slate-700 p-3 rounded-lg">
                                    <h4 class="text-sm font-semibold mb-2 text-slate-300">Modo True Color (RGB 24-bit)</h4>
                                    
                                    <div class="color-picker-container mb-2">
                                        <div class="color-field">
                                            <canvas id="fg-color-canvas" class="rounded-lg"></canvas>
                                            <div id="fg-color-selector" class="color-selector hidden"></div>
                                        </div>
                                        <div class="hue-slider">
                                            <canvas id="fg-hue-canvas" class="rounded-lg"></canvas>
                                            <div id="fg-hue-selector" class="hue-selector hidden"></div>
                                        </div>
                                        <div class="flex flex-col justify-between ml-4">
                                            <div id="fg-rgb-preview" class="h-10 w-10 rounded-lg border-2 border-slate-600 mb-2"></div>
                                            <div class="text-xs">
                                                <div class="flex justify-between">R: <input type="number" id="fg-rgb-r" min="0" max="255" class="w-10 bg-slate-900 rounded px-1 text-center" readonly></div>
                                                <div class="flex justify-between">G: <input type="number" id="fg-rgb-g" min="0" max="255" class="w-10 bg-slate-900 rounded px-1 text-center" readonly></div>
                                                <div class="flex justify-between">B: <input type="number" id="fg-rgb-b" min="0" max="255" class="w-10 bg-slate-900 rounded px-1 text-center" readonly></div>
                                            </div>
                                        </div>
                                    </div>
                                    <button id="fg-rgb-apply" data-color-type="fg" class="w-full px-3 py-1.5 text-sm rounded-md bg-blue-600 hover:bg-blue-700">Aplicar Color Seleccionado</button>
                                </div>
                            </div>
                            
                            <!-- Color de Fondo --><div class="space-y-4">
                                <label class="control-label">Color de Fondo</label>
                                <div class="bg-slate-700 p-3 rounded-lg">
                                    <h4 class="text-sm font-semibold mb-2 text-slate-300">Modo Básico (16 colores)</h4>
                                    <div id="bg-colors-basic" class="grid grid-cols-5 md:grid-cols-9 gap-2"></div>
                                </div>

                                <!-- Modo 256 --><div class="bg-slate-700 p-3 rounded-lg">
                                    <h4 class="text-sm font-semibold mb-2 text-slate-300">Modo 256 Colores (Índice 0-255)</h4>
                                    <div class="flex space-x-2">
                                        <input type="number" id="bg-256-input" min="0" max="255" placeholder="0-255" class="w-full px-3 py-1.5 text-sm rounded-md bg-slate-900 border border-slate-700 focus:ring-blue-500 focus:border-blue-500 text-white">
                                        <button id="bg-256-apply" data-color-type="bg" class="px-3 py-1.5 text-sm rounded-md bg-blue-600 hover:bg-blue-700">Aplicar</button>
                                    </div>
                                </div>

                                <!-- Nuevo Modo RGB (Color Picker) --><div class="bg-slate-700 p-3 rounded-lg">
                                    <h4 class="text-sm font-semibold mb-2 text-slate-300">Modo True Color (RGB 24-bit)</h4>
                                    
                                    <div class="color-picker-container mb-2">
                                        <div class="color-field">
                                            <canvas id="bg-color-canvas" class="rounded-lg"></canvas>
                                            <div id="bg-color-selector" class="color-selector hidden"></div>
                                        </div>
                                        <div class="hue-slider">
                                            <canvas id="bg-hue-canvas" class="rounded-lg"></canvas>
                                            <div id="bg-hue-selector" class="hue-selector hidden"></div>
                                        </div>
                                        <div class="flex flex-col justify-between ml-4">
                                            <div id="bg-rgb-preview" class="h-10 w-10 rounded-lg border-2 border-slate-600 mb-2"></div>
                                            <div class="text-xs">
                                                <div class="flex justify-between">R: <input type="number" id="bg-rgb-r" min="0" max="255" class="w-10 bg-slate-900 rounded px-1 text-center" readonly></div>
                                                <div class="flex justify-between">G: <input type="number" id="bg-rgb-g" min="0" max="255" class="w-10 bg-slate-900 rounded px-1 text-center" readonly></div>
                                                <div class="flex justify-between">B: <input type="number" id="bg-rgb-b" min="0" max="255" class="w-10 bg-slate-900 rounded px-1 text-center" readonly></div>
                                            </div>
                                        </div>
                                    </div>
                                    <button id="bg-rgb-apply" data-color-type="bg" class="w-full px-3 py-1.5 text-sm rounded-md bg-blue-600 hover:bg-blue-700">Aplicar Color Seleccionado</button>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Panel de Visualización --><div class="flex flex-col">
                        <div>
                            <h3 class="text-lg font-semibold text-white mb-2">Vista Previa</h3>
                            <div class="bg-black rounded-lg p-6 min-h-[120px] flex items-center justify-center">
                                <pre><code id="preview-text" class="font-mono text-xl">Hola, Mundo ANSI!</code></pre>
                            </div>
                        </div>
                        <div class="mt-6 flex-grow flex flex-col">
                           <h3 class="text-lg font-semibold text-white mb-2">Código Generado</h3>
                            
                            <!-- Selector de Lenguaje --><div id="language-selector" class="flex mb-3 space-x-2 p-1 bg-slate-700 rounded-lg">
                                <button data-lang="java" class="w-1/2 text-sm px-3 py-1 rounded-md bg-blue-600">Java</button>
                                <button data-lang="python" class="w-1/2 text-sm px-3 py-1 rounded-md bg-slate-700 hover:bg-slate-600">Python</button>
                            </div>
                            
                            <div class="bg-black rounded-lg p-4 flex-grow relative">
                                <pre><code id="generated-code-java" class="font-mono text-sm whitespace-pre-wrap"></code></pre>
                                <pre><code id="generated-code-python" class="font-mono text-sm whitespace-pre-wrap hidden"></code></pre>
                                <button id="copy-button" class="absolute top-3 right-3 bg-slate-700 hover:bg-slate-600 text-slate-300 text-xs px-2 py-1 rounded-md">Copiar</button>
                            </div>
                        </div>
                    </div>
                </div>
            </section>

            <!-- Sección Explorador ASCII --><section id="ascii-explorer" class="bg-slate-800 rounded-xl shadow-lg p-6 md:p-8">
                <h2 class="text-2xl font-bold text-white mb-4 text-center">Explorador de Caracteres ASCII</h2>
                <p class="text-slate-400 mb-8 text-center max-w-3xl mx-auto">
                    Pasa el cursor sobre cualquier carácter para ver sus códigos decimal, hexadecimal y su descripción. Los primeros 32 caracteres son de control y no son visibles.
                </p>
                <div id="ascii-grid" class="grid grid-cols-8 sm:grid-cols-12 md:grid-cols-16 gap-1 font-mono text-center">
                    <!-- El grid se llenará dinámicamente con JavaScript --></div>
            </section>

        </main>
        
        <footer class="text-center mt-12 text-slate-500 text-sm">
            <p>Creado para demostrar las capacidades de las secuencias de escape ANSI de forma interactiva.</p>
        </footer>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // --- Estado de la aplicación ---
            const state = {
                styles: new Set(),
                fg: { mode: 'basic', params: null, picker: { h: 0, s: 1, v: 1, r: 255, g: 0, b: 0 } },
                bg: { mode: 'basic', params: null, picker: { h: 0, s: 1, v: 1, r: 255, g: 0, b: 0 } },
                language: 'java'
            };

            // --- Datos ---
            const colors = [
                { name: 'Negro', code: 30, bgCode: 40, hex: '#000000' }, { name: 'Rojo', code: 31, bgCode: 41, hex: '#AA0000' },
                { name: 'Verde', code: 32, bgCode: 42, hex: '#00AA00' }, { name: 'Amarillo', code: 33, bgCode: 43, hex: '#AA5500' },
                { name: 'Azul', code: 34, bgCode: 44, hex: '#0000AA' }, { name: 'Magenta', code: 35, bgCode: 45, hex: '#AA00AA' },
                { name: 'Cian', code: 36, bgCode: 46, hex: '#00AAAA' }, { name: 'Blanco', code: 37, bgCode: 47, hex: '#AAAAAA' }
            ];
            const brightColors = [
                { name: 'Gris Oscuro', code: 90, bgCode: 100, hex: '#555555' }, { name: 'Rojo Brillante', code: 91, bgCode: 101, hex: '#FF5555' },
                { name: 'Verde Brillante', code: 92, bgCode: 102, hex: '#55FF55' }, { name: 'Amarillo Brillante', code: 93, bgCode: 103, hex: '#FFFF55' },
                { name: 'Azul Brillante', code: 94, bgCode: 104, hex: '#5555FF' }, { name: 'Magenta Brillante', code: 95, bgCode: 105, hex: '#FF55FF' },
                { name: 'Cian Brillante', code: 96, bgCode: 106, hex: '#55FFFF' }, { name: 'Blanco Brillante', code: 97, bgCode: 107, hex: '#FFFFFF' }
            ];
            const allColors = [...colors, ...brightColors];

            // --- Elementos del DOM ---
            const styleControls = document.getElementById('style-controls');
            const fgColorContainer = document.getElementById('fg-colors-basic');
            const bgColorContainer = document.getElementById('bg-colors-basic');
            const previewText = document.getElementById('preview-text');
            const copyButton = document.getElementById('copy-button');
            const asciiGrid = document.getElementById('ascii-grid');
            const languageSelector = document.getElementById('language-selector');

            // Selectores de códigos
            const generatedCodeJava = document.getElementById('generated-code-java');
            const generatedCodePython = document.getElementById('generated-code-python');
            
            // RGB/256 Elements
            const fg256Input = document.getElementById('fg-256-input');
            const fg256Apply = document.getElementById('fg-256-apply');
            const bg256Input = document.getElementById('bg-256-input');
            const bg256Apply = document.getElementById('bg-256-apply');

            // --- Elementos del Selector de Color FG ---
            const fgColorCanvas = document.getElementById('fg-color-canvas');
            const fgHueCanvas = document.getElementById('fg-hue-canvas');
            const fgColorSelector = document.getElementById('fg-color-selector');
            const fgHueSelector = document.getElementById('fg-hue-selector');
            const fgRgbR = document.getElementById('fg-rgb-r');
            const fgRgbG = document.getElementById('fg-rgb-g');
            const fgRgbB = document.getElementById('fg-rgb-b');
            const fgRgbPreview = document.getElementById('fg-rgb-preview');
            const fgRgbApply = document.getElementById('fg-rgb-apply');

            // --- Elementos del Selector de Color BG ---
            const bgColorCanvas = document.getElementById('bg-color-canvas');
            const bgHueCanvas = document.getElementById('bg-hue-canvas');
            const bgColorSelector = document.getElementById('bg-color-selector');
            const bgHueSelector = document.getElementById('bg-hue-selector');
            const bgRgbR = document.getElementById('bg-rgb-r');
            const bgRgbG = document.getElementById('bg-rgb-g');
            const bgRgbB = document.getElementById('bg-rgb-b');
            const bgRgbPreview = document.getElementById('bg-rgb-preview');
            const bgRgbApply = document.getElementById('bg-rgb-apply');


            // --- Lógica del Selector de Color (HSV <-> RGB) ---

            /**
             * Convierte HSV (0-360, 0-1, 0-1) a RGB (0-255)
             */
            function hsvToRgb(h, s, v) {
                let r, g, b;
                let i = Math.floor(h * 6);
                let f = h * 6 - i;
                let p = v * (1 - s);
                let q = v * (1 - f * s);
                let t = v * (1 - (1 - f) * s);

                switch (i % 6) {
                    case 0: r = v; g = t; b = p; break;
                    case 1: r = q; g = v; b = p; break;
                    case 2: r = p; g = v; b = t; break;
                    case 3: r = p; g = q; b = v; break;
                    case 4: r = t; g = p; b = v; break;
                    case 5: r = v; g = p; b = q; break;
                }

                return {
                    r: Math.round(r * 255),
                    g: Math.round(g * 255),
                    b: Math.round(b * 255)
                };
            }

            /**
             * Dibuja el gradiente de tono (Hue) en el canvas lateral
             */
            function drawHue(canvas) {
                const ctx = canvas.getContext('2d');
                const width = canvas.width;
                const height = canvas.height;
                const gradient = ctx.createLinearGradient(0, 0, 0, height);

                // Ciclo de color RGB/HSV
                gradient.addColorStop(0,    '#FF0000'); // Rojo
                gradient.addColorStop(0.166, '#FFFF00'); // Amarillo
                gradient.addColorStop(0.333, '#00FF00'); // Verde
                gradient.addColorStop(0.5,   '#00FFFF'); // Cian
                gradient.addColorStop(0.666, '#0000FF'); // Azul
                gradient.addColorStop(0.833, '#FF00FF'); // Magenta
                gradient.addColorStop(1,    '#FF0000'); // Rojo

                ctx.fillStyle = gradient;
                ctx.fillRect(0, 0, width, height);
            }
            
            /**
             * Dibuja el campo de color (Saturación y Valor) para un tono dado
             */
            function drawColorField(canvas, h) {
                const ctx = canvas.getContext('2d');
                const width = canvas.width;
                const height = canvas.height;

                // 1. Dibuja el color base (Hue)
                const { r, g, b } = hsvToRgb(h / 360, 1, 1);
                ctx.fillStyle = `rgb(${r}, ${g}, ${b})`;
                ctx.fillRect(0, 0, width, height);

                // 2. Dibuja el gradiente de Saturación (Blanco a Tono)
                const saturationGradient = ctx.createLinearGradient(0, 0, width, 0);
                saturationGradient.addColorStop(0, 'rgba(255, 255, 255, 1)'); // Blanco
                saturationGradient.addColorStop(1, 'rgba(255, 255, 255, 0)'); // Transparente
                ctx.fillStyle = saturationGradient;
                ctx.fillRect(0, 0, width, height);
                
                // 3. Dibuja el gradiente de Valor/Brillo (Negro)
                const valueGradient = ctx.createLinearGradient(0, height, 0, 0);
                valueGradient.addColorStop(0, 'rgba(0, 0, 0, 1)'); // Negro
                valueGradient.addColorStop(1, 'rgba(0, 0, 0, 0)'); // Transparente
                ctx.fillStyle = valueGradient;
                ctx.fillRect(0, 0, width, height);
            }

            /**
             * Obtiene el color RGB de un punto en el canvas de color
             */
            function getColorFromCanvas(canvas, x, y) {
                const ctx = canvas.getContext('2d');
                const imageData = ctx.getImageData(x, y, 1, 1).data;
                return { r: imageData[0], g: imageData[1], b: imageData[2] };
            }

            /**
             * Procesa la interacción en el selector de color
             */
            function handleColorSelection(e, type) {
                const canvas = type === 'fg' ? fgColorCanvas : bgHueCanvas;
                const rect = canvas.getBoundingClientRect();
                let x = e.clientX - rect.left;
                let y = e.clientY - rect.top;

                // Clamp values
                x = Math.max(0, Math.min(x, canvas.width));
                y = Math.max(0, Math.min(y, canvas.height));

                const currentSelection = type === 'fg' ? state.fg.picker : state.bg.picker;
                
                // 1. Si el evento es en el campo de color (S y V)
                if (e.target.id.includes('color-canvas')) {
                    const selector = type === 'fg' ? fgColorSelector : bgColorSelector;
                    const rgb = getColorFromCanvas(canvas, x, y);

                    // Actualiza el estado con S y V (manteniendo H)
                    currentSelection.s = 1 - (x / canvas.width);
                    currentSelection.v = 1 - (y / canvas.height);
                    currentSelection.r = rgb.r;
                    currentSelection.g = rgb.g;
                    currentSelection.b = rgb.b;
                    
                    // Mueve el selector de S/V
                    selector.style.left = `${x}px`;
                    selector.style.top = `${y}px`;
                    selector.classList.remove('hidden');

                // 2. Si el evento es en el slider de tono (H)
                } else if (e.target.id.includes('hue-canvas')) {
                    const hueCanvas = type === 'fg' ? fgHueCanvas : bgHueCanvas;
                    const colorCanvas = type === 'fg' ? fgColorCanvas : bgHueCanvas;
                    const hueSelector = type === 'fg' ? fgHueSelector : bgHueSelector;

                    // Calcula H (360 * posición vertical)
                    currentSelection.h = (y / hueCanvas.height) * 360;
                    
                    // Dibuja el nuevo campo de color basado en el nuevo H
                    drawColorField(colorCanvas, currentSelection.h);

                    // Mueve el selector de H
                    hueSelector.style.top = `${y}px`;
                    hueSelector.classList.remove('hidden');

                    // Recalcula RGB y actualiza el selector de S/V (si estaba visible)
                    const colorX = parseFloat(selector.style.left) || 0;
                    const colorY = parseFloat(selector.style.top) || 0;
                    if (!selector.classList.contains('hidden')) {
                        const rgb = getColorFromCanvas(colorCanvas, colorX, colorY);
                        currentSelection.r = rgb.r;
                        currentSelection.g = rgb.g;
                        currentSelection.b = rgb.b;
                    } else {
                        // Si no hay selector S/V activo, forzar a la esquina superior derecha (S=1, V=1)
                        const rgb = hsvToRgb(currentSelection.h / 360, 1, 1);
                        currentSelection.r = rgb.r;
                        currentSelection.g = rgb.g;
                        currentSelection.b = rgb.b;
                    }
                }
                
                // Actualiza los campos de texto y la vista previa del color seleccionado
                updateRgbInputs(type, currentSelection.r, currentSelection.g, currentSelection.b);
            }

            /**
             * Dibuja los selectores de color RGB iniciales
             */
            function initializeColorPickers(type) {
                const colorCanvas = type === 'fg' ? fgColorCanvas : bgHueCanvas;
                const hueCanvas = type === 'fg' ? fgHueCanvas : bgHueCanvas;
                
                // Establecer dimensiones
                colorCanvas.width = 150; colorCanvas.height = 150;
                hueCanvas.width = 25; hueCanvas.height = 150;

                drawHue(hueCanvas);
                drawColorField(colorCanvas, state[type].picker.h);
                
                // Configurar eventos de click/drag
                let isDragging = false;
                
                function startDrag(e) {
                    isDragging = true;
                    e.preventDefault();
                    handleColorSelection(e, type);
                }

                function doDrag(e) {
                    if (isDragging) {
                        handleColorSelection(e, type);
                    }
                }

                function stopDrag() {
                    isDragging = false;
                }

                colorCanvas.addEventListener('mousedown', startDrag);
                hueCanvas.addEventListener('mousedown', startDrag);
                document.addEventListener('mousemove', doDrag);
                document.addEventListener('mouseup', stopDrag);
                
                // Inicializar la vista previa y los inputs
                updateRgbInputs(type, state[type].picker.r, state[type].picker.g, state[type].picker.b);
            }
            
            /**
             * Actualiza los inputs de RGB y la pequeña preview box.
             */
            function updateRgbInputs(type, r, g, b) {
                const rEl = type === 'fg' ? fgRgbR : bgRgbR;
                const gEl = type === 'fg' ? fgRgbG : bgRgbG;
                const bEl = type === 'fg' ? fgRgbB : bgRgbB;
                const previewEl = type === 'fg' ? fgRgbPreview : bgRgbPreview;

                rEl.value = r;
                gEl.value = g;
                bEl.value = b;
                previewEl.style.backgroundColor = `rgb(${r}, ${g}, ${b})`;
            }

            // --- Funciones de vista previa y generación de código (mantenidas) ---

            function updatePreview() {
                const styleParams = [...state.styles];
                const fgParam = state.fg.params ? state.fg.params : null;
                const bgParam = state.bg.params ? state.bg.params : null;
                
                const allParams = [...styleParams];
                if (fgParam && fgParam !== '39') allParams.push(fgParam);
                if (bgParam && bgParam !== '49') allParams.push(bgParam);

                const escapeCode = allParams.length > 0 ? `\u001b[${allParams.join(';')}m` : '';
                const resetCode = (allParams.length > 0) ? `\u001b[0m` : '';
                
                previewText.textContent = `${escapeCode}Hola, Mundo ANSI!${resetCode}`;
                
                // Generación de código Java
                if (allParams.length > 0) {
                    const javaParams = allParams.join(';');
                    generatedCodeJava.textContent = 
                        `// Java utiliza \\u001b (el carácter de escape Unicode)\nfinal String ANSI_STYLE = "\\u001b[${javaParams}m";\nfinal String RESET = "\\u001b[0m";\nSystem.out.println(ANSI_STYLE + "Hola, Mundo ANSI!" + RESET);`;
                } else {
                    generatedCodeJava.textContent = '// Sin parámetros ANSI seleccionados\nSystem.out.println("Hola, Mundo ANSI!");';
                }

                // Generación de código Python
                if (allParams.length > 0) {
                    const pythonParams = allParams.join(';');
                    generatedCodePython.textContent = 
                        `# Python utiliza \\033 o \\x1b (el carácter de escape octal/hexadecimal)\nANSI_STYLE = '\\033[${pythonParams}m'\nRESET = '\\033[0m'\nprint(f"{ANSI_STYLE}Hola, Mundo ANSI!{RESET}")`;
                } else {
                    generatedCodePython.textContent = '# Sin parámetros ANSI seleccionados\nprint("Hola, Mundo ANSI!")';
                }
                
                updateLanguageView();
            }
            
            function updateLanguageView() {
                if (state.language === 'java') {
                    generatedCodeJava.classList.remove('hidden');
                    generatedCodePython.classList.add('hidden');
                    languageSelector.querySelector('[data-lang="java"]').classList.add('bg-blue-600', 'hover:bg-blue-700');
                    languageSelector.querySelector('[data-lang="java"]').classList.remove('bg-slate-700', 'hover:bg-slate-600');
                    languageSelector.querySelector('[data-lang="python"]').classList.remove('bg-blue-600', 'hover:bg-blue-700');
                    languageSelector.querySelector('[data-lang="python"]').classList.add('bg-slate-700', 'hover:bg-slate-600');
                } else {
                    generatedCodePython.classList.remove('hidden');
                    generatedCodeJava.classList.add('hidden');
                    languageSelector.querySelector('[data-lang="python"]').classList.add('bg-blue-600', 'hover:bg-blue-700');
                    languageSelector.querySelector('[data-lang="python"]').classList.remove('bg-slate-700', 'hover:bg-slate-600');
                    languageSelector.querySelector('[data-lang="java"]').classList.remove('bg-blue-600', 'hover:bg-blue-700');
                    languageSelector.querySelector('[data-lang="java"]').classList.add('bg-slate-700', 'hover:bg-slate-600');
                }
            }

            function clearBasicSelection(type) {
                const container = type === 'fg' ? fgColorContainer : bgColorContainer;
                const currentSelected = container.querySelector('.selected');
                if (currentSelected) {
                    currentSelected.classList.remove('selected');
                }
                const clearXButton = container.querySelector('.clear-color-x');
                if (clearXButton) {
                    clearXButton.classList.remove('selected');
                }
            }

            // --- Inicialización de Controles ---
            
            function setupStyleControls() {
                styleControls.addEventListener('click', (e) => {
                    if (e.target.tagName === 'BUTTON') {
                        const styleCode = e.target.dataset.style;
                        e.target.classList.toggle('bg-blue-600');
                        e.target.classList.toggle('bg-slate-700');
                        
                        if (state.styles.has(styleCode)) {
                            state.styles.delete(styleCode);
                        } else {
                            state.styles.add(styleCode);
                        }
                        updatePreview();
                    }
                });
            }

            function createColorPalette(container, type) {
                const colorMap = type === 'fg' ? allColors : allColors.map(c => ({...c, code: c.bgCode, hex: c.hex}));
                
                colorMap.forEach(color => {
                    const code = color.code;
                    const box = document.createElement('div');
                    
                    box.className = `color-box w-full h-8 rounded-md border-2 border-transparent`;
                    box.style.backgroundColor = color.hex;
                    box.dataset.code = code;
                    box.title = color.name;
                    
                    box.addEventListener('click', () => {
                        clearBasicSelection(type);
                        
                        // Resetea 256/RGB inputs/selectors al usar básico
                        if (type === 'fg') {
                            fg256Input.value = '';
                            fgColorSelector.classList.add('hidden'); fgHueSelector.classList.add('hidden');
                        } else {
                            bg256Input.value = '';
                            bgColorSelector.classList.add('hidden'); bgHueSelector.classList.add('hidden');
                        }

                        const isCurrentlySelected = box.classList.contains('selected');
                        
                        if (isCurrentlySelected) {
                            box.classList.remove('selected');
                            if (type === 'fg') state.fg.params = '39';
                            if (type === 'bg') state.bg.params = '49';
                        } else {
                            box.classList.add('selected');
                            if (type === 'fg') state.fg = { ...state.fg, mode: 'basic', params: code.toString() };
                            if (type === 'bg') state.bg = { ...state.bg, mode: 'basic', params: code.toString() };
                        }

                        updatePreview();
                    });
                    container.appendChild(box);
                });

                // Add the 'X' button for clearing colors
                const clearXButton = document.createElement('div');
                clearXButton.className = `color-box w-full h-8 rounded-md clear-color-x`;
                clearXButton.textContent = 'X';
                clearXButton.title = 'Quitar Color';
                clearXButton.dataset.code = type === 'fg' ? '39' : '49';
                
                clearXButton.addEventListener('click', () => {
                    clearBasicSelection(type); 
                    clearXButton.classList.add('selected'); 
                    
                    // Resetea 256/RGB inputs/selectors al limpiar
                    if (type === 'fg') {
                        state.fg = { ...state.fg, mode: 'basic', params: '39' };
                        fg256Input.value = '';
                        fgColorSelector.classList.add('hidden'); fgHueSelector.classList.add('hidden');
                    } else {
                        state.bg = { ...state.bg, mode: 'basic', params: '49' };
                        bg256Input.value = '';
                        bgColorSelector.classList.add('hidden'); bgHueSelector.classList.add('hidden');
                    }

                    updatePreview();
                });
                container.appendChild(clearXButton);
            }

            // Lógica para 256
            function handleAdvancedColor(type, mode, value) {
                clearBasicSelection(type);
                
                // Clear RGB selectors and inputs
                const colorSelector = type === 'fg' ? fgColorSelector : bgColorSelector;
                const hueSelector = type === 'fg' ? fgHueSelector : bgHueSelector;

                if (type === 'fg') {
                    colorSelector.classList.add('hidden'); hueSelector.classList.add('hidden');
                } else {
                    colorSelector.classList.add('hidden'); hueSelector.classList.add('hidden');
                }
                
                // Update state
                const base = type === 'fg' ? 38 : 48;
                const params = `${base};5;${value}`;

                if (type === 'fg') {
                    state.fg = { ...state.fg, mode, params };
                } else {
                    state.bg = { ...state.bg, mode, params };
                }

                updatePreview();
            }

            // Handlers de aplicación (256 y RGB)
            fg256Apply.addEventListener('click', () => {
                const index = parseInt(fg256Input.value);
                if (index >= 0 && index <= 255) {
                    handleAdvancedColor('fg', '256', index);
                }
            });

            bg256Apply.addEventListener('click', () => {
                const index = parseInt(bg256Input.value);
                if (index >= 0 && index <= 255) {
                    handleAdvancedColor('bg', '256', index);
                }
            });

            fgRgbApply.addEventListener('click', () => {
                const {r, g, b} = state.fg.picker;
                const base = 38;
                const params = `${base};2;${r};${g};${b}`;
                handleAdvancedColor('fg', 'rgb', {r, g, b}); // Llama a la función, aunque reusamos la lógica de generación
                state.fg.params = params;
                updatePreview();
            });

            bgRgbApply.addEventListener('click', () => {
                const {r, g, b} = state.bg.picker;
                const base = 48;
                const params = `${base};2;${r};${g};${b}`;
                handleAdvancedColor('bg', 'rgb', {r, g, b}); // Llama a la función, aunque reusamos la lógica de generación
                state.bg.params = params;
                updatePreview();
            });
            
            function setupLanguageSelector() {
                languageSelector.addEventListener('click', (e) => {
                    if (e.target.tagName === 'BUTTON') {
                        const newLang = e.target.dataset.lang;
                        if (newLang !== state.language) {
                            state.language = newLang;
                            updateLanguageView();
                        }
                    }
                });
            }

            function setupCopyButton() {
                copyButton.addEventListener('click', async () => {
                    let codeToCopy = '';
                    if (state.language === 'java') {
                        codeToCopy = generatedCodeJava.textContent;
                    } else {
                        codeToCopy = generatedCodePython.textContent;
                    }
                    
                    try {
                        await navigator.clipboard.writeText(codeToCopy);
                        copyButton.textContent = '¡Copiado!';
                    } catch (err) {
                        copyButton.textContent = 'Error al Copiar';
                        console.error('Error al copiar el texto: ', err);
                    }
                    
                    setTimeout(() => {
                        copyButton.textContent = 'Copiar';
                    }, 2000);
                });
            }

            function setupAsciiExplorer() {
                 const controlChars = {0: "NUL", 1: "SOH", 2: "STX", 3: "ETX", 4: "EOT", 5: "ENQ", 6: "ACK", 7: "BEL", 8: "BS", 9: "HT", 10: "LF", 11: "VT", 12: "FF", 13: "CR", 14: "SO", 15: "SI", 16: "DLE", 17: "DC1", 18: "DC2", 19: "DC3", 20: "DC4", 21: "NAK", 22: "SYN", 23: "ETB", 24: "CAN", 25: "EM", 26: "SUB", 27: "ESC", 28: "FS", 29: "GS", 30: "RS", 31: "US", 32: "Space", 127: "DEL" };

                for (let i = 0; i <= 127; i++) {
                    const charContainer = document.createElement('div');
                    charContainer.className = 'relative p-2 rounded bg-slate-700/50 ascii-char flex items-center justify-center h-12';
                    
                    const char = (i >= 33 && i <= 126) ? String.fromCharCode(i) : ' ';
                    charContainer.textContent = char;

                    if (i < 33 || i === 127) {
                        charContainer.classList.add('text-slate-500');
                        charContainer.textContent = controlChars[i] || 'CTL';
                    }
                    
                    const tooltip = document.createElement('div');
                    tooltip.className = 'ascii-tooltip absolute bottom-full mb-2 w-max p-2 text-xs bg-slate-900 text-white rounded-md shadow-lg opacity-0 invisible transition-opacity duration-200 z-10';
                    const hex = `0x${i.toString(16).toUpperCase().padStart(2, '0')}`;
                    const description = controlChars[i] ? `Control: ${controlChars[i]}` : `Carácter: '${char}'`;
                    tooltip.innerHTML = `<strong>${description}</strong><br>Decimal: ${i}<br>Hex: ${hex}`;
                    
                    charContainer.appendChild(tooltip);
                    asciiGrid.appendChild(charContainer);
                }
            }
            
            // --- Arranque ---
            setupStyleControls();
            createColorPalette(fgColorContainer, 'fg');
            createColorPalette(bgColorContainer, 'bg');
            setupLanguageSelector();
            setupCopyButton();
            setupAsciiExplorer();
            
            // Inicializar los selectores de color HSV
            initializeColorPickers('fg');
            initializeColorPickers('bg');

            updatePreview(); 
        });
    </script>
</body>
</html>
