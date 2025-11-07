<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sonnensystem-Simulation (6. Klasse)</title>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <style>
        body {
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Inter", sans-serif;
            background-color: #000;
            color: #fff;
            overflow: hidden;
        }

        #container {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }

        #ui-container {
            position: fixed;
            top: 10px;
            left: 10px;
            background: rgba(0, 0, 0, 0.75);
            padding: 15px;
            border-radius: 12px;
            max-width: 320px;
            width: calc(100% - 40px);
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.5);
            border: 1px solid rgba(255, 255, 255, 0.1);
            max-height: calc(100vh - 20px);
            overflow-y: auto;
            transition: max-width 0.3s ease, padding 0.3s ease;
            z-index: 1000; /* Sicherstellen, dass UI über 3D-Szene ist */
        }

        /* Neu: Für minimierbare UI */
        #toggle-ui {
            width: 100%;
            margin-bottom: 10px;
            font-size: 12px;
            padding: 6px;
        }
        #ui-container.minimized {
            max-width: 100px;
            padding: 10px 5px;
            overflow: hidden;
        }
        #ui-container.minimized .control-group {
            display: none;
        }
        #ui-container.minimized #toggle-ui {
            margin-bottom: 0;
        }
        /* Ende Neu */

        .control-group {
            margin-bottom: 15px;
        }

        .control-group h3 {
            font-size: 14px;
            font-weight: 600;
            margin: 0 0 10px 0;
            border-bottom: 1px solid #444;
            padding-bottom: 5px;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-size: 12px;
            font-weight: 500;
        }

        .btn {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 8px 12px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            transition: background-color 0.2s ease, transform 0.1s ease;
        }

        .btn:hover {
            background-color: #0056b3;
        }
        
        .btn:active {
            transform: scale(0.98);
        }

        .btn-secondary {
            background-color: #4a4a4a;
        }
        .btn-secondary:hover {
            background-color: #666;
        }
        
        .btn-warning {
            background-color: #ffc107;
            color: #000;
        }
        .btn-warning:hover {
            background-color: #e0a800;
        }
        
        .btn-danger { /* Neuer Button-Style */
            background-color: #dc3545;
            color: white;
        }
        .btn-danger:hover {
            background-color: #c82333;
        }


        .btn-group {
            display: flex;
            gap: 5px;
            flex-wrap: wrap;
        }
        
        .btn-group button {
            flex-grow: 1;
            font-size: 12px;
            padding: 6px 8px;
        }

        input[type="range"] {
            width: 100%;
            cursor: pointer;
        }
        
        #info-box {
            background: rgba(255, 255, 255, 0.1);
            padding: 10px;
            border-radius: 6px;
            font-size: 13px;
            font-weight: 500;
            margin-top: 10px;
            min-height: 2.5em;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
        }

        .checkbox-label {
            display: flex;
            align-items: center;
            gap: 8px;
            font-size: 13px;
        }
        
        #moon-phases-ui {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 5px;
        }
        
        #moon-phases-ui button {
            background: #333;
            border: 1px solid #555;
            color: #fff;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.2s ease, border-color 0.2s ease;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            padding: 5px;
            height: 42px; 
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            line-height: 1;
        }
        
        #moon-phases-ui button:hover {
            background: #444;
        }
        
        #moon-phases-ui button.active {
            background: #4A4A4A; 
            border-color: #ffc107; 
            font-weight: bold;
            box-shadow: 0 0 8px 1px rgba(255, 193, 7, 0.7); 
        }

        #demo-speed-control {
            display: none; 
            margin-top: 10px;
        }

    </style>
