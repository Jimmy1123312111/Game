# Rocket Cars

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Rocket Arena 3D — DFH Stadium Sunset Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800;900&family=Orbitron:wght@600;700;800;900&display=swap" rel="stylesheet">
    <!-- Three.js r128 + Post-Processing Passes -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/EffectComposer.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/RenderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/ShaderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/CopyShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/LuminosityHighPassShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/UnrealBloomPass.js"></script>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
            -webkit-touch-callout: none;
        }
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #090d16;
            font-family: 'Inter', sans-serif;
        }
        .font-display {
            font-family: 'Orbitron', sans-serif;
        }
        #game-container {
            position: fixed;
            inset: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }
        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }
        /* Glassmorphic HUD & Overlays */
        .hud-glass {
            background: rgba(10, 15, 28, 0.72);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.14);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.45);
        }
        .btn-arcade {
            transition: transform 0.14s cubic-bezier(0.22, 1, 0.36, 1), box-shadow 0.14s ease, filter 0.14s ease;
        }
        .btn-arcade:hover {
            transform: translateY(-2px);
            filter: brightness(1.08);
        }
        .btn-arcade:active {
            transform: translateY(1px) scale(0.98);
        }
        @keyframes pulseGlow {
            0%, 100% { opacity: 0.92; transform: scale(1); }
            50% { opacity: 1; transform: scale(1.03); }
        }
        .animate-pulse-glow {
            animation: pulseGlow 2s infinite ease-in-out;
        }
        @keyframes bannerPop {
            0% { opacity: 0; transform: scale(0.6) translateY(16px); }
            18% { opacity: 1; transform: scale(1.06) translateY(0); }
            82% { opacity: 1; transform: scale(1) translateY(0); }
            100% { opacity: 0; transform: scale(0.95) translateY(-12px); }
        }
        .banner-animate {
            animation: bannerPop 2.4s cubic-bezier(0.16, 1, 0.3, 1) forwards;
        }
    </style>
