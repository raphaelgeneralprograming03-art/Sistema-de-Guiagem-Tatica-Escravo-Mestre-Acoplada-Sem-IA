
<html lang="pt-BR" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SISTEMA TÁTICO WEBRTC & DRONE ESCRAVO-MESTRE SEM IA</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        tactical: {
                            dark: '#030712',
                            panel: '#0a101d',
                            border: '#1b2a3f',
                            cyan: '#06b6d4',
                            emerald: '#10b981',
                            amber: '#f59e0b',
                            rose: '#f43f5e',
                            purple: '#a855f7'
                        }
                    },
                    fontFamily: {
                        mono: ['Consolas', 'Monaco', 'Courier New', 'monospace'],
                        sans: ['Inter', 'sans-serif']
                    }
                }
            }
        }
    </script>
    <style>
        body {
            background-color: #020617;
            color: #f1f5f9;
            overflow-x: hidden;
            user-select: none;
        }
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: #0a101d; }
        ::-webkit-scrollbar-thumb { background: #1b2a3f; border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: #06b6d4; }
        
        .canvas-container {
            position: relative;
            width: 100%;
            height: 100%;
            min-height: 520px;
        }
        .pip-hud {
            background: rgba(3, 11, 24, 0.90);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(6, 182, 212, 0.4);
        }
        .radar-scan {
            background: conic-gradient(from 0deg, rgba(6, 182, 212, 0.25), transparent 60deg);
            animation: spin 2.5s linear infinite;
        }
        @keyframes spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }
        .target-btn { transition: all 0.2s ease; }
        .target-btn.active {
            border-color: #06b6d4;
            background-color: rgba(6, 182, 212, 0.18);
            box-shadow: 0 0 12px rgba(6, 182, 212, 0.35);
        }
        .tab-btn.active {
            border-bottom: 2px solid #06b6d4;
            color: #06b6d4;
            font-weight: bold;
        }
    </style>