</head>
<body>

    <div id="container"></div>

    <div id="ui-container">
        
        <button id="toggle-ui" class="btn btn-secondary">Verbergen</button>

        <div class="control-group">
            <h3>Steuerung</h3>
            <div class="btn-group" style="margin-bottom: 15px;">
                <button id="play-pause-btn" class="btn">Pause</button>
            </div>
            
            <label for="day-slider">Tag im Jahr: <span id="day-label">0</span></label>
            <input type="range" id="day-slider" min="0" max="365" value="0" step="0.1">
            
            <label for="speed-slider" style="margin-top: 10px;">Geschwindigkeit: <span id="speed-label">1.0</span>x</label>
            <input type="range" id="speed-slider" min="0" max="100" value="90" step="1">

            <label class="checkbox-label" style="margin-top: 15px;">
                <input type="checkbox" id="orbit-checkbox" checked>
                Umlaufbahnen anzeigen
            </label>
            
            <!-- KORREKTUR: Checkbox durch Slider ersetzt -->
            <label for="darkness-slider" style="margin-top: 10px;">Helligkeit (Nachtseite): <span id="darkness-label">0.30</span></label>
            <input type="range" id="darkness-slider" min="0" max="0.9" value="0.3" step="0.01">
        </div>

        <div class="control-group">
            <h3>Kamera-Fokus</h3>
            <div class="btn-group">
                <button id="focus-system" class="btn btn-secondary">Sonnensystem</button>
                <button id="focus-earth" class="btn btn-secondary">Erde</button>
                <button id="focus-moon" class="btn btn-secondary">Mond</button>
                <!-- NEU: Ekliptik-Sicht Button -->
                <button id="focus-ecliptic" class="btn btn-secondary">Ekliptik-Sicht</button>
            </div>
        </div>

        <div class="control-group">
            <h3>Ereignisse</h3>
            <div class="btn-group" id="demo-buttons">
                <button id="demo-sofi" class="btn btn-warning">Demo: SoFi</button>
                <button id="demo-mofi" class="btn btn-warning">Demo: MoFi</button>
            </div>
            <div class="btn-group" id="demo-control-buttons" style="display: none; margin-top: 10px;">
                <button id="end-demo-btn" class="btn btn-danger">Demo beenden</button>
            </div>
            
            <div id="demo-speed-control">
                 <label for="demo-speed-slider" style="margin-top: 10px;">Demo-Geschw.: <span id="demo-speed-label">1.0</span>x</label>
                 <input type="range" id="demo-speed-slider" min="0" max="100" value="50" step="1">
            </div>
            
            <div id="info-box">Tag: 0</div>
        </div>

        <div class="control-group">
            <h3>Mondphasen</h3>
            <div id="moon-phases-ui">
                <button id="phase-0" data-phase-index="0" class="moon-phase-btn" title="Neumond (Springen)">🌑</button>
                <button id="phase-1" data-phase-index="1" class="moon-phase-btn" title="Zunehmende Sichel (Springen)">🌒</button>
                <button id="phase-2" data-phase-index="2" class="moon-phase-btn" title="Zunehmender Halbmond (Springen)">🌓</button>
                <button id="phase-3" data-phase-index="3" class="moon-phase-btn" title="Zunehmendes Drittel (Springen)">🌔</button>
                <button id="phase-4" data-phase-index="4" class="moon-phase-btn" title="Vollmond (Springen)">🌕</button>
                <button id="phase-5" data-phase-index="5" class="moon-phase-btn" title="Abnehmendes Drittel (Springen)">🌖</button>
                <button id="phase-6" data-phase-index="6" class="moon-phase-btn" title="Abnehmender Halbmond (Springen)">🌗</button>
                <button id="phase-7" data-phase-index="7" class="moon-phase-btn" title="Abnehmende Sichel (Springen)">🌘</button>
            </div>
        </div>
    </div>

    <script>
        // --- Globale Variablen ---
        let scene, camera, renderer, controls;
        let sun, earth, moon, earthPivot, moonPivot;
        let earthOrbitLine, moonOrbitLine;
        let starField; // NEU: Für Sternenhimmel
        
        let isPlaying = true;
        let currentDay = 0;
        let speed = 1.0;
        let cameraFocus = 'system';
        let lastCameraTargetPos = new THREE.Vector3();
        let isDemoActive = false;
        let originalMoonMaterial;
        let moonPhaseButtons = [];

        // Demo-spezifische Variablen
        let demoLoopStartDay = 0;
        let demoLoopEndDay = 0;
        let demoLoopSpeed = 0.1; 
        let demoType = ''; 
        let demoShadowBrightness = 0.0; 
        let demoRedOverlay = 0.0; 
        let earthDemoRotationOffset = 0.0; // Für manuelle Rotation der Erde in Demos

        // --- Konstanten ---
        const SCENE_SCALE = 1.0;
        const SUN_RADIUS = 5 * SCENE_SCALE; 
        const EARTH_RADIUS = 2.5 * SCENE_SCALE;
        const MOON_RADIUS = 0.7 * SCENE_SCALE;
        
        const EARTH_DISTANCE = 150 * SCENE_SCALE;
        const MOON_DISTANCE = 15 * SCENE_SCALE;

        const EARTH_TILT_RAD = (23.5 * Math.PI) / 180;
        const MOON_TILT_RAD = (5.1 * Math.PI) / 180; 

        const EARTH_YEAR_DAYS = 365.25;
        const LUNAR_MONTH_DAYS = 29.53; 
        
        const PHASE_OFFSET_DAYS = 5; 
        
        const PHASE_DAY_MAP = [
            LUNAR_MONTH_DAYS * 0.02  + PHASE_OFFSET_DAYS, // Neumond
            LUNAR_MONTH_DAYS * 0.145 + PHASE_OFFSET_DAYS, // Zun. Sichel (Index 1)
            LUNAR_MONTH_DAYS * 0.27  + PHASE_OFFSET_DAYS, // Zun. Halbmond (Index 2)
            LUNAR_MONTH_DAYS * 0.395 + PHASE_OFFSET_DAYS, // Zun. Drittel (Index 3)
            LUNAR_MONTH_DAYS * 0.55  + PHASE_OFFSET_DAYS, // Vollmond
            LUNAR_MONTH_DAYS * 0.71  + PHASE_OFFSET_DAYS, // Abn. Drittel (Index 5)
            LUNAR_MONTH_DAYS * 0.85  + PHASE_OFFSET_DAYS, // Abn. Halbmond (Index 6)
            LUNAR_MONTH_DAYS * 0.97  + PHASE_OFFSET_DAYS  // Abn. Sichel (Index 7)
        ];
        
        // --- DOM-Elemente ---
        const container = document.getElementById('container');
        const playPauseBtn = document.getElementById('play-pause-btn');
        const daySlider = document.getElementById('day-slider');
        const dayLabel = document.getElementById('day-label');
        const speedSlider = document.getElementById('speed-slider');
        const speedLabel = document.getElementById('speed-label');
        const orbitCheckbox = document.getElementById('orbit-checkbox');
        const infoBox = document.getElementById('info-box');
        // const darknessCheckbox = document.getElementById('darkness-checkbox'); // KORREKTUR: Entfernt, da ID nicht existiert
        
        // NEU: Slider für Dunkelheit
        const darknessSlider = document.getElementById('darkness-slider');
        const darknessLabel = document.getElementById('darkness-label');

        const demoButtons = document.getElementById('demo-buttons');
        const demoControlButtons = document.getElementById('demo-control-buttons');
        const endDemoBtn = document.getElementById('end-demo-btn');
        const demoSpeedControl = document.getElementById('demo-speed-control');
        const demoSpeedSlider = document.getElementById('demo-speed-slider');
        const demoSpeedLabel = document.getElementById('demo-speed-label');
        
        // --- Shader (Erde) ---
        const earthVertexShader = `
            varying vec2 vUv;
            varying vec3 vWorldPosition;
            
            void main() {
                vUv = uv;
                vWorldPosition = (modelMatrix * vec4(position, 1.0)).xyz;
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
        `;
        const earthFragmentShader = `
            uniform sampler2D dayTexture;
            uniform vec3 uSunPosition;         // Position der Sonne (Weltkoordinaten)
            uniform vec3 uObjectWorldPosition; // Position des Objekt-Zentrums (Weltkoordinaten)
            uniform float uNightBrightness;    // 0.3 oder 0.07
            
            // NEU FÜR SOFI SHADER
            uniform bool uSofiDemoActive;      // Ist SoFi-Demo aktiv?
            uniform vec3 uMoonPosition;
            uniform float uMoonRadius;
            uniform float uSunRadius;
            
            varying vec2 vUv;
            varying vec3 vWorldPosition;
            
            // NEU: Helper-Funktion für SoFi (kopiert von MoFi, Radien angepasst)
            // Berechnet Helligkeit (0.0 = Schatten, 1.0 = Licht)
            float calculateMoonShadowIntensity(vec3 pixelPos, vec3 moonPos, float moonRadius, vec3 sunPos, float sunRadius) {
                
                vec3 sunToMoon = moonPos - sunPos;
                vec3 shadowAxis = normalize(sunToMoon);

                vec3 sunToPixel = pixelPos - sunPos;
                
                float depthInShadow = dot(sunToPixel, shadowAxis);
                if (depthInShadow < 0.0) return 1.0; 

                vec3 closestPointOnAxis = sunPos + shadowAxis * depthInShadow;
                
                float pixelDistanceToAxis = distance(pixelPos, closestPointOnAxis);

                // --- HIER KANN GRÖSSE/DECKKRAFT (SoFi) ANGEPASST WERDEN ---
                // Kernschatten (Umbra)
                float umbraRadius = moonRadius * 0.1; // Z.B. 50% des Mondradius
                
                // Halbschatten (Penumbra)
                float penumbraRadius = moonRadius * 1.0; // Z.B. 150% des Mondradius

                if (pixelDistanceToAxis > penumbraRadius) {
                    return 1.0; // Volles Licht
                }

                if (pixelDistanceToAxis < umbraRadius) {
                    return 0.0; // Kernschatten (100% Schwarz)
                }

                // Halbschatten (Linearer Übergang)
                float progress = (pixelDistanceToAxis - umbraRadius) / (penumbraRadius - umbraRadius);
                return mix(0.4, 0.9, progress); // 0.0 (Schatten) -> 1.0 (Licht)
            }
            
            void main() {
                vec3 objectToSun = normalize(uSunPosition - uObjectWorldPosition);
                vec3 objectToFragment = normalize(vWorldPosition - uObjectWorldPosition);
                
                float intensity = (dot(objectToFragment, objectToSun) + 1.0) / 2.0;
                
                float nightBrightness = uNightBrightness;
                float lightMix = smoothstep(0.48, 0.59, intensity); // Weichere Grenze
                
                vec4 dayColor = texture2D(dayTexture, vUv);
                vec4 nightColor = dayColor * nightBrightness;
                
                vec4 finalColor = mix(nightColor, dayColor, lightMix);

                // --- SoFi-Schatten Logik (NEU) ---
                if (uSofiDemoActive) {
                    // Berechne den Mondschatten nur, wenn der Pixel auf der Tagseite ist
                    if (intensity > 0.01) { 
                        float moonShadowIntensity = calculateMoonShadowIntensity(
                            vWorldPosition, 
                            uMoonPosition, 
                            uMoonRadius,
                            uSunPosition,
                            uSunRadius
                        );
                        
                        // Wende die Schwärzung des Mondschattens an
                        // (0.0 = Schwarz, 1.0 = Volles Licht)
                        finalColor.rgb *= moonShadowIntensity;
                    }
                }

                gl_FragColor = finalColor;
            }
        `;
        
        // --- Shader (Mond) ---
        const moonVertexShader = `
            varying vec2 vUv;
            varying vec3 vWorldPosition;
            
            void main() {
                vUv = uv;
                vWorldPosition = (modelMatrix * vec4(position, 1.0)).xyz;
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
        `;
        const moonFragmentShader = `
            uniform sampler2D dayTexture;
            uniform vec3 uSunPosition;         // Sonne (Lichtquelle)
            uniform vec3 uEarthPosition;       // Erde (Schattenquelle)
            uniform vec3 uObjectWorldPosition; // Mond (Dieses Objekt)
            
            uniform float uEarthRadius;
            uniform float uSunRadius;
            uniform float uMoonRadius;
            uniform float uNightBrightness;    // NEU: 0.3 oder 0.07
            
            // Demo-spezifische Uniforms
            uniform bool uDemoActive;          // Ist MoFi-Demo aktiv?
            uniform float uShadowBrightness; // Helligkeit des Schattens (0.0 = 100% Schwarz, 0.5 = Totalität)
            uniform float uRedOverlayIntensity; // Deckkraft des roten Overlays (0.0 bis 1.0)

            varying vec2 vUv;
            varying vec3 vWorldPosition; // Position des Pixels auf dem Mond (Welt)

            // Helper: Berechnet die Helligkeit des Erdschattens (0.0 = 100% Schwarz, 1.0 = volles Licht)
            float calculateShadowIntensity(vec3 pixelPos, vec3 earthPos, float earthRadius, vec3 sunPos, float sunRadius) {
                
                // KORREKTUR: Schattenachse war invertiert
                // Vektor von der Sonne zur Erde (Schattenachse)
                vec3 sunToEarth = earthPos - sunPos;
                float earthSunDist = length(sunToEarth);
                vec3 shadowAxis = normalize(sunToEarth);

                // Vektor von der Sonne zum Pixel auf dem Mond
                vec3 sunToPixel = pixelPos - sunPos;
                
                // Projiziere sunToPixel auf die Schattenachse, um die "Tiefe" im Schatten zu finden
                float depthInShadow = dot(sunToPixel, shadowAxis);
                if (depthInShadow < 0.0) return 1.0; // Pixel ist vor der Sonne

                // Finde den Punkt auf der Schattenachse, der dem Pixel am nächsten ist
                vec3 closestPointOnAxis = sunPos + shadowAxis * depthInShadow;
                
                // Distanz des Pixels zu diesem Punkt (Radius im Schattenkegel)
                float pixelDistanceToAxis = distance(pixelPos, closestPointOnAxis);

                // --- Didaktisch vereinfachtes Modell (feste Radien) ---
                float umbraRadius = uEarthRadius * 0.9; // Kernschatten (ca. 90% der Erde)
                
                // KORREKTUR: Halbschatten (Penumbra) drastisch reduziert (scharfe Kante)
                // float penumbraRadius = umbraRadius * 1.01; // Sehr schmaler Halbschatten
                
                // NEU: Weiche Kante mit smoothstep
                float startRatio = 0.95; 
                float endRatio = 1.05;   
                
                // WICHTIG: smoothstep glättet den Übergang von 0.0 (Schatten) zu 1.0 (Licht)
                float intensity = smoothstep(umbraRadius * startRatio, umbraRadius * endRatio, pixelDistanceToAxis);
                
                return intensity; 
            }

            void main() {
                vec4 textureColor = texture2D(dayTexture, vUv);
                
                // 1. Normale Tag/Nacht-Beleuchtung (durch die Sonne)
                vec3 objectToSun = normalize(uSunPosition - uObjectWorldPosition);
                vec3 objectToFragment = normalize(vWorldPosition - uObjectWorldPosition);
                float sunLightIntensity = (dot(objectToFragment, objectToSun) + 1.0) / 2.0;
                float nightBrightness = uNightBrightness; // KORRIGIERT
                float sunLightMix = smoothstep(0.5, 0.6, sunLightIntensity); // KORRIGIERT: Weichere Grenze
                
                vec4 sunLitColor = mix(textureColor * nightBrightness, textureColor, sunLightMix);

                // Nur wenn die MoFi-Demo aktiv ist, berechne Schatten und Rot
                if (uDemoActive) {
                    // KORRIGIERTE REIHENFOLGE
                    
                    // 1. Beginne mit der Tag/Nacht-Farbe
                    vec4 finalColor = vec4(sunLitColor.rgb, 1.0); // Startet mit Tag/Nacht

                    // 2. Erdschatten-Berechnung (MoFi)
                    float earthShadowIntensity = calculateShadowIntensity(
                        vWorldPosition, 
                        uEarthPosition, 
                        uEarthRadius, 
                        uSunPosition, 
                        uSunRadius
                    );

                    // 3. Wende den Erdschatten (Abdunklung) AN
                    if (earthShadowIntensity < 1.0) { // Wenn IRGENDEIN Schatten vorhanden ist
                        
                        if (earthShadowIntensity == 0.0) {
                            // Im Kernschatten (Umbra): Wende die Helligkeit (z.B. 0.05) NUR auf die Tagseite an
                            finalColor.rgb = mix(finalColor.rgb, finalColor.rgb * uShadowBrightness, sunLightMix);
                        } else {
                            // Im Halbschatten (Penumbra): Abdunklung nur auf die Tagseite des Mondes anwenden
                             finalColor.rgb *= (1.0 - (1.0 - earthShadowIntensity) * sunLightMix); 
                        }
                    }
                    
                    // 4. Rotes Overlay (Blutmond) ZULETZT anwenden
                    vec3 bloodMoonColor = vec3(1.0, 0.1, 0.0); // Rot
                    float redMixAmount = uRedOverlayIntensity * sunLightMix * 0.5; // 50% Deckkraft
                    
                    // Mische das Rot ÜBER die (jetzt abgedunkelte) Farbe
                    finalColor.rgb = mix(finalColor.rgb, bloodMoonColor, redMixAmount);
                    
                    gl_FragColor = finalColor;

                } else {
                    // Normale Ansicht (keine Demo)
                    gl_FragColor = sunLitColor;
                }
            }
        `;

        // --- Initialisierung ---
        function init() {
            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 5000);
            camera.position.set(EARTH_DISTANCE * 1.5, EARTH_DISTANCE * 0.7, EARTH_DISTANCE * 1.5);
            camera.lookAt(0, 0, 0);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(window.devicePixelRatio);
            container.appendChild(renderer.domElement);

            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.1;

            createSolarSystem();
            createOrbits();
            setupUI();

            animate();
            
            window.addEventListener('resize', onWindowResize);
            
            updatePositions(currentDay);
            updateCamera(true); 
        }

        // --- Himmelskörper erstellen ---
        function createSolarSystem() {
            const textureLoader = new THREE.TextureLoader();

            // NEU: Sternenhimmel
            const starGeometry = new THREE.SphereGeometry(3000, 32, 32); // Riesige Kugel
            const starTexture = textureLoader.load('https://threejs.org/examples/textures/starmap.jpg');
            const starMaterial = new THREE.MeshBasicMaterial({
                map: starTexture,
                side: THREE.BackSide // Wichtig: Textur auf der Innenseite der Kugel
            });
            starField = new THREE.Mesh(starGeometry, starMaterial);
            scene.add(starField);

            // 1. Sonne
            const sunMaterial = new THREE.MeshBasicMaterial({ color: 0xFFFF00 });
            sun = new THREE.Mesh(new THREE.SphereGeometry(SUN_RADIUS, 32, 32), sunMaterial);
            scene.add(sun);
            
            // 2. Erde
            earthPivot = new THREE.Group();
            scene.add(earthPivot);

            const earthMaterial = new THREE.ShaderMaterial({
                vertexShader: earthVertexShader,
                fragmentShader: earthFragmentShader,
                uniforms: {
                    dayTexture: { value: textureLoader.load('https://stemkoski.github.io/Three.js/images/earth-day.jpg') },
                    uSunPosition: { value: new THREE.Vector3(0, 0, 0) },
                    uObjectWorldPosition: { value: new THREE.Vector3() },
                    uNightBrightness: { value: 0.3 }, 
                    // NEU FÜR SOFI SHADER
                    uSofiDemoActive: { value: false },
                    uMoonPosition: { value: new THREE.Vector3() },
                    uMoonRadius: { value: MOON_RADIUS },
                    uSunRadius: { value: SUN_RADIUS } 
                }
            });
            earth = new THREE.Mesh(new THREE.SphereGeometry(EARTH_RADIUS, 32, 32), earthMaterial);
            earth.position.x = EARTH_DISTANCE;
            earth.rotation.order = 'YXZ'; 
            earth.rotation.z = EARTH_TILT_RAD; 
            earthPivot.add(earth);

            // 3. Mond
            moonPivot = new THREE.Group();
            moonPivot.rotation.x = MOON_TILT_RAD; 
            scene.add(moonPivot); 

            originalMoonMaterial = new THREE.ShaderMaterial({
                vertexShader: moonVertexShader,
                fragmentShader: moonFragmentShader,
                uniforms: {
                    dayTexture: { value: textureLoader.load('https://threejs.org/examples/textures/planets/moon_1024.jpg') },
                    uSunPosition: { value: new THREE.Vector3(0, 0, 0) },
                    uEarthPosition: { value: new THREE.Vector3() },
                    uObjectWorldPosition: { value: new THREE.Vector3() },
                    uEarthRadius: { value: EARTH_RADIUS },
                    uSunRadius: { value: SUN_RADIUS },
                    uMoonRadius: { value: MOON_RADIUS },
                    uNightBrightness: { value: 0.3 },
                    uDemoActive: { value: false },
                    uShadowBrightness: { value: 0.0 }, 
                    uRedOverlayIntensity: { value: 0.0 } 
                }
            });
            moon = new THREE.Mesh(new THREE.SphereGeometry(MOON_RADIUS, 32, 32), originalMoonMaterial);
            moon.position.x = -MOON_DISTANCE; 
            moonPivot.add(moon);
            
            // 4. SoFi-Schatten (auf der Erde) - ENTFERNT (jetzt im Shader)
        }

        // --- Umlaufbahnen (Linien) ---
        function createOrbits() {
            const createOrbitLine = (radius, color, segments = 128) => {
                const points = [];
                for (let i = 0; i <= segments; i++) {
                    const angle = (i / segments) * Math.PI * 2;
                    points.push(new THREE.Vector3(Math.cos(angle) * radius, 0, Math.sin(angle) * radius));
                }
                const geometry = new THREE.BufferGeometry().setFromPoints(points);
                const material = new THREE.LineBasicMaterial({ color: color });
                return new THREE.Line(geometry, material);
            };

            earthOrbitLine = createOrbitLine(EARTH_DISTANCE, 0x0000FF);
            scene.add(earthOrbitLine);

            moonOrbitLine = createOrbitLine(MOON_DISTANCE, 0xFFFF00);
            moonOrbitLine.rotation.x = MOON_TILT_RAD; 
            scene.add(moonOrbitLine); 
        }

        // --- UI-Einrichtung ---
        function setupUI() {
            //Verbergen Button toggle
             const uiContainer = document.getElementById('ui-container');
            const toggleBtn = document.getElementById('toggle-ui');
            toggleBtn.addEventListener('click', () => {
                uiContainer.classList.toggle('minimized');
                toggleBtn.textContent = uiContainer.classList.contains('minimized') ? 'Anzeigen' : 'Verbergen';
            });
            //Orbit Checkbox
            orbitCheckbox.addEventListener('change', (e) => {
                earthOrbitLine.visible = e.target.checked;
                moonOrbitLine.visible = e.target.checked;
            });
            // NEU: Event-Listener für Dunkelheits-Slider
            darknessSlider.addEventListener('input', (e) => {
                const brightness = parseFloat(e.target.value);
                earth.material.uniforms.uNightBrightness.value = brightness;
                originalMoonMaterial.uniforms.uNightBrightness.value = brightness;
                darknessLabel.textContent = brightness.toFixed(2);
            });
            //Speedslider Fix
            speedSlider.addEventListener('input', onSpeedSliderChange);
            onSpeedSliderChange(); 
            //Play Button fix
            playPauseBtn.addEventListener('click', togglePlay);
            daySlider.addEventListener('input', () => {
                pauseSimulation();
                currentDay = parseFloat(daySlider.value);
                updatePositions(currentDay);
                updateUI();
            });
            // Initialwert setzen
            const initialBrightness = parseFloat(darknessSlider.value);
            earth.material.uniforms.uNightBrightness.value = initialBrightness;
            originalMoonMaterial.uniforms.uNightBrightness.value = initialBrightness;
            darknessLabel.textContent = initialBrightness.toFixed(2);


            document.getElementById('focus-system').addEventListener('click', () => setFocus('system'));
            document.getElementById('focus-earth').addEventListener('click', () => setFocus('earth'));
            document.getElementById('focus-moon').addEventListener('click', () => setFocus('moon'));

            // NEU: Listener für Ekliptik-Sicht
            document.getElementById('focus-ecliptic').addEventListener('click', () => {
                cameraFocus = 'ecliptic_side_view'; // NEUER Modus
                
                let earthPos = new THREE.Vector3();
                earth.getWorldPosition(earthPos);
                
                // KORREKTUR: Kamera blickt auf die Sonne (0,0,0)
                controls.target.set(0, 0, 0); 
                
                // KORREKTUR: Kamera ist auf Augenhöhe (Y=0) auf der Erdumlaufbahn
                let offset = earthPos.clone().normalize().multiplyScalar(40); // KORRIGIERT: 40 Einheiten "hinter" der Erde
                let sideOffset = new THREE.Vector3(-earthPos.z, 0, earthPos.x).normalize().multiplyScalar(4); // KORRIGIERT: 4 Einheiten "seitlich"
                
                camera.position.copy(earthPos).add(offset).add(sideOffset); // Position = Erde + Hinten + Seitlich
            });

            document.getElementById('demo-sofi').addEventListener('click', () => startDemo('sofi'));
            document.getElementById('demo-mofi').addEventListener('click', () => startDemo('mofi'));
            endDemoBtn.addEventListener('click', endDemo);
            
            demoSpeedSlider.addEventListener('input', onDemoSpeedSliderChange);
            onDemoSpeedSliderChange();

            document.querySelectorAll('#moon-phases-ui button').forEach(btn => {
                const index = parseInt(btn.dataset.phaseIndex);
                btn.addEventListener('click', () => jumpToPhase(index));
                moonPhaseButtons.push(btn);
            });
        }
        
        // --- Event Handler ---

        function togglePlay() {
            isPlaying = !isPlaying;
            playPauseBtn.textContent = isPlaying ? 'Pause' : 'Play';
            
            controls.enableZoom = true; // Zoom ist IMMER an
        }

        function pauseSimulation() {
            if (isPlaying) {
                togglePlay();
            }
        }
        
        function onSpeedSliderChange() {
            const value = parseFloat(speedSlider.value); 
            const minSpeed = 0.00005; 
            const normalSpeed = 1.0; 
            const maxSpeed = 10.0;
            
            if (value <= 90) {
                speed = minSpeed + (value / 90) * (normalSpeed - minSpeed);
            } else {
                speed = normalSpeed + ((value - 90) / 10) * (maxSpeed - normalSpeed);
            }
            speedLabel.textContent = speed.toFixed(4) + 'x'; 
        }
        
        function onDemoSpeedSliderChange() {
            const value = parseFloat(demoSpeedSlider.value); 
            const minSpeed = 0.01;
            const normalSpeed = 0.1;
            const maxSpeed = 1.0;
            
            if (value <= 50) {
                demoLoopSpeed = minSpeed + (value / 50) * (normalSpeed - minSpeed);
            } else {
                demoLoopSpeed = normalSpeed + ((value - 50) / 50) * (maxSpeed - normalSpeed);
            }
            demoSpeedLabel.textContent = demoLoopSpeed.toFixed(2) + 'x';
        }

        function setFocus(target) {
            cameraFocus = target; 
            let targetObj;
            let zoomFactor = 1.0;

            if (target === 'earth') {
                targetObj = earth;
                zoomFactor = 2.0;
            } else if (target === 'moon') {
                targetObj = moon;
                zoomFactor = 10.0;
            } else { // 'system'
                targetObj = sun; 
                zoomFactor = 0.2;
            }

            const targetPos = new THREE.Vector3();
            targetObj.getWorldPosition(targetPos);
            
            const offset = camera.position.clone().sub(controls.target).normalize().multiplyScalar(EARTH_DISTANCE / zoomFactor);
            camera.position.copy(targetPos).add(offset);
            controls.target.copy(targetPos);
            lastCameraTargetPos.copy(targetPos);

            controls.enableZoom = true;
        }

        function resetToRealMode() {
            isDemoActive = false;
            demoType = '';
            controls.enableZoom = true;

            demoButtons.style.display = 'flex';
            demoControlButtons.style.display = 'none';
            demoSpeedControl.style.display = 'none';

            earth.rotation.z = EARTH_TILT_RAD;
            earth.rotation.x = 0; 
            moonPivot.rotation.x = MOON_TILT_RAD; 
            
            // NEU: SoFi-Shader deaktivieren
            earth.material.uniforms.uSofiDemoActive.value = false;
            
            if (moon.material.uniforms) {
                originalMoonMaterial.uniforms.uDemoActive.value = false; 
                originalMoonMaterial.uniforms.uShadowBrightness.value = 0.0;
                originalMoonMaterial.uniforms.uRedOverlayIntensity.value = 0.0;
            }
        }
        
        // --- Demo-Start ---
        function startDemo(type) {
            pauseSimulation(); 
            resetToRealMode(); 
            
            isDemoActive = true;
            demoType = type;
            isPlaying = true; 
            playPauseBtn.textContent = 'Pause';
            controls.enableZoom = true; 

            demoButtons.style.display = 'none'; 
            demoControlButtons.style.display = 'flex'; 
            demoSpeedControl.style.display = 'block'; 
            
            // NEU: Erdrotation (z) wird für SoFi gespiegelt
            if (type === 'sofi') {
                earth.rotation.z = -EARTH_TILT_RAD;
            } else {
                earth.rotation.z = 0;
            }
            
            
            
            let demoDurationDays;

            if (type === 'sofi') {
                demoDurationDays = 2; 
                currentDay = LUNAR_MONTH_DAYS * 0.0 - (demoDurationDays / 2); 
                demoLoopStartDay = currentDay;
                demoLoopEndDay = LUNAR_MONTH_DAYS * 0.0 + (demoDurationDays / 2); 
                
                // NEU: SoFi-Shader aktivieren
                earth.material.uniforms.uSofiDemoActive.value = true;
                
                earthDemoRotationOffset = 4 * Math.PI / 4; 
                
                moon.position.y = 0.3; 

                setFocus('earth'); 
            } else if (type === 'mofi') {
                demoDurationDays = 17.5 - 14.3; 
                currentDay = 14.3; 
                demoLoopStartDay = currentDay;
                demoLoopEndDay = 17.5; 

                moonPivot.position.y = MOON_RADIUS * 1.0; 
                
                originalMoonMaterial.uniforms.uDemoActive.value = true;
                
                setFocus('moon'); 
            }
            
            updatePositions(currentDay);
            daySlider.value = currentDay;
            updateUI();
        }

        // --- Demo-Beenden ---
        function endDemo() {
            resetToRealMode();
            currentDay = 0; 
            daySlider.value = currentDay;
            moonPivot.position.y = 0; 
            
            earthDemoRotationOffset = 0.0; 
            moon.position.y = 0; 
            
            updatePositions(currentDay);
            updateUI();
        }
        
        // --- Zu Mondphase springen ---
        function jumpToPhase(index) {
            pauseSimulation(); 
            resetToRealMode(); 
            
            earthDemoRotationOffset = 0.0; 
            moonPivot.position.y = 0; 
            moon.position.y = 0;
            
            currentDay = PHASE_DAY_MAP[index];

            // --- NEUE HIGHLIGHT-LOGIK ---
            moonPhaseButtons.forEach(btn => {
                btn.classList.remove('active');
            });
            
            const clickedButton = moonPhaseButtons.find(btn => parseInt(btn.dataset.phaseIndex) === index);
            if (clickedButton) {
                clickedButton.classList.add('active');
            }
            
            updatePositions(currentDay);
            daySlider.value = currentDay;
            updateUI();

            // --- NEU: Kamera-Sprung & Modus-Wechsel ---
            cameraFocus = 'earthView'; 
            
            let earthPos = new THREE.Vector3();
            let moonPos = new THREE.Vector3();
            earth.getWorldPosition(earthPos);
            moon.getWorldPosition(moonPos);
            
            camera.position.copy(earthPos); 
            controls.target.copy(moonPos); 
            lastCameraTargetPos.copy(moonPos);
        }

        // --- Animations-Loop ---
        function animate() {
            requestAnimationFrame(animate);

            let actualSpeed = speed;

            if (isDemoActive) {
                actualSpeed = demoLoopSpeed;
            }

            if (isPlaying) {
                const deltaDays = (actualSpeed * (1 / 60)) * (isDemoActive ? 1 : EARTH_YEAR_DAYS / 60); 
                currentDay += deltaDays;
                
                if (isDemoActive) {
                    if (currentDay > demoLoopEndDay) {
                        currentDay = demoLoopStartDay; 
                    } else if (currentDay < demoLoopStartDay) { 
                        currentDay = demoLoopEndDay;
                    }
                } else {
                    if (currentDay > EARTH_YEAR_DAYS) {
                        currentDay -= EARTH_YEAR_DAYS;
                    }
                }
                
                daySlider.value = currentDay; 
                updateUI();
            }

            updatePositions(currentDay);
            updateCamera(false);
            
            controls.update();
            renderer.render(scene, camera);
        }

        // --- Update-Funktionen ---

        function updatePositions(day) {
            // 1. Umlaufbahn der Erde (um die Sonne)
            const earthOrbitAngle = (day / EARTH_YEAR_DAYS) * Math.PI * 2;
            earthPivot.rotation.y = earthOrbitAngle;

            // 2. Eigendrehung der Erde (1x pro Tag)
            let rotationFactor = 1.0;
            
            if (isDemoActive && demoType === 'sofi') {
                rotationFactor = 0.1; 
            }
            
            earth.rotation.y = (day * rotationFactor) * Math.PI * 2; 
            earth.rotation.y += earthDemoRotationOffset; 

            // 3. Umlaufbahn des Mondes (um die Erde)
            let moonOrbitAngle;
            if (isDemoActive) {
                moonOrbitAngle = (day / LUNAR_MONTH_DAYS) * Math.PI * 2;
            } else {
                moonOrbitAngle = ((day - PHASE_OFFSET_DAYS) / LUNAR_MONTH_DAYS) * Math.PI * 2;
            }
            
            let earthPos = new THREE.Vector3();
            earth.getWorldPosition(earthPos);
            
            moonPivot.position.copy(earthPos);
            moonPivot.rotation.y = moonOrbitAngle; 
            moonOrbitLine.position.copy(earthPos);
            
            // 4. Eigendrehung des Mondes (Gebundene Rotation)
            moon.rotation.y = 0;

            // 5. Shader-Uniforms aktualisieren
            let tempVec = new THREE.Vector3();
            earth.getWorldPosition(tempVec);
            earth.material.uniforms.uObjectWorldPosition.value.copy(tempVec);
            
            // NEU: Befülle Erde-Shader mit Mondposition für SoFi-Schattenberechnung
            let moonWorldPosition = new THREE.Vector3();
            moon.getWorldPosition(moonWorldPosition);
            earth.material.uniforms.uMoonPosition.value.copy(moonWorldPosition);

            // MoFi-Demo Logik
            if (isDemoActive && demoType === 'mofi') {
                const animProgress = (currentDay - demoLoopStartDay) / (demoLoopEndDay - demoLoopStartDay);
                
                // --- Fading-Logik (State-Based) ---
                const dayToProgress = (d) => (d - demoLoopStartDay) / (demoLoopEndDay - demoLoopStartDay);

                const peakDay = 15.7; 
                const peakProgress = dayToProgress(peakDay); 

                // Rot
                const redFadeInStart = peakProgress - 0.05; 
                const redFadeOutStart = dayToProgress(16.4); 
                const redFadeOutEnd = dayToProgress(16.5);   
                
                // Schatten
                const shadowFadeInStart = peakProgress - 0.05; 
                const shadowFadeInEnd = peakProgress;         

                const shadowFadeOutStart = dayToProgress(16.2); // 0.6875
                const shadowFadeOutEnd = dayToProgress(16.5);   // 0.71875
                
                const shadowTargetBrightness = 0.6; // KORRIGIERT: 40% Deckkraft (1.0 - 0.4 = 0.6 Helligkeit)

                let redIntensity = 0.0;
                let shadowBrightness = 0.0; 

                // Rot Fade-In (Start -> Peak 15.5)
                if (animProgress <= peakProgress) {
                    const fadeProgress = Math.max(0.0, (animProgress - redFadeInStart) / (peakProgress - redFadeInStart));
                    redIntensity = (fadeProgress * fadeProgress); 
                }
                // Rot Fade-Out (15.5 -> 16.8)
                else if (animProgress > peakProgress && animProgress <= redFadeOutEnd) {
                    const fadeProgress = (animProgress - peakProgress) / (redFadeOutEnd - peakProgress);
                    redIntensity = 1.0 - (fadeProgress * fadeProgress); 
                }

                // Schatten-Deckkraft Fade-In (15.34 -> 15.5)
                if (animProgress > shadowFadeInStart && animProgress <= shadowFadeInEnd) {
                    const fadeProgress = (animProgress - shadowFadeInStart) / (shadowFadeInEnd - shadowFadeInStart);
                    shadowBrightness = (fadeProgress * fadeProgress) * shadowTargetBrightness; // 0.0 -> 0.6
                }
                // Plateau (15.5 -> 16.5)
                else if (animProgress > shadowFadeInEnd && animProgress <= shadowFadeOutStart) {
                     shadowBrightness = shadowTargetBrightness; // Bleibt 40% Deckkraft
                }
                // Schatten-Deckkraft Fade-Out (16.5 -> 16.6)
                else if (animProgress > shadowFadeOutStart && animProgress <= shadowFadeOutEnd) {
                    const fadeProgress = (animProgress - shadowFadeOutStart) / (shadowFadeOutEnd - shadowFadeOutStart);
                    shadowBrightness = shadowTargetBrightness - ((fadeProgress * fadeProgress) * shadowTargetBrightness); // 0.6 -> 0.0
                }
                
                originalMoonMaterial.uniforms.uShadowBrightness.value = shadowBrightness;
                originalMoonMaterial.uniforms.uRedOverlayIntensity.value = redIntensity;
            }
            
            moon.getWorldPosition(tempVec);
            if (moon.material.uniforms) {
                originalMoonMaterial.uniforms.uObjectWorldPosition.value.copy(tempVec);
                earth.getWorldPosition(tempVec); 
                originalMoonMaterial.uniforms.uEarthPosition.value.copy(tempVec);
            }
        }

        function updateCamera(isJump = false) {
            let targetObj;
            let currentTargetPos = new THREE.Vector3();

            // NEUER 'earthView'-Modus
            if (cameraFocus === 'earthView') {
                earth.getWorldPosition(currentTargetPos); 
                
                if (isJump) {
                    lastCameraTargetPos.copy(currentTargetPos);
                } else if (isPlaying) { 
                    const delta = currentTargetPos.clone().sub(lastCameraTargetPos);
                    camera.position.add(delta);
                }
                lastCameraTargetPos.copy(currentTargetPos);
                
                let moonPos = new THREE.Vector3();
                moon.getWorldPosition(moonPos);
                controls.target.copy(moonPos);
                return;
            }

            // NEU: 'ecliptic_side_view'-Modus (KORRIGIERT)
            if (cameraFocus === 'ecliptic_side_view') {
                targetObj = earth; // Die Kamera folgt der Erde
                
                targetObj.getWorldPosition(currentTargetPos);

                if (isJump) {
                    lastCameraTargetPos.copy(currentTargetPos);
                } else if (isPlaying || isDemoActive) { 
                    const delta = currentTargetPos.clone().sub(lastCameraTargetPos);
                    
                    // KORREKTUR: Kamera folgt der Erde, bleibt aber auf Y=0
                    
                    // Positioniere 40 Einheiten "hinter" und 4 "seitlich" der Erde (relativ zur Sonne)
                    let offset = currentTargetPos.clone().normalize().multiplyScalar(40); // KORRIGIERT: 40 Einheiten "hinter" der Erde
                    let sideOffset = new THREE.Vector3(-currentTargetPos.z, 0, currentTargetPos.x).normalize().multiplyScalar(4); // KORRIGIERT: 4 Einheiten seitlich
                    
                    camera.position.copy(currentTargetPos).add(offset).add(sideOffset); 
                    camera.position.y = 0; // Garantiert auf der Ekliptik bleiben
                }
                lastCameraTargetPos.copy(currentTargetPos);
                
                // Wichtig: Das Ziel (controls.target) bleibt immer die Sonne
                controls.target.set(0, 0, 0);
                return;
            } // <--- Diese Klammer hat gefehlt!
            // Normale Fokus-Logik (KORRIGIERT: Ekliptik-Modus entfernt)
            if (cameraFocus === 'system') {
                targetObj = sun;
            } else if (cameraFocus === 'earth') { 
                targetObj = earth;
            } else { // 'moon'
                targetObj = moon;
            }
            
            targetObj.getWorldPosition(currentTargetPos);

            if (isJump) {
                lastCameraTargetPos.copy(currentTargetPos);
            }
            // KORREKTUR: Kamera-Drift soll immer stattfinden (auch wenn pausiert),
            // außer wenn die normale Simulation pausiert ist.
            else if (isPlaying || isDemoActive) {
                const delta = currentTargetPos.clone().sub(lastCameraTargetPos);
                camera.position.add(delta);
                controls.target.add(delta);
            }
            
            lastCameraTargetPos.copy(currentTargetPos);
        }
        
        function updateUI() {
            dayLabel.textContent = Math.floor(currentDay % EARTH_YEAR_DAYS);
            
            let sunPos = new THREE.Vector3();
            let earthPos = new THREE.Vector3();
            let moonPos = new THREE.Vector3();
            
            sun.getWorldPosition(sunPos);
            earth.getWorldPosition(earthPos);
            moon.getWorldPosition(moonPos);

            const vecEarthToMoon = new THREE.Vector3().subVectors(moonPos, earthPos);
            const vecEarthToSun = new THREE.Vector3().subVectors(sunPos, earthPos); 
            
            let angle = Math.atan2(vecEarthToSun.z, vecEarthToSun.x) - Math.atan2(vecEarthToMoon.z, vecEarthToMoon.x);
            angle = (angle + Math.PI * 2) % (Math.PI * 2);
            
            let phaseIndex = 0;
            let normalizedAngle = angle;
            
            phaseIndex = Math.floor(normalizedAngle / (Math.PI * 2) * 8 + 0.5) % 8;
            
            const phaseNames = ["Neumond", "Zun. Sichel", "Zun. Halbmond", "Zun. Drittel", "Vollmond", "Abn. Drittel", "Abn. Halbmond", "Abn. Sichel"];
            let phaseName = phaseNames[phaseIndex];

            let infoText = `Tag: ${Math.floor(currentDay % EARTH_YEAR_DAYS)} | ${phaseName}`;
            
            // Finsternis-Logik (Prüfung)
            if (isDemoActive) {
                if (demoType === 'sofi') {
                    infoText = "SONNENFINSTERNIS (Demo)";
                } else if (demoType === 'mofi') {
                    infoText = "MONDFINSTERNIS (Demo)";
                }
            } else {
                let moonWorldY = moon.getWorldPosition(new THREE.Vector3()).y;
                let earthWorldY = earth.getWorldPosition(new THREE.Vector3()).y; 
                
                let yDiff = Math.abs(moonWorldY - earthWorldY); // KORRIGIERT: yDiff deklariert
                
                const ECLIPSE_TOLERANCE = (MOON_RADIUS + EARTH_RADIUS) / 3; 

                if (phaseIndex === 0 && yDiff < ECLIPSE_TOLERANCE) { 
                    infoText += " (Mögliche SoFi)";
                }
                if (phaseIndex === 4 && yDiff < ECLIPSE_TOLERANCE) { 
                     infoText += " (Mögliche MoFi)";
                }
            }
            infoBox.textContent = infoText;
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // --- Start ---
        init();
    </script>
</body>
</html>