</head>
<body class="text-white">
    <!-- 3D WebGL Canvas Container -->
    <div id="game-container"></div>

    <!-- Top HUD Bar (Scoreboard + Controls) -->
    <div id="hud-top" class="fixed top-0 left-0 right-0 z-20 pointer-events-none flex items-start justify-between p-3 sm:p-4 gap-2">
        <!-- Left Quick Status -->
        <div class="hidden md:flex items-center gap-2 pointer-events-auto">
            <div class="hud-glass rounded-xl px-3.5 py-2 flex items-center gap-2.5 text-xs font-semibold text-slate-200">
                <span class="w-2 h-2 rounded-full bg-orange-400 animate-ping"></span>
                <span id="hud-mode-label">DFH STADIUM • 1V1 SUNSET</span>
            </div>
        </div>

        <!-- Center Scoreboard -->
        <div class="mx-auto flex items-center hud-glass rounded-2xl overflow-hidden border border-white/15 shadow-2xl">
            <!-- Blue Team -->
            <div class="flex items-center gap-2.5 sm:gap-3.5 px-3.5 sm:px-5 py-2 bg-gradient-to-r from-blue-600/80 to-blue-500/40">
                <span class="text-xs sm:text-sm font-extrabold tracking-wider uppercase text-blue-100">BLUE</span>
                <span id="score-blue" class="font-display text-2xl sm:text-3xl font-black text-white drop-shadow">0</span>
            </div>
            <!-- Match Clock -->
            <div class="px-4 sm:px-6 py-1.5 bg-slate-950/85 border-x border-white/10 flex flex-col items-center justify-center min-w-[84px] sm:min-w-[104px]">
                <span id="match-timer" class="font-display text-lg sm:text-2xl font-extrabold tracking-wider text-amber-300">3:00</span>
                <span id="overtime-badge" class="hidden text-[10px] font-black tracking-widest uppercase text-rose-400">OVERTIME</span>
            </div>
            <!-- Orange Team -->
            <div class="flex items-center gap-2.5 sm:gap-3.5 px-3.5 sm:px-5 py-2 bg-gradient-to-l from-orange-600/80 to-orange-500/40">
                <span id="score-orange" class="font-display text-2xl sm:text-3xl font-black text-white drop-shadow">0</span>
                <span class="text-xs sm:text-sm font-extrabold tracking-wider uppercase text-orange-100">ORANGE</span>
            </div>
        </div>

        <!-- Right Utility Buttons -->
        <div class="flex items-center gap-1.5 sm:gap-2 pointer-events-auto">
            <button id="btn-cam" title="Toggle Ball Cam (C)" class="hud-glass btn-arcade rounded-xl px-2.5 sm:px-3.5 py-2 text-xs font-bold tracking-wide flex items-center gap-1.5 text-sky-300 hover:text-white">
                <span id="cam-dot" class="w-2 h-2 rounded-full bg-sky-400"></span>
                <span id="cam-label">BALL CAM</span>
            </button>
            <button id="btn-quality" title="Toggle Graphics Quality" class="hud-glass btn-arcade rounded-xl px-2.5 sm:px-3 py-2 text-xs font-bold tracking-wide flex items-center gap-1.5 text-amber-300 hover:text-white">
                <span id="quality-dot" class="w-2.5 h-2.5 rounded-full bg-amber-400 shadow-[0_0_8px_#fbbf24]"></span>
                <span id="quality-label" class="hidden sm:inline">ULTRA HD</span>
            </button>
            <button id="btn-mute" title="Mute / Unmute Audio (M)" class="hud-glass btn-arcade rounded-xl p-2 text-slate-200 hover:text-white" aria-label="Toggle Audio">
                <svg id="icon-sound-on" class="w-4 h-4 sm:w-5 sm:h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M11 5L6 9H2v6h4l5 4V5zM15.54 8.46a5 5 0 010 7.07M19.07 4.93a10 10 0 010 14.14"/>
                </svg>
                <svg id="icon-sound-off" class="w-4 h-4 sm:w-5 sm:h-5 hidden text-rose-400" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M11 5L6 9H2v6h4l5 4V5zM23 9l-6 6M17 9l6 6"/>
                </svg>
            </button>
            <button id="btn-pause" title="Pause Match (Esc)" class="hud-glass btn-arcade rounded-xl p-2 text-slate-200 hover:text-white" aria-label="Pause Game">
                <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M10 9v6m4-6v6m7-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
                </svg>
            </button>
        </div>
    </div>

    <!-- Center Event Banner (Countdown, GOAL!, Overtime) -->
    <div id="center-banner-wrap" class="fixed inset-0 z-20 pointer-events-none flex items-center justify-center">
        <div id="center-banner" class="hidden text-center px-6">
            <div id="banner-title" class="font-display text-5xl sm:text-7xl md:text-8xl font-black tracking-wider uppercase text-white drop-shadow-[0_6px_28px_rgba(251,146,60,0.85)]">3</div>
            <div id="banner-sub" class="mt-2 text-sm sm:text-xl font-extrabold tracking-widest uppercase text-amber-300 drop-shadow"></div>
        </div>
    </div>

    <!-- Bottom Right Circular Boost Gauge -->
    <div id="hud-boost" class="fixed bottom-4 right-4 sm:bottom-6 sm:right-6 z-20 pointer-events-none flex flex-col items-center">
        <div class="relative w-24 h-24 sm:w-28 sm:h-28 rounded-full hud-glass flex items-center justify-center shadow-2xl border-2 border-amber-400/30">
            <svg class="w-full h-full -rotate-90 transform" viewBox="0 0 100 100">
                <circle cx="50" cy="50" r="41" stroke="rgba(255,255,255,0.1)" stroke-width="8" fill="none" />
                <circle id="boost-arc" cx="50" cy="50" r="41" stroke="url(#boostGrad)" stroke-width="8" stroke-linecap="round" fill="none" stroke-dasharray="257.6" stroke-dashoffset="172" />
                <defs>
                    <linearGradient id="boostGrad" x1="0%" y1="0%" x2="100%" y2="0%">
                        <stop offset="0%" stop-color="#f59e0b" />
                        <stop offset="50%" stop-color="#f97316" />
                        <stop offset="100%" stop-color="#ef4444" />
                    </linearGradient>
                </defs>
            </svg>
            <div class="absolute inset-0 flex flex-col items-center justify-center">
                <span id="boost-value" class="font-display text-2xl sm:text-3xl font-black text-white leading-none">33</span>
                <span class="text-[9px] sm:text-[10px] font-extrabold tracking-widest uppercase text-amber-300 mt-1">BOOST</span>
            </div>
        </div>
    </div>

    <!-- Bottom Left Desktop Keybind Hint Pill -->
    <div id="hud-controls-hint" class="hidden lg:flex fixed bottom-5 left-5 z-20 pointer-events-none hud-glass rounded-2xl px-4 py-2.5 text-[11px] text-slate-300 gap-4 items-center">
        <span><strong class="text-white">WASD / Arrows</strong> Drive &amp; Pitch</span>
        <span><strong class="text-amber-300">Shift</strong> Boost</span>
        <span><strong class="text-sky-300">Space</strong> Jump / Dodge Flip</span>
        <span><strong class="text-purple-300">Q / E</strong> Air-Roll</span>
        <span><strong class="text-emerald-300">C</strong> Ball Cam</span>
    </div>

    <!-- Responsive Mobile / Touch Controls -->
    <div id="mobile-controls" class="lg:hidden fixed inset-x-0 bottom-3 z-20 pointer-events-none px-3 flex items-end justify-between">
        <!-- Left Steering & Air-Roll Pad -->
        <div class="flex flex-col gap-2 pointer-events-auto">
            <div class="flex gap-2">
                <button id="touch-roll-l" class="hud-glass active:bg-purple-500/40 rounded-xl px-3.5 py-2.5 text-xs font-extrabold text-purple-200 border border-purple-400/30">↺ ROLL</button>
                <button id="touch-roll-r" class="hud-glass active:bg-purple-500/40 rounded-xl px-3.5 py-2.5 text-xs font-extrabold text-purple-200 border border-purple-400/30">ROLL ↻</button>
            </div>
            <div class="flex gap-2">
                <button id="touch-left" class="hud-glass active:bg-white/25 rounded-2xl w-16 h-16 flex items-center justify-center text-2xl font-black text-white">◀</button>
                <button id="touch-right" class="hud-glass active:bg-white/25 rounded-2xl w-16 h-16 flex items-center justify-center text-2xl font-black text-white">▶</button>
            </div>
        </div>

        <!-- Right Gas, Brake, Jump & Boost -->
        <div class="flex flex-col items-end gap-2 pointer-events-auto mr-28 sm:mr-32">
            <div class="flex gap-2">
                <button id="touch-jump" class="hud-glass active:bg-sky-500/40 rounded-2xl px-4 py-3 text-xs font-black tracking-wider text-sky-200 border border-sky-400/40">JUMP</button>
                <button id="touch-boost" class="hud-glass active:bg-amber-500/50 rounded-2xl px-4 py-3 text-xs font-black tracking-wider text-amber-200 border border-amber-400/40">BOOST</button>
            </div>
            <div class="flex gap-2">
                <button id="touch-brake" class="hud-glass active:bg-rose-500/40 rounded-2xl w-14 h-14 flex items-center justify-center text-xs font-extrabold text-rose-200">REV</button>
                <button id="touch-gas" class="hud-glass active:bg-emerald-500/40 rounded-2xl w-16 h-14 flex items-center justify-center text-xs font-extrabold text-emerald-200 border border-emerald-400/40">GAS</button>
            </div>
        </div>
    </div>

    <!-- Main Start Menu Overlay -->
    <div id="menu-overlay" class="fixed inset-0 z-30 flex items-center justify-center p-4 bg-slate-950/65 backdrop-blur-md">
        <div class="hud-glass rounded-3xl max-w-xl w-full p-6 sm:p-8 border border-white/15 text-center shadow-2xl">
            <div class="inline-flex items-center gap-2 px-3.5 py-1 rounded-full bg-amber-500/15 border border-amber-400/30 text-amber-300 text-xs font-extrabold tracking-widest uppercase mb-3">
                <span>DFH STADIUM • SUNSET ULTRA HD</span>
            </div>
            <h1 class="font-display text-3xl sm:text-5xl font-black tracking-tight bg-gradient-to-r from-sky-400 via-white to-orange-400 bg-clip-text text-transparent">
                ROCKET ARENA 3D
            </h1>
            <p class="mt-2.5 text-sm sm:text-base text-slate-300 max-w-md mx-auto">
                High-octane supersonic vehicular soccer. Ride the curved netted walls, master aerial air-rolls, and blast the ball into the net!
            </p>

            <!-- Match Settings -->
            <div class="mt-6 space-y-3 text-left">
                <div class="bg-slate-900/75 rounded-2xl p-3.5 border border-white/10">
                    <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-2">Team Mode (Rocket League Spawns)</label>
                    <div class="grid grid-cols-3 gap-2" id="mode-selector">
                        <button data-size="1" class="mode-btn rounded-xl py-2 text-xs font-extrabold tracking-wide bg-gradient-to-r from-indigo-600 to-blue-500 text-white shadow">1v1 DUEL</button>
                        <button data-size="2" class="mode-btn rounded-xl py-2 text-xs font-extrabold tracking-wide bg-slate-800 text-slate-300 hover:bg-slate-700">2v2 DOUBLES</button>
                        <button data-size="3" class="mode-btn rounded-xl py-2 text-xs font-extrabold tracking-wide bg-slate-800 text-slate-300 hover:bg-slate-700">3v3 STANDARD</button>
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div class="bg-slate-900/75 rounded-2xl p-3.5 border border-white/10">
                        <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-2">AI Difficulty</label>
                        <div class="grid grid-cols-3 gap-1.5" id="diff-selector">
                            <button data-diff="rookie" class="diff-btn rounded-xl py-2 text-xs font-bold bg-slate-800 text-slate-300 hover:bg-slate-700">Rookie</button>
                            <button data-diff="pro" class="diff-btn rounded-xl py-2 text-xs font-bold bg-gradient-to-r from-blue-600 to-sky-500 text-white shadow">Pro</button>
                            <button data-diff="allstar" class="diff-btn rounded-xl py-2 text-xs font-bold bg-slate-800 text-slate-300 hover:bg-slate-700">All-Star</button>
                        </div>
                    </div>

                    <div class="bg-slate-900/75 rounded-2xl p-3.5 border border-white/10">
                        <label class="block text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-2">Match Duration</label>
                        <div class="grid grid-cols-3 gap-1.5" id="time-selector">
                            <button data-time="90" class="time-btn rounded-xl py-2 text-xs font-bold bg-slate-800 text-slate-300 hover:bg-slate-700">1:30</button>
                            <button data-time="180" class="time-btn rounded-xl py-2 text-xs font-bold bg-gradient-to-r from-orange-500 to-amber-500 text-white shadow">3:00</button>
                            <button data-time="300" class="time-btn rounded-xl py-2 text-xs font-bold bg-slate-800 text-slate-300 hover:bg-slate-700">5:00</button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Controls Summary Grid -->
            <div class="mt-4 bg-slate-900/60 rounded-2xl p-3.5 border border-white/10 grid grid-cols-2 sm:grid-cols-3 gap-2 text-xs text-slate-300">
                <div><span class="font-bold text-white">W A S D</span> : Drive / Pitch</div>
                <div><span class="font-bold text-amber-300">Shift</span> : Rocket Boost</div>
                <div><span class="font-bold text-sky-300">Space</span> : Jump / Dodge</div>
                <div><span class="font-bold text-purple-300">Q / E</span> : Air-Roll Left/Right</div>
                <div><span class="font-bold text-pink-300">Ctrl</span> : Powerslide / Roll</div>
                <div><span class="font-bold text-emerald-300">C / Tab</span> : Ball Cam</div>
            </div>

            <button id="btn-play" class="mt-6 w-full btn-arcade animate-pulse-glow rounded-2xl py-4 px-6 font-display text-lg sm:text-xl font-black tracking-wider uppercase bg-gradient-to-r from-blue-600 via-indigo-500 to-orange-500 text-white shadow-[0_10px_30px_rgba(249,115,22,0.4)]">
                KICKOFF MATCH
            </button>
        </div>
    </div>

    <!-- Pause Overlay -->
    <div id="pause-overlay" class="hidden fixed inset-0 z-30 flex items-center justify-center p-4 bg-slate-950/70 backdrop-blur-md">
        <div class="hud-glass rounded-3xl max-w-sm w-full p-6 border border-white/15 text-center shadow-2xl">
            <h2 class="font-display text-3xl font-black tracking-wider text-white">MATCH PAUSED</h2>
            <p class="mt-1.5 text-sm text-slate-300">DFH Stadium • Sunset</p>
            <div class="mt-6 flex flex-col gap-3">
                <button id="btn-resume" class="btn-arcade rounded-2xl py-3.5 font-display font-bold tracking-wider uppercase bg-gradient-to-r from-sky-500 to-blue-600 text-white">
                    RESUME MATCH
                </button>
                <button id="btn-reset-kickoff" class="btn-arcade rounded-2xl py-3 font-bold text-sm bg-slate-800 hover:bg-slate-700 text-slate-200 border border-white/10">
                    RESET KICKOFF POSITION
                </button>
                <button id="btn-quit-menu" class="btn-arcade rounded-2xl py-3 font-bold text-sm bg-rose-600/80 hover:bg-rose-600 text-white">
                    RETURN TO MENU
                </button>
            </div>
        </div>
    </div>

    <!-- Post-Match Summary Overlay -->
    <div id="gameover-overlay" class="hidden fixed inset-0 z-30 flex items-center justify-center p-4 bg-slate-950/75 backdrop-blur-md">
        <div class="hud-glass rounded-3xl max-w-md w-full p-7 border border-white/15 text-center shadow-2xl">
            <div id="winner-badge" class="inline-block px-4 py-1 rounded-full text-xs font-black tracking-widest uppercase bg-blue-500/20 text-blue-300 border border-blue-400/30 mb-2">
                VICTORY
            </div>
            <h2 id="winner-title" class="font-display text-3xl sm:text-4xl font-black tracking-wide text-white">BLUE TEAM WINS!</h2>
            <div class="mt-4 flex items-center justify-center gap-6 bg-slate-900/80 rounded-2xl py-4 px-6 border border-white/10">
                <div>
                    <div class="text-xs font-bold uppercase text-blue-400">Blue</div>
                    <div id="final-blue" class="font-display text-4xl font-black text-white">0</div>
                </div>
                <div class="font-display text-xl font-bold text-slate-500">VS</div>
                <div>
                    <div class="text-xs font-bold uppercase text-orange-400">Orange</div>
                    <div id="final-orange" class="font-display text-4xl font-black text-white">0</div>
                </div>
            </div>
            <div class="mt-6 flex flex-col sm:flex-row gap-3">
                <button id="btn-rematch" class="flex-1 btn-arcade rounded-2xl py-3.5 font-display font-bold text-sm tracking-wider uppercase bg-gradient-to-r from-blue-600 to-orange-500 text-white">
                    PLAY AGAIN
                </button>
                <button id="btn-menu-end" class="flex-1 btn-arcade rounded-2xl py-3.5 font-bold text-sm bg-slate-800 hover:bg-slate-700 text-slate-200 border border-white/10">
                    MAIN MENU
                </button>
            </div>
        </div>
    </div>

    <script>
    (() => {
        'use strict';

        // =========================================================================
        // 1. ARENA DIMENSIONS (2X SCALE) & ROCKET LEAGUE PHYSICS CONSTANTS
        // =========================================================================
        const ARENA = {
            HALF_W: 108,         // 2x Sideline X boundary
            HALF_L: 164,         // 2x Endline Z boundary
            CHAMFER: 48,         // 2x 45-degree corner chamfer size
            WALL_H: 36,          // Scaled Arena ceiling height
            RAMP_R: 9.0,         // Floor-to-wall quarter-pipe radius
            TOP_RAMP_R: 8.0,     // Wall-to-ceiling quarter-pipe radius
            GOAL_W: 28,          // Goal mouth width (-14 to +14)
            GOAL_H: 12.5,        // Goal crossbar height
            GOAL_D: 13.0         // Recessed goal depth
        };

        const PHYSICS = {
            GRAVITY: -36.0,
            CAR_MAX_SPEED: 42.0,
            CAR_BOOST_MAX_SPEED: 56.0,
            CAR_ACCEL: 54.0,
            CAR_BRAKE: 68.0,
            CAR_BOOST_ACCEL: 48.0,
            CAR_TURN_SPEED: 2.55,
            CAR_DRIFT_TURN_MULT: 1.48,
            CAR_JUMP_IMPULSE: 16.5,
            CAR_DODGE_IMPULSE: 21.0,
            CAR_AIR_PITCH_SPEED: 3.1,
            CAR_AIR_YAW_SPEED: 2.6,
            CAR_AIR_ROLL_SPEED: 4.1,
            WALL_ADHESION_FORCE: 26.0,

            // Paced Ball Physics (Half-speed feel + Zero unwanted ground pop-up)
            BALL_RADIUS: 2.35,
            BALL_GRAVITY: -26.0,
            BALL_MAX_SPEED: 45.0,
            BALL_BOUNCE: 0.62,
            BALL_GROUND_FRICTION: 0.992,
            BALL_AIR_DRAG: 0.997,
            BALL_HIT_MULT: 1.18,
            BALL_DODGE_HIT_BONUS: 1.28
        };

        // =========================================================================
        // 2. GAME STATE & INPUT TRACKING
        // =========================================================================
        const state = {
            mode: 'menu',        // 'menu' | 'countdown' | 'playing' | 'goal' | 'paused' | 'gameover'
            prevMode: 'playing',
            teamSize: 1,         // 1 (1v1), 2 (2v2), or 3 (3v3)
            kickoffIndex: 0,
            difficulty: 'pro',
            matchDuration: 180,
            timeLeft: 180,
            overtime: false,
            scoreBlue: 0,
            scoreOrange: 0,
            countdownTimer: 0,
            goalPauseTimer: 0,
            ballCam: true,
            ultraHD: true,
            muted: false,
            crowdCheerTimer: 0
        };

        const keys = {
            forward: false,
            backward: false,
            left: false,
            right: false,
            boost: false,
            jump: false,
            drift: false,
            rollLeft: false,
            rollRight: false
        };

        let jumpPressedEdge = false;
        let lastCountdownSec = 4;

        // =========================================================================
        // 3. PROCEDURAL WEB AUDIO SYNTHESIZER (ZERO EXTERNAL URLS, ZERO LAG)
        // =========================================================================
        class SoundSynthesizer {
            constructor() {
                this.ctx = null;
                this.engineOsc = null;
                this.engineGain = null;
                this.boostGain = null;
            }

            init() {
                if (this.ctx) {
                    if (this.ctx.state === 'suspended') this.ctx.resume();
                    return;
                }
                const AudioCtx = window.AudioContext || window.webkitAudioContext;
                if (!AudioCtx) return;
                this.ctx = new AudioCtx();

                // Continuous subtle engine hum
                this.engineOsc = this.ctx.createOscillator();
                this.engineGain = this.ctx.createGain();
                this.engineOsc.type = 'sawtooth';
                this.engineOsc.frequency.value = 55;
                this.engineGain.gain.value = 0.0;

                const engineFilter = this.ctx.createBiquadFilter();
                engineFilter.type = 'lowpass';
                engineFilter.frequency.value = 210;

                this.engineOsc.connect(engineFilter);
                engineFilter.connect(this.engineGain);
                this.engineGain.connect(this.ctx.destination);
                this.engineOsc.start();

                // Continuous filtered noise for rocket boost
                const bufferSize = this.ctx.sampleRate * 1.5;
                const noiseBuffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = noiseBuffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;

                const noise = this.ctx.createBufferSource();
                noise.buffer = noiseBuffer;
                noise.loop = true;

                const boostFilter = this.ctx.createBiquadFilter();
                boostFilter.type = 'bandpass';
                boostFilter.frequency.value = 420;
                boostFilter.Q.value = 1.4;

                this.boostGain = this.ctx.createGain();
                this.boostGain.gain.value = 0;

                noise.connect(boostFilter);
                boostFilter.connect(this.boostGain);
                this.boostGain.connect(this.ctx.destination);
                noise.start();
            }

            updateContinuous(speedRatio, isBoosting, active) {
                if (!this.ctx || !this.engineGain || !this.boostGain) return;
                const now = this.ctx.currentTime;
                if (state.muted || !active) {
                    this.engineGain.gain.setTargetAtTime(0, now, 0.06);
                    this.boostGain.gain.setTargetAtTime(0, now, 0.06);
                    return;
                }
                this.engineOsc.frequency.setTargetAtTime(48 + speedRatio * 75, now, 0.08);
                this.engineGain.gain.setTargetAtTime(0.022 + speedRatio * 0.024, now, 0.08);
                this.boostGain.gain.setTargetAtTime(isBoosting ? 0.14 : 0.0, now, 0.05);
            }

            playTone(freq, type, duration, vol = 0.14, slideFreq = null, delay = 0) {
                if (state.muted) return;
                this.init();
                if (!this.ctx) return;
                const now = this.ctx.currentTime + delay;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = type;
                osc.frequency.setValueAtTime(freq, now);
                if (slideFreq) {
                    osc.frequency.exponentialRampToValueAtTime(Math.max(20, slideFreq), now + duration);
                }
                gain.gain.setValueAtTime(vol, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + duration);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + duration);
            }

            playJump(isDodge = false) {
                if (isDodge) {
                    this.playTone(185, 'sawtooth', 0.22, 0.13, 460);
                } else {
                    this.playTone(135, 'triangle', 0.16, 0.13, 310);
                }
            }

            playBallHit(intensity = 0.6) {
                const vol = Math.min(0.26, 0.07 + intensity * 0.18);
                this.playTone(108 + intensity * 65, 'sine', 0.18, vol, 44);
                this.playTone(240, 'triangle', 0.08, vol * 0.55, 85);
            }

            playBoostPickup(isFull = false) {
                if (isFull) {
                    this.playTone(440, 'sine', 0.24, 0.14, 880, 0);
                    this.playTone(660, 'triangle', 0.24, 0.11, 1120, 0.065);
                } else {
                    this.playTone(520, 'sine', 0.14, 0.08, 760, 0);
                }
            }

            playCountdown(go = false) {
                if (go) {
                    this.playTone(587.33, 'triangle', 0.36, 0.18, 880);
                } else {
                    this.playTone(440, 'sine', 0.18, 0.14);
                }
            }

            playGoal() {
                this.playTone(110, 'sawtooth', 0.65, 0.22, 42);
                this.playTone(220, 'square', 0.45, 0.12, 330, 0.05);
                this.playTone(330, 'triangle', 0.55, 0.14, 440, 0.15);
            }
        }

        const sfx = new SoundSynthesizer();

        // =========================================================================
        // 4. THREE.JS SCENE, RENDERER, SUNSET HDR LIGHTING & POST-PROCESSING
        // =========================================================================
        const container = document.getElementById('game-container');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x2a1b2e, 0.0012);

        const camera = new THREE.PerspectiveCamera(62, window.innerWidth / window.innerHeight, 0.5, 1650);

        const renderer = new THREE.WebGLRenderer({
            antialias: true,
            powerPreference: 'high-performance'
        });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        renderer.outputEncoding = THREE.sRGBEncoding;
        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.toneMappingExposure = 0.94;
        renderer.physicallyCorrectLights = true;
        container.appendChild(renderer.domElement);

        // Optional HDR Bloom Post-Processing Composer
        let composer = null;
        try {
            if (THREE.EffectComposer && THREE.RenderPass && THREE.UnrealBloomPass) {
                composer = new THREE.EffectComposer(renderer);
                const renderPass = new THREE.RenderPass(scene, camera);
                composer.addPass(renderPass);

                const bloomPass = new THREE.UnrealBloomPass(
                    new THREE.Vector2(window.innerWidth, window.innerHeight),
                    0.34, // strength
                    0.55, // radius
                    0.86  // threshold
                );
                composer.addPass(bloomPass);
            }
        } catch (err) {
            composer = null;
        }

        // Sunset Sky Dome & Real-Time PBR Environment Map
        function buildSunsetSkyDome() {
            const skyCanvas = document.createElement('canvas');
            skyCanvas.width = 512;
            skyCanvas.height = 256;
            const sctx = skyCanvas.getContext('2d');
            const grad = sctx.createLinearGradient(0, 0, 0, 256);
            grad.addColorStop(0.00, '#1e293b');
            grad.addColorStop(0.28, '#312e81');
            grad.addColorStop(0.46, '#7e22ce');
            grad.addColorStop(0.55, '#f97316');
            grad.addColorStop(0.64, '#fdba74');
            grad.addColorStop(1.00, '#334155');
            sctx.fillStyle = grad;
            sctx.fillRect(0, 0, 512, 256);

            // Setting sun disc on the horizon
            const sunGrad = sctx.createRadialGradient(355, 142, 4, 355, 142, 62);
            sunGrad.addColorStop(0.0, 'rgba(255, 251, 235, 1.0)');
            sunGrad.addColorStop(0.25, 'rgba(253, 186, 116, 0.85)');
            sunGrad.addColorStop(0.65, 'rgba(249, 115, 22, 0.35)');
            sunGrad.addColorStop(1.0, 'rgba(249, 115, 22, 0.0)');
            sctx.fillStyle = sunGrad;
            sctx.fillRect(280, 75, 150, 140);

            const skyTex = new THREE.CanvasTexture(skyCanvas);
            skyTex.encoding = THREE.sRGBEncoding;

            const skyDome = new THREE.Mesh(
                new THREE.SphereGeometry(780, 32, 24),
                new THREE.MeshBasicMaterial({ map: skyTex, side: THREE.BackSide, fog: false })
            );
            scene.add(skyDome);

            // Generate PMREM Environment Reflections for Metallic Cars, Ball & Walls
            try {
                const pmrem = new THREE.PMREMGenerator(renderer);
                pmrem.compileEquirectangularShader();
                const envScene = new THREE.Scene();
                envScene.add(new THREE.Mesh(
                    new THREE.SphereGeometry(60, 24, 16),
                    new THREE.MeshBasicMaterial({ map: skyTex, side: THREE.BackSide })
                ));
                // Overhead stadium floodlight strips in reflection map
                const lightBarMat = new THREE.MeshBasicMaterial({ color: 0xfff7ed });
                for (let s = -1; s <= 1; s += 2) {
                    const bar = new THREE.Mesh(new THREE.BoxGeometry(8, 2, 70), lightBarMat);
                    bar.position.set(s * 34, 32, 0);
                    envScene.add(bar);
                }
                const envRT = pmrem.fromScene(envScene, 0.035);
                scene.environment = envRT.texture;
                pmrem.dispose();
            } catch (e) {
                // Graceful fallback if PMREM isn't supported
            }
        }

        buildSunsetSkyDome();

        // Mid-Twilight Sunset Lighting Rig (Balanced Moody Atmosphere + Crisp Car/Ball Readability)
        const hemiLight = new THREE.HemisphereLight(0xdbeafe, 0x334155, 0.95);
        scene.add(hemiLight);

        const sunLight = new THREE.DirectionalLight(0xff944d, 2.25);
        sunLight.position.set(195, 115, -240);
        sunLight.castShadow = true;
        sunLight.shadow.mapSize.set(4096, 4096);
        sunLight.shadow.camera.near = 30;
        sunLight.shadow.camera.far = 560;
        sunLight.shadow.camera.left = -190;
        sunLight.shadow.camera.right = 190;
        sunLight.shadow.camera.top = 210;
        sunLight.shadow.camera.bottom = -210;
        sunLight.shadow.bias = -0.0004;
        sunLight.shadow.normalBias = 0.025;
        scene.add(sunLight);

        // Cool Twilight Rim Light for Crisp Metallic Separation
        const rimLight = new THREE.DirectionalLight(0x7dd3fc, 1.05);
        rimLight.position.set(-180, 110, 210);
        scene.add(rimLight);

        // =========================================================================
        // 5. PROCEDURAL PBR TEXTURES (CROSS-MOWED TURF, METAL NETTING, FLAKE)
        // =========================================================================
        function createTurfTextures() {
            const w = 1024;
            const h = 1536;
            const canvas = document.createElement('canvas');
            canvas.width = w;
            canvas.height = h;
            const ctx = canvas.getContext('2d');

            // Cross-mowed emerald turf stripes
            const stripeRows = 24;
            const stripeCols = 14;
            const rowH = h / stripeRows;
            const colW = w / stripeCols;

            for (let r = 0; r < stripeRows; r++) {
                for (let c = 0; c < stripeCols; c++) {
                    const even = (r + c) % 2 === 0;
                    ctx.fillStyle = even ? '#1e6b34' : '#257c3e';
                    ctx.fillRect(c * colW, r * rowH, colW + 1, rowH + 1);
                }
            }

            // Fine grass blade micro-texture
            for (let i = 0; i < 28000; i++) {
                const gx = Math.random() * w;
                const gy = Math.random() * h;
                ctx.fillStyle = Math.random() > 0.5 ? 'rgba(134, 239, 172, 0.06)' : 'rgba(6, 40, 18, 0.08)';
                ctx.fillRect(gx, gy, 2, 4);
            }

            // Crisp White DFH Stadium Soccer Pitch Markings
            ctx.strokeStyle = 'rgba(248, 250, 252, 0.88)';
            ctx.lineWidth = 7;
            const padX = 56;
            const padY = 56;
            const pitchW = w - padX * 2;
            const pitchH = h - padY * 2;

            // Outer boundary & halfway line
            ctx.strokeRect(padX, padY, pitchW, pitchH);
            ctx.beginPath();
            ctx.moveTo(padX, h / 2);
            ctx.lineTo(w - padX, h / 2);
            ctx.stroke();

            // Center circle & kickoff spot
            ctx.beginPath();
            ctx.arc(w / 2, h / 2, 120, 0, Math.PI * 2);
            ctx.stroke();

            ctx.fillStyle = 'rgba(248, 250, 252, 0.9)';
            ctx.beginPath();
            ctx.arc(w / 2, h / 2, 9, 0, Math.PI * 2);
            ctx.fill();

            // Penalty boxes & 6-yard boxes (Top = Orange end, Bottom = Blue end)
            const boxW = 420;
            const boxH = 195;
            const sixW = 210;
            const sixH = 80;
            ctx.strokeRect((w - boxW) / 2, padY, boxW, boxH);
            ctx.strokeRect((w - sixW) / 2, padY, sixW, sixH);
            ctx.strokeRect((w - boxW) / 2, h - padY - boxH, boxW, boxH);
            ctx.strokeRect((w - sixW) / 2, h - padY - sixH, sixW, sixH);

            // Team Half Subtle Accent Rings
            ctx.strokeStyle = 'rgba(249, 115, 22, 0.45)';
            ctx.beginPath();
            ctx.arc(w / 2, padY + boxH, 70, 0, Math.PI);
            ctx.stroke();

            ctx.strokeStyle = 'rgba(56, 189, 248, 0.45)';
            ctx.beginPath();
            ctx.arc(w / 2, h - padY - boxH, 70, Math.PI, Math.PI * 2);
            ctx.stroke();

            const map = new THREE.CanvasTexture(canvas);
            map.encoding = THREE.sRGBEncoding;
            map.anisotropy = 8;

            // Bump/roughness texture for micro-grass sheen
            const bumpCanvas = document.createElement('canvas');
            bumpCanvas.width = 256;
            bumpCanvas.height = 256;
            const bctx = bumpCanvas.getContext('2d');
            bctx.fillStyle = '#808080';
            bctx.fillRect(0, 0, 256, 256);
            for (let i = 0; i < 6000; i++) {
                const v = 100 + Math.floor(Math.random() * 60);
                bctx.fillStyle = `rgb(${v},${v},${v})`;
                bctx.fillRect(Math.random() * 256, Math.random() * 256, 1.5, 3);
            }
            const bumpMap = new THREE.CanvasTexture(bumpCanvas);
            bumpMap.wrapS = bumpMap.wrapT = THREE.RepeatWrapping;
            bumpMap.repeat.set(48, 72);

            return { map, bumpMap };
        }

        function createMetalNettingTextures() {
            const size = 256;
            const canvas = document.createElement('canvas');
            canvas.width = size;
            canvas.height = size;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, size, size);

            // Hexagonal / Diamond Steel Wire Mesh Pattern
            ctx.strokeStyle = '#f8fafc';
            ctx.lineWidth = 4.5;
            const step = 32;
            for (let y = -step; y <= size + step; y += step) {
                for (let x = -step; x <= size + step; x += step) {
                    ctx.beginPath();
                    ctx.moveTo(x + step * 0.5, y);
                    ctx.lineTo(x + step, y + step * 0.5);
                    ctx.lineTo(x + step * 0.5, y + step);
                    ctx.lineTo(x, y + step * 0.5);
                    ctx.closePath();
                    ctx.stroke();
                }
            }

            const map = new THREE.CanvasTexture(canvas);
            map.wrapS = map.wrapT = THREE.RepeatWrapping;
            map.anisotropy = 8;
            return { map };
        }

        function createFlakeNormalMap() {
            const size = 128;
            const canvas = document.createElement('canvas');
            canvas.width = size;
            canvas.height = size;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = 'rgb(128, 128, 255)';
            ctx.fillRect(0, 0, size, size);
            for (let i = 0; i < 1600; i++) {
                const r = 118 + Math.floor(Math.random() * 20);
                const g = 118 + Math.floor(Math.random() * 20);
                ctx.fillStyle = `rgb(${r},${g},255)`;
                ctx.fillRect(Math.random() * size, Math.random() * size, 1.5, 1.5);
            }
            const tex = new THREE.CanvasTexture(canvas);
            tex.wrapS = tex.wrapT = THREE.RepeatWrapping;
            tex.repeat.set(16, 16);
            return tex;
        }

        const flakeNormalMap = createFlakeNormalMap();

        // =========================================================================
        // 6. DFH STADIUM BUILDER (2X SCALE RAMPED NETTED WALLS, GRANDSTANDS, GOALS)
        // =========================================================================
        let blueGoalLight = null;
        let orangeGoalLight = null;
        let crowdInstancedMesh = null;
        const crowdBaseMatrices = [];
        const crowdPhases = [];

        function buildDFHStadium() {
            const stadiumGroup = new THREE.Group();
            const turfTextures = createTurfTextures();
            const netTextures = createMetalNettingTextures();

            // 1. Main Soccer Pitch Turf
            const pitchGeo = new THREE.PlaneGeometry(ARENA.HALF_W * 2, ARENA.HALF_L * 2);
            pitchGeo.rotateX(-Math.PI / 2);
            const pitchMat = new THREE.MeshStandardMaterial({
                map: turfTextures.map,
                bumpMap: turfTextures.bumpMap,
                bumpScale: 0.08,
                roughness: 0.78,
                metalness: 0.04
            });
            const pitch = new THREE.Mesh(pitchGeo, pitchMat);
            pitch.receiveShadow = true;
            stadiumGroup.add(pitch);

            // Outer Stadium Grass Apron
            const apronGeo = new THREE.PlaneGeometry(720, 840);
            apronGeo.rotateX(-Math.PI / 2);
            const apronMat = new THREE.MeshStandardMaterial({ color: 0x113a1d, roughness: 0.88 });
            const apron = new THREE.Mesh(apronGeo, apronMat);
            apron.position.y = -0.08;
            apron.receiveShadow = true;
            stadiumGroup.add(apron);

            // 2. Shared Structural Steel, Ramp & Metal Netting Materials
            const darkSteelMat = new THREE.MeshStandardMaterial({
                color: 0x475569,
                metalness: 0.65,
                roughness: 0.30
            });
            const chromeTrimMat = new THREE.MeshStandardMaterial({
                color: 0xe2e8f0,
                metalness: 0.85,
                roughness: 0.16
            });
            const rampMetalMat = new THREE.MeshPhysicalMaterial({
                color: 0x64748b,
                metalness: 0.55,
                roughness: 0.28,
                clearcoat: 0.55,
                clearcoatRoughness: 0.22
            });
            const metalNetMat = new THREE.MeshPhysicalMaterial({
                color: 0xcbd5e1,
                emissive: 0x1e293b,
                emissiveIntensity: 0.16,
                map: netTextures.map,
                alphaMap: netTextures.map,
                transparent: true,
                opacity: 0.72,
                metalness: 0.55,
                roughness: 0.24,
                side: THREE.DoubleSide,
                depthWrite: false
            });

            // Helper to build a ramped wall segment with metal netting, steel pillars & neon rail
            function buildWallSegment(x1, z1, x2, z2, teamTintHex = 0x38bdf8, trimEnds = 0) {
                const dx = x2 - x1;
                const dz = z2 - z1;
                const fullLen = Math.hypot(dx, dz);
                const segLen = Math.max(2, fullLen - trimEnds * 2);
                const cx = (x1 + x2) * 0.5;
                const cz = (z1 + z2) * 0.5;
                const angleY = Math.atan2(-dz, dx);

                const segGroup = new THREE.Group();
                segGroup.position.set(cx, 0, cz);
                segGroup.rotation.y = angleY;

                // Lower Curved Quarter-Pipe Transition Ramp (faces inward toward +Z local)
                const R = ARENA.RAMP_R;
                const botShape = new THREE.Shape();
                botShape.moveTo(0, 0);
                botShape.lineTo(R, 0);
                botShape.absarc(R, R, R, -Math.PI / 2, Math.PI, true);
                botShape.lineTo(0, 0);

                const botGeo = new THREE.ExtrudeGeometry(botShape, {
                    depth: segLen,
                    bevelEnabled: false,
                    curveSegments: 14
                });
                botGeo.translate(0, 0, -segLen * 0.5);
                botGeo.rotateY(Math.PI / 2);

                const botRamp = new THREE.Mesh(botGeo, rampMetalMat);
                botRamp.receiveShadow = true;
                segGroup.add(botRamp);

                // Glowing Trim Rail along Ramp Top Lip
                const railMat = new THREE.MeshStandardMaterial({
                    color: teamTintHex,
                    emissive: teamTintHex,
                    emissiveIntensity: 1.6,
                    roughness: 0.2
                });
                const rail = new THREE.Mesh(new THREE.BoxGeometry(segLen, 0.32, 0.34), railMat);
                rail.position.set(0, R, 0.16);
                segGroup.add(rail);

                // Upper Wall-to-Ceiling Curved Ramp
                const TR = ARENA.TOP_RAMP_R;
                const topShape = new THREE.Shape();
                topShape.moveTo(0, ARENA.WALL_H);
                topShape.lineTo(0, ARENA.WALL_H - TR);
                topShape.absarc(TR, ARENA.WALL_H - TR, TR, Math.PI, Math.PI / 2, true);
                topShape.lineTo(0, ARENA.WALL_H);

                const topGeo = new THREE.ExtrudeGeometry(topShape, {
                    depth: segLen,
                    bevelEnabled: false,
                    curveSegments: 14
                });
                topGeo.translate(0, 0, -segLen * 0.5);
                topGeo.rotateY(Math.PI / 2);

                const topRamp = new THREE.Mesh(topGeo, rampMetalMat);
                segGroup.add(topRamp);

                // Metal Netting Panel between Lower & Upper Ramps
                const netH = ARENA.WALL_H - R - TR;
                const segNetTex = netTextures.map.clone();
                segNetTex.needsUpdate = true;
                segNetTex.repeat.set(Math.max(2, Math.round(segLen / 3.6)), Math.max(2, Math.round(netH / 3.6)));

                const segNetMat = metalNetMat.clone();
                segNetMat.map = segNetTex;
                segNetMat.alphaMap = segNetTex;

                const netPanel = new THREE.Mesh(new THREE.PlaneGeometry(segLen, netH), segNetMat);
                netPanel.position.set(0, R + netH * 0.5, 0.05);
                segGroup.add(netPanel);

                // Horizontal Structural Mid-Beam
                const midBeam = new THREE.Mesh(new THREE.BoxGeometry(segLen, 0.52, 0.55), darkSteelMat);
                midBeam.position.set(0, R + netH * 0.5, -0.22);
                segGroup.add(midBeam);

                // Vertical Structural Steel Pillars
                const numPillars = Math.max(2, Math.round(segLen / 18) + 1);
                for (let p = 0; p < numPillars; p++) {
                    const px = -segLen * 0.5 + (p / (numPillars - 1)) * segLen;
                    const pillar = new THREE.Mesh(new THREE.BoxGeometry(0.85, ARENA.WALL_H, 0.95), darkSteelMat);
                    pillar.position.set(px, ARENA.WALL_H * 0.5, -0.42);
                    pillar.castShadow = true;
                    segGroup.add(pillar);

                    const chromeStrip = new THREE.Mesh(new THREE.BoxGeometry(0.30, ARENA.WALL_H - R - TR, 0.18), chromeTrimMat);
                    chromeStrip.position.set(px, R + netH * 0.5, 0.02);
                    segGroup.add(chromeStrip);
                }

                stadiumGroup.add(segGroup);
            }

            const W = ARENA.HALF_W;
            const L = ARENA.HALF_L;
            const C = ARENA.CHAMFER;
            const GW = ARENA.GOAL_W * 0.5;

            // Sidelines (West X = -W, East X = +W)
            buildWallSegment(-W, -L + C, -W, L - C, 0x94a3b8, 2.0);
            buildWallSegment(W, L - C, W, -L + C, 0x94a3b8, 2.0);

            // 45-Degree Chamfered Corners
            buildWallSegment(-W + C, -L, -W, -L + C, 0x38bdf8, 0);
            buildWallSegment(W, -L + C, W - C, -L, 0x38bdf8, 0);
            buildWallSegment(-W, L - C, -W + C, L, 0xf97316, 0);
            buildWallSegment(W - C, L, W, L - C, 0xf97316, 0);

            // Blue Endline Flanking Walls (Z = -L)
            buildWallSegment(-GW, -L, -W + C, -L, 0x38bdf8, 0.8);
            buildWallSegment(W - C, -L, GW, -L, 0x38bdf8, 0.8);

            // Orange Endline Flanking Walls (Z = +L)
            buildWallSegment(-W + C, L, -GW, L, 0xf97316, 0.8);
            buildWallSegment(GW, L, W - C, L, 0xf97316, 0.8);

            // Overhead Translucent Arena Ceiling Netting
            const ceilTex = netTextures.map.clone();
            ceilTex.needsUpdate = true;
            ceilTex.repeat.set(64, 96);
            const ceilMat = metalNetMat.clone();
            ceilMat.map = ceilTex;
            ceilMat.alphaMap = ceilTex;
            ceilMat.opacity = 0.42;

            const ceiling = new THREE.Mesh(new THREE.PlaneGeometry(W * 2, L * 2), ceilMat);
            ceiling.rotation.x = Math.PI / 2;
            ceiling.position.y = ARENA.WALL_H;
            stadiumGroup.add(ceiling);

            // 3. Recessed Blue & Orange Goals + Goal-Line Forcefield Curtains
            function buildGoal(isBlue) {
                const zSign = isBlue ? -1 : 1;
                const teamColor = isBlue ? 0x3b82f6 : 0xf97316;
                const goalGroup = new THREE.Group();
                goalGroup.position.set(0, 0, zSign * L);

                const frameMat = new THREE.MeshStandardMaterial({
                    color: teamColor,
                    emissive: teamColor,
                    emissiveIntensity: 1.4,
                    metalness: 0.6,
                    roughness: 0.2
                });

                // Left & Right Goal Posts + Crossbar
                const postGeo = new THREE.BoxGeometry(1.05, ARENA.GOAL_H + 0.6, 1.05);
                const leftPost = new THREE.Mesh(postGeo, frameMat);
                leftPost.position.set(-GW, ARENA.GOAL_H * 0.5, 0);
                goalGroup.add(leftPost);

                const rightPost = new THREE.Mesh(postGeo, frameMat);
                rightPost.position.set(GW, ARENA.GOAL_H * 0.5, 0);
                goalGroup.add(rightPost);

                const crossbar = new THREE.Mesh(new THREE.BoxGeometry(ARENA.GOAL_W + 1.05, 1.05, 1.05), frameMat);
                crossbar.position.set(0, ARENA.GOAL_H, 0);
                goalGroup.add(crossbar);

                // Recessed Goal Chamber Netting
                const goalHexTex = netTextures.map.clone();
                goalHexTex.needsUpdate = true;
                goalHexTex.repeat.set(10, 5);

                const chamberNetMat = new THREE.MeshPhysicalMaterial({
                    color: teamColor,
                    emissive: teamColor,
                    emissiveIntensity: 0.55,
                    map: goalHexTex,
                    alphaMap: goalHexTex,
                    transparent: true,
                    opacity: 0.65,
                    side: THREE.DoubleSide,
                    depthWrite: false
                });

                const backNet = new THREE.Mesh(new THREE.PlaneGeometry(ARENA.GOAL_W, ARENA.GOAL_H), chamberNetMat);
                backNet.position.set(0, ARENA.GOAL_H * 0.5, zSign * ARENA.GOAL_D);
                goalGroup.add(backNet);

                const sideNetGeo = new THREE.PlaneGeometry(ARENA.GOAL_D, ARENA.GOAL_H);
                sideNetGeo.rotateY(Math.PI / 2);
                const leftSideNet = new THREE.Mesh(sideNetGeo, chamberNetMat);
                leftSideNet.position.set(-GW, ARENA.GOAL_H * 0.5, zSign * ARENA.GOAL_D * 0.5);
                goalGroup.add(leftSideNet);

                const rightSideNet = new THREE.Mesh(sideNetGeo, chamberNetMat);
                rightSideNet.position.set(GW, ARENA.GOAL_H * 0.5, zSign * ARENA.GOAL_D * 0.5);
                goalGroup.add(rightSideNet);

                const roofNet = new THREE.Mesh(new THREE.PlaneGeometry(ARENA.GOAL_W, ARENA.GOAL_D), chamberNetMat);
                roofNet.rotation.x = Math.PI / 2;
                roofNet.position.set(0, ARENA.GOAL_H, zSign * ARENA.GOAL_D * 0.5);
                goalGroup.add(roofNet);

                // Glowing Goal-Line Curtain
                const curtainMat = new THREE.MeshBasicMaterial({
                    color: teamColor,
                    map: goalHexTex,
                    alphaMap: goalHexTex,
                    transparent: true,
                    opacity: 0.25,
                    side: THREE.DoubleSide,
                    depthWrite: false
                });
                const curtain = new THREE.Mesh(new THREE.PlaneGeometry(ARENA.GOAL_W, ARENA.GOAL_H), curtainMat);
                curtain.position.set(0, ARENA.GOAL_H * 0.5, 0);
                goalGroup.add(curtain);

                // Goal Floor Turf Extension
                const goalFloor = new THREE.Mesh(
                    new THREE.PlaneGeometry(ARENA.GOAL_W, ARENA.GOAL_D),
                    new THREE.MeshStandardMaterial({ color: isBlue ? 0x1d4ed8 : 0xc2410c, roughness: 0.6 })
                );
                goalFloor.rotation.x = -Math.PI / 2;
                goalFloor.position.set(0, 0.02, zSign * ARENA.GOAL_D * 0.5);
                goalGroup.add(goalFloor);

                // Dynamic Team Goal Point Light
                const gLight = new THREE.PointLight(teamColor, 48, 86, 1.5);
                gLight.position.set(0, ARENA.GOAL_H * 0.65, zSign * 6.5);
                goalGroup.add(gLight);
                if (isBlue) blueGoalLight = gLight;
                else orangeGoalLight = gLight;

                stadiumGroup.add(goalGroup);
            }

            buildGoal(true);
            buildGoal(false);

            // 4. DFH Stadium Multi-Tiered Grandstands & Cheering Instanced Crowd
            const concreteMat = new THREE.MeshStandardMaterial({ color: 0x334155, roughness: 0.82 });
            const seatTierMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.75 });
            const dummy = new THREE.Object3D();

            // Left & Right Grandstand Bowls
            for (let side = -1; side <= 1; side += 2) {
                for (let tier = 0; tier < 6; tier++) {
                    const stepW = 5.2;
                    const stepH = 2.8;
                    const bx = side * (W + 5.5 + tier * stepW);
                    const by = 2.4 + tier * stepH;
                    const bench = new THREE.Mesh(new THREE.BoxGeometry(stepW, stepH * 1.6, L * 1.85), tier % 2 === 0 ? concreteMat : seatTierMat);
                    bench.position.set(bx, by, 0);
                    stadiumGroup.add(bench);

                    // Populate Spectator Positions
                    for (let sz = -L * 0.85; sz <= L * 0.85; sz += 4.6) {
                        if (Math.random() > 0.16) {
                            crowdBaseMatrices.push({
                                x: bx + (Math.random() - 0.5) * 1.6,
                                y: by + stepH * 0.8 + 0.75,
                                z: sz + (Math.random() - 0.5) * 1.0,
                                ry: side < 0 ? Math.PI / 2 : -Math.PI / 2,
                                isBlueSide: sz < 0
                            });
                            crowdPhases.push(Math.random() * Math.PI * 2);
                        }
                    }
                }
            }

            if (crowdBaseMatrices.length > 0) {
                const specGeo = new THREE.CylinderGeometry(0.42, 0.52, 1.35, 6);
                const specMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.65 });
                crowdInstancedMesh = new THREE.InstancedMesh(specGeo, specMat, crowdBaseMatrices.length);
                const cBlue = new THREE.Color(0x38bdf8);
                const cOrange = new THREE.Color(0xfb923c);
                const cWhite = new THREE.Color(0xf1f5f9);

                crowdBaseMatrices.forEach((b, idx) => {
                    dummy.position.set(b.x, b.y, b.z);
                    dummy.rotation.set(0, b.ry, 0);
                    dummy.updateMatrix();
                    crowdInstancedMesh.setMatrixAt(idx, dummy.matrix);
                    const r = Math.random();
                    crowdInstancedMesh.setColorAt(idx, r < 0.42 ? (b.isBlueSide ? cBlue : cOrange) : (r < 0.75 ? cWhite : (b.isBlueSide ? cOrange : cBlue)));
                });
                crowdInstancedMesh.instanceMatrix.needsUpdate = true;
                if (crowdInstancedMesh.instanceColor) crowdInstancedMesh.instanceColor.needsUpdate = true;
                stadiumGroup.add(crowdInstancedMesh);
            }

            // 5. Cantilevered Steel Roof Trusses, Floodlights & Jumbotron Scoreboards
            const trussMat = new THREE.MeshStandardMaterial({ color: 0x475569, metalness: 0.7, roughness: 0.35 });
            for (let side = -1; side <= 1; side += 2) {
                for (let z = -140; z <= 140; z += 40) {
                    const mast = new THREE.Mesh(new THREE.BoxGeometry(2.8, 58, 2.8), trussMat);
                    mast.position.set(side * (W + 36), 29, z);
                    stadiumGroup.add(mast);

                    const overhang = new THREE.Mesh(new THREE.BoxGeometry(44, 2.2, 2.4), trussMat);
                    overhang.position.set(side * (W + 17), 57, z);
                    overhang.rotation.z = side * -0.08;
                    stadiumGroup.add(overhang);

                    // Glowing Stadium Floodlight Pod
                    const lamp = new THREE.Mesh(
                        new THREE.BoxGeometry(4.0, 2.0, 6.8),
                        new THREE.MeshBasicMaterial({ color: 0xffedd5 })
                    );
                    lamp.position.set(side * (W + 2.5), 54, z);
                    lamp.rotation.z = side * 0.45;
                    stadiumGroup.add(lamp);
                }
            }

            // Endline Jumbotron Scoreboards
            for (let end = -1; end <= 1; end += 2) {
                const jumboGroup = new THREE.Group();
                jumboGroup.position.set(0, 46, end * (L + 22));
                const frame = new THREE.Mesh(new THREE.BoxGeometry(52, 16, 3.0), trussMat);
                jumboGroup.add(frame);

                const screenMat = new THREE.MeshBasicMaterial({
                    color: end < 0 ? 0x2563eb : 0xea580c
                });
                const screen = new THREE.Mesh(new THREE.PlaneGeometry(48, 13), screenMat);
                screen.position.z = -end * 1.65;
                if (end > 0) screen.rotation.y = Math.PI;
                jumboGroup.add(screen);
                stadiumGroup.add(jumboGroup);
            }

            // Distant Horizon Hills & City Towers
            const hillMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.9 });
            for (let i = 0; i < 32; i++) {
                const a = (i / 32) * Math.PI * 2;
                const dist = 390 + (i % 4) * 44;
                const bh = 36 + ((i * 19) % 75);
                const bw = 22 + ((i * 11) % 28);
                const tower = new THREE.Mesh(new THREE.BoxGeometry(bw, bh, bw), hillMat);
                tower.position.set(Math.cos(a) * dist, bh * 0.5 - 4, Math.sin(a) * dist);
                stadiumGroup.add(tower);
            }

            scene.add(stadiumGroup);
        }

        // =========================================================================
        // 7. ZERO-LAG ULTRA HD ROUNDED BOOST PADS (2X STADIUM NETWORK)
        // =========================================================================
        const boostPads = [];

        function setBoostPadActive(pad, active) {
            pad.active = active;
            pad.orb.visible = active;
            if (pad.ring1) pad.ring1.visible = active;
            if (pad.ring2) pad.ring2.visible = active;
            if (pad.beam) pad.beam.visible = active;
            // Modulate intensity only — never toggle .visible on PointLights to prevent shader recompilation stutter
            if (pad.light) {
                pad.light.intensity = (active && state.ultraHD) ? 20 : 0;
            }
            if (!active) {
                pad.cooldown = pad.isFull ? 10.0 : 4.0;
            }
        }

        function initBoostPads() {
            const padCoords = [
                // 6 Full 100% Corner & Midfield Boost Orbs (2x DFH Stadium Layout)
                { x: -82, z: -124, isFull: true },
                { x: 82, z: -124, isFull: true },
                { x: -90, z: 0, isFull: true },
                { x: 90, z: 0, isFull: true },
                { x: -82, z: 124, isFull: true },
                { x: 82, z: 124, isFull: true },

                // 12 Small 12% Tactical & Kickoff Pads (2x DFH Stadium Layout)
                { x: 0, z: -122, isFull: false },
                { x: 0, z: -88, isFull: false },
                { x: 0, z: -48, isFull: false },
                { x: -42, z: -64, isFull: false },
                { x: 42, z: -64, isFull: false },
                { x: -44, z: 0, isFull: false },
                { x: 44, z: 0, isFull: false },
                { x: -42, z: 64, isFull: false },
                { x: 42, z: 64, isFull: false },
                { x: 0, z: 48, isFull: false },
                { x: 0, z: 88, isFull: false },
                { x: 0, z: 122, isFull: false }
            ];

            // Smooth Lathe-Turned Titanium Housing Profile
            const lathePts = [
                new THREE.Vector2(0.0, 0.0),
                new THREE.Vector2(1.55, 0.0),
                new THREE.Vector2(1.48, 0.14),
                new THREE.Vector2(1.22, 0.26),
                new THREE.Vector2(0.85, 0.30),
                new THREE.Vector2(0.0, 0.24)
            ];
            const housingGeo = new THREE.LatheGeometry(lathePts, 28);
            const sharedHousingMat = new THREE.MeshPhysicalMaterial({
                color: 0x94a3b8,
                metalness: 0.78,
                roughness: 0.22,
                clearcoat: 0.7,
                clearcoatRoughness: 0.15
            });

            const emitterMat = new THREE.MeshBasicMaterial({ color: 0xfbbf24 });
            const orbMat = new THREE.MeshPhysicalMaterial({
                color: 0xfde047,
                emissive: 0xf59e0b,
                emissiveIntensity: 1.5,
                roughness: 0.12,
                metalness: 0.2,
                clearcoat: 1.0
            });
            const gyroMat = new THREE.MeshBasicMaterial({ color: 0xfef08a });
            const beamMat = new THREE.MeshBasicMaterial({
                color: 0xfbbf24,
                transparent: true,
                opacity: 0.14,
                side: THREE.DoubleSide,
                depthWrite: false
            });

            padCoords.forEach((cfg, idx) => {
                const scale = cfg.isFull ? 1.25 : 0.85;
                const group = new THREE.Group();
                group.position.set(cfg.x, 0.02, cfg.z);

                const base = new THREE.Mesh(housingGeo, sharedHousingMat);
                base.scale.set(scale, scale, scale);
                base.receiveShadow = true;
                group.add(base);

                // Glowing Inlaid Torus Emitter Ring
                const emitterTorus = new THREE.Mesh(
                    new THREE.TorusGeometry(1.08 * scale, 0.09 * scale, 12, 28),
                    emitterMat
                );
                emitterTorus.rotation.x = Math.PI / 2;
                emitterTorus.position.y = 0.26 * scale;
                group.add(emitterTorus);

                // Floating Plasma Orb
                const orb = new THREE.Mesh(
                    new THREE.SphereGeometry(cfg.isFull ? 0.72 : 0.38, 20, 16),
                    orbMat
                );
                orb.position.y = cfg.isFull ? 1.85 : 0.55;
                group.add(orb);

                let ring1 = null;
                let ring2 = null;
                let beam = null;
                let light = null;

                if (cfg.isFull) {
                    ring1 = new THREE.Mesh(new THREE.TorusGeometry(1.05, 0.06, 10, 28), gyroMat);
                    ring1.position.y = 1.85;
                    group.add(ring1);

                    ring2 = new THREE.Mesh(new THREE.TorusGeometry(0.92, 0.05, 10, 28), gyroMat);
                    ring2.position.y = 1.85;
                    ring2.rotation.x = Math.PI / 3;
                    group.add(ring2);

                    beam = new THREE.Mesh(
                        new THREE.CylinderGeometry(0.95, 1.25, 3.6, 20, 1, true),
                        beamMat
                    );
                    beam.position.y = 1.85;
                    group.add(beam);

                    light = new THREE.PointLight(0xfbbf24, 20, 18, 1.6);
                    light.position.set(0, 2.2, 0);
                    group.add(light);
                } else {
                    ring1 = new THREE.Mesh(new THREE.TorusGeometry(0.62, 0.045, 8, 24), gyroMat);
                    ring1.rotation.x = Math.PI / 2;
                    ring1.position.y = 0.55;
                    group.add(ring1);
                }

                scene.add(group);
                boostPads.push({
                    x: cfg.x,
                    z: cfg.z,
                    isFull: cfg.isFull,
                    radius: cfg.isFull ? 3.2 : 2.3,
                    active: true,
                    cooldown: 0,
                    phase: idx * 0.7,
                    orb,
                    ring1,
                    ring2,
                    beam,
                    light
                });
            });
        }

        // =========================================================================
        // 8. SOCCER BALL ENTITY (BRIGHT TITANIUM FINISH + GROUND RING)
        // =========================================================================
        const ball = {
            pos: new THREE.Vector3(0, PHYSICS.BALL_RADIUS, 0),
            vel: new THREE.Vector3(0, 0, 0),
            angVel: new THREE.Vector3(0, 0, 0),
            mesh: null,
            ring: null,
            glowLight: null
        };

        function initBall() {
            const bCanvas = document.createElement('canvas');
            bCanvas.width = 512;
            bCanvas.height = 256;
            const bctx = bCanvas.getContext('2d');
            bctx.fillStyle = '#f8fafc';
            bctx.fillRect(0, 0, 512, 256);

            bctx.strokeStyle = '#475569';
            bctx.lineWidth = 6;
            for (let y = 0; y <= 256; y += 32) {
                for (let x = 0; x <= 512; x += 32) {
                    bctx.strokeRect(x + 2, y + 2, 28, 28);
                }
            }
            // Glowing Equator Circuit Band
            bctx.fillStyle = '#38bdf8';
            bctx.fillRect(0, 120, 512, 16);

            const ballTex = new THREE.CanvasTexture(bCanvas);
            ballTex.encoding = THREE.sRGBEncoding;

            const ballMat = new THREE.MeshPhysicalMaterial({
                map: ballTex,
                metalness: 0.52,
                roughness: 0.20,
                clearcoat: 1.0,
                clearcoatRoughness: 0.08,
                emissive: 0x1e293b,
                emissiveIntensity: 0.14,
                envMapIntensity: 1.75
            });

            ball.mesh = new THREE.Mesh(new THREE.SphereGeometry(PHYSICS.BALL_RADIUS, 36, 28), ballMat);
            ball.mesh.castShadow = true;
            ball.mesh.receiveShadow = true;
            scene.add(ball.mesh);

            // Ground Height Indicator Ring
            const ringGeo = new THREE.RingGeometry(PHYSICS.BALL_RADIUS * 0.72, PHYSICS.BALL_RADIUS * 1.05, 36);
            ringGeo.rotateX(-Math.PI / 2);
            const ringMat = new THREE.MeshBasicMaterial({
                color: 0xf8fafc,
                transparent: true,
                opacity: 0.65,
                side: THREE.DoubleSide,
                depthWrite: false
            });
            ball.ring = new THREE.Mesh(ringGeo, ringMat);
            ball.ring.position.y = 0.05;
            scene.add(ball.ring);

            ball.glowLight = new THREE.PointLight(0xfbbf24, 10, 22, 1.6);
            scene.add(ball.glowLight);
        }

        // =========================================================================
        // 9. ROUNDED METALLIC CLEARCOAT ROCKET CARS (OCTANE-INSPIRED HYPERCAR)
        // =========================================================================
        function createRocketCarMesh(isBlue) {
            const root = new THREE.Group();

            const primaryHex = isBlue ? 0x3b82f6 : 0xf97316;
            const accentHex = isBlue ? 0x7dd3fc : 0xfde047;

            // High-Gloss Bright Metallic Flake Clearcoat Body Paint
            const bodyPaintMat = new THREE.MeshPhysicalMaterial({
                color: primaryHex,
                metalness: 0.55,
                roughness: 0.18,
                clearcoat: 1.0,
                clearcoatRoughness: 0.06,
                clearcoatNormalMap: flakeNormalMap,
                clearcoatNormalScale: new THREE.Vector2(0.08, 0.08),
                envMapIntensity: 2.2
            });

            const carbonFiberMat = new THREE.MeshStandardMaterial({
                color: 0x334155,
                metalness: 0.45,
                roughness: 0.35
            });

            const canopyMat = new THREE.MeshPhysicalMaterial({
                color: 0x0f172a,
                metalness: 0.15,
                roughness: 0.05,
                clearcoat: 1.0,
                clearcoatRoughness: 0.04,
                envMapIntensity: 2.6
            });

            const stripeMat = new THREE.MeshStandardMaterial({
                color: 0xf8fafc,
                metalness: 0.35,
                roughness: 0.2
            });

            // Rounded Extruded Aerodynamic Body Profile
            const bodyProfile = new THREE.Shape();
            bodyProfile.moveTo(-2.15, 0.34);
            bodyProfile.quadraticCurveTo(-2.25, 0.75, -1.75, 1.05);
            bodyProfile.lineTo(-0.85, 1.15);
            bodyProfile.quadraticCurveTo(-0.15, 1.56, 0.65, 1.38);
            bodyProfile.quadraticCurveTo(1.55, 1.10, 2.22, 0.74);
            bodyProfile.quadraticCurveTo(2.35, 0.52, 2.18, 0.34);
            bodyProfile.closePath();

            const bodyGeo = new THREE.ExtrudeGeometry(bodyProfile, {
                depth: 1.72,
                bevelEnabled: true,
                bevelThickness: 0.22,
                bevelSize: 0.18,
                bevelSegments: 6,
                curveSegments: 16
            });
            bodyGeo.translate(0, 0, -0.86);
            bodyGeo.rotateY(-Math.PI / 2);

            const chassis = new THREE.Mesh(bodyGeo, bodyPaintMat);
            chassis.castShadow = true;
            chassis.receiveShadow = true;
            root.add(chassis);

            // Twin White Racing Stripes along Hood & Roof
            for (let s = -1; s <= 1; s += 2) {
                const stripe = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.06, 2.65), stripeMat);
                stripe.position.set(s * 0.26, 1.18, 0.25);
                stripe.rotation.x = 0.17;
                root.add(stripe);
            }

            // Curved Teardrop Tinted Glass Canopy
            const canopy = new THREE.Mesh(new THREE.SphereGeometry(0.95, 24, 18), canopyMat);
            canopy.scale.set(0.88, 0.54, 1.32);
            canopy.position.set(0, 1.16, 0.08);
            root.add(canopy);

            // Carbon-Fiber Front Splitter & Side Skirts
            const splitter = new THREE.Mesh(new THREE.BoxGeometry(2.12, 0.14, 0.55), carbonFiberMat);
            splitter.position.set(0, 0.32, 2.15);
            root.add(splitter);

            for (let side = -1; side <= 1; side += 2) {
                const skirt = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.16, 2.75), carbonFiberMat);
                skirt.position.set(side * 1.06, 0.34, 0);
                root.add(skirt);
            }

            // Rear Spoiler Wing & Struts
            const wing = new THREE.Mesh(new THREE.BoxGeometry(2.15, 0.12, 0.52), bodyPaintMat);
            wing.position.set(0, 1.56, -1.92);
            wing.rotation.x = -0.14;
            wing.castShadow = true;
            root.add(wing);

            for (let side = -1; side <= 1; side += 2) {
                const strut = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.08, 0.58, 8), carbonFiberMat);
                strut.position.set(side * 0.65, 1.28, -1.88);
                strut.rotation.x = -0.18;
                root.add(strut);
            }

            // Twin Rear Rocket Thruster Nozzles & Boost Flame Cones
            const nozzleMat = new THREE.MeshStandardMaterial({ color: 0x475569, metalness: 0.85, roughness: 0.25 });
            const flameMat = new THREE.MeshBasicMaterial({ color: 0xfbbf24, transparent: true, opacity: 0.92 });
            const flames = [];

            for (let side = -1; side <= 1; side += 2) {
                const nozzle = new THREE.Mesh(new THREE.CylinderGeometry(0.22, 0.26, 0.38, 16), nozzleMat);
                nozzle.rotation.x = Math.PI / 2;
                nozzle.position.set(side * 0.44, 0.68, -2.22);
                root.add(nozzle);

                const flame = new THREE.Mesh(new THREE.ConeGeometry(0.22, 1.45, 14), flameMat);
                flame.rotation.x = -Math.PI / 2;
                flame.position.set(side * 0.44, 0.68, -3.02);
                flame.visible = false;
                root.add(flame);
                flames.push(flame);
            }

            // Glowing LED Daytime Headlights & Brake Taillight Bar
            const headlightMat = new THREE.MeshBasicMaterial({ color: 0xf0f9ff });
            for (let side = -1; side <= 1; side += 2) {
                const hl = new THREE.Mesh(new THREE.BoxGeometry(0.38, 0.12, 0.1), headlightMat);
                hl.position.set(side * 0.66, 0.68, 2.24);
                hl.rotation.y = side * -0.18;
                root.add(hl);
            }

            const taillightMat = new THREE.MeshBasicMaterial({ color: 0xef4444 });
            const taillight = new THREE.Mesh(new THREE.BoxGeometry(1.42, 0.12, 0.1), taillightMat);
            taillight.position.set(0, 0.86, -2.24);
            root.add(taillight);

            // 4 Multi-Spoke Metallic Alloy Wheels & Treaded Tires
            const tireMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.72 });
            const rimMat = new THREE.MeshStandardMaterial({
                color: accentHex,
                emissive: accentHex,
                emissiveIntensity: 0.45,
                metalness: 0.85,
                roughness: 0.15
            });
            const wheels = [];
            const wheelPositions = [
                { x: -1.12, y: 0.46, z: 1.38, scale: 0.94 },
                { x: 1.12, y: 0.46, z: 1.38, scale: 0.94 },
                { x: -1.16, y: 0.50, z: -1.35, scale: 1.05 },
                { x: 1.16, y: 0.50, z: -1.35, scale: 1.05 }
            ];

            wheelPositions.forEach(wp => {
                const wGroup = new THREE.Group();
                wGroup.position.set(wp.x, wp.y, wp.z);
                wGroup.scale.setScalar(wp.scale);

                const tire = new THREE.Mesh(new THREE.CylinderGeometry(0.46, 0.46, 0.36, 22), tireMat);
                tire.rotation.z = Math.PI / 2;
                tire.castShadow = true;
                wGroup.add(tire);

                const rim = new THREE.Mesh(new THREE.CylinderGeometry(0.30, 0.30, 0.38, 16), rimMat);
                rim.rotation.z = Math.PI / 2;
                wGroup.add(rim);

                root.add(wGroup);
                wheels.push(wGroup);
            });

            // Soft Ground Contact Shadow & Team Underglow Light
            const shadowDecal = new THREE.Mesh(
                new THREE.PlaneGeometry(2.8, 4.9),
                new THREE.MeshBasicMaterial({ color: 0x020617, transparent: true, opacity: 0.42, depthWrite: false })
            );
            shadowDecal.rotation.x = -Math.PI / 2;
            shadowDecal.position.y = 0.04;
            scene.add(shadowDecal);

            const underglowLight = new THREE.PointLight(primaryHex, 7, 10, 1.8);
            underglowLight.position.set(0, 0.35, 0);
            root.add(underglowLight);

            scene.add(root);
            return { root, wheels, flames, taillightMat, shadowDecal, underglowLight };
        }

        function createCarEntity(isBlue, slotIndex = 0) {
            const visuals = createRocketCarMesh(isBlue);
            return {
                isBlue,
                slotIndex,
                active: true,
                pos: new THREE.Vector3(0, 0.5, isBlue ? -110 : 110),
                vel: new THREE.Vector3(0, 0, 0),
                quat: new THREE.Quaternion(),
                surfaceNormal: new THREE.Vector3(0, 1, 0),
                halfExtents: new THREE.Vector3(1.15, 0.85, 2.25),
                boost: 33,
                grounded: true,
                hasDoubleJump: true,
                flipTimer: 0,
                flipTorque: new THREE.Vector3(),
                supersonic: false,
                visuals,
                underglowLight: visuals.underglowLight
            };
        }

        // Pre-allocate 3 Blue & 3 Orange Rocket Cars for 1v1, 2v2, and 3v3 Modes
        const blueCars = [
            createCarEntity(true, 0),  // Human Player
            createCarEntity(true, 1),  // Blue AI Teammate 1
            createCarEntity(true, 2)   // Blue AI Teammate 2
        ];
        const orangeCars = [
            createCarEntity(false, 0), // Orange AI 1
            createCarEntity(false, 1), // Orange AI 2
            createCarEntity(false, 2)  // Orange AI 3
        ];
        const playerCar = blueCars[0];
        const allCars = [...blueCars, ...orangeCars];
        let activeCars = [];

        // Canonical Rocket League Kickoff Spawn Formations (Mirrored for Blue & Orange)
        // Each slot: { x, z } on Blue half (z < 0), automatically angled toward center ball (0, 0)
        const ROCKET_LEAGUE_KICKOFFS = {
            1: [
                // 1v1 Presets: Diagonal Duel, Off-Center Straight, Far Center Straight
                [{ x: 42, z: -64 }],
                [{ x: -42, z: -64 }],
                [{ x: 14, z: -112 }],
                [{ x: -14, z: -112 }],
                [{ x: 0, z: -132 }]
            ],
            2: [
                // 2v2 Presets: Diagonal + Goalie, Double Off-Center, Opposite Diagonal + Off-Center
                [{ x: -42, z: -64 }, { x: 0, z: -136 }],
                [{ x: 42, z: -64 }, { x: 0, z: -136 }],
                [{ x: -14, z: -112 }, { x: 14, z: -112 }],
                [{ x: 42, z: -64 }, { x: -14, z: -112 }]
            ],
            3: [
                // 3v3 Presets: Classic Double Diagonal + Goalie, Diagonal + Off-Center + Goalie, Double Off-Center + Goalie
                [{ x: -42, z: -64 }, { x: 42, z: -64 }, { x: 0, z: -136 }],
                [{ x: 42, z: -64 }, { x: -14, z: -112 }, { x: 0, z: -136 }],
                [{ x: -42, z: -64 }, { x: 14, z: -112 }, { x: 0, z: -136 }],
                [{ x: -14, z: -112 }, { x: 14, z: -112 }, { x: 0, z: -136 }]
            ]
        };

        function placeCarAtKickoffSpawn(car, spawnX, spawnZ) {
            car.pos.set(spawnX, 0.5, spawnZ);
            car.vel.set(0, 0, 0);
            // Angle car directly toward center kickoff spot (0, 0) just like Rocket League!
            const yawToBall = Math.atan2(-spawnX, -spawnZ);
            car.quat.setFromAxisAngle(new THREE.Vector3(0, 1, 0), yawToBall);
            car.surfaceNormal.set(0, 1, 0);
            car.boost = 33;
            car.grounded = true;
            car.hasDoubleJump = true;
            car.flipTimer = 0;
            car.supersonic = false;
            car.visuals.root.position.copy(car.pos);
            car.visuals.root.quaternion.copy(car.quat);
            car.visuals.shadowDecal.position.set(car.pos.x, 0.04, car.pos.z);
        }

        function resetKickoffPositions(advancePreset = false) {
            ball.pos.set(0, PHYSICS.BALL_RADIUS, 0);
            ball.vel.set(0, 0, 0);
            ball.angVel.set(0, 0, 0);
            if (ball.mesh) ball.mesh.position.copy(ball.pos);
            if (ball.ring) ball.ring.position.set(0, 0.05, 0);

            const n = state.teamSize;
            const presets = ROCKET_LEAGUE_KICKOFFS[n] || ROCKET_LEAGUE_KICKOFFS[1];
            if (advancePreset) {
                state.kickoffIndex = (state.kickoffIndex + 1) % presets.length;
            }
            const formation = presets[state.kickoffIndex % presets.length];

            activeCars = [];
            for (let i = 0; i < 3; i++) {
                const bCar = blueCars[i];
                const oCar = orangeCars[i];
                if (i < n) {
                    bCar.active = true;
                    bCar.visuals.root.visible = true;
                    bCar.visuals.shadowDecal.visible = true;
                    bCar.underglowLight.intensity = state.ultraHD ? 7 : 0;

                    oCar.active = true;
                    oCar.visuals.root.visible = true;
                    oCar.visuals.shadowDecal.visible = true;
                    oCar.underglowLight.intensity = state.ultraHD ? 7 : 0;

                    const spot = formation[i];
                    // Mirror diagonally across center (0, 0) for Orange team just like Rocket League!
                    placeCarAtKickoffSpawn(bCar, spot.x, spot.z);
                    placeCarAtKickoffSpawn(oCar, -spot.x, -spot.z);

                    activeCars.push(bCar, oCar);
                } else {
                    bCar.active = false;
                    bCar.visuals.root.visible = false;
                    bCar.visuals.shadowDecal.visible = false;
                    bCar.underglowLight.intensity = 0;

                    oCar.active = false;
                    oCar.visuals.root.visible = false;
                    oCar.visuals.shadowDecal.visible = false;
                    oCar.underglowLight.intensity = 0;
                }
            }
        }

        // =========================================================================
        // 10. PARTICLE EFFECTS & GOAL EXPLOSIONS
        // =========================================================================
        const particles = [];
        const particleGeo = new THREE.SphereGeometry(0.32, 8, 8);

        function spawnParticle(pos, vel, colorHex, scale = 1.0, maxLife = 0.45) {
            if (particles.length > 220) return;
            const mat = new THREE.MeshBasicMaterial({
                color: colorHex,
                transparent: true,
                opacity: 0.85
            });
            const mesh = new THREE.Mesh(particleGeo, mat);
            mesh.position.copy(pos);
            mesh.scale.setScalar(scale);
            scene.add(mesh);
            particles.push({ mesh, vel: vel.clone(), life: maxLife, maxLife, baseScale: scale });
        }

        function triggerGoalExplosion(zPos, isBlueScorer) {
            const origin = new THREE.Vector3(0, 5.5, zPos);
            const colors = isBlueScorer ? [0x38bdf8, 0x3b82f6, 0xe0f2fe] : [0xfb923c, 0xf97316, 0xfef08a];
            for (let i = 0; i < 75; i++) {
                const dir = new THREE.Vector3(
                    (Math.random() - 0.5) * 2,
                    Math.random() * 1.4,
                    (Math.random() - 0.5) * 2
                ).normalize();
                const speed = 16 + Math.random() * 38;
                spawnParticle(
                    origin,
                    dir.multiplyScalar(speed),
                    colors[i % colors.length],
                    1.6 + Math.random() * 2.4,
                    0.95 + Math.random() * 0.75
                );
            }
        }

        // =========================================================================
        // 11. CONTINUOUS ROCKET LEAGUE ARENA SDF COLLISION (RAMPS, WALLS, CEILING)
        // =========================================================================
        function evaluateRocketLeagueArenaSDF(pos) {
            const W = ARENA.HALF_W;
            const L = ARENA.HALF_L;
            const C = ARENA.CHAMFER;
            const H = ARENA.WALL_H;
            const GW = ARENA.GOAL_W * 0.5;
            const GH = ARENA.GOAL_H;
            const GD = ARENA.GOAL_D;

            // Check if inside recessed Blue or Orange Goal Box
            const inGoalMouthX = Math.abs(pos.x) < GW - 0.35;
            const inGoalMouthY = pos.y < GH - 0.35;
            if (inGoalMouthX && inGoalMouthY && Math.abs(pos.z) > L - 1.5) {
                // Goal scored detection if ball crosses goal line completely
                if (pos.z < -L - PHYSICS.BALL_RADIUS * 0.85) {
                    return { dist: -1, normal: new THREE.Vector3(0, 0, 1), goalScored: 'orange' };
                }
                if (pos.z > L + PHYSICS.BALL_RADIUS * 0.85) {
                    return { dist: -1, normal: new THREE.Vector3(0, 0, -1), goalScored: 'blue' };
                }

                // Confine within recessed goal chamber walls/ceiling/floor
                let minD = 999;
                const n = new THREE.Vector3(0, 1, 0);
                if (pos.y < minD) { minD = pos.y; n.set(0, 1, 0); }
                if (GH - pos.y < minD) { minD = GH - pos.y; n.set(0, -1, 0); }
                if (GW - Math.abs(pos.x) < minD) { minD = GW - Math.abs(pos.x); n.set(-Math.sign(pos.x), 0, 0); }
                const backDist = (L + GD) - Math.abs(pos.z);
                if (backDist < minD) { minD = backDist; n.set(0, 0, -Math.sign(pos.z)); }
                return { dist: minD, normal: n, goalScored: null };
            }

            // Compute inward horizontal distance to the 8 octagonal arena walls
            const dWest = pos.x + W;
            const dEast = W - pos.x;
            const dNorth = pos.z + L;
            const dSouth = L - pos.z;

            // 45-degree chamfered corner planes: |x| + |z| <= W + L - C
            const invSqrt2 = 0.70710678;
            const cornerLimit = (W + L - C) * invSqrt2;
            const sx = pos.x >= 0 ? 1 : -1;
            const sz = pos.z >= 0 ? 1 : -1;
            const dCorner = cornerLimit - (sx * pos.x + sz * pos.z) * invSqrt2;

            // Pick closest horizontal wall
            let dWall = dWest;
            const wallN = new THREE.Vector3(1, 0, 0);

            if (dEast < dWall) { dWall = dEast; wallN.set(-1, 0, 0); }
            if (dNorth < dWall) { dWall = dNorth; wallN.set(0, 0, 1); }
            if (dSouth < dWall) { dWall = dSouth; wallN.set(0, 0, -1); }
            if (dCorner < dWall) { dWall = dCorner; wallN.set(-sx * invSqrt2, 0, -sz * invSqrt2); }

            const dFloor = pos.y;
            const dCeil = H - pos.y;

            // Taper ramp radius smoothly near goal posts so goal openings remain crisp
            let R = ARENA.RAMP_R;
            if (Math.abs(pos.z) > L - 15 && Math.abs(pos.x) < GW + 6.0) {
                const t = THREE.MathUtils.clamp((Math.abs(pos.x) - GW) / 6.0, 0, 1);
                R *= t;
            }

            // Bottom Quarter-Pipe Floor-to-Wall Transition Ramp
            if (R > 0.2 && dWall < R && dFloor < R) {
                const dx = R - dWall;
                const dy = R - dFloor;
                const len = Math.hypot(dx, dy);
                if (len > 0.0001) {
                    const dist = R - len;
                    const normal = new THREE.Vector3()
                        .addScaledVector(wallN, dx / len)
                        .add(new THREE.Vector3(0, dy / len, 0))
                        .normalize();
                    return { dist, normal, goalScored: null };
                }
            }

            // Top Quarter-Pipe Wall-to-Ceiling Transition Ramp
            const TR = ARENA.TOP_RAMP_R;
            if (dWall < TR && dCeil < TR) {
                const dx = TR - dWall;
                const dy = TR - dCeil;
                const len = Math.hypot(dx, dy);
                if (len > 0.0001) {
                    const dist = TR - len;
                    const normal = new THREE.Vector3()
                        .addScaledVector(wallN, dx / len)
                        .add(new THREE.Vector3(0, -dy / len, 0))
                        .normalize();
                    return { dist, normal, goalScored: null };
                }
            }

            // Standard Planar Floor, Wall, or Ceiling
            if (dFloor <= dWall && dFloor <= dCeil) {
                return { dist: dFloor, normal: new THREE.Vector3(0, 1, 0), goalScored: null };
            }
            if (dCeil <= dWall) {
                return { dist: dCeil, normal: new THREE.Vector3(0, -1, 0), goalScored: null };
            }
            return { dist: dWall, normal: wallN, goalScored: null };
        }

        function resolveArenaCollision(pos, vel, radius, restitution, isBall = false) {
            const sdf = evaluateRocketLeagueArenaSDF(pos);
            if (isBall && sdf.goalScored) {
                return { hit: true, normal: sdf.normal, goalScored: sdf.goalScored };
            }
            if (sdf.dist < radius) {
                const pen = radius - sdf.dist;
                pos.addScaledVector(sdf.normal, pen);
                const vn = vel.dot(sdf.normal);
                if (vn < 0) {
                    vel.addScaledVector(sdf.normal, -(1 + restitution) * vn);
                }
                return { hit: true, normal: sdf.normal, goalScored: null };
            }
            return { hit: false, normal: sdf.normal, goalScored: null };
        }

        // =========================================================================
        // 12. CAR PHYSICS, TEAMMATE-AWARE AI & BALL COLLISION (ZERO GROUND POP-UP)
        // =========================================================================
        function updateAIController(car, dt) {
            const cfg = {
                rookie: { speedMult: 0.72, boostThresh: 72, predict: 0.25 },
                pro: { speedMult: 0.92, boostThresh: 42, predict: 0.45 },
                allstar: { speedMult: 1.06, boostThresh: 18, predict: 0.65 }
            }[state.difficulty] || { speedMult: 0.92, boostThresh: 42, predict: 0.45 };

            const teamSign = car.isBlue ? -1 : 1; // Blue defends -Z, Orange defends +Z
            const ownGoalZ = teamSign * ARENA.HALF_L;
            const enemyGoalZ = -teamSign * ARENA.HALF_L;

            // Rank teammates by distance to the ball so 1st man attacks, 2nd man supports, 3rd man guards goal
            const teammates = (car.isBlue ? blueCars : orangeCars).filter(c => c.active);
            const myDistToBall = car.pos.distanceTo(ball.pos);
            let rank = 0;
            for (let i = 0; i < teammates.length; i++) {
                if (teammates[i] !== car && teammates[i].pos.distanceTo(ball.pos) < myDistToBall) {
                    rank++;
                }
            }

            const predictedBall = ball.pos.clone().addScaledVector(ball.vel, cfg.predict);
            const targetPos = new THREE.Vector3();

            if (rank === 0) {
                // 1st Man (Striker / Kickoff Rusher): Attack ball toward enemy goal
                const enemyGoal = new THREE.Vector3(0, 0, enemyGoalZ);
                const ballToGoal = enemyGoal.sub(predictedBall).normalize();
                targetPos.copy(predictedBall).sub(ballToGoal.multiplyScalar(2.6));

                // Fall back to clear if ball is behind car toward own goal
                if ((car.isBlue && ball.pos.z < car.pos.z - 6) || (!car.isBlue && ball.pos.z > car.pos.z + 6)) {
                    targetPos.set(
                        THREE.MathUtils.clamp(ball.pos.x * 0.55, -16, 16),
                        0,
                        THREE.MathUtils.clamp(ball.pos.z + teamSign * 14, -ARENA.HALF_L + 8, ARENA.HALF_L - 8)
                    );
                }
            } else if (rank === 1) {
                // 2nd Man (Midfield Wing Support): Hold center-offset passing lane behind 1st man
                const laneSide = car.slotIndex % 2 === 0 ? -1 : 1;
                targetPos.set(
                    THREE.MathUtils.clamp(predictedBall.x * 0.4 + laneSide * 26, -ARENA.HALF_W + 18, ARENA.HALF_W - 18),
                    0,
                    THREE.MathUtils.clamp(predictedBall.z + teamSign * 44, -ARENA.HALF_L + 22, ARENA.HALF_L - 22)
                );
            } else {
                // 3rd Man (Goalie / Sweeper): Patrol defensive third & intercept incoming shots
                const ballThreat = (car.isBlue && ball.pos.z < -45) || (!car.isBlue && ball.pos.z > 45);
                if (ballThreat) {
                    targetPos.copy(predictedBall);
                } else {
                    targetPos.set(
                        THREE.MathUtils.clamp(ball.pos.x * 0.35, -14, 14),
                        0,
                        ownGoalZ - teamSign * 18
                    );
                }
            }

            const toTarget = targetPos.sub(car.pos);
            toTarget.y = 0;
            const dist = toTarget.length();

            const forward = new THREE.Vector3(0, 0, 1).applyQuaternion(car.quat);
            forward.y = 0;
            if (forward.lengthSq() > 0.001) forward.normalize();

            const right = new THREE.Vector3().crossVectors(new THREE.Vector3(0, 1, 0), forward).normalize();
            const dir = dist > 0.1 ? toTarget.normalize() : forward;

            const align = forward.dot(dir);
            const side = right.dot(dir);
            const isKickoff = ball.pos.lengthSq() < 1.0 && ball.vel.lengthSq() < 0.1 && rank === 0;

            return {
                throttle: align > -0.2 ? cfg.speedMult : -0.65,
                steer: THREE.MathUtils.clamp(side * 2.4, -1, 1),
                boost: isKickoff || (align > 0.88 && dist > 22 && car.boost > cfg.boostThresh),
                jump: rank === 0 && dist < 7.0 && ball.pos.y > 2.6 && ball.pos.y < 6.5 && car.grounded,
                drift: Math.abs(side) > 0.72 && dist > 14,
                pitch: 0,
                roll: 0
            };
        }

        function updateCarPhysics(car, dt) {
            let input;
            if (car === playerCar) {
                const throttle = (keys.forward ? 1 : 0) - (keys.backward ? 1 : 0);
                const steer = (keys.left ? 1 : 0) - (keys.right ? 1 : 0);
                let roll = (keys.rollRight ? 1 : 0) - (keys.rollLeft ? 1 : 0);
                if (!car.grounded && keys.drift && steer !== 0) {
                    roll = -steer;
                }
                input = {
                    throttle,
                    steer,
                    boost: keys.boost,
                    jump: jumpPressedEdge,
                    drift: keys.drift,
                    pitch: throttle,
                    roll
                };
                jumpPressedEdge = false;
            } else {
                input = updateAIController(car, dt);
            }

            // Local car axes
            const forward = new THREE.Vector3(0, 0, 1).applyQuaternion(car.quat).normalize();
            const up = new THREE.Vector3(0, 1, 0).applyQuaternion(car.quat).normalize();
            const right = new THREE.Vector3(1, 0, 0).applyQuaternion(car.quat).normalize();

            // Check if car is near floor, curved ramp, or wall
            const sdf = evaluateRocketLeagueArenaSDF(car.pos);
            const rideRadius = 0.68;
            if (sdf.dist <= rideRadius + 0.12 && car.flipTimer <= 0) {
                car.grounded = true;
                car.hasDoubleJump = true;
                car.surfaceNormal.lerp(sdf.normal, Math.min(1, dt * 18)).normalize();

                // Align car's local UP smoothly to the ramp/wall/floor normal
                const alignQuat = new THREE.Quaternion().setFromUnitVectors(up, car.surfaceNormal);
                car.quat.premultiply(alignQuat).normalize();
            } else {
                car.grounded = false;
                car.surfaceNormal.lerp(new THREE.Vector3(0, 1, 0), Math.min(1, dt * 4)).normalize();
            }

            // Ground / Wall Driving & Steering
            if (car.grounded) {
                const forwardSpeed = car.vel.dot(forward);
                const reverseSign = forwardSpeed < -1.0 ? -1 : 1;
                const turnRate = PHYSICS.CAR_TURN_SPEED * (input.drift ? PHYSICS.CAR_DRIFT_TURN_MULT : 1.0);
                const speedFactor = THREE.MathUtils.clamp(Math.abs(forwardSpeed) / 10, 0.25, 1.0);
                const yawDelta = input.steer * turnRate * speedFactor * reverseSign * dt;
                const yawQuat = new THREE.Quaternion().setFromAxisAngle(car.surfaceNormal, yawDelta);
                car.quat.premultiply(yawQuat).normalize();

                const newFwd = new THREE.Vector3(0, 0, 1).applyQuaternion(car.quat).normalize();
                const newRight = new THREE.Vector3(1, 0, 0).applyQuaternion(car.quat).normalize();

                if (input.throttle > 0) {
                    car.vel.addScaledVector(newFwd, input.throttle * PHYSICS.CAR_ACCEL * dt);
                } else if (input.throttle < 0) {
                    const decel = forwardSpeed > 1.0 ? PHYSICS.CAR_BRAKE : PHYSICS.CAR_ACCEL * 0.7;
                    car.vel.addScaledVector(newFwd, input.throttle * decel * dt);
                } else {
                    car.vel.multiplyScalar(Math.pow(0.94, dt * 60));
                }

                // Lateral tire grip (looser when powersliding/drifting)
                const lateralSpeed = car.vel.dot(newRight);
                const grip = input.drift ? 0.94 : 0.76;
                car.vel.addScaledVector(newRight, -lateralSpeed * (1 - Math.pow(grip, dt * 60)));

                // Wall tire downforce adhesion so cars stay planted on vertical walls & ceiling
                if (car.surfaceNormal.y < 0.85) {
                    car.vel.addScaledVector(car.surfaceNormal, -PHYSICS.WALL_ADHESION_FORCE * dt);
                }
            } else {
                // Aerial Pitch, Yaw & Air-Roll Rotation
                const pitchQ = new THREE.Quaternion().setFromAxisAngle(right, input.pitch * PHYSICS.CAR_AIR_PITCH_SPEED * dt);
                const yawQ = new THREE.Quaternion().setFromAxisAngle(up, input.steer * PHYSICS.CAR_AIR_YAW_SPEED * dt);
                const rollQ = new THREE.Quaternion().setFromAxisAngle(forward, input.roll * PHYSICS.CAR_AIR_ROLL_SPEED * dt);
                car.quat.premultiply(pitchQ).premultiply(yawQ).premultiply(rollQ).normalize();

                // Gravity in mid-air
                car.vel.y += PHYSICS.GRAVITY * dt;
                car.vel.multiplyScalar(Math.pow(0.994, dt * 60));
            }

            // Rocket Boost Thrusters
            const isBoosting = input.boost && car.boost > 0.5;
            if (isBoosting) {
                car.boost = Math.max(0, car.boost - 31 * dt);
                const boostDir = new THREE.Vector3(0, 0, 1).applyQuaternion(car.quat).normalize();
                car.vel.addScaledVector(boostDir, PHYSICS.CAR_BOOST_ACCEL * dt);

                if (Math.random() < 0.75) {
                    const exhaustPos = car.pos.clone().addScaledVector(boostDir, -2.5);
                    const pVel = boostDir.clone().multiplyScalar(-12 - Math.random() * 8);
                    spawnParticle(exhaustPos, pVel, Math.random() > 0.4 ? 0xf97316 : 0xfde047, 0.95, 0.28);
                }
            }

            // Jump & Directional Dodge Flip
            if (input.jump) {
                if (car.grounded) {
                    car.grounded = false;
                    car.pos.addScaledVector(car.surfaceNormal, 0.35);
                    car.vel.addScaledVector(car.surfaceNormal, PHYSICS.CAR_JUMP_IMPULSE);
                    if (car === playerCar) sfx.playJump(false);
                } else if (car.hasDoubleJump) {
                    car.hasDoubleJump = false;
                    car.vel.y = Math.max(car.vel.y * 0.25, 4.5);

                    const flatFwd = new THREE.Vector3(forward.x, 0, forward.z).normalize();
                    const flatRight = new THREE.Vector3(right.x, 0, right.z).normalize();
                    const dodgeDir = new THREE.Vector3();

                    if (Math.abs(input.throttle) > 0.1 || Math.abs(input.steer) > 0.1) {
                        dodgeDir.addScaledVector(flatFwd, input.throttle);
                        dodgeDir.addScaledVector(flatRight, -input.steer);
                        dodgeDir.normalize();
                        car.vel.addScaledVector(dodgeDir, PHYSICS.CAR_DODGE_IMPULSE);
                        car.flipTimer = 0.48;
                        car.flipTorque.set(input.throttle * 12.5, 0, input.steer * 12.5);
                    } else {
                        car.vel.y += PHYSICS.CAR_JUMP_IMPULSE * 0.9;
                    }
                    if (car === playerCar) sfx.playJump(true);
                }
            }

            if (car.flipTimer > 0) {
                car.flipTimer -= dt;
                const fPitch = new THREE.Quaternion().setFromAxisAngle(right, car.flipTorque.x * dt);
                const fRoll = new THREE.Quaternion().setFromAxisAngle(forward, car.flipTorque.z * dt);
                car.quat.premultiply(fPitch).premultiply(fRoll).normalize();
            }

            // Speed Cap & Supersonic State
            const maxSpd = isBoosting ? PHYSICS.CAR_BOOST_MAX_SPEED : PHYSICS.CAR_MAX_SPEED;
            const spd = car.vel.length();
            if (spd > maxSpd) {
                car.vel.setLength(maxSpd);
            }
            car.supersonic = spd > PHYSICS.CAR_MAX_SPEED * 1.12;

            // Integrate Position & Resolve Arena Walls/Ramps
            car.pos.addScaledVector(car.vel, dt);
            resolveArenaCollision(car.pos, car.vel, rideRadius, 0.15, false);

            // Boost Pad Pickups
            for (let i = 0; i < boostPads.length; i++) {
                const pad = boostPads[i];
                if (pad.active && car.boost < 99.5) {
                    const dx = car.pos.x - pad.x;
                    const dz = car.pos.z - pad.z;
                    if (car.pos.y < 3.5 && dx * dx + dz * dz < pad.radius * pad.radius) {
                        car.boost = Math.min(100, car.boost + (pad.isFull ? 100 : 12));
                        setBoostPadActive(pad, false);
                        if (car === playerCar) sfx.playBoostPickup(pad.isFull);
                    }
                }
            }

            // Update Car Visuals, Wheels, Exhaust Flames & Shadow Decal
            car.visuals.root.position.copy(car.pos);
            car.visuals.root.quaternion.copy(car.quat);

            const wheelSpin = car.vel.dot(forward) * dt * 2.1;
            car.visuals.wheels.forEach((w, idx) => {
                w.children[0].rotation.x += wheelSpin;
                w.children[1].rotation.x += wheelSpin;
                if (idx < 2) {
                    w.rotation.y = input.steer * 0.36;
                }
            });

            car.visuals.flames.forEach(f => {
                f.visible = isBoosting;
                if (isBoosting) {
                    f.scale.set(1, 0.8 + Math.random() * 0.55, 1);
                }
            });

            car.visuals.taillightMat.color.setHex(input.throttle < 0 ? 0xff2a2a : 0x991b1b);
            car.visuals.shadowDecal.position.set(car.pos.x, 0.04, car.pos.z);
            car.visuals.shadowDecal.visible = car.pos.y < 14;
            const shadowScale = THREE.MathUtils.clamp(1 - car.pos.y / 16, 0.35, 1.0);
            car.visuals.shadowDecal.scale.setScalar(shadowScale);

            if (car === playerCar) {
                sfx.updateContinuous(spd / PHYSICS.CAR_BOOST_MAX_SPEED, isBoosting, state.mode === 'playing');
            }
        }

        // Oriented 3D Car Bounding Box vs Sphere Ball Collision (Zero Unwanted Ground Pop-Up!)
        function resolveCarBallCollision(car) {
            const invQuat = car.quat.clone().invert();
            const localBallPos = ball.pos.clone().sub(car.pos).applyQuaternion(invQuat);

            const hx = car.halfExtents.x;
            const hy = car.halfExtents.y;
            const hz = car.halfExtents.z;

            // Use taller effective vertical clearance so a grounded ball doesn't clip the top bevel of the low chassis box
            const effectiveHy = Math.max(hy, PHYSICS.BALL_RADIUS * 0.85);

            // Closest point on the car's oriented chassis box to the ball center
            const closestLocal = new THREE.Vector3(
                THREE.MathUtils.clamp(localBallPos.x, -hx, hx),
                THREE.MathUtils.clamp(localBallPos.y, -effectiveHy, effectiveHy),
                THREE.MathUtils.clamp(localBallPos.z, -hz, hz)
            );

            const deltaLocal = localBallPos.clone().sub(closestLocal);
            const distSq = deltaLocal.lengthSq();

            if (distSq < PHYSICS.BALL_RADIUS * PHYSICS.BALL_RADIUS) {
                const dist = Math.sqrt(distSq);
                let normalLocal;
                if (dist > 0.0001) {
                    normalLocal = deltaLocal.divideScalar(dist);
                } else {
                    normalLocal = new THREE.Vector3(0, 0, 1);
                }

                // Transform impact normal back to world space
                const hitNormal = normalLocal.applyQuaternion(car.quat).normalize();

                // When driving on flat turf and striking a grounded ball, keep the hit normal purely horizontal
                // so the ball shoots cleanly along the ground instead of popping straight up
                const onFlatTurf = car.grounded && car.surfaceNormal.y > 0.85;
                const ballNearGround = ball.pos.y <= PHYSICS.BALL_RADIUS + 0.65;
                if (onFlatTurf && ballNearGround && car.flipTimer <= 0) {
                    hitNormal.y = 0;
                    if (hitNormal.lengthSq() > 0.0001) {
                        hitNormal.normalize();
                    } else {
                        hitNormal.set(0, 0, 1).applyQuaternion(car.quat);
                        hitNormal.y = 0;
                        hitNormal.normalize();
                    }
                }

                const penetration = PHYSICS.BALL_RADIUS - dist;
                ball.pos.addScaledVector(hitNormal, penetration + 0.04);

                // Compute relative impact velocity
                const relVel = ball.vel.clone().sub(car.vel);
                const vn = relVel.dot(hitNormal);

                if (vn < 0) {
                    const powerMult = car.flipTimer > 0 ? PHYSICS.BALL_DODGE_HIT_BONUS : PHYSICS.BALL_HIT_MULT;
                    const impulse = -(1 + 0.48) * vn * powerMult;
                    ball.vel.addScaledVector(hitNormal, impulse);

                    // Transfer forward momentum cleanly along the shot direction
                    const carSpeed = car.vel.length();
                    if (carSpeed > 4) {
                        ball.vel.addScaledVector(hitNormal, carSpeed * 0.36);
                    }

                    // Only add upward loft if airborne, dodge-flipping, driving up a wall ramp, or hitting an airborne ball
                    if (!onFlatTurf || car.flipTimer > 0 || !ballNearGround) {
                        ball.vel.y += Math.max(0, hitNormal.y * 5.5);
                    } else {
                        ball.vel.y = 0;
                    }

                    // Impart spin torque from off-center contact
                    ball.angVel.crossVectors(hitNormal, ball.vel).multiplyScalar(0.18);

                    // Small recoil on car
                    car.vel.addScaledVector(hitNormal, -impulse * 0.14);

                    if (ball.vel.length() > PHYSICS.BALL_MAX_SPEED) {
                        ball.vel.setLength(PHYSICS.BALL_MAX_SPEED);
                    }

                    sfx.playBallHit(Math.min(1, impulse / 32));
                }
            }
        }

        function updateBallPhysics(dt) {
            // Gravity & Air Drag
            ball.vel.y += PHYSICS.BALL_GRAVITY * dt;
            ball.vel.multiplyScalar(Math.pow(PHYSICS.BALL_AIR_DRAG, dt * 60));

            // Subtle Magnus Spin Curve in Flight
            if (ball.pos.y > PHYSICS.BALL_RADIUS + 0.3 && ball.angVel.lengthSq() > 0.1) {
                const magnus = new THREE.Vector3().crossVectors(ball.angVel, ball.vel).multiplyScalar(0.0045 * dt);
                ball.vel.add(magnus);
            }

            if (ball.vel.length() > PHYSICS.BALL_MAX_SPEED) {
                ball.vel.setLength(PHYSICS.BALL_MAX_SPEED);
            }

            ball.pos.addScaledVector(ball.vel, dt);

            // Resolve Arena Ramp, Wall, Ceiling & Goal Collisions
            const col = resolveArenaCollision(ball.pos, ball.vel, PHYSICS.BALL_RADIUS, PHYSICS.BALL_BOUNCE, true);
            if (col.hit) {
                if (col.goalScored) {
                    handleGoalScored(col.goalScored);
                    return;
                }
                // Ground rolling friction & spin alignment
                if (col.normal.y > 0.7) {
                    ball.vel.x *= Math.pow(PHYSICS.BALL_GROUND_FRICTION, dt * 60);
                    ball.vel.z *= Math.pow(PHYSICS.BALL_GROUND_FRICTION, dt * 60);
                    ball.angVel.set(ball.vel.z / PHYSICS.BALL_RADIUS, 0, -ball.vel.x / PHYSICS.BALL_RADIUS);
                }
            }

            // Car vs Car bump resolution across all active cars
            for (let i = 0; i < activeCars.length; i++) {
                for (let j = i + 1; j < activeCars.length; j++) {
                    const cA = activeCars[i];
                    const cB = activeCars[j];
                    const carDelta = cA.pos.clone().sub(cB.pos);
                    const carDist = carDelta.length();
                    if (carDist < 3.2 && carDist > 0.001) {
                        carDelta.divideScalar(carDist);
                        const overlap = 3.2 - carDist;
                        cA.pos.addScaledVector(carDelta, overlap * 0.5);
                        cB.pos.addScaledVector(carDelta, -overlap * 0.5);
                        const relV = cA.vel.clone().sub(cB.vel).dot(carDelta);
                        if (relV < 0) {
                            cA.vel.addScaledVector(carDelta, -relV * 0.65);
                            cB.vel.addScaledVector(carDelta, relV * 0.65);
                        }
                    }
                }
            }

            // Car vs Ball Collisions for all active cars
            for (let i = 0; i < activeCars.length; i++) {
                resolveCarBallCollision(activeCars[i]);
            }

            // Update Ball Mesh, Ground Ring & Glow Light
            ball.mesh.position.copy(ball.pos);
            const rollAxis = new THREE.Vector3(ball.vel.z, 0, -ball.vel.x);
            if (rollAxis.lengthSq() > 0.01) {
                rollAxis.normalize();
                ball.mesh.rotateOnWorldAxis(rollAxis, (ball.vel.length() * dt) / PHYSICS.BALL_RADIUS);
            }

            ball.ring.position.set(ball.pos.x, 0.05, ball.pos.z);
            const heightScale = THREE.MathUtils.clamp(1 - (ball.pos.y - PHYSICS.BALL_RADIUS) / 40, 0.35, 1.0);
            ball.ring.scale.setScalar(heightScale);
            ball.ring.material.opacity = 0.25 + heightScale * 0.55;

            ball.glowLight.position.copy(ball.pos);
            ball.glowLight.color.setHex(ball.pos.z < -16 ? 0x38bdf8 : (ball.pos.z > 16 ? 0xf97316 : 0xfbbf24));
        }

        // =========================================================================
        // 13. MATCH FLOW, BANNERS, CAMERA CONTROLLER & HUD UPDATES
        // =========================================================================
        const bannerWrap = document.getElementById('center-banner');
        const bannerTitle = document.getElementById('banner-title');
        const bannerSub = document.getElementById('banner-sub');

        function showBanner(title, sub = '', colorClass = 'text-white') {
            bannerTitle.textContent = title;
            bannerTitle.className = `font-display text-5xl sm:text-7xl md:text-8xl font-black tracking-wider uppercase drop-shadow-[0_6px_28px_rgba(251,146,60,0.85)] ${colorClass}`;
            bannerSub.textContent = sub;
            bannerWrap.classList.remove('hidden', 'banner-animate');
            void bannerWrap.offsetWidth;
            bannerWrap.classList.add('banner-animate');
        }

        function handleGoalScored(scoringTeam) {
            if (state.mode !== 'playing') return;
            state.mode = 'goal';
            state.goalPauseTimer = 2.6;
            state.crowdCheerTimer = 2.8;
            sfx.playGoal();

            if (scoringTeam === 'blue') {
                state.scoreBlue++;
                document.getElementById('score-blue').textContent = state.scoreBlue;
                triggerGoalExplosion(ARENA.HALF_L + 3, true);
                showBanner('BLUE SCORES!', 'WHAT A SAVE! • GOAL EXPLOSION', 'text-sky-400');
            } else {
                state.scoreOrange++;
                document.getElementById('score-orange').textContent = state.scoreOrange;
                triggerGoalExplosion(-ARENA.HALF_L - 3, false);
                showBanner('ORANGE SCORES!', 'SUPERB STRIKE!', 'text-orange-400');
            }
        }

        function startMatch() {
            sfx.init();
            state.scoreBlue = 0;
            state.scoreOrange = 0;
            state.kickoffIndex = 0;
            state.timeLeft = state.matchDuration;
            state.overtime = false;
            document.getElementById('score-blue').textContent = '0';
            document.getElementById('score-orange').textContent = '0';
            document.getElementById('overtime-badge').classList.add('hidden');
            document.getElementById('hud-mode-label').textContent = `DFH STADIUM • ${state.teamSize}V${state.teamSize} SUNSET`;

            document.getElementById('menu-overlay').classList.add('hidden');
            document.getElementById('pause-overlay').classList.add('hidden');
            document.getElementById('gameover-overlay').classList.add('hidden');

            startCountdown(false);
        }

        function startCountdown(advancePreset = true) {
            resetKickoffPositions(advancePreset);
            state.mode = 'countdown';
            state.countdownTimer = 3.0;
            lastCountdownSec = 4;
        }

        function endMatch() {
            state.mode = 'gameover';
            sfx.updateContinuous(0, false, false);
            document.getElementById('final-blue').textContent = state.scoreBlue;
            document.getElementById('final-orange').textContent = state.scoreOrange;

            const blueWon = state.scoreBlue > state.scoreOrange;
            const badge = document.getElementById('winner-badge');
            const title = document.getElementById('winner-title');
            if (blueWon) {
                badge.textContent = 'CHAMPION';
                badge.className = 'inline-block px-4 py-1 rounded-full text-xs font-black tracking-widest uppercase bg-blue-500/20 text-blue-300 border border-blue-400/30 mb-2';
                title.textContent = 'BLUE TEAM WINS!';
            } else {
                badge.textContent = 'FINAL WHISTLE';
                badge.className = 'inline-block px-4 py-1 rounded-full text-xs font-black tracking-widest uppercase bg-orange-500/20 text-orange-300 border border-orange-400/30 mb-2';
                title.textContent = 'ORANGE TEAM WINS!';
            }
            document.getElementById('gameover-overlay').classList.remove('hidden');
        }

        // Camera Controller (Ball Cam, Car Cam & Attract Orbit)
        const camPos = new THREE.Vector3(0, 24, -120);
        const camLook = new THREE.Vector3(0, 4, 0);

        function updateCamera(dt, now) {
            if (state.mode === 'menu') {
                const ang = now * 0.00022;
                camPos.set(Math.cos(ang) * 116, 42 + Math.sin(ang * 2) * 6, Math.sin(ang) * 148);
                camLook.set(0, 6, 0);
                camera.position.copy(camPos);
                camera.lookAt(camLook);
                return;
            }

            const carForward = new THREE.Vector3(0, 0, 1).applyQuaternion(playerCar.quat);
            const desiredPos = new THREE.Vector3();
            const desiredLook = new THREE.Vector3();

            if (state.ballCam) {
                const carToBall = ball.pos.clone().sub(playerCar.pos);
                const horizDir = new THREE.Vector3(carToBall.x, 0, carToBall.z);
                if (horizDir.lengthSq() < 0.1) horizDir.copy(carForward);
                horizDir.normalize();

                desiredPos.copy(playerCar.pos)
                    .addScaledVector(horizDir, -11.5)
                    .add(new THREE.Vector3(0, 4.4, 0));

                desiredLook.copy(ball.pos).lerp(playerCar.pos, 0.25);
                desiredLook.y = Math.max(1.5, desiredLook.y);
            } else {
                const flatFwd = new THREE.Vector3(carForward.x, 0, carForward.z);
                if (flatFwd.lengthSq() < 0.01) flatFwd.set(0, 0, 1);
                flatFwd.normalize();

                desiredPos.copy(playerCar.pos)
                    .addScaledVector(flatFwd, -10.8)
                    .add(new THREE.Vector3(0, 4.2, 0));
                desiredLook.copy(playerCar.pos).addScaledVector(flatFwd, 12).add(new THREE.Vector3(0, 1.6, 0));
            }

            desiredPos.x = THREE.MathUtils.clamp(desiredPos.x, -ARENA.HALF_W + 3, ARENA.HALF_W - 3);
            desiredPos.z = THREE.MathUtils.clamp(desiredPos.z, -ARENA.HALF_L + 3, ARENA.HALF_L - 3);
            desiredPos.y = THREE.MathUtils.clamp(desiredPos.y, 1.8, ARENA.WALL_H - 2.5);

            camPos.lerp(desiredPos, Math.min(1, dt * 11));
            camLook.lerp(desiredLook, Math.min(1, dt * 14));
            camera.position.copy(camPos);
            camera.lookAt(camLook);
        }

        function updateHUD() {
            const boostInt = Math.round(playerCar.boost);
            document.getElementById('boost-value').textContent = boostInt;
            const circumference = 257.6;
            const offset = circumference - (playerCar.boost / 100) * circumference;
            document.getElementById('boost-arc').style.strokeDashoffset = offset.toFixed(1);

            const totalSec = Math.ceil(Math.max(0, state.timeLeft));
            const mins = Math.floor(totalSec / 60);
            const secs = totalSec % 60;
            document.getElementById('match-timer').textContent = `${mins}:${secs < 10 ? '0' : ''}${secs}`;
        }

        // =========================================================================
        // 14. UI & INPUT EVENT LISTENERS
        // =========================================================================
        function toggleBallCam() {
            state.ballCam = !state.ballCam;
            const lbl = document.getElementById('cam-label');
            const dot = document.getElementById('cam-dot');
            if (state.ballCam) {
                lbl.textContent = 'BALL CAM';
                dot.className = 'w-2 h-2 rounded-full bg-sky-400';
            } else {
                lbl.textContent = 'CAR CAM';
                dot.className = 'w-2 h-2 rounded-full bg-emerald-400';
            }
        }

        function toggleQuality() {
            state.ultraHD = !state.ultraHD;
            const ind = document.getElementById('quality-dot');
            const lbl = document.getElementById('quality-label');
            if (state.ultraHD) {
                renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
                sunLight.shadow.mapSize.set(4096, 4096);
                if (sunLight.shadow.map) { sunLight.shadow.map.dispose(); sunLight.shadow.map = null; }
                blueGoalLight.intensity = 48;
                orangeGoalLight.intensity = 48;
                ball.glowLight.intensity = 10;
                activeCars.forEach(c => { c.underglowLight.intensity = 7; });
                boostPads.forEach(p => { if (p.light) p.light.intensity = p.active ? 20 : 0; });
                ind.className = 'w-2.5 h-2.5 rounded-full bg-amber-400 shadow-[0_0_8px_#fbbf24]';
                lbl.textContent = 'ULTRA HD';
            } else {
                renderer.setPixelRatio(1);
                sunLight.shadow.mapSize.set(1024, 1024);
                if (sunLight.shadow.map) { sunLight.shadow.map.dispose(); sunLight.shadow.map = null; }
                blueGoalLight.intensity = 0;
                orangeGoalLight.intensity = 0;
                ball.glowLight.intensity = 0;
                allCars.forEach(c => { c.underglowLight.intensity = 0; });
                boostPads.forEach(p => { if (p.light) p.light.intensity = 0; });
                ind.className = 'w-2.5 h-2.5 rounded-full bg-sky-400 shadow-[0_0_8px_#38bdf8]';
                lbl.textContent = 'HIGH';
            }
        }

        function toggleMute() {
            state.muted = !state.muted;
            document.getElementById('icon-sound-on').classList.toggle('hidden', state.muted);
            document.getElementById('icon-sound-off').classList.toggle('hidden', !state.muted);
        }

        function togglePause() {
            if (state.mode === 'playing' || state.mode === 'countdown') {
                state.prevMode = state.mode;
                state.mode = 'paused';
                sfx.updateContinuous(0, false, false);
                document.getElementById('pause-overlay').classList.remove('hidden');
            } else if (state.mode === 'paused') {
                state.mode = state.prevMode || 'playing';
                document.getElementById('pause-overlay').classList.add('hidden');
            }
        }

        window.addEventListener('keydown', (e) => {
            if (['Space', 'ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'Tab'].includes(e.code)) {
                e.preventDefault();
            }
            switch (e.code) {
                case 'KeyW': case 'ArrowUp': keys.forward = true; break;
                case 'KeyS': case 'ArrowDown': keys.backward = true; break;
                case 'KeyA': case 'ArrowLeft': keys.left = true; break;
                case 'KeyD': case 'ArrowRight': keys.right = true; break;
                case 'ShiftLeft': case 'ShiftRight': keys.boost = true; break;
                case 'Space':
                    if (!keys.jump) jumpPressedEdge = true;
                    keys.jump = true;
                    break;
                case 'ControlLeft': case 'ControlRight': keys.drift = true; break;
                case 'KeyQ': keys.rollLeft = true; break;
                case 'KeyE': keys.rollRight = true; break;
                case 'KeyC': case 'Tab': toggleBallCam(); break;
                case 'KeyM': toggleMute(); break;
                case 'Escape': togglePause(); break;
            }
        });

        window.addEventListener('keyup', (e) => {
            switch (e.code) {
                case 'KeyW': case 'ArrowUp': keys.forward = false; break;
                case 'KeyS': case 'ArrowDown': keys.backward = false; break;
                case 'KeyA': case 'ArrowLeft': keys.left = false; break;
                case 'KeyD': case 'ArrowRight': keys.right = false; break;
                case 'ShiftLeft': case 'ShiftRight': keys.boost = false; break;
                case 'Space': keys.jump = false; break;
                case 'ControlLeft': case 'ControlRight': keys.drift = false; break;
                case 'KeyQ': keys.rollLeft = false; break;
                case 'KeyE': keys.rollRight = false; break;
            }
        });

        // Mobile Touch Bindings
        function bindHoldButton(id, keyName, isEdge = false) {
            const el = document.getElementById(id);
            if (!el) return;
            const down = (e) => {
                e.preventDefault();
                sfx.init();
                if (isEdge && !keys[keyName]) jumpPressedEdge = true;
                keys[keyName] = true;
            };
            const up = (e) => {
                e.preventDefault();
                keys[keyName] = false;
            };
            el.addEventListener('touchstart', down, { passive: false });
            el.addEventListener('touchend', up, { passive: false });
            el.addEventListener('touchcancel', up, { passive: false });
            el.addEventListener('mousedown', down);
            el.addEventListener('mouseup', up);
            el.addEventListener('mouseleave', up);
        }

        bindHoldButton('touch-left', 'left');
        bindHoldButton('touch-right', 'right');
        bindHoldButton('touch-gas', 'forward');
        bindHoldButton('touch-brake', 'backward');
        bindHoldButton('touch-boost', 'boost');
        bindHoldButton('touch-jump', 'jump', true);
        bindHoldButton('touch-roll-l', 'rollLeft');
        bindHoldButton('touch-roll-r', 'rollRight');

        // HUD & Menu Button Clicks
        document.getElementById('btn-cam').addEventListener('click', toggleBallCam);
        document.getElementById('btn-quality').addEventListener('click', toggleQuality);
        document.getElementById('btn-mute').addEventListener('click', toggleMute);
        document.getElementById('btn-pause').addEventListener('click', togglePause);
        document.getElementById('btn-play').addEventListener('click', startMatch);
        document.getElementById('btn-rematch').addEventListener('click', startMatch);
        document.getElementById('btn-resume').addEventListener('click', togglePause);
        document.getElementById('btn-reset-kickoff').addEventListener('click', () => {
            document.getElementById('pause-overlay').classList.add('hidden');
            startCountdown(false);
        });
        document.getElementById('btn-quit-menu').addEventListener('click', () => {
            state.mode = 'menu';
            sfx.updateContinuous(0, false, false);
            document.getElementById('pause-overlay').classList.add('hidden');
            document.getElementById('menu-overlay').classList.remove('hidden');
        });
        document.getElementById('btn-menu-end').addEventListener('click', () => {
            state.mode = 'menu';
            document.getElementById('gameover-overlay').classList.add('hidden');
            document.getElementById('menu-overlay').classList.remove('hidden');
        });

        // Team Mode, Difficulty & Duration Selectors
        document.querySelectorAll('.mode-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                state.teamSize = parseInt(btn.dataset.size, 10);
                document.querySelectorAll('.mode-btn').forEach(b => {
                    b.className = 'mode-btn rounded-xl py-2 text-xs font-extrabold tracking-wide bg-slate-800 text-slate-300 hover:bg-slate-700';
                });
                btn.className = 'mode-btn rounded-xl py-2 text-xs font-extrabold tracking-wide bg-gradient-to-r from-indigo-600 to-blue-500 text-white shadow';
                document.getElementById('hud-mode-label').textContent = `DFH STADIUM • ${state.teamSize}V${state.teamSize} SUNSET`;
                resetKickoffPositions(false);
            });
        });

        document.querySelectorAll('.diff-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                state.difficulty = btn.dataset.diff;
                document.querySelectorAll('.diff-btn').forEach(b => {
                    b.className = 'diff-btn rounded-xl py-2 text-xs font-bold bg-slate-800 text-slate-300 hover:bg-slate-700';
                });
                btn.className = 'diff-btn rounded-xl py-2 text-xs font-bold bg-gradient-to-r from-blue-600 to-sky-500 text-white shadow';
            });
        });

        document.querySelectorAll('.time-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                state.matchDuration = parseInt(btn.dataset.time, 10);
                document.querySelectorAll('.time-btn').forEach(b => {
                    b.className = 'time-btn rounded-xl py-2 text-xs font-bold bg-slate-800 text-slate-300 hover:bg-slate-700';
                });
                btn.className = 'time-btn rounded-xl py-2 text-xs font-bold bg-gradient-to-r from-orange-500 to-amber-500 text-white shadow';
            });
        });

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            if (composer) composer.setSize(window.innerWidth, window.innerHeight);
        });

        // =========================================================================
        // 15. INITIALIZE WORLD & MAIN ANIMATION LOOP
        // =========================================================================
        buildDFHStadium();
        initBoostPads();
        initBall();
        resetKickoffPositions(false);

        const dummyCrowdObj = new THREE.Object3D();
        let lastFrameTime = performance.now();

        function animate(now) {
            requestAnimationFrame(animate);

            const dt = Math.min(0.05, (now - lastFrameTime) / 1000);
            lastFrameTime = now;

            // Animate Boost Pads (Orb hover, gyroscopic rings & respawns)
            for (let i = 0; i < boostPads.length; i++) {
                const pad = boostPads[i];
                if (!pad.active) {
                    pad.cooldown -= dt;
                    if (pad.cooldown <= 0) {
                        setBoostPadActive(pad, true);
                    }
                } else {
                    const t = now * 0.003 + pad.phase;
                    if (pad.isFull) {
                        pad.orb.position.y = 1.85 + Math.sin(t) * 0.18;
                        pad.ring1.rotation.y = t * 1.4;
                        pad.ring2.rotation.z = -t * 1.6;
                    } else {
                        pad.orb.position.y = 0.55 + Math.sin(t * 1.4) * 0.06;
                        pad.ring1.rotation.z = t * 1.5;
                    }
                }
            }

            // Animate Cheering Spectators after Goals
            if (state.crowdCheerTimer > 0 && crowdInstancedMesh) {
                state.crowdCheerTimer -= dt;
                for (let i = 0; i < crowdBaseMatrices.length; i++) {
                    const b = crowdBaseMatrices[i];
                    const bounce = Math.abs(Math.sin(now * 0.012 + crowdPhases[i])) * 0.85;
                    dummyCrowdObj.position.set(b.x, b.y + bounce, b.z);
                    dummyCrowdObj.rotation.set(0, b.ry, 0);
                    dummyCrowdObj.updateMatrix();
                    crowdInstancedMesh.setMatrixAt(i, dummyCrowdObj.matrix);
                }
                crowdInstancedMesh.instanceMatrix.needsUpdate = true;
            }

            // Update Active Particles
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.life -= dt;
                if (p.life <= 0) {
                    scene.remove(p.mesh);
                    p.mesh.material.dispose();
                    particles.splice(i, 1);
                } else {
                    p.mesh.position.addScaledVector(p.vel, dt);
                    const ratio = p.life / p.maxLife;
                    p.mesh.scale.setScalar(p.baseScale * ratio);
                    p.mesh.material.opacity = ratio * 0.85;
                }
            }

            // Match State Machine
            if (state.mode === 'countdown') {
                state.countdownTimer -= dt;
                const sec = Math.ceil(state.countdownTimer);
                if (sec < lastCountdownSec && sec >= 1) {
                    lastCountdownSec = sec;
                    sfx.playCountdown(false);
                    showBanner(String(sec), `${state.teamSize}V${state.teamSize} KICKOFF`, 'text-amber-300');
                }
                if (state.countdownTimer <= 0) {
                    state.mode = 'playing';
                    sfx.playCountdown(true);
                    showBanner('GO!', 'SUPERSONIC KICKOFF', 'text-emerald-400');
                }
            } else if (state.mode === 'playing') {
                for (let i = 0; i < activeCars.length; i++) {
                    updateCarPhysics(activeCars[i], dt);
                }
                updateBallPhysics(dt);

                if (!state.overtime) {
                    state.timeLeft -= dt;
                    if (state.timeLeft <= 0) {
                        state.timeLeft = 0;
                        if (state.scoreBlue === state.scoreOrange) {
                            state.overtime = true;
                            document.getElementById('overtime-badge').classList.remove('hidden');
                            showBanner('OVERTIME!', 'NEXT GOAL WINS', 'text-rose-400');
                            startCountdown(true);
                        } else {
                            endMatch();
                        }
                    }
                } else {
                    state.timeLeft += dt;
                }
            } else if (state.mode === 'goal') {
                state.goalPauseTimer -= dt;
                for (let i = 0; i < activeCars.length; i++) {
                    updateCarPhysics(activeCars[i], dt * 0.45);
                }
                if (state.goalPauseTimer <= 0) {
                    if (state.overtime) {
                        endMatch();
                    } else {
                        startCountdown(true);
                    }
                }
            }

            updateCamera(dt, now);
            updateHUD();

            if (composer && state.ultraHD) {
                composer.render();
            } else {
                renderer.render(scene, camera);
            }
        }

        window.addEventListener('load', () => {
            lastFrameTime = performance.now();
            requestAnimationFrame(animate);
        });
    })();
    </script>
</body>
</html>
