<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Dawn Ops - 3D Tactical Action Shooter</title>
    <!-- Tailwind CSS for modern responsive UI overlay -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Three.js Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #050505;
            font-family: 'Rajdhani', sans-serif;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none;
        }
        #canvas-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }
        .hud-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none;
        }
        .interactive-ui {
            pointer-events: auto;
        }
        /* Custom crosshair styling */
        .crosshair {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 24px;
            height: 24px;
            pointer-events: none;
            z-index: 15;
            transition: transform 0.1s ease;
        }
        .crosshair-dot {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 4px;
            height: 4px;
            background-color: rgba(255, 230, 0, 0.9);
            border-radius: 50%;
            box-shadow: 0 0 6px rgba(255, 180, 0, 0.8);
        }
        .crosshair-line {
            position: absolute;
            background-color: rgba(255, 255, 255, 0.8);
            box-shadow: 0 0 4px rgba(255, 200, 0, 0.5);
        }
        .crosshair-top { top: 0; left: 10px; width: 4px; height: 7px; }
        .crosshair-bottom { bottom: 0; left: 10px; width: 4px; height: 7px; }
        .crosshair-left { top: 10px; left: 0; width: 7px; height: 4px; }
        .crosshair-right { top: 10px; right: 0; width: 7px; height: 4px; }

        /* Joystick styles */
        .joystick-base {
            position: absolute;
            bottom: 40px;
            left: 40px;
            width: 130px;
            height: 130px;
            background: rgba(255, 255, 255, 0.1);
            border: 2px solid rgba(255, 255, 255, 0.25);
            border-radius: 50%;
            z-index: 20;
            touch-action: none;
            backdrop-filter: blur(4px);
        }
        .joystick-knob {
            position: absolute;
            top: 35px;
            left: 35px;
            width: 60px;
            height: 60px;
            background: linear-gradient(135deg, rgba(255, 180, 0, 0.8), rgba(200, 100, 0, 0.9));
            border: 2px solid rgba(255, 255, 255, 0.6);
            border-radius: 50%;
            box-shadow: 0 4px 15px rgba(0,0,0,0.4);
            pointer-events: none;
            transition: transform 0.05s linear;
        }
        /* Hit damage vignette */
        #damage-vignette {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            box-shadow: inset 0 0 100px rgba(220, 20, 20, 0);
            z-index: 12;
            pointer-events: none;
            transition: box-shadow 0.3s ease;
        }
        /* Scanline / HUD grid decoration */
        .hud-panel {
            background: linear-gradient(135deg, rgba(15, 23, 42, 0.75), rgba(30, 41, 59, 0.6));
            border: 1px solid rgba(255, 180, 0, 0.3);
            backdrop-filter: blur(8px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
        }
        /* Animations */
        @keyframes pulseGlow {
            0%, 100% { opacity: 0.8; transform: scale(1); }
            50% { opacity: 1; transform: scale(1.02); }
        }
        .pulse-btn {
            animation: pulseGlow 2s infinite ease-in-out;
        }
    </style>
</head>
<body class="bg-gray-950 text-white select-none overflow-hidden">

    <!-- ThreeJS WebGL Canvas Container -->
    <div id="canvas-container"></div>

    <!-- Damage / Hit Vignette Overlay -->
    <div id="damage-vignette"></div>

    <!-- Main HUD Layer -->
    <div class="hud-layer flex flex-col justify-between p-4 sm:p-6">
        <!-- Top Bar: Stats & Objectives -->
        <div class="flex justify-between items-start w-full max-w-7xl mx-auto">
            <!-- Player Health & Shield -->
            <div class="hud-panel rounded-xl p-3 sm:p-4 w-48 sm:w-64">
                <div class="flex justify-between text-xs sm:text-sm font-bold mb-1 tracking-wider text-amber-400">
                    <span>AGENT VITALS</span>
                    <span id="health-text">100%</span>
                </div>
                <div class="w-full bg-gray-900 h-3 rounded-full overflow-hidden p-0.5 border border-amber-500/30">
                    <div id="health-bar" class="bg-gradient-to-r from-amber-600 to-amber-400 h-full rounded-full transition-all duration-300" style="width: 100%;"></div>
                </div>
                <div class="mt-2 text-[10px] sm:text-xs text-gray-400 flex justify-between">
                    <span>ARMOR: <strong class="text-amber-300">ACTIVE</strong></span>
                    <span>ZONE: <strong class="text-emerald-400">SECTOR 7</strong></span>
                </div>
            </div>

            <!-- Wave & Score Info -->
            <div class="hud-panel rounded-xl px-4 sm:px-6 py-2 sm:py-3 text-center">
                <div class="text-[10px] sm:text-xs tracking-widest text-amber-400 font-bold uppercase">Mission Wave</div>
                <div id="wave-display" class="text-2xl sm:text-3xl font-extrabold tracking-wider text-white">WAVE 1</div>
                <div class="text-[10px] sm:text-xs text-gray-400 mt-0.5">HOSTILES ELIMINATED: <span id="score-display" class="text-amber-400 font-bold">0</span></div>
            </div>

            <!-- Tactical Radar / Ammo Counter -->
            <div class="hud-panel rounded-xl p-3 sm:p-4 text-right w-40 sm:w-56">
                <div class="text-[10px] sm:text-xs font-bold tracking-wider text-amber-400 mb-1">ASSAULT RIFLE MK.II</div>
                <div class="text-2xl sm:text-3xl font-black text-white"><span id="ammo-display" class="text-amber-400">30</span><span class="text-sm text-gray-500">/120</span></div>
                <div id="reload-indicator" class="text-[10px] text-amber-300 font-bold tracking-widest mt-1 hidden animate-pulse">RELOADING...</div>
            </div>
        </div>

        <!-- Center Crosshair -->
        <div class="crosshair" id="crosshair-element">
            <div class="crosshair-dot"></div>
            <div class="crosshair-line crosshair-top"></div>
            <div class="crosshair-line crosshair-bottom"></div>
            <div class="crosshair-line crosshair-left"></div>
            <div class="crosshair-line crosshair-right"></div>
        </div>

        <!-- Bottom Bar: Controls & Action Buttons -->
        <div class="flex justify-between items-end w-full max-w-7xl mx-auto pb-4">
            <!-- Mobile Virtual Joystick (Left) -->
            <div id="virtual-joystick" class="joystick-base interactive-ui sm:hidden">
                <div id="joystick-knob" class="joystick-knob"></div>
            </div>
            <div class="hidden sm:block text-xs text-gray-400 bg-black/40 px-3 py-2 rounded-lg border border-white/10 backdrop-blur-sm">
                <span class="text-amber-400 font-bold">CONTROLS:</span> WASD / Arrows to Move | Mouse Look | Left Click to Fire | R to Reload | Space to Jump
            </div>

            <!-- Action Buttons (Right) -->
            <div class="flex items-end gap-3 interactive-ui">
                <!-- Scope / Aim Button -->
                <button id="btn-aim" class="hud-panel w-14 h-14 sm:w-16 sm:h-16 rounded-2xl flex flex-col items-center justify-center text-amber-400 font-bold hover:bg-amber-500/20 active:scale-95 transition-all shadow-lg border border-amber-500/40">
                    <svg class="w-6 h-6 sm:w-7 sm:h-7" fill="none" stroke="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="9" stroke-width="2"/><circle cx="12" cy="12" r="3" stroke-width="2"/><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v3m0 12v3M3 12h3m12 0h3"/></svg>
                    <span class="text-[9px] mt-0.5">AIM</span>
                </button>
                <!-- Reload Button -->
                <button id="btn-reload" class="hud-panel w-14 h-14 sm:w-16 sm:h-16 rounded-2xl flex flex-col items-center justify-center text-amber-400 font-bold hover:bg-amber-500/20 active:scale-95 transition-all shadow-lg border border-amber-500/40">
                    <svg class="w-6 h-6 sm:w-7 sm:h-7" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"/></svg>
                    <span class="text-[9px] mt-0.5">RELOAD</span>
                </button>
                <!-- Primary Fire Button -->
                <button id="btn-fire" class="w-20 h-20 sm:w-24 sm:h-24 bg-gradient-to-br from-amber-500 via-amber-600 to-orange-700 rounded-3xl flex flex-col items-center justify-center text-white font-black shadow-2xl active:scale-90 border-2 border-amber-300/60 pulse-btn">
                    <svg class="w-9 h-9 sm:w-11 sm:h-11" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"/></svg>
                    <span class="text-[10px] tracking-widest mt-0.5">FIRE</span>
                </button>
            </div>
        </div>
    </div>

    <!-- Start / Pause / Game Over Menu Modal -->
    <div id="game-modal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/85 backdrop-blur-md transition-opacity duration-300">
        <div class="hud-panel p-8 sm:p-10 rounded-3xl max-w-md w-full mx-4 text-center border border-amber-500/40 shadow-2xl">
            <div class="inline-block p-3 bg-amber-500/20 rounded-2xl mb-4 border border-amber-500/30 text-amber-400">
                <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"/></svg>
            </div>
            <h1 id="modal-title" class="text-3xl sm:text-4xl font-extrabold tracking-wider text-white mb-2">DAWN OPS: SECTOR 7</h1>
            <p id="modal-desc" class="text-gray-300 text-sm sm:text-base mb-6">Eliminate hostile insurgent squads in realistic morning patrol sectors. Survive incoming waves.</p>
            
            <div id="final-stats" class="hidden mb-6 p-4 bg-black/40 rounded-xl border border-white/10 text-sm">
                <div class="text-gray-400">MISSION DEBRIEF</div>
                <div class="text-xl font-bold text-amber-400 mt-1" id="stat-summary">WAVE 1 REACHED | 0 ELIMINATED</div>
            </div>

            <button id="start-btn" class="w-full py-4 bg-gradient-to-r from-amber-500 via-amber-600 to-orange-600 hover:from-amber-400 hover:to-orange-500 text-white font-extrabold text-lg tracking-widest rounded-2xl shadow-xl transition-all active:scale-98 border border-amber-300/50">
                DEPLOY TO COMBAT
            </button>
        </div>
    </div>

    <!-- Web Audio & Game Logic -->
    <script>
        // --- AUDIO GENERATOR (Procedural Web Audio API for immersive sound effects) ---
        class SoundEngine {
            constructor() {
                this.ctx = null;
            }
            init() {
                if (!this.ctx) {
                    const AudioContext = window.AudioContext || window.webkitAudioContext;
                    this.ctx = new AudioContext();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }
            playGunshot() {
                this.init();
                if (!this.ctx) return;
                const t = this.ctx.currentTime;
                // Noise buffer for gunshot blast
                const bufferSize = this.ctx.sampleRate * 0.3;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = Math.random() * 2 - 1;
                }
                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(800, t);
                filter.frequency.exponentialRampToValueAtTime(100, t + 0.3);

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(1.0, t);
                gain.gain.exponentialRampToValueAtTime(0.01, t + 0.3);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);

                noise.start(t);
                noise.stop(t + 0.3);
            }
            playHit() {
                this.init();
                if (!this.ctx) return;
                const t = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(300, t);
                osc.frequency.exponentialRampToValueAtTime(80, t + 0.15);

                gain.gain.setValueAtTime(0.5, t);
                gain.gain.exponentialRampToValueAtTime(0.01, t + 0.15);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(t);
                osc.stop(t + 0.15);
            }
            playReload() {
                this.init();
                if (!this.ctx) return;
                const t = this.ctx.currentTime;
                // Mechanical click sounds
                [0, 0.4, 0.9].forEach((delay, idx) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(idx === 1 ? 1200 : 800, t + delay);
                    gain.gain.setValueAtTime(0.3, t + delay);
                    gain.gain.exponentialRampToValueAtTime(0.01, t + delay + 0.08);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(t + delay);
                    osc.stop(t + delay + 0.08);
                });
            }
            playDamage() {
                this.init();
                if (!this.ctx) return;
                const t = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(150, t);
                osc.frequency.linearRampToValueAtTime(40, t + 0.4);

                gain.gain.setValueAtTime(0.8, t);
                gain.gain.exponentialRampToValueAtTime(0.01, t + 0.4);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(t);
                osc.stop(t + 0.4);
            }
        }
        const soundEngine = new SoundEngine();

        // --- GAME STATE & CONFIGURATION ---
        const gameState = {
            isPlaying: false,
            health: 100,
            maxHealth: 100,
            score: 0,
            wave: 1,
            ammo: 30,
            maxAmmo: 30,
            isReloading: false,
            isAiming: false,
            enemiesRemaining: 0
        };

        // --- THREE.JS SCENE SETUP ---
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0xfff0dd); // Warm morning sky tint
        scene.fog = new THREE.FogExp2(0xffe5cc, 0.015); // Realistic morning mist/haze

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 2.2, 10);

        const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        container.appendChild(renderer.domElement);

        // --- LIGHTING (Morning Atmosphere) ---
        const ambientLight = new THREE.AmbientLight(0xffeedd, 0.6);
        scene.add(ambientLight);

        const sunLight = new THREE.DirectionalLight(0xffa54f, 1.8);
        sunLight.position.set(50, 40, -50);
        sunLight.castShadow = true;
        sunLight.shadow.mapSize.width = 2048;
        sunLight.shadow.mapSize.height = 2048;
        sunLight.shadow.camera.near = 0.5;
        sunLight.shadow.camera.far = 250;
        const d = 80;
        sunLight.shadow.camera.left = -d;
        sunLight.shadow.camera.right = d;
        sunLight.shadow.camera.top = d;
        sunLight.shadow.camera.bottom = -d;
        sunLight.shadow.bias = -0.0005;
        scene.add(sunLight);

        // Sky dome gradient simulation
        const skyGeo = new THREE.SphereGeometry(500, 32, 15);
        const skyMat = new THREE.MeshBasicMaterial({ color: 0xffd1a9, side: THREE.BackSide });
        const sky = new THREE.Mesh(skyGeo, skyMat);
        scene.add(sky);

        // --- ENVIRONMENT: TERRAIN & COVER STRUCTURES ---
        // Ground plane
        const groundGeo = new THREE.PlaneGeometry(400, 400, 64, 64);
        // Slight height variations for realism
        const pos = groundGeo.attributes.position;
        for (let i = 0; i < pos.count; i++) {
            const vx = pos.getX(i);
            const vy = pos.getY(i);
            const z = Math.sin(vx * 0.05) * Math.cos(vy * 0.05) * 0.8;
            pos.setZ(i, z);
        }
        groundGeo.computeVertexNormals();
        const groundMat = new THREE.MeshStandardMaterial({ 
            color: 0x3d4e33, 
            roughness: 0.9,
            metalness: 0.1
        });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        // Procedural Buildings & Tactical Cover Obstacles
        const obstacles = [];
        const obstacleMat = new THREE.MeshStandardMaterial({ color: 0x8c7b6d, roughness: 0.8 });
        const concreteMat = new THREE.MeshStandardMaterial({ color: 0x6e6e6e, roughness: 0.6 });

        function createBuilding(x, z, width, height, depth) {
            const group = new THREE.Group();
            const bGeo = new THREE.BoxGeometry(width, height, depth);
            const bMesh = new THREE.Mesh(bGeo, obstacleMat);
            bMesh.position.y = height / 2;
            bMesh.castShadow = true;
            bMesh.receiveShadow = true;
            group.add(bMesh);

            // Add roof detail
            const rGeo = new THREE.BoxGeometry(width + 0.4, 0.4, depth + 0.4);
            const rMesh = new THREE.Mesh(rGeo, concreteMat);
            rMesh.position.y = height + 0.2;
            rMesh.castShadow = true;
            group.add(rMesh);

            group.position.set(x, 0, z);
            scene.add(group);
            obstacles.push({ x, z, radius: Math.max(width, depth) / 2 + 1 });
        }

        // Spawn tactical buildings around arena
        createBuilding(-25, -25, 10, 8, 12);
        createBuilding(25, -30, 12, 10, 10);
        createBuilding(-30, 25, 8, 6, 14);
        createBuilding(28, 25, 14, 9, 10);
        createBuilding(0, -45, 16, 7, 8);

        // Concrete Jersey barriers for cover
        for (let i = 0; i < 12; i++) {
            const angle = (i / 12) * Math.PI * 2;
            const dist = 18 + (i % 2) * 6;
            const bx = Math.cos(angle) * dist;
            const bz = Math.sin(angle) * dist;
            const barrierGeo = new THREE.BoxGeometry(3, 1.2, 1);
            const barrierMesh = new THREE.Mesh(barrierGeo, concreteMat);
            barrierMesh.position.set(bx, 0.6, bz);
            barrierMesh.rotation.y = angle;
            barrierMesh.castShadow = true;
            barrierMesh.receiveShadow = true;
            scene.add(barrierMesh);
            obstacles.push({ x: bx, z: bz, radius: 2 });
        }

        // --- PLAYER WEAPON VIEW MODEL ---
        const weaponGroup = new THREE.Group();
        // Rifle body
        const rifleBodyMat = new THREE.MeshStandardMaterial({ color: 0x1f2421, roughness: 0.4, metalness: 0.8 });
        const rifleWoodMat = new THREE.MeshStandardMaterial({ color: 0x5c3a21, roughness: 0.7 });
        
        const receiverGeo = new THREE.BoxGeometry(0.12, 0.18, 0.8);
        const receiver = new THREE.Mesh(receiverGeo, rifleBodyMat);
        weaponGroup.add(receiver);

        const barrelGeo = new THREE.CylinderGeometry(0.03, 0.03, 0.9, 12);
        barrelGeo.rotateX(Math.PI / 2);
        const barrel = new THREE.Mesh(barrelGeo, rifleBodyMat);
        barrel.position.set(0, 0.04, -0.7);
        weaponGroup.add(barrel);

        const stockGeo = new THREE.BoxGeometry(0.1, 0.15, 0.5);
        const stock = new THREE.Mesh(stockGeo, rifleWoodMat);
        stock.position.set(0, -0.02, 0.55);
        weaponGroup.add(stock);

        const magGeo = new THREE.BoxGeometry(0.08, 0.35, 0.15);
        const mag = new THREE.Mesh(magGeo, rifleBodyMat);
        mag.position.set(0, -0.25, -0.15);
        mag.rotation.x = 0.2;
        weaponGroup.add(mag);

        // Position weapon relative to camera (First-person view)
        weaponGroup.position.set(0.35, -0.3, -0.6);
        camera.add(weaponGroup);
        scene.add(camera);

        // Muzzle Flash Particle
        const flashGeo = new THREE.SphereGeometry(0.08, 8, 8);
        const flashMat = new THREE.MeshBasicMaterial({ color: 0xffea00 });
        const muzzleFlash = new THREE.Mesh(flashGeo, flashMat);
        muzzleFlash.position.set(0, 0.04, -1.2);
        muzzleFlash.visible = false;
        weaponGroup.add(muzzleFlash);

        // --- ENEMY SYSTEM (Human Figure Tactical Insurgents) ---
        const enemies = [];
        const enemyMaterials = {
            skin: new THREE.MeshStandardMaterial({ color: 0xd4a373, roughness: 0.8 }),
            uniform: new THREE.MeshStandardMaterial({ color: 0x2b3a28, roughness: 0.7 }), // Olive tactical gear
            vest: new THREE.MeshStandardMaterial({ color: 0x1a1a1a, roughness: 0.5, metalness: 0.3 }),
            boots: new THREE.MeshStandardMaterial({ color: 0x111111, roughness: 0.9 })
        };

        function createHumanEnemy() {
            const enemyGroup = new THREE.Group();

            // Torso & Tactical Vest
            const torsoGeo = new THREE.BoxGeometry(0.6, 0.8, 0.35);
            const torso = new THREE.Mesh(torsoGeo, enemyMaterials.uniform);
            torso.position.y = 1.1;
            torso.castShadow = true;
            enemyGroup.add(torso);

            const vestGeo = new THREE.BoxGeometry(0.62, 0.5, 0.38);
            const vest = new THREE.Mesh(vestGeo, enemyMaterials.vest);
            vest.position.y = 1.15;
            vest.castShadow = true;
            enemyGroup.add(vest);

            // Head & Helmet
            const headGeo = new THREE.SphereGeometry(0.2, 12, 12);
            const head = new THREE.Mesh(headGeo, enemyMaterials.skin);
            head.position.y = 1.75;
            head.castShadow = true;
            enemyGroup.add(head);

            const helmetGeo = new THREE.SphereGeometry(0.22, 12, 12, 0, Math.PI * 2, 0, Math.PI * 1.1);
            const helmet = new THREE.Mesh(helmetGeo, enemyMaterials.uniform);
            helmet.position.y = 1.8;
            helmet.castShadow = true;
            enemyGroup.add(helmet);

            // Limbs (Legs & Arms)
            const legGeo = new THREE.CylinderGeometry(0.12, 0.1, 0.9, 8);
            
            const leftLeg = new THREE.Mesh(legGeo, enemyMaterials.boots);
            leftLeg.position.set(-0.18, 0.45, 0);
            leftLeg.castShadow = true;
            enemyGroup.add(leftLeg);

            const rightLeg = new THREE.Mesh(legGeo, enemyMaterials.boots);
            rightLeg.position.set(0.18, 0.45, 0);
            rightLeg.castShadow = true;
            enemyGroup.add(rightLeg);

            const armGeo = new THREE.CylinderGeometry(0.09, 0.08, 0.8, 8);
            const leftArm = new THREE.Mesh(armGeo, enemyMaterials.uniform);
            leftArm.position.set(-0.38, 1.1, 0);
            leftArm.rotation.z = 0.2;
            enemyGroup.add(leftArm);

            const rightArm = new THREE.Mesh(armGeo, enemyMaterials.uniform);
            rightArm.position.set(0.38, 1.1, 0);
            rightArm.rotation.z = -0.2;
            enemyGroup.add(rightArm);

            // Spawn at random perimeter distance
            const angle = Math.random() * Math.PI * 2;
            const dist = 35 + Math.random() * 25;
            const ex = Math.cos(angle) * dist;
            const ez = Math.sin(angle) * dist;

            enemyGroup.position.set(ex, 0, ez);
            scene.add(enemyGroup);

            return {
                mesh: enemyGroup,
                health: 100,
                maxHealth: 100,
                speed: 0.04 + Math.random() * 0.03,
                attackCooldown: 0,
                state: 'advance' // 'advance', 'patrol', 'hit'
            };
        }

        function spawnWave() {
            const count = 3 + gameState.wave * 2;
            gameState.enemiesRemaining = count;
            for (let i = 0; i < count; i++) {
                enemies.push(createHumanEnemy());
            }
        }

        // --- CONTROLS & INPUT MANAGEMENT ---
        const keys = { w: false, a: false, s: false, d: false, space: false };
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false;
        
        // Mouse Look setup
        let pitch = 0;
        let yaw = 0;
        let isPointerLocked = false;

        document.addEventListener('keydown', (e) => {
            if (!gameState.isPlaying) return;
            if (e.code === 'KeyW' || e.code === 'ArrowUp') moveForward = true;
            if (e.code === 'KeyS' || e.code === 'ArrowDown') moveBackward = true;
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') moveLeft = true;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') moveRight = true;
            if (e.code === 'KeyR') reloadWeapon();
            if (e.code === 'Space') triggerJump();
        });

        document.addEventListener('keyup', (e) => {
            if (e.code === 'KeyW' || e.code === 'ArrowUp') moveForward = false;
            if (e.code === 'KeyS' || e.code === 'ArrowDown') moveBackward = false;
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') moveLeft = false;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') moveRight = false;
        });

        // Mouse look on desktop
        renderer.domElement.addEventListener('click', () => {
            if (gameState.isPlaying && !isPointerLocked) {
                renderer.domElement.requestPointerLock();
            }
        });

        document.addEventListener('pointerlockchange', () => {
            isPointerLocked = document.pointerLockElement === renderer.domElement;
        });

        document.addEventListener('mousemove', (e) => {
            if (!gameState.isPlaying || !isPointerLocked) return;
            const sensitivity = 0.002;
            yaw -= e.movementX * sensitivity;
            pitch -= e.movementY * sensitivity;
            pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, pitch));
        });

        // --- TOUCH / VIRTUAL JOYSTICK (MOBILE) ---
        let joystickActive = false;
        let joystickCenter = { x: 0, y: 0 };
        let joystickVector = { x: 0, y: 0 };

        const joystickEl = document.getElementById('virtual-joystick');
        const knobEl = document.getElementById('joystick-knob');

        joystickEl.addEventListener('touchstart', (e) => {
            joystickActive = true;
            const rect = joystickEl.getBoundingClientRect();
            joystickCenter = { x: rect.left + rect.width / 2, y: rect.top + rect.height / 2 };
            handleTouchMove(e.touches[0]);
        });

        window.addEventListener('touchmove', (e) => {
            if (!joystickActive) return;
            for (let i = 0; i < e.touches.length; i++) {
                const touch = e.touches[i];
                // Check if touch is near left bottom
                if (touch.clientX < window.innerWidth / 2) {
                    handleTouchMove(touch);
                }
            }
        });

        window.addEventListener('touchend', (e) => {
            if (joystickActive && e.touches.length === 0) {
                joystickActive = false;
                joystickVector = { x: 0, y: 0 };
                knobEl.style.transform = `translate(0px, 0px)`;
            }
        });

        function handleTouchMove(touch) {
            const dx = touch.clientX - joystickCenter.x;
            const dy = touch.clientY - joystickCenter.y;
            const dist = Math.sqrt(dx * dx + dy * dy);
            const maxRadius = 45;

            const angle = Math.atan2(dy, dx);
            const clampedDist = Math.min(dist, maxRadius);

            const kx = Math.cos(angle) * clampedDist;
            const ky = Math.sin(angle) * clampedDist;
            knobEl.style.transform = `translate(${kx}px, ${ky}px)`;

            joystickVector.x = kx / maxRadius;
            joystickVector.y = ky / maxRadius;
        }

        // Look around with touch swipe on right side of screen
        let touchLookPrev = { x: 0, y: 0 };
        renderer.domElement.addEventListener('touchstart', (e) => {
            const touch = e.touches[0];
            if (touch.clientX > window.innerWidth / 3) {
                touchLookPrev = { x: touch.clientX, y: touch.clientY };
            }
        });

        renderer.domElement.addEventListener('touchmove', (e) => {
            for (let i = 0; i < e.touches.length; i++) {
                const touch = e.touches[i];
                if (touch.clientX > window.innerWidth / 3) {
                    const dx = touch.clientX - touchLookPrev.x;
                    const dy = touch.clientY - touchLookPrev.y;
                    yaw -= dx * 0.004;
                    pitch -= dy * 0.004;
                    pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, pitch));
                    touchLookPrev = { x: touch.clientX, y: touch.clientY };
                }
            }
        });

        // --- PLAYER ACTIONS: SHOOT, RELOAD, AIM ---
        const crosshairEl = document.getElementById('crosshair-element');

        document.getElementById('btn-fire').addEventListener('click', shootWeapon);
        document.addEventListener('mousedown', (e) => {
            if (e.button === 0 && gameState.isPlaying && isPointerLocked) {
                shootWeapon();
            }
        });

        document.getElementById('btn-reload').addEventListener('click', reloadWeapon);
        document.getElementById('btn-aim').addEventListener('click', toggleAim);

        function shootWeapon() {
            if (!gameState.isPlaying || gameState.isReloading) return;
            if (gameState.ammo <= 0) {
                reloadWeapon();
                return;
            }

            gameState.ammo--;
            updateHUD();
            soundEngine.playGunshot();

            // Muzzle flash animation
            muzzleFlash.visible = true;
            setTimeout(() => { muzzleFlash.visible = false; }, 60);

            // Recoil punch
            pitch += 0.02;

            // Raycast for hits
            const raycaster = new THREE.Raycaster();
            const centerVector = new THREE.Vector2(0, 0);
            raycaster.setFromCamera(centerVector, camera);

            // Collect enemy meshes
            const enemyMeshes = enemies.map(e => e.mesh);
            const intersects = raycaster.intersectObjects(enemyMeshes, true);

            if (intersects.length > 0) {
                const hitMesh = intersects[0].object;
                // Find parent enemy object
                const targetEnemy = enemies.find(en => {
                    let found = false;
                    en.mesh.traverse(child => { if (child === hitMesh) found = true; });
                    return found;
                });

                if (targetEnemy) {
                    targetEnemy.health -= 35;
                    soundEngine.playHit();

                    // Hit effect color flash on enemy
                    targetEnemy.mesh.traverse(child => {
                        if (child.material && child.material.color) {
                            const orig = child.material.color.getHex();
                            child.material.color.setHex(0xff0000);
                            setTimeout(() => { child.material.color.setHex(orig); }, 100);
                        }
                    });

                    if (targetEnemy.health <= 0) {
                        // Eliminate enemy
                        scene.remove(targetEnemy.mesh);
                        const idx = enemies.indexOf(targetEnemy);
                        if (idx > -1) enemies.splice(idx, 1);

                        gameState.score += 100;
                        gameState.enemiesRemaining--;
                        updateHUD();

                        if (enemies.length === 0) {
                            // Wave completed!
                            setTimeout(nextWave, 1000);
                        }
                    }
                }
            }
        }

        function reloadWeapon() {
            if (gameState.isReloading || gameState.ammo === gameState.maxAmmo) return;
            gameState.isReloading = true;
            document.getElementById('reload-indicator').classList.remove('hidden');
            soundEngine.playReload();

            setTimeout(() => {
                gameState.ammo = gameState.maxAmmo;
                gameState.isReloading = false;
                document.getElementById('reload-indicator').classList.add('hidden');
                updateHUD();
            }, 1200);
        }

        function toggleAim() {
            gameState.isAiming = !gameState.isAiming;
            const btn = document.getElementById('btn-aim');
            if (gameState.isAiming) {
                camera.fov = 45; // Zoom in
                btn.classList.add('bg-amber-500/30', 'border-amber-400');
                crosshairEl.style.transform = 'translate(-50%, -50%) scale(0.6)';
            } else {
                camera.fov = 75; // Normal FOV
                btn.classList.remove('bg-amber-500/30', 'border-amber-400');
                crosshairEl.style.transform = 'translate(-50%, -50%) scale(1)';
            }
            camera.updateProjectionMatrix();
        }

        let verticalVelocity = 0;
        const gravity = 0.015;
        function triggerJump() {
            if (camera.position.y <= 2.21) {
                verticalVelocity = 0.25;
            }
        }

        function updateHUD() {
            document.getElementById('health-text').innerText = `${gameState.health}%`;
            document.getElementById('health-bar').style.width = `${(gameState.health / gameState.maxHealth) * 100}%`;
            document.getElementById('ammo-display').innerText = gameState.ammo;
            document.getElementById('score-display').innerText = gameState.score;
            document.getElementById('wave-display').innerText = `WAVE ${gameState.wave}`;
        }

        function nextWave() {
            gameState.wave++;
            gameState.health = Math.min(100, gameState.health + 30); // Heal slightly between waves
            updateHUD();
            spawnWave();

            // Display wave notification banner in modal briefly or alert
            const modal = document.getElementById('game-modal');
            document.getElementById('modal-title').innerText = `WAVE ${gameState.wave} INCOMING`;
            document.getElementById('modal-desc').innerText = `Hostile insurgent squad approaching sector. Prepare defenses!`;
            document.getElementById('start-btn').innerText = `CONTINUE COMBAT`;
            document.getElementById('final-stats').classList.add('hidden');
            modal.classList.remove('opacity-0', 'pointer-events-none');
            gameState.isPlaying = false;
            if (document.pointerLockElement) document.exitPointerLock();
        }

        function gameOver() {
            gameState.isPlaying = false;
            if (document.pointerLockElement) document.exitPointerLock();
            const modal = document.getElementById('game-modal');
            document.getElementById('modal-title').innerText = `MISSION FAILED`;
            document.getElementById('modal-desc').innerText = `You were overwhelmed by enemy hostiles in Sector 7.`;
            document.getElementById('start-btn').innerText = `DEPLOY AGAIN`;
            document.getElementById('stat-summary').innerText = `REACHED WAVE ${gameState.wave} | SCORE: ${gameState.score}`;
            document.getElementById('final-stats').classList.remove('hidden');
            modal.classList.remove('opacity-0', 'pointer-events-none');
        }

        // --- START GAME BUTTON ---
        document.getElementById('start-btn').addEventListener('click', () => {
            const modal = document.getElementById('game-modal');
            modal.classList.add('opacity-0');
            setTimeout(() => modal.classList.add('pointer-events-none'), 300);

            if (!gameState.isPlaying && gameState.health <= 0) {
                // Reset stats
                gameState.health = 100;
                gameState.score = 0;
                gameState.wave = 1;
                gameState.ammo = 30;
                // Clear existing enemies
                enemies.forEach(en => scene.remove(en.mesh));
                enemies.length = 0;
                spawnWave();
            } else if (enemies.length === 0 && gameState.wave === 1) {
                spawnWave();
            }

            gameState.isPlaying = true;
            updateHUD();
            renderer.domElement.requestPointerLock();
        });

        // --- GAME LOOP & PHYSICS UPDATE ---
        const clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame(animate);

            const delta = clock.getDelta();

            if (gameState.isPlaying) {
                // Camera rotation based on yaw & pitch
                const euler = new THREE.Euler(pitch, yaw, 0, 'YXZ');
                camera.quaternion.setFromEuler(euler);

                // Player Movement Calculation
                const moveSpeed = 0.12;
                const dir = new THREE.Vector3();

                let forwardInput = 0;
                let sideInput = 0;

                if (moveForward || keys.w) forwardInput += 1;
                if (moveBackward || keys.s) forwardInput -= 1;
                if (moveLeft || keys.a) sideInput -= 1;
                if (moveRight || keys.d) sideInput += 1;

                // Add virtual joystick input
                forwardInput -= joystickVector.y;
                sideInput += joystickVector.x;

                const forwardDir = new THREE.Vector3(0, 0, -1).applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
                const sideDir = new THREE.Vector3(1, 0, 0).applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);

                const moveVector = new THREE.Vector3()
                    .addScaledVector(forwardDir, forwardInput * moveSpeed)
                    .addScaledVector(sideDir, sideInput * moveSpeed);

                camera.position.add(moveVector);

                // Vertical Jump / Gravity physics
                camera.position.y += verticalVelocity;
                verticalVelocity -= gravity;
                if (camera.position.y < 2.2) {
                    camera.position.y = 2.2;
                    verticalVelocity = 0;
                }

                // Boundary collision with obstacles & arena walls
                obstacles.forEach(obs => {
                    const dx = camera.position.x - obs.x;
                    const dz = camera.position.z - obs.z;
                    const dist = Math.sqrt(dx * dx + dz * dz);
                    if (dist < obs.radius + 0.8) {
                        camera.position.x += (dx / dist) * 0.1;
                        camera.position.z += (dz / dist) * 0.1;
                    }
                });

                // Arena bounds limit
                camera.position.x = Math.max(-180, Math.min(180, camera.position.x));
                camera.position.z = Math.max(-180, Math.min(180, camera.position.z));

                // --- ENEMY AI BEHAVIOR ---
                enemies.forEach((enemy, idx) => {
                    const eMesh = enemy.mesh;
                    // Move towards player camera
                    const targetPos = new THREE.Vector3(camera.position.x, 0, camera.position.z);
                    eMesh.lookAt(targetPos);

                    const currentPos = eMesh.position;
                    const distanceToPlayer = currentPos.distanceTo(targetPos);

                    if (distanceToPlayer > 2.5) {
                        // Advance towards player
                        eMesh.translateZ(enemy.speed);
                    } else {
                        // Attack player if close
                        enemy.attackCooldown += delta;
                        if (enemy.attackCooldown > 1.2) {
                            enemy.attackCooldown = 0;
                            gameState.health -= 12;
                            soundEngine.playDamage();
                            updateHUD();

                            // Flash damage vignette
                            const vignette = document.getElementById('damage-vignette');
                            vignette.style.boxShadow = 'inset 0 0 120px rgba(220, 20, 20, 0.7)';
                            setTimeout(() => {
                                vignette.style.boxShadow = 'inset 0 0 100px rgba(220, 20, 20, 0)';
                            }, 250);

                            if (gameState.health <= 0) {
                                gameOver();
                            }
                        }
                    }
                });
            }

            renderer.render(scene, camera);
        }

        // Handle Window Resize
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // Initialize first wave in background & start render loop
        spawnWave();
        updateHUD();
        window.onload = function() {
            animate();
        };
    </script>
</body>
</html>