</head>
<body class="font-sans antialiased text-slate-200">

    <div class="min-h-screen flex flex-col justify-between p-2 sm:p-4 gap-3 max-w-[1920px] mx-auto">
        
        <!-- HEADER PRINCIPAL -->
        <header class="bg-tactical-panel/90 border border-tactical-border rounded-xl p-3 sm:p-4 backdrop-blur shadow-2xl flex flex-col lg:flex-row justify-between items-center gap-4">
            <div class="flex items-center gap-3">
                <div class="p-3 bg-cyan-500/10 border border-cyan-500/40 rounded-xl text-cyan-400">
                    <i class="fa-solid fa-satellite-dish text-2xl animate-pulse"></i>
                </div>
                <div>
                    <h1 class="text-lg sm:text-2xl font-bold bg-gradient-to-r from-cyan-400 via-teal-300 to-emerald-400 bg-clip-text text-transparent">
                        SISTEMA DE TRANSMISSÃO WEBRTC & GUIAGEM ESCRAVO-MESTRE
                    </h1>
                    <p class="text-xs text-slate-400 font-mono">
                        Multiplataforma (Windows, macOS, Linux, Android, iOS) | Servomecanismo Óptico Sem IA
                    </p>
                </div>
            </div>
            
            <div class="flex flex-wrap items-center gap-2 bg-slate-950/80 p-2 rounded-lg border border-slate-800 text-xs font-mono">
                <div class="flex items-center gap-2 px-2.5 border-r border-slate-800">
                    <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 animate-ping"></span>
                    <span class="text-slate-300">WEBRTC P2P: <span id="webrtcLatency" class="text-emerald-400 font-bold">1.2 ms</span></span>
                </div>
                <div class="flex items-center gap-2 px-2.5 border-r border-slate-800">
                    <i class="fa-solid fa-crosshairs text-cyan-400"></i>
                    <span class="text-slate-300">SENSOR TRAVA: <span id="statusLockMode" class="text-cyan-400 font-bold">IMAGE LOCK OK</span></span>
                </div>
                <div class="flex items-center gap-2 px-2.5">
                    <i class="fa-solid fa-shield-halved text-amber-400"></i>
                    <span class="text-slate-300">ESTADO DA MISSÃO: <span id="statusMissionState" class="text-amber-400 font-bold">EM ESPERA</span></span>
                </div>
            </div>
        </header>

        <!-- NAVEGAÇÃO DE ABAS: VISÃO GERAL DO SISTEMA E SIMULADOR -->
        <div class="flex border-b border-tactical-border font-mono text-xs text-slate-400 gap-4 px-2">
            <button id="tabSim" class="tab-btn active pb-2 px-3 flex items-center gap-2">
                <i class="fa-solid fa-gamepad"></i> Simulador Tático Integrado
            </button>
            <button id="tabWebRTC" class="tab-btn pb-2 px-3 flex items-center gap-2">
                <i class="fa-solid fa-desktop"></i> Transmissão de Tela WebRTC (Estúdio)
            </button>
            <button id="tabOverview" class="tab-btn pb-2 px-3 flex items-center gap-2">
                <i class="fa-solid fa-sitemap"></i> Visão Geral da Arquitetura sem IA
            </button>
        </div>

        <!-- ABA 1: SIMULADOR TÁTICO INTEGRADO -->
        <div id="viewSim" class="flex flex-col gap-3">
            
            <section class="grid grid-cols-1 xl:grid-cols-12 gap-3">
                <!-- SELETOR DE ALVOS -->
                <div class="xl:col-span-8 bg-tactical-panel/90 border border-tactical-border rounded-xl p-3 shadow-xl">
                    <h2 class="text-xs font-mono uppercase text-cyan-400 tracking-wider mb-2 font-bold flex items-center justify-between">
                        <span><i class="fa-solid fa-bullseye mr-1.5"></i> 1. Seleção de Alvo Operacional</span>
                        <span class="text-[10px] text-slate-400">Transmissão Sincronizada do Drone Observador</span>
                    </h2>
                    
                    <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-6 gap-2 text-xs font-mono">
                        <button data-target="tank" class="target-btn active bg-slate-900/80 border border-slate-700 hover:border-cyan-500 p-2 rounded-lg flex flex-col items-center gap-1.5">
                            <i class="fa-solid fa-tank text-lg text-amber-400"></i>
                            <span class="font-bold text-slate-200 text-[11px]">Tanque</span>
                            <span class="text-[9px] text-slate-400">Blindado Móvel</span>
                        </button>

                        <button data-target="drone" class="target-btn bg-slate-900/80 border border-slate-700 hover:border-cyan-500 p-2 rounded-lg flex flex-col items-center gap-1.5">
                            <i class="fa-solid fa-plane-slash text-lg text-rose-400"></i>
                            <span class="font-bold text-slate-200 text-[11px]">Drone Inimigo</span>
                            <span class="text-[9px] text-slate-400">Ameaça Aérea</span>
                        </button>

                        <button data-target="sniper" class="target-btn bg-slate-900/80 border border-slate-700 hover:border-cyan-500 p-2 rounded-lg flex flex-col items-center gap-1.5">
                            <i class="fa-solid fa-building text-lg text-purple-400"></i>
                            <span class="font-bold text-slate-200 text-[11px]">Prédio / Sniper</span>
                            <span class="text-[9px] text-slate-400">Urbano Elevado</span>
                        </button>

                        <button data-target="artillery" class="target-btn bg-slate-900/80 border border-slate-700 hover:border-cyan-500 p-2 rounded-lg flex flex-col items-center gap-1.5">
                            <i class="fa-solid fa-burst text-lg text-orange-400"></i>
                            <span class="font-bold text-slate-200 text-[11px]">Artilharia</span>
                            <span class="text-[9px] text-slate-400">Bateria Estática</span>
                        </button>

                        <button data-target="warship" class="target-btn bg-slate-900/80 border border-slate-700 hover:border-cyan-500 p-2 rounded-lg flex flex-col items-center gap-1.5">
                            <i class="fa-solid fa-ship text-lg text-blue-400"></i>
                            <span class="font-bold text-slate-200 text-[11px]">Navio / Fragata</span>
                            <span class="text-[9px] text-slate-400">Superfície Marítima</span>
                        </button>

                        <button data-target="base" class="target-btn bg-slate-900/80 border border-slate-700 hover:border-cyan-500 p-2 rounded-lg flex flex-col items-center gap-1.5">
                            <i class="fa-solid fa-fort-awesome text-lg text-emerald-400"></i>
                            <span class="font-bold text-slate-200 text-[11px]">Base Inimiga</span>
                            <span class="text-[9px] text-slate-400">Comando Fortificado</span>
                        </button>
                    </div>
                </div>

                <!-- CONTROLE DE DECOLAGEM E MISSÃO -->
                <div class="xl:col-span-4 bg-tactical-panel/90 border border-tactical-border rounded-xl p-3 shadow-xl flex flex-col justify-between gap-2">
                    <h2 class="text-xs font-mono uppercase text-amber-400 tracking-wider font-bold flex items-center justify-between">
                        <span><i class="fa-solid fa-play mr-1.5"></i> 2. Execução da Missão Tática</span>
                        <span id="missionTypeTag" class="text-[10px] text-rose-400 font-bold">MODO: ATAQUE</span>
                    </h2>

                    <div class="grid grid-cols-2 gap-2 text-xs font-mono">
                        <button id="btnModeStrike" class="bg-rose-600/30 border border-rose-500 text-rose-200 py-1.5 px-2 rounded-lg font-bold flex items-center justify-center gap-1.5 hover:bg-rose-600/50">
                            <i class="fa-solid fa-explosion"></i> Ataque Tático
                        </button>
                        <button id="btnModeLand" class="bg-slate-800 border border-slate-700 text-slate-300 py-1.5 px-2 rounded-lg font-bold flex items-center justify-center gap-1.5 hover:bg-emerald-600/30 hover:border-emerald-500">
                            <i class="fa-solid fa-plane-arrival"></i> Pouso de Precisão
                        </button>
                    </div>

                    <div class="flex items-center gap-2">
                        <button id="btnLaunch" class="flex-1 bg-gradient-to-r from-cyan-500 to-emerald-500 hover:from-cyan-400 hover:to-emerald-400 text-slate-950 font-bold py-2 px-3 rounded-lg font-mono text-xs shadow-lg flex items-center justify-center gap-2">
                            <i class="fa-solid fa-rocket text-base"></i> DECOLAR E EXECUTAR MISSÃO
                        </button>
                        <button id="btnReset" class="bg-slate-800 hover:bg-slate-700 text-slate-300 border border-slate-700 p-2 rounded-lg text-xs font-mono">
                            <i class="fa-solid fa-rotate-right"></i>
                        </button>
                    </div>
                </div>
            </section>

            <main class="grid grid-cols-1 lg:grid-cols-12 gap-3 flex-1">
                
                <!-- CANVAS DE SIMULAÇÃO CENTRAL COM HUD PIP -->
                <div class="lg:col-span-8 flex flex-col bg-tactical-panel/90 border border-tactical-border rounded-xl p-3 shadow-2xl relative overflow-hidden">
                    
                    <div class="flex justify-between items-center mb-2 pb-2 border-b border-slate-800">
                        <div class="flex items-center gap-2">
                            <span class="w-2.5 h-2.5 rounded-full bg-cyan-400 animate-ping"></span>
                            <h2 class="text-xs font-mono font-bold text-slate-200 uppercase tracking-wider">
                                Visualizador Tático ISO - Sincronização por Transmissão de Painel
                            </h2>
                        </div>
                        <div class="flex items-center gap-3 text-xs font-mono text-slate-400">
                            <span class="flex items-center gap-1"><i class="fa-solid fa-location-pin text-cyan-400"></i> Local de Saída</span>
                            <span class="flex items-center gap-1"><i class="fa-solid fa-flag-checkered text-emerald-400"></i> Alvo/Destino</span>
                        </div>
                    </div>

                    <div class="canvas-container flex-1 bg-slate-950 rounded-lg overflow-hidden border border-slate-800/80 relative">
                        <canvas id="simCanvas" class="w-full h-full block cursor-crosshair"></canvas>

                        <!-- HUD PIP: CÂMERA DO TRIPÉ ACOPLADA NA BARRIGA DO DRONE -->
                        <div class="pip-hud absolute bottom-3 right-3 w-56 sm:w-64 p-2.5 rounded-lg shadow-2xl pointer-events-none">
                            <div class="flex justify-between items-center border-b border-cyan-500/30 pb-1 mb-1.5">
                                <span class="text-[10px] font-mono text-cyan-400 font-bold flex items-center gap-1">
                                    <i class="fa-solid fa-camera text-rose-400"></i> FEED CÂMERA ACOPLADA
                                </span>
                                <span id="pipLockBadge" class="text-[9px] font-mono text-emerald-400 font-bold animate-pulse">LOCK ACTIVE</span>
                            </div>
                            
                            <div class="relative w-full h-28 bg-slate-950 rounded border border-slate-800 overflow-hidden flex items-center justify-center">
                                <div class="radar-scan absolute inset-0 opacity-20"></div>
                                <div class="absolute inset-0 flex items-center justify-center">
                                    <div class="w-full h-[1px] bg-cyan-500/30"></div>
                                    <div class="h-full w-[1px] bg-cyan-500/30"></div>
                                </div>
                                <div id="pipTargetBox" class="absolute w-8 h-8 border-2 border-emerald-400 rounded transition-all duration-75 flex items-center justify-center">
                                    <span class="w-1.5 h-1.5 bg-emerald-400 rounded-full"></span>
                                </div>
                                <div class="absolute top-1 left-1 text-[8px] font-mono text-cyan-300 bg-black/60 px-1 rounded">
                                    SENSOR: IMAGE LOCK
                                </div>
                            </div>

                            <div class="grid grid-cols-2 gap-1 mt-1.5 text-[10px] font-mono text-slate-300">
                                <div>DELTA X: <span id="pipDeltaX" class="text-amber-400 font-bold">+0.0px</span></div>
                                <div>DELTA Y: <span id="pipDeltaY" class="text-amber-400 font-bold">+0.0px</span></div>
                            </div>
                        </div>
                    </div>

                    <!-- FERRAMENTAS AMBIENTAIS -->
                    <div class="mt-2.5 grid grid-cols-1 sm:grid-cols-3 gap-2 bg-slate-950/80 p-2 rounded-lg border border-slate-800 text-xs font-mono">
                        <div class="flex flex-col gap-1">
                            <label class="text-slate-400 text-[11px] flex justify-between">
                                <span>Velocidade do Vento:</span>
                                <span id="windSpeedVal" class="text-amber-400 font-bold">14.0 km/h</span>
                            </label>
                            <input id="sliderWindSpeed" type="range" min="0" max="60" value="14" step="1" class="accent-amber-500 cursor-pointer h-1.5 bg-slate-800 rounded">
                        </div>

                        <div class="flex flex-col gap-1">
                            <label class="text-slate-400 text-[11px] flex justify-between">
                                <span>Direção do Vento:</span>
                                <span id="windDirVal" class="text-amber-400 font-bold">60° (NE)</span>
                            </label>
                            <input id="sliderWindDir" type="range" min="0" max="360" value="60" step="5" class="accent-amber-500 cursor-pointer h-1.5 bg-slate-800 rounded">
                        </div>

                        <div class="flex items-end">
                            <button id="btnGustWind" class="w-full bg-amber-600/30 hover:bg-amber-600/50 border border-amber-500/50 text-amber-200 py-1 px-2 rounded flex items-center justify-center gap-1.5 text-[11px]">
                                <i class="fa-solid fa-wind"></i> Simular Rajada de Vento
                            </button>
                        </div>
                    </div>
                </div>

                <!-- DASHBOARD DE TELEMETRIA E MOTORES -->
                <div class="lg:col-span-4 flex flex-col gap-3">
                    
                    <div class="bg-tactical-panel/90 border border-tactical-border rounded-xl p-3 shadow-xl flex flex-col gap-2">
                        <h3 class="text-xs font-mono uppercase text-cyan-400 font-bold flex items-center justify-between border-b border-slate-800 pb-1.5">
                            <span><i class="fa-solid fa-tower-cell mr-1"></i> Telemetria & Estação Meteorológica</span>
                            <span class="text-[10px] text-slate-500">Link RF</span>
                        </h3>

                        <div class="grid grid-cols-2 gap-2 text-xs font-mono">
                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">TEMPERATURA</div>
                                <div class="text-slate-200 font-bold flex items-center gap-1 mt-0.5">
                                    <i class="fa-solid fa-temperature-half text-rose-400"></i>
                                    <span id="telemetryTemp">24.8 °C</span>
                                </div>
                            </div>

                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">UMIDADE AR</div>
                                <div class="text-slate-200 font-bold flex items-center gap-1 mt-0.5">
                                    <i class="fa-solid fa-droplet text-blue-400"></i>
                                    <span id="telemetryHumidity">62 %</span>
                                </div>
                            </div>

                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">ALTITUDE DO DRONE</div>
                                <div class="text-cyan-400 font-bold flex items-center gap-1 mt-0.5">
                                    <i class="fa-solid fa-arrows-up-to-line text-cyan-400"></i>
                                    <span id="telemetryAltitude">0.0 m</span>
                                </div>
                            </div>

                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">DISTÂNCIA ALVO</div>
                                <div class="text-emerald-400 font-bold flex items-center gap-1 mt-0.5">
                                    <i class="fa-solid fa-ruler-horizontal text-emerald-400"></i>
                                    <span id="telemetryDistance">180.0 m</span>
                                </div>
                            </div>
                        </div>

                        <div class="bg-slate-950/80 p-2 rounded border border-slate-800 text-xs font-mono flex justify-between items-center">
                            <div>
                                <div class="text-slate-500 text-[10px]">COORDENADAS GPS</div>
                                <div class="text-slate-300 font-bold text-[11px]">23°33'04.2"S 46°37'51.0"W</div>
                            </div>
                            <i class="fa-solid fa-map-location-dot text-cyan-400 text-lg"></i>
                        </div>
                    </div>

                    <div class="bg-tactical-panel/90 border border-tactical-border rounded-xl p-3 shadow-xl flex flex-col gap-2">
                        <h3 class="text-xs font-mono uppercase text-amber-400 font-bold flex items-center justify-between border-b border-slate-800 pb-1.5">
                            <span><i class="fa-solid fa-sliders mr-1"></i> Tripé Servo & Armação Mestre</span>
                            <span class="text-[10px] text-emerald-400 font-bold">ACOPLADO</span>
                        </h3>

                        <div class="grid grid-cols-2 gap-2 text-xs font-mono">
                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">GIMBAL PAN (YAW)</div>
                                <div id="anglePan" class="text-amber-400 font-bold">+0.0°</div>
                            </div>
                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">GIMBAL TILT (PITCH)</div>
                                <div id="angleTilt" class="text-amber-400 font-bold">+0.0°</div>
                            </div>
                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">ARMAÇÃO ME. PITCH</div>
                                <div id="framePitch" class="text-purple-400 font-bold">+0.0°</div>
                            </div>
                            <div class="bg-slate-950/80 p-2 rounded border border-slate-800">
                                <div class="text-slate-500 text-[10px]">ARMAÇÃO ME. YAW</div>
                                <div id="frameYaw" class="text-purple-400 font-bold">+0.0°</div>
                            </div>
                        </div>
                    </div>

                    <div class="bg-tactical-panel/90 border border-tactical-border rounded-xl p-3 shadow-xl flex flex-col gap-2 flex-1">
                        <h3 class="text-xs font-mono uppercase text-teal-400 font-bold flex items-center justify-between border-b border-slate-800 pb-1.5">
                            <span><i class="fa-solid fa-fan mr-1"></i> Motores de Sustentação (RPM)</span>
                            <span class="text-[10px] text-slate-400">Misturador Direto</span>
                        </h3>

                        <div class="space-y-2 font-mono text-xs">
                            <div>
                                <div class="flex justify-between text-slate-300 mb-0.5">
                                    <span>M1 (Frontal Esquerdo)</span>
                                    <span id="m1RpmText" class="font-bold text-teal-400">0 RPM</span>
                                </div>
                                <div class="w-full bg-slate-950 h-2 rounded-full overflow-hidden border border-slate-800">
                                    <div id="m1RpmBar" class="bg-gradient-to-r from-teal-500 to-emerald-400 h-full transition-all duration-75" style="width: 0%"></div>
                                </div>
                            </div>

                            <div>
                                <div class="flex justify-between text-slate-300 mb-0.5">
                                    <span>M2 (Frontal Direito)</span>
                                    <span id="m2RpmText" class="font-bold text-teal-400">0 RPM</span>
                                </div>
                                <div class="w-full bg-slate-950 h-2 rounded-full overflow-hidden border border-slate-800">
                                    <div id="m2RpmBar" class="bg-gradient-to-r from-teal-500 to-emerald-400 h-full transition-all duration-75" style="width: 0%"></div>
                                </div>
                            </div>

                            <div>
                                <div class="flex justify-between text-slate-300 mb-0.5">
                                    <span>M3 (Traseiro Esquerdo)</span>
                                    <span id="m3RpmText" class="font-bold text-teal-400">0 RPM</span>
                                </div>
                                <div class="w-full bg-slate-950 h-2 rounded-full overflow-hidden border border-slate-800">
                                    <div id="m3RpmBar" class="bg-gradient-to-r from-teal-500 to-emerald-400 h-full transition-all duration-75" style="width: 0%"></div>
                                </div>
                            </div>

                            <div>
                                <div class="flex justify-between text-slate-300 mb-0.5">
                                    <span>M4 (Traseiro Direito)</span>
                                    <span id="m4RpmText" class="font-bold text-teal-400">0 RPM</span>
                                </div>
                                <div class="w-full bg-slate-950 h-2 rounded-full overflow-hidden border border-slate-800">
                                    <div id="m4RpmBar" class="bg-gradient-to-r from-teal-500 to-emerald-400 h-full transition-all duration-75" style="width: 0%"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </main>
        </div>

        <!-- ABA 2: ESTÚDIO DE TRANSMISSÃO DE TELA WEBRTC -->
        <div id="viewWebRTC" class="hidden flex-col gap-3 bg-tactical-panel/90 border border-tactical-border rounded-xl p-4 shadow-xl">
            <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-2 border-b border-slate-800 pb-3">
                <div>
                    <h2 class="text-sm font-bold font-mono text-cyan-400 flex items-center gap-2">
                        <i class="fa-solid fa-desktop"></i> TRANSMISSOR DE PAINÉIS EM TEMPO REAL (WEBRTC / CANVAS)
                    </h2>
                    <p class="text-xs text-slate-400 font-mono">
                        Captura telas de Windows, macOS, Linux, Android e iOS usando a API nativa HTML5 getDisplayMedia
                    </p>
                </div>
                <div class="flex gap-2 font-mono text-xs">
                    <button id="btnStartScreenCapture" class="bg-cyan-500 hover:bg-cyan-400 text-slate-950 font-bold py-1.5 px-3 rounded flex items-center gap-1.5">
                        <i class="fa-solid fa-play"></i> Capturar Tela Real (getDisplayMedia)
                    </button>
                    <button id="btnProcessCanvas" class="bg-slate-800 hover:bg-slate-700 text-cyan-400 border border-cyan-500/40 font-bold py-1.5 px-3 rounded flex items-center gap-1.5">
                        <i class="fa-solid fa-microchip"></i> Renderizar no Canvas (ms)
                    </button>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 my-2">
                <!-- Entrada de Vídeo do Display -->
                <div class="bg-slate-950 p-3 rounded-lg border border-slate-800 flex flex-col gap-2">
                    <h3 class="text-xs font-mono font-bold text-slate-300 flex justify-between">
                        <span>1. ENTRADA DE VÍDEO DO PAINEL</span>
                        <span id="captureStatusText" class="text-amber-400">AGUARDANDO SELEÇÃO</span>
                    </h3>
                    <div class="relative aspect-video bg-slate-900 rounded overflow-hidden flex items-center justify-center border border-slate-800">
                        <video id="videoScreenSource" autoplay playsinline muted class="w-full h-full object-contain"></video>
                        <div id="videoPlaceholder" class="absolute inset-0 flex flex-col items-center justify-center text-slate-600 font-mono text-xs gap-2">
                            <i class="fa-solid fa-display text-3xl text-slate-700"></i>
                            <span>Nenhum painel selecionado. Clique em "Capturar Tela Real".</span>
                        </div>
                    </div>
                </div>

                <!-- Saída Canvas Processada em Milissegundos -->
                <div class="bg-slate-950 p-3 rounded-lg border border-slate-800 flex flex-col gap-2">
                    <h3 class="text-xs font-mono font-bold text-slate-300 flex justify-between">
                        <span>2. CANVAS RECEPTOR COM RETÍCULO SINCRONIZADO</span>
                        <span id="canvasLatencyText" class="text-emerald-400">LATÊNCIA: -- ms</span>
                    </h3>
                    <div class="relative aspect-video bg-slate-900 rounded overflow-hidden flex items-center justify-center border border-slate-800">
                        <canvas id="canvasScreenOutput" class="w-full h-full block"></canvas>
                    </div>
                </div>
            </div>
        </div>

        <!-- ABA 3: VISÃO GERAL DA ARQUITETURA SEM IA -->
        <div id="viewOverview" class="hidden flex-col gap-3 bg-tactical-panel/90 border border-tactical-border rounded-xl p-4 shadow-xl font-mono">
            <h2 class="text-sm font-bold text-amber-400 border-b border-slate-800 pb-2 flex items-center gap-2">
                <i class="fa-solid fa-sitemap"></i> ARQUITETURA DO SISTEMA DE GUIAGEM ESCRAVO-MESTRE ACOPLADA
            </h2>
            
            <div class="grid grid-cols-1 md:grid-cols-5 gap-3 text-xs my-2">
                
                <div class="bg-slate-950 p-3 rounded border border-cyan-500/30 flex flex-col gap-1.5">
                    <div class="text-cyan-400 font-bold flex items-center gap-1.5">
                        <span class="w-5 h-5 rounded-full bg-cyan-500/20 flex items-center justify-center text-[10px]">1</span>
                        DRONE OBSERVADOR
                    </div>
                    <p class="text-slate-400 text-[11px]">
                        Em altitude (500m), mapeia o alvo e local de chegada com telescópio estabilizado.
                    </p>
                </div>

                <div class="bg-slate-950 p-3 rounded border border-cyan-500/30 flex flex-col gap-1.5">
                    <div class="text-cyan-400 font-bold flex items-center gap-1.5">
                        <span class="w-5 h-5 rounded-full bg-cyan-500/20 flex items-center justify-center text-[10px]">2</span>
                        TRANSMISSÃO DE PAINEL
                    </div>
                    <p class="text-slate-400 text-[11px]">
                        Transmita o feed em milissegundos via WebRTC P2P para telas/painéis do sistema.
                    </p>
                </div>

                <div class="bg-slate-950 p-3 rounded border border-amber-500/30 flex flex-col gap-1.5">
                    <div class="text-amber-400 font-bold flex items-center gap-1.5">
                        <span class="w-5 h-5 rounded-full bg-amber-500/20 flex items-center justify-center text-[10px]">3</span>
                        CÂMERA ACOPLADA NO TRIPÉ
                    </div>
                    <p class="text-slate-400 text-[11px]">
                        Fixada na barriga do drone principal. Trava o centro da imagem na tela do painel.
                    </p>
                </div>

                <div class="bg-slate-950 p-3 rounded border border-purple-500/30 flex flex-col gap-1.5">
                    <div class="text-purple-400 font-bold flex items-center gap-1.5">
                        <span class="w-5 h-5 rounded-full bg-purple-500/20 flex items-center justify-center text-[10px]">4</span>
                        ARMAÇÃO ESCRAVO-MESTRE
                    </div>
                    <p class="text-slate-400 text-[11px]">
                        Segue o mini-tripé fisicamente, reorientando o vetor mecânico de comando (Pitch/Yaw).
                    </p>
                </div>

                <div class="bg-slate-950 p-3 rounded border border-teal-500/30 flex flex-col gap-1.5">
                    <div class="text-teal-400 font-bold flex items-center gap-1.5">
                        <span class="w-5 h-5 rounded-full bg-teal-500/20 flex items-center justify-center text-[10px]">5</span>
                        ROTORES & HÉLICES
                    </div>
                    <p class="text-slate-400 text-[11px]">
                        Misturador de canais ajusta a velocidade de rotação (RPM M1-M4) sem auxílio de IA.
                    </p>
                </div>

            </div>

            <div class="bg-slate-950 p-3 rounded border border-slate-800 text-xs text-slate-300">
                <span class="text-amber-400 font-bold">EQUAÇÃO DO MISTURADOR DE CANAIS (MOTOR MIXER):</span>
                <p class="mt-1 font-mono text-[11px] text-slate-400">
                    RPM_M1 = Base_RPM + (Pitch_Cmd × Kp) + (Yaw_Cmd × Ky)<br>
                    RPM_M2 = Base_RPM + (Pitch_Cmd × Kp) - (Yaw_Cmd × Ky)<br>
                    RPM_M3 = Base_RPM - (Pitch_Cmd × Kp) + (Yaw_Cmd × Ky)<br>
                    RPM_M4 = Base_RPM - (Pitch_Cmd × Kp) - (Yaw_Cmd × Ky)
                </p>
            </div>
        </div>

        <footer class="text-center text-[10px] font-mono text-slate-500 py-1">
            Simulação Aeroespacial e Estúdio de Transmissão WebRTC em HTML5, CSS3 e Canvas sem Inteligência Artificial.
        </footer>

    </div>

    <script>
        // ESTADO GLOBAL
        const sim = {
            launchPad: { x: 90, y: 0 },
            targetType: 'tank',
            targetPos: { x: 0, y: 0 },
            missionType: 'strike',
            missionState: 'IDLE',
            
            drone: {
                x: 0, y: 0, altitude: 0, speed: 0,
                baseRPM: 0, m1RPM: 0, m2RPM: 0, m3RPM: 0, m4RPM: 0, propAngle: 0
            },
            
            observerDrone: { x: 0, y: 45 },
            tripod: { panDeg: 0, tiltDeg: 0, deltaX: 0, deltaY: 0 },
            masterFrame: { pitch: 0, yaw: 0 },
            environment: { windSpeed: 14.0, windDirDeg: 60, gustFactor: 0 },
            explosion: null
        };

        // CANVAS DE SIMULAÇÃO
        const canvas = document.getElementById('simCanvas');
        const ctx = canvas.getContext('2d');

        // ABAS
        const tabSim = document.getElementById('tabSim');
        const tabWebRTC = document.getElementById('tabWebRTC');
        const tabOverview = document.getElementById('tabOverview');
        const viewSim = document.getElementById('viewSim');
        const viewWebRTC = document.getElementById('viewWebRTC');
        const viewOverview = document.getElementById('viewOverview');

        tabSim.addEventListener('click', () => switchTab(tabSim, viewSim));
        tabWebRTC.addEventListener('click', () => switchTab(tabWebRTC, viewWebRTC));
        tabOverview.addEventListener('click', () => switchTab(tabOverview, viewOverview));

        function switchTab(btn, view) {
            [tabSim, tabWebRTC, tabOverview].forEach(b => b.classList.remove('active'));
            [viewSim, viewWebRTC, viewOverview].forEach(v => v.classList.add('hidden'));
            btn.classList.add('active');
            view.classList.remove('hidden');
            if (view === viewSim) resizeCanvas();
        }

        // BOTOES E DOM
        const btnLaunch = document.getElementById('btnLaunch');
        const btnReset = document.getElementById('btnReset');
        const btnModeStrike = document.getElementById('btnModeStrike');
        const btnModeLand = document.getElementById('btnModeLand');
        const missionTypeTag = document.getElementById('missionTypeTag');

        const statusMissionState = document.getElementById('statusMissionState');
        const sliderWindSpeed = document.getElementById('sliderWindSpeed');
        const sliderWindDir = document.getElementById('sliderWindDir');
        const windSpeedVal = document.getElementById('windSpeedVal');
        const windDirVal = document.getElementById('windDirVal');
        const btnGustWind = document.getElementById('btnGustWind');

        const pipTargetBox = document.getElementById('pipTargetBox');
        const pipDeltaX = document.getElementById('pipDeltaX');
        const pipDeltaY = document.getElementById('pipDeltaY');

        const telemetryAltitude = document.getElementById('telemetryAltitude');
        const telemetryDistance = document.getElementById('telemetryDistance');
        const anglePan = document.getElementById('anglePan');
        const angleTilt = document.getElementById('angleTilt');
        const framePitch = document.getElementById('framePitch');
        const frameYaw = document.getElementById('frameYaw');

        const m1RpmText = document.getElementById('m1RpmText');
        const m1RpmBar = document.getElementById('m1RpmBar');
        const m2RpmText = document.getElementById('m2RpmText');
        const m2RpmBar = document.getElementById('m2RpmBar');
        const m3RpmText = document.getElementById('m3RpmText');
        const m3RpmBar = document.getElementById('m3RpmBar');
        const m4RpmText = document.getElementById('m4RpmText');
        const m4RpmBar = document.getElementById('m4RpmBar');

        function resizeCanvas() {
            const rect = canvas.parentElement.getBoundingClientRect();
            canvas.width = rect.width;
            canvas.height = rect.height;

            sim.launchPad.x = 100;
            sim.launchPad.y = canvas.height * 0.75;

            if (sim.missionState === 'IDLE') {
                resetDroneToPad();
                resetTargetPosition();
            }
        }

        function resetDroneToPad() {
            sim.drone.x = sim.launchPad.x;
            sim.drone.y = sim.launchPad.y;
            sim.drone.altitude = 0;
            sim.drone.baseRPM = 0;
            sim.drone.m1RPM = 0; sim.drone.m2RPM = 0; sim.drone.m3RPM = 0; sim.drone.m4RPM = 0;
            sim.missionState = 'IDLE';
            sim.explosion = null;
            statusMissionState.textContent = "EM ESPERA";
            statusMissionState.className = "text-amber-400 font-bold";
        }

        function resetTargetPosition() {
            sim.targetPos.x = canvas.width - 130;
            sim.targetPos.y = canvas.height * (sim.targetType === 'drone' ? 0.35 : sim.targetType === 'sniper' ? 0.5 : 0.75);
        }

        window.addEventListener('resize', resizeCanvas);

        // SELEÇÃO DE ALVOS
        document.querySelectorAll('.target-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelectorAll('.target-btn').forEach(b => b.classList.remove('active'));
                btn.classList.add('active');
                sim.targetType = btn.getAttribute('data-target');
                resetTargetPosition();
            });
        });

        // MODO DA MISSÃO
        btnModeStrike.addEventListener('click', () => {
            sim.missionType = 'strike';
            btnModeStrike.className = "bg-rose-600/30 border border-rose-500 text-rose-200 py-1.5 px-2 rounded-lg font-bold flex items-center justify-center gap-1.5 hover:bg-rose-600/50";
            btnModeLand.className = "bg-slate-800 border border-slate-700 text-slate-300 py-1.5 px-2 rounded-lg font-bold flex items-center justify-center gap-1.5 hover:bg-emerald-600/30 hover:border-emerald-500";
            missionTypeTag.textContent = "MODO: ATAQUE";
            missionTypeTag.className = "text-[10px] text-rose-400 font-bold";
        });

        btnModeLand.addEventListener('click', () => {
            sim.missionType = 'land';
            btnModeLand.className = "bg-emerald-600/30 border border-emerald-500 text-emerald-200 py-1.5 px-2 rounded-lg font-bold flex items-center justify-center gap-1.5 hover:bg-emerald-600/50";
            btnModeStrike.className = "bg-slate-800 border border-slate-700 text-slate-300 py-1.5 px-2 rounded-lg font-bold flex items-center justify-center gap-1.5 hover:bg-rose-600/30 hover:border-rose-500";
            missionTypeTag.textContent = "MODO: POUSO";
            missionTypeTag.className = "text-[10px] text-emerald-400 font-bold";
        });

        btnLaunch.addEventListener('click', () => {
            if (sim.missionState === 'IDLE' || sim.missionState === 'COMPLETED') {
                resetDroneToPad();
                sim.missionState = 'TAKEOFF';
                statusMissionState.textContent = "DECOLANDO...";
                statusMissionState.className = "text-cyan-400 font-bold animate-pulse";
            }
        });

        btnReset.addEventListener('click', () => resetDroneToPad());

        sliderWindSpeed.addEventListener('input', (e) => {
            sim.environment.windSpeed = parseFloat(e.target.value);
            windSpeedVal.textContent = sim.environment.windSpeed.toFixed(1) + " km/h";
        });

        sliderWindDir.addEventListener('input', (e) => {
            sim.environment.windDirDeg = parseInt(e.target.value);
            windDirVal.textContent = sim.environment.windDirDeg + "°";
        });

        btnGustWind.addEventListener('click', () => {
            sim.environment.gustFactor = 30;
            setTimeout(() => { sim.environment.gustFactor = 0; }, 2000);
        });

        function updatePhysics() {
            sim.observerDrone.x = canvas.width * 0.5;

            if (sim.missionState === 'TAKEOFF') {
                sim.drone.baseRPM = 5500;
                sim.drone.altitude += 1.8;
                sim.drone.y -= 1.8;
                if (sim.drone.altitude >= 60) {
                    sim.missionState = 'CRUISE';
                    statusMissionState.textContent = "EM NAVEGAÇÃO";
                    statusMissionState.className = "text-emerald-400 font-bold";
                }
            } else if (sim.missionState === 'CRUISE') {
                sim.drone.baseRPM = 5000;
                const targetX = sim.targetPos.x - (sim.missionType === 'land' ? 50 : 0);
                const targetY = sim.targetPos.y - (sim.missionType === 'land' ? 40 : 20);
                const dx = targetX - sim.drone.x;
                const dy = targetY - sim.drone.y;
                const dist = Math.hypot(dx, dy);

                if (dist > 40) {
                    sim.drone.x += (dx / dist) * 3.5;
                    sim.drone.y += (dy / dist) * 2.0;
                } else {
                    sim.missionState = 'TERMINAL';
                    statusMissionState.textContent = sim.missionType === 'strike' ? "FASE TERMINAL" : "APROXIMAÇÃO DE POUSO";
                    statusMissionState.className = "text-rose-400 font-bold animate-pulse";
                }
            } else if (sim.missionState === 'TERMINAL') {
                if (sim.missionType === 'strike') {
                    const dx = sim.targetPos.x - sim.drone.x;
                    const dy = sim.targetPos.y - sim.drone.y;
                    const dist = Math.hypot(dx, dy);
                    sim.drone.x += (dx / dist) * 7.0;
                    sim.drone.y += (dy / dist) * 7.0;

                    if (dist < 12) {
                        triggerExplosion(sim.targetPos.x, sim.targetPos.y);
                        sim.missionState = 'COMPLETED';
                        statusMissionState.textContent = "ALVO NEUTRALIZADO";
                        statusMissionState.className = "text-rose-500 font-bold";
                    }
                } else {
                    const landX = sim.targetPos.x - 50;
                    const landY = sim.targetPos.y;
                    sim.drone.x += (landX - sim.drone.x) * 0.08;
                    sim.drone.y += (landY - sim.drone.y) * 0.08;
                    sim.drone.altitude *= 0.92;

                    if (sim.drone.altitude < 2) {
                        sim.drone.altitude = 0;
                        sim.drone.baseRPM = 0;
                        sim.missionState = 'COMPLETED';
                        statusMissionState.textContent = "POUSO CONCLUÍDO";
                        statusMissionState.className = "text-emerald-400 font-bold";
                    }
                }
            }

            // CÁLCULO DO TRIPÉ E CÂMERA ACOPLADA
            const deltaX = sim.targetPos.x - sim.drone.x;
            const deltaY = sim.targetPos.y - (sim.drone.y + 15);
            sim.tripod.deltaX = deltaX;
            sim.tripod.deltaY = deltaY;

            const targetPan = (deltaX / (canvas.width * 0.5)) * 45;
            const targetTilt = (deltaY / (canvas.height * 0.5)) * 35;
            sim.tripod.panDeg += (targetPan - sim.tripod.panDeg) * 0.15;
            sim.tripod.tiltDeg += (targetTilt - sim.tripod.tiltDeg) * 0.15;

            // ARMAÇÃO ESCRAVO-MESTRE + CORREÇÃO DO VENTO
            const effectiveWind = sim.environment.windSpeed + sim.environment.gustFactor;
            const windRad = (sim.environment.windDirDeg * Math.PI) / 180;
            const windCorrection = Math.cos(windRad) * (effectiveWind * 0.15);

            sim.masterFrame.pitch = sim.tripod.tiltDeg + windCorrection;
            sim.masterFrame.yaw = sim.tripod.panDeg;

            // MISTURADOR DOS MOTORES (M1-M4)
            if (sim.drone.baseRPM > 0) {
                const pitchEffect = sim.masterFrame.pitch * 18;
                const yawEffect = sim.masterFrame.yaw * 14;

                sim.drone.m1RPM = Math.min(8000, Math.max(1000, sim.drone.baseRPM + pitchEffect + yawEffect));
                sim.drone.m2RPM = Math.min(8000, Math.max(1000, sim.drone.baseRPM + pitchEffect - yawEffect));
                sim.drone.m3RPM = Math.min(8000, Math.max(1000, sim.drone.baseRPM - pitchEffect + yawEffect));
                sim.drone.m4RPM = Math.min(8000, Math.max(1000, sim.drone.baseRPM - pitchEffect - yawEffect));
                sim.drone.propAngle += 0.5;
            } else {
                sim.drone.m1RPM = 0; sim.drone.m2RPM = 0; sim.drone.m3RPM = 0; sim.drone.m4RPM = 0;
            }

            if (sim.explosion) {
                sim.explosion.particles.forEach(p => {
                    p.x += p.vx; p.y += p.vy; p.life -= 0.02;
                });
                sim.explosion.particles = sim.explosion.particles.filter(p => p.life > 0);
            }

            updateDashboardUI();
        }

        function triggerExplosion(x, y) {
            const particles = [];
            for (let i = 0; i < 45; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = 1 + Math.random() * 6;
                particles.push({
                    x, y,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    color: ['#f43f5e', '#f59e0b', '#fbbf24', '#ef4444'][Math.floor(Math.random() * 4)],
                    size: 2 + Math.random() * 5,
                    life: 1.0
                });
            }
            sim.explosion = { particles };
        }

        function updateDashboardUI() {
            telemetryAltitude.textContent = (sim.drone.altitude * 0.4).toFixed(1) + " m";
            const dist = Math.hypot(sim.targetPos.x - sim.drone.x, sim.targetPos.y - sim.drone.y);
            telemetryDistance.textContent = (dist * 0.8).toFixed(1) + " m";

            anglePan.textContent = (sim.tripod.panDeg >= 0 ? "+" : "") + sim.tripod.panDeg.toFixed(1) + "°";
            angleTilt.textContent = (sim.tripod.tiltDeg >= 0 ? "+" : "") + sim.tripod.tiltDeg.toFixed(1) + "°";

            framePitch.textContent = (sim.masterFrame.pitch >= 0 ? "+" : "") + sim.masterFrame.pitch.toFixed(1) + "°";
            frameYaw.textContent = (sim.masterFrame.yaw >= 0 ? "+" : "") + sim.masterFrame.yaw.toFixed(1) + "°";

            pipDeltaX.textContent = (sim.tripod.deltaX >= 0 ? "+" : "") + Math.round(sim.tripod.deltaX) + "px";
            pipDeltaY.textContent = (sim.tripod.deltaY >= 0 ? "+" : "") + Math.round(sim.tripod.deltaY) + "px";

            const pipX = Math.max(-40, Math.min(40, sim.tripod.deltaX * 0.2));
            const pipY = Math.max(-25, Math.min(25, sim.tripod.deltaY * 0.2));
            pipTargetBox.style.transform = `translate(${pipX}px, ${pipY}px)`;

            updateMotorGauge(sim.drone.m1RPM, m1RpmText, m1RpmBar);
            updateMotorGauge(sim.drone.m2RPM, m2RpmText, m2RpmBar);
            updateMotorGauge(sim.drone.m3RPM, m3RpmText, m3RpmBar);
            updateMotorGauge(sim.drone.m4RPM, m4RpmText, m4RpmBar);
        }

        function updateMotorGauge(rpm, textElem, barElem) {
            textElem.textContent = Math.round(rpm) + " RPM";
            const pct = Math.min(100, (rpm / 8000) * 100);
            barElem.style.width = pct + "%";
        }

        function drawScene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Redemoinhos e Grade
            ctx.strokeStyle = "rgba(15, 23, 42, 0.6)"; ctx.lineWidth = 1;
            for (let x = 0; x < canvas.width; x += 40) {
                ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, canvas.height); ctx.stroke();
            }
            for (let y = 0; y < canvas.height; y += 40) {
                ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
            }

            // Local de Saída
            const lx = sim.launchPad.x, ly = sim.launchPad.y;
            ctx.fillStyle = "#0f172a"; ctx.strokeStyle = "#06b6d4"; ctx.lineWidth = 2;
            ctx.beginPath(); ctx.ellipse(lx, ly + 15, 35, 12, 0, 0, Math.PI * 2); ctx.fill(); ctx.stroke();
            ctx.font = "bold 12px Consolas, monospace"; ctx.fillStyle = "#38bdf8"; ctx.textAlign = "center";
            ctx.fillText("H", lx, ly + 19);

            // Alvo Escolhido
            drawTargetObject(sim.targetPos.x, sim.targetPos.y);

            // Drone Observador
            ctx.save(); ctx.translate(sim.observerDrone.x, sim.observerDrone.y);
            ctx.fillStyle = "#0284c7"; ctx.beginPath(); ctx.arc(0, 0, 14, 0, Math.PI * 2); ctx.fill();
            ctx.strokeStyle = "#38bdf8"; ctx.lineWidth = 2; ctx.stroke();
            ctx.fillStyle = "rgba(56, 189, 248, 0.4)"; ctx.fillRect(-30, -2, 60, 4);
            ctx.restore();

            // Feixe de Mapeamento
            if (sim.missionState !== 'COMPLETED' || sim.missionType === 'land') {
                ctx.strokeStyle = "rgba(6, 182, 212, 0.25)"; ctx.setLineDash([4, 4]);
                ctx.beginPath(); ctx.moveTo(sim.observerDrone.x, sim.observerDrone.y + 10);
                ctx.lineTo(sim.drone.x, sim.drone.y - 15); ctx.stroke(); ctx.setLineDash([]);
            }

            // Drone Principal com Tripé e Armação Acoplados
            if (sim.missionState !== 'COMPLETED' || sim.missionType === 'land') {
                drawDroneInstance(sim.drone.x, sim.drone.y);
            }

            // Partículas da Explosão
            if (sim.explosion) {
                sim.explosion.particles.forEach(p => {
                    ctx.save(); ctx.fillStyle = p.color; ctx.globalAlpha = p.life;
                    ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2); ctx.fill();
                    ctx.restore();
                });
            }
        }

        function drawTargetObject(x, y) {
            ctx.save(); ctx.translate(x, y);
            if (sim.targetType === 'tank') {
                ctx.fillStyle = "#334155"; ctx.fillRect(-22, -10, 44, 20);
                ctx.fillStyle = "#0284c7"; ctx.fillRect(-10, -6, 20, 12);
                ctx.strokeStyle = "#38bdf8"; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(0, 0); ctx.lineTo(25, 0); ctx.stroke();
            } else if (sim.targetType === 'drone') {
                ctx.fillStyle = "#f43f5e"; ctx.beginPath(); ctx.arc(0, 0, 12, 0, Math.PI * 2); ctx.fill();
            } else if (sim.targetType === 'sniper') {
                ctx.fillStyle = "#1e293b"; ctx.fillRect(-25, -35, 50, 50);
                ctx.strokeStyle = "#8b5cf6"; ctx.lineWidth = 2; ctx.strokeRect(-25, -35, 50, 50);
            } else if (sim.targetType === 'artillery') {
                ctx.fillStyle = "#475569"; ctx.fillRect(-20, -12, 40, 24);
                ctx.strokeStyle = "#f97316"; ctx.lineWidth = 4;
                ctx.beginPath(); ctx.moveTo(-5, 0); ctx.lineTo(22, -15); ctx.stroke();
            } else if (sim.targetType === 'warship') {
                ctx.fillStyle = "#0f172a"; ctx.beginPath();
                ctx.moveTo(-35, -5); ctx.lineTo(25, -5); ctx.lineTo(38, 10); ctx.lineTo(-30, 10); ctx.closePath(); ctx.fill();
            } else if (sim.targetType === 'base') {
                ctx.fillStyle = "#064e3b"; ctx.fillRect(-30, -20, 60, 35);
                ctx.strokeStyle = "#10b981"; ctx.lineWidth = 2; ctx.strokeRect(-30, -20, 60, 35);
            }
            ctx.restore();
        }

        function drawDroneInstance(x, y) {
            ctx.save(); ctx.translate(x, y);

            ctx.strokeStyle = "#334155"; ctx.lineWidth = 5;
            ctx.beginPath();
            ctx.moveTo(-35, -25); ctx.lineTo(35, 25);
            ctx.moveTo(35, -25); ctx.lineTo(-35, 25);
            ctx.stroke();

            drawRotor(-35, -25, sim.drone.m1RPM);
            drawRotor(35, -25, sim.drone.m2RPM);
            drawRotor(-35, 25, sim.drone.m3RPM);
            drawRotor(35, 25, sim.drone.m4RPM);

            ctx.fillStyle = "#0f172a"; ctx.beginPath(); ctx.arc(0, 0, 18, 0, Math.PI * 2); ctx.fill();
            ctx.strokeStyle = "#06b6d4"; ctx.lineWidth = 2; ctx.stroke();

            ctx.strokeStyle = "#a855f7"; ctx.lineWidth = 1.5; ctx.strokeRect(-12, -10, 24, 20);

            ctx.save(); ctx.rotate((sim.tripod.panDeg * Math.PI) / 180);
            ctx.fillStyle = "#f59e0b"; ctx.fillRect(-6, 6, 12, 8);
            ctx.fillStyle = "#1e293b"; ctx.fillRect(-8, 14, 16, 10);
            ctx.fillStyle = "#f43f5e"; ctx.beginPath(); ctx.arc(0, 19, 3, 0, Math.PI * 2); ctx.fill();
            ctx.restore(); ctx.restore();
        }

        function drawRotor(px, py, rpm) {
            ctx.save(); ctx.translate(px, py);
            ctx.fillStyle = "#1e293b"; ctx.beginPath(); ctx.arc(0, 0, 5, 0, Math.PI * 2); ctx.fill();
            if (rpm > 0) {
                ctx.rotate(sim.drone.propAngle * (rpm / 1500));
                ctx.fillStyle = "rgba(6, 182, 212, 0.7)";
                ctx.beginPath(); ctx.ellipse(0, 0, 16, 4, 0, 0, Math.PI * 2); ctx.fill();
            }
            ctx.restore();
        }

        // WEBRTC SCREEN CAPTURE HANDLER
        const videoScreenSource = document.getElementById('videoScreenSource');
        const videoPlaceholder = document.getElementById('videoPlaceholder');
        const canvasScreenOutput = document.getElementById('canvasScreenOutput');
        const ctxScreen = canvasScreenOutput.getContext('2d');
        const btnStartScreenCapture = document.getElementById('btnStartScreenCapture');
        const btnProcessCanvas = document.getElementById('btnProcessCanvas');
        const captureStatusText = document.getElementById('captureStatusText');
        const canvasLatencyText = document.getElementById('canvasLatencyText');

        let isScreenProcessing = false;
        let lastFrameTime = performance.now();

        btnStartScreenCapture.addEventListener('click', async () => {
            try {
                const stream = await navigator.mediaDevices.getDisplayMedia({
                    video: { frameRate: { max: 60 } },
                    audio: false
                });
                videoScreenSource.srcObject = stream;
                videoPlaceholder.classList.add('hidden');
                captureStatusText.textContent = "TRANSMISSÃO WEBRTC ATIVA";
                captureStatusText.className = "text-emerald-400 font-bold";
            } catch (err) {
                alert("Erro ou cancelamento do compartilhamento de tela: " + err.message);
            }
        });

        btnProcessCanvas.addEventListener('click', () => {
            if (!videoScreenSource.srcObject) {
                alert("Primeiro inicie a captura de tela clicando em 'Capturar Tela Real'!");
                return;
            }
            canvasScreenOutput.width = videoScreenSource.videoWidth || 1280;
            canvasScreenOutput.height = videoScreenSource.videoHeight || 720;
            isScreenProcessing = true;
            processScreenCanvasLoop();
        });

        function processScreenCanvasLoop() {
            if (!isScreenProcessing) return;

            const now = performance.now();
            const deltaMs = (now - lastFrameTime).toFixed(2);
            lastFrameTime = now;

            ctxScreen.drawImage(videoScreenSource, 0, 0, canvasScreenOutput.width, canvasScreenOutput.height);

            // Retículo
            const cx = canvasScreenOutput.width / 2;
            const cy = canvasScreenOutput.height / 2;
            ctxScreen.strokeStyle = '#00ffcc'; ctxScreen.lineWidth = 2;
            ctxScreen.beginPath(); ctxScreen.arc(cx, cy, 50, 0, Math.PI * 2);
            ctxScreen.moveTo(cx - 70, cy); ctxScreen.lineTo(cx + 70, cy);
            ctxScreen.moveTo(cx, cy - 70); ctxScreen.lineTo(cx, cy + 70); ctxScreen.stroke();

            canvasLatencyText.textContent = `LATÊNCIA: ${deltaMs} ms`;
            requestAnimationFrame(processScreenCanvasLoop);
        }

        // ANIMATION LOOP
        function mainLoop() {
            updatePhysics();
            drawScene();
            requestAnimationFrame(mainLoop);
        }

        window.onload = function() {
            resizeCanvas();
            mainLoop();
        };
    </script>
</body>
</html>
