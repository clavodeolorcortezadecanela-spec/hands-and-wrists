<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Simulación Biomecánica: Muñeca y Mano | Clase Invertida</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Segoe UI', 'Poppins', 'Roboto', sans-serif;
        }
        /* Panel de control - Interfaz moderna y educativa */
        .controls-panel {
            position: absolute;
            bottom: 20px;
            left: 20px;
            right: 20px;
            background: rgba(10, 20, 30, 0.85);
            backdrop-filter: blur(12px);
            border-radius: 28px;
            padding: 16px 24px;
            color: white;
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            gap: 16px;
            z-index: 10;
            border: 1px solid rgba(255,255,255,0.2);
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
            pointer-events: auto;
            font-weight: 500;
        }
        .control-group {
            flex: 1;
            min-width: 150px;
            background: rgba(0,0,0,0.5);
            border-radius: 20px;
            padding: 12px 18px;
            transition: all 0.2s ease;
            border-left: 4px solid #ff8c42;
        }
        .control-group label {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
            font-size: 0.85rem;
            letter-spacing: 1px;
            text-transform: uppercase;
            font-weight: 600;
        }
        input {
            width: 100%;
            cursor: pointer;
            background: #2c3e44;
            height: 4px;
            border-radius: 5px;
            accent-color: #ff8c42;
        }
        .angle-value {
            color: #ffaa66;
            background: #1e2a32;
            padding: 2px 10px;
            border-radius: 30px;
            font-family: monospace;
            font-size: 0.9rem;
        }
        h3 {
            margin: 0 0 6px 0;
            font-size: 1rem;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .badge {
            background: #ff8c42;
            border-radius: 40px;
            padding: 2px 10px;
            font-size: 0.7rem;
            color: #1e2a32;
            font-weight: bold;
        }
        .info-text {
            position: absolute;
            top: 20px;
            right: 20px;
            background: rgba(0,0,0,0.7);
            backdrop-filter: blur(8px);
            padding: 12px 20px;
            border-radius: 20px;
            color: white;
            max-width: 260px;
            font-size: 0.8rem;
            pointer-events: none;
            z-index: 10;
            border-right: 3px solid #ff8c42;
            font-family: monospace;
            line-height: 1.4;
        }
        .info-text strong {
            color: #ffaa66;
        }
        button {
            background: #ff8c42;
            border: none;
            padding: 6px 16px;
            border-radius: 40px;
            font-weight: bold;
            cursor: pointer;
            transition: 0.2s;
            margin-top: 6px;
            font-size: 0.75rem;
        }
        button:hover {
            background: #ffa25e;
            transform: scale(1.02);
        }
        @media (max-width: 800px) {
            .controls-panel { flex-direction: column; bottom: 10px; left: 10px; right: 10px; padding: 12px; gap: 8px; }
            .info-text { display: none; }
            .control-group { min-width: auto; }
        }
        .title-intro {
            position: absolute;
            bottom: 100px;
            left: 20px;
            background: none;
            pointer-events: none;
            z-index: 10;
            font-size: 0.7rem;
            color: #ccc;
        }
    </style>
</head>
<body>
    <div class="info-text">
        🔬 <strong>BIOMECÁNICA DE MUÑECA Y MANO</strong><br>
        • Flexión/Extensión: movimiento en plano sagital<br>
        • Desviación radial/cubital: plano frontal<br>
        • Flexión digital (agarre): articulaciones MCP, PIP<br>
        🖱️ <strong>Interactúa</strong>: arrastra para rotar/zoom. Ajusta los sliders y observa los rangos fisiológicos.
    </div>
    <div class="controls-panel">
        <div class="control-group">
            <h3>🌀 MUÑECA <span class="badge">Flexión / Extensión</span></h3>
            <label>Flexión plantar / Dorsal <span id="flexVal" class="angle-value">0°</span></label>
            <input type="range" id="wristFlex" min="-60" max="60" step="1" value="0">
            <div style="font-size:12px; opacity:0.7;">⬅️ Extensión | Flexión ➡️</div>
        </div>
        <div class="control-group">
            <h3>↔️ MUÑECA <span class="badge">Desviación</span></h3>
            <label>Radial (-) / Cubital (+) <span id="devVal" class="angle-value">0°</span></label>
            <input type="range" id="wristDev" min="-25" max="25" step="1" value="0">
            <div style="font-size:12px;">Desviación radial / cubital</div>
        </div>
        <div class="control-group">
            <h3>✊ MANO <span class="badge">Flexión de dedos</span></h3>
            <label>Agarre / Extensión <span id="fingerVal" class="angle-value">0%</span></label>
            <input type="range" id="fingerFlex" min="0" max="100" step="1" value="30">
            <button id="resetBtn">⟳ Restablecer postura neutra</button>
        </div>
    </div>
    <div class="title-intro">
        ⚡ Simulación 3D interactiva | Estudiante de Biomecánica | Modelo anatómico funcional
    </div>

    <!-- Importación de Three.js y extensiones -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- Configuración inicial de escena ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x071a2b);
        scene.fog = new THREE.FogExp2(0x071a2b, 0.008);

        // Cámara
        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0.8, 0.6, 1.2);
        camera.lookAt(0, 0.1, 0.3);

        // Renderers
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // sombras suaves
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);
        
        // CSS2DRenderer para etiquetas anatómicas
        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.left = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(labelRenderer.domElement);

        // Controles de órbita (permite explorar desde cualquier ángulo)
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.rotateSpeed = 1.2;
        controls.zoomSpeed = 1.2;
        controls.enableZoom = true;
        controls.autoRotate = false;
        controls.target.set(0, 0.1, 0.35);

        // --- Iluminación realista ---
        // Luz ambiental
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        // Luz principal direccional
        const mainLight = new THREE.DirectionalLight(0xfff5e6, 1.2);
        mainLight.position.set(1, 2, 1.5);
        mainLight.castShadow = true;
        mainLight.receiveShadow = false;
        mainLight.shadow.mapSize.width = 1024;
        scene.add(mainLight);
        // Luz de relleno cálida desde atrás
        const backLight = new THREE.PointLight(0xcc9966, 0.5);
        backLight.position.set(-0.5, 0.4, -0.8);
        scene.add(backLight);
        // Luz fría lateral
        const fillLight = new THREE.PointLight(0x88aaff, 0.4);
        fillLight.position.set(0.8, 0.3, 0.6);
        scene.add(fillLight);
        // Luz inferior suave
        const rimLight = new THREE.PointLight(0xffaa77, 0.3);
        rimLight.position.set(0, -0.4, 0.4);
        scene.add(rimLight);
        
        // Plano de suelo sutil (referencia visual)
        const gridHelper = new THREE.GridHelper(2.5, 20, 0x88aaff, 0x335588);
        gridHelper.position.y = -0.35;
        gridHelper.material.transparent = true;
        gridHelper.material.opacity = 0.25;
        scene.add(gridHelper);
        
        // --- Estructura jerárquica para la muñeca y mano ---
        // Grupo fijo: antebrazo (no rota con muñeca)
        const forearmGroup = new THREE.Group();
        scene.add(forearmGroup);
        
        // Antebrazo: cilindro estilizado
        const forearmGeo = new THREE.CylinderGeometry(0.12, 0.1, 0.65, 24);
        const forearmMat = new THREE.MeshStandardMaterial({ color: 0xd9b48b, roughness: 0.4, metalness: 0.1 });
        const forearmMesh = new THREE.Mesh(forearmGeo, forearmMat);
        forearmMesh.castShadow = true;
        forearmMesh.receiveShadow = false;
        forearmMesh.position.set(0, -0.02, -0.32);
        forearmMesh.rotation.x = 0.05;
        forearmGroup.add(forearmMesh);
        
        // Hueso radio/ulna decorativo (línea lateral)
        const boneLineMat = new THREE.MeshStandardMaterial({ color: 0xc2a070, emissive: 0x221100 });
        const radiusStick = new THREE.Mesh(new THREE.CylinderGeometry(0.025, 0.025, 0.6, 6), boneLineMat);
        radiusStick.position.set(0.07, -0.05, -0.3);
        radiusStick.rotation.z = 0.2;
        forearmGroup.add(radiusStick);
        
        // Punto de articulación de muñeca (esfera indicadora)
        const wristJointMarker = new THREE.Mesh(
            new THREE.SphereGeometry(0.045, 16, 16),
            new THREE.MeshStandardMaterial({ color: 0xff8866, emissive: 0x442200, roughness: 0.3 })
        );
        wristJointMarker.position.set(0, 0.01, 0);
        wristJointMarker.castShadow = true;
        scene.add(wristJointMarker);
        
        // --- Grupo de muñeca: representa el complejo radiocarpiano, permite movimientos ---
        const wristGroup = new THREE.Group();
        wristGroup.position.set(0, 0.01, 0);
        scene.add(wristGroup);
        
        // Base de la mano (carpo + metacarpo)
        const handBaseGeo = new THREE.BoxGeometry(0.28, 0.12, 0.32);
        const handMat = new THREE.MeshStandardMaterial({ color: 0xe0b487, roughness: 0.35, flatShading: false });
        const handBase = new THREE.Mesh(handBaseGeo, handMat);
        handBase.castShadow = true;
        handBase.receiveShadow = true;
        handBase.position.set(0, 0.03, 0.12);
        wristGroup.add(handBase);
        
        // --- Modelado de dedos con articulaciones (dos falanges por dedo + pulgar especial) ---
        // Configuración de posiciones de los dedos (coordenadas relativas a la mano)
        const fingerPositions = [
            { name: "Índice", pos: [0.07, 0.055, 0.28], scale: [0.045, 0.045, 0.13], color: 0xe0b487, angleFactor: 1.0 },
            { name: "Medio", pos: [0.02, 0.058, 0.29], scale: [0.048, 0.048, 0.14], color: 0xe0b487, angleFactor: 1.0 },
            { name: "Anular", pos: [-0.03, 0.055, 0.28], scale: [0.046, 0.046, 0.13], color: 0xe0b487, angleFactor: 1.0 },
            { name: "Meñique", pos: [-0.075, 0.048, 0.26], scale: [0.038, 0.038, 0.11], color: 0xe0b487, angleFactor: 0.85 }
        ];
        
        // Almacenar referencias de grupos de dedos para animación de flexión
        const fingerJoints = []; // cada elemento: { proximal, distal, basePos, factor }
        
        // Crear cada dedo con dos segmentos (falange proximal y media/distal combinada visualmente pero articulada)
        fingerPositions.forEach((finger, idx) => {
            const fingerGroup = new THREE.Group();
            fingerGroup.position.set(finger.pos[0], finger.pos[1], finger.pos[2]);
            wristGroup.add(fingerGroup);
            
            // Falange proximal (articulación MCP)
            const proxGeo = new THREE.BoxGeometry(finger.scale[0], finger.scale[1], finger.scale[2]);
            const proxMat = new THREE.MeshStandardMaterial({ color: finger.color, roughness: 0.3 });
            const proximal = new THREE.Mesh(proxGeo, proxMat);
            proximal.castShadow = true;
            proximal.position.set(0, 0, finger.scale[2] / 2);
            fingerGroup.add(proximal);
            
            // Falange media/distal (articulación PIP)
            const distalGeo = new THREE.BoxGeometry(finger.scale[0] * 0.85, finger.scale[1] * 0.9, finger.scale[2] * 0.8);
            const distalMat = new THREE.MeshStandardMaterial({ color: 0xd4a373, roughness: 0.4 });
            const distal = new THREE.Mesh(distalGeo, distalMat);
            distal.castShadow = true;
            distal.position.set(0, 0, finger.scale[2] + 0.055);
            fingerGroup.add(distal);
            
            // Articulaciones esféricas decorativas (realce biomecánico)
            const jointSphere = new THREE.Mesh(
                new THREE.SphereGeometry(0.018, 8, 8),
                new THREE.MeshStandardMaterial({ color: 0xff9966, emissive: 0x331100 })
            );
            jointSphere.position.set(0, 0, finger.scale[2]);
            fingerGroup.add(jointSphere);
            
            // Guardar referencias para control de flexión
            fingerJoints.push({
                group: fingerGroup,
                proximal: proximal,
                distal: distal,
                baseLen: finger.scale[2],
                factor: finger.angleFactor
            });
        });
        
        // --- PULGAR (articulación especial con orientación diferente) ---
        const thumbGroup = new THREE.Group();
        thumbGroup.position.set(0.11, 0.02, 0.21);
        wristGroup.add(thumbGroup);
        // Rotación basal del pulgar (oposición simulada)
        thumbGroup.rotation.z = -0.35;
        thumbGroup.rotation.x = 0.2;
        
        const thumbProxGeo = new THREE.BoxGeometry(0.045, 0.045, 0.11);
        const thumbProx = new THREE.Mesh(thumbProxGeo, new THREE.MeshStandardMaterial({ color: 0xdcb185, roughness: 0.3 }));
        thumbProx.castShadow = true;
        thumbProx.position.set(0, 0, 0.055);
        thumbGroup.add(thumbProx);
        
        const thumbDistGeo = new THREE.BoxGeometry(0.04, 0.04, 0.09);
        const thumbDist = new THREE.Mesh(thumbDistGeo, new THREE.MeshStandardMaterial({ color: 0xc9996e }));
        thumbDist.castShadow = true;
        thumbDist.position.set(0, 0, 0.12);
        thumbGroup.add(thumbDist);
        
        // Articulación pulgar
        const thumbJoint = new THREE.Mesh(new THREE.SphereGeometry(0.016, 6, 6), new THREE.MeshStandardMaterial({ color: 0xffaa77 }));
        thumbJoint.position.set(0, 0, 0.105);
        thumbGroup.add(thumbJoint);
        
        // Añadir referencias para flexión del pulgar también
        const thumbParts = { group: thumbGroup, proximal: thumbProx, distal: thumbDist };
        
        // --- Etiquetas CSS2D: aprendizaje visual ---
        function makeLabel(text, position, color = "#ffcc99") {
            const div = document.createElement('div');
            div.textContent = text;
            div.style.color = color;
            div.style.fontSize = "14px";
            div.style.fontWeight = "bold";
            div.style.textShadow = "1px 1px 0px black";
            div.style.background = "rgba(20,30,45,0.75)";
            div.style.padding = "2px 8px";
            div.style.borderRadius = "20px";
            div.style.borderLeft = `3px solid ${color}`;
            div.style.fontFamily = "monospace";
            div.style.backdropFilter = "blur(4px)";
            const label = new CSS2DObject(div);
            label.position.copy(position);
            scene.add(label);
            return label;
        }
        
        makeLabel("🔹 Radio / Cúbito", new THREE.Vector3(0.15, -0.12, -0.45), "#ffaa88");
        makeLabel("🔸 Articulación de la Muñeca (Radiocarpiana)", new THREE.Vector3(0, 0.12, 0.02), "#ff9966");
        makeLabel("🖐️ Carpo / Metacarpo", new THREE.Vector3(0, 0.12, 0.2), "#ffaa77");
        makeLabel("📌 Flexión / Extensión", new THREE.Vector3(0.35, 0.05, 0.15), "#88ccff");
        makeLabel("📌 Desviación Radial/Cubital", new THREE.Vector3(-0.32, 0.08, 0.05), "#88ccff");
        makeLabel("✋ Falanges proximales y medias", new THREE.Vector3(0.15, 0.17, 0.45), "#ffccaa");
        
        // --- Control de variables biomecánicas ---
        // Valores actuales
        let currentFlex = 0;    // grados
        let currentDev = 0;
        let currentGrip = 0.3;  // 0-1
        
        // Elementos DOM
        const flexSlider = document.getElementById('wristFlex');
        const devSlider = document.getElementById('wristDev');
        const fingerSlider = document.getElementById('fingerFlex');
        const flexValSpan = document.getElementById('flexVal');
        const devValSpan = document.getElementById('devVal');
        const fingerValSpan = document.getElementById('fingerVal');
        const resetBtn = document.getElementById('resetBtn');
        
        // Función para actualizar rotación de muñeca (flexión/extensión y desviación)
        function updateWrist() {
            const flexRad = currentFlex * Math.PI / 180;
            const devRad = currentDev * Math.PI / 180;
            // Orden de rotaciones: primero desviación (Z) luego flexión (X) para simular movimientos naturales
            wristGroup.rotation.x = flexRad;
            wristGroup.rotation.z = devRad;
            // Actualizar marcador esférico: pequeño efecto visual de movimiento
            wristJointMarker.position.set(devRad * 0.03, flexRad * 0.02, 0);
        }
        
        // Función para actualizar flexión de dedos y pulgar (simula contracción de flexores)
        function updateFingers() {
            // El valor grip entre 0 y 1 mapea a ángulos de las articulaciones MCP y PIP (máx ~80°)
            const flexAngleProx = currentGrip * 1.2;  // radianes (máximo ~70°)
            const flexAngleDist = currentGrip * 0.9;
            
            fingerJoints.forEach(joint => {
                // Aplicar rotación en cada grupo considerando factor individual
                const proxRot = Math.min(flexAngleProx * joint.factor, 1.3);
                const distRot = Math.min(flexAngleDist * joint.factor, 1.0);
                // Rotación de la falange proximal (articulación MCP) alrededor del eje X
                if (joint.proximal) {
                    joint.proximal.rotation.x = proxRot;
                }
                // Rotación distal relativa (PIP)
                if (joint.distal) {
                    joint.distal.rotation.x = distRot;
                }
                // Pequeño desplazamiento estético para evitar interpenetraciones
                if (joint.distal) {
                    joint.distal.position.z = joint.baseLen + 0.045 + (proxRot * 0.008);
                }
            });
            
            // Pulgar: flexión independiente (agarre)
            const thumbAngle = currentGrip * 0.9;
            thumbParts.proximal.rotation.x = thumbAngle * 0.7;
            thumbParts.distal.rotation.x = thumbAngle * 0.8;
            // Ajustar posición por realce
            thumbParts.distal.position.z = 0.12 + (thumbAngle * 0.01);
        }
        
        // --- Eventos de los sliders ---
        flexSlider.addEventListener('input', (e) => {
            currentFlex = parseFloat(e.target.value);
            flexValSpan.textContent = currentFlex + "°";
            updateWrist();
        });
        devSlider.addEventListener('input', (e) => {
            currentDev = parseFloat(e.target.value);
            devValSpan.textContent = currentDev + "°";
            updateWrist();
        });
        fingerSlider.addEventListener('input', (e) => {
            currentGrip = parseFloat(e.target.value) / 100;
            fingerValSpan.textContent = e.target.value + "%";
            updateFingers();
        });
        
        resetBtn.addEventListener('click', () => {
            // Resetear sliders
            flexSlider.value = "0";
            devSlider.value = "0";
            fingerSlider.value = "30";
            currentFlex = 0;
            currentDev = 0;
            currentGrip = 0.3;
            flexValSpan.textContent = "0°";
            devValSpan.textContent = "0°";
            fingerValSpan.textContent = "30%";
            updateWrist();
            updateFingers();
            // Pequeña animación de rebote en la cámara (opcional)
            camera.position.set(0.8, 0.6, 1.2);
            controls.target.set(0, 0.1, 0.35);
            controls.update();
        });
        
        // Inicializar postura neutra pero con ligera flexión inicial para aspecto natural
        currentGrip = 0.3;
        fingerSlider.value = "30";
        fingerValSpan.textContent = "30%";
        updateFingers();
        updateWrist();
        
        // Detalles estéticos: agregar tendones / venas sutiles (líneas decorativas)
        const addTendonStripe = (start, end, parent, color=0xccaa88) => {
            const points = [new THREE.Vector3(start[0], start[1], start[2]), new THREE.Vector3(end[0], end[1], end[2])];
            const geometry = new THREE.BufferGeometry().setFromPoints(points);
            const material = new THREE.LineBasicMaterial({ color: color });
            const line = new THREE.Line(geometry, material);
            parent.add(line);
        };
        // Pequeños detalles en la mano (simulación de pliegues)
        const detailGroup = new THREE.Group();
        wristGroup.add(detailGroup);
        const tendonMat = new THREE.LineBasicMaterial({ color: 0xc9a87c });
        const pointsHand = [
            [[-0.05,0.07,0.22],[0,0.09,0.26]], [[0.02,0.09,0.27],[0.07,0.07,0.25]], [[-0.08,0.05,0.23],[-0.04,0.08,0.25]]
        ];
        pointsHand.forEach(p => {
            const geom = new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(p[0][0], p[0][1], p[0][2]), new THREE.Vector3(p[1][0], p[1][1], p[1][2])]);
            const line = new THREE.Line(geom, tendonMat);
            wristGroup.add(line);
        });
        
        // Añadir un pequeño anillo alrededor de la muñeca para destacar el eje articular
        const wristRingGeo = new THREE.TorusGeometry(0.075, 0.008, 32, 64);
        const ringMat = new THREE.MeshStandardMaterial({ color: 0xffaa77, emissive: 0x442200 });
        const wristRing = new THREE.Mesh(wristRingGeo, ringMat);
        wristRing.rotation.x = Math.PI / 2;
        wristRing.position.set(0, 0.01, 0.04);
        wristGroup.add(wristRing);
        
        // --- Animación suave partículas / brillo (opcional) ---
        const particleGeo = new THREE.BufferGeometry();
        const particleCount = 300;
        const particlePositions = new Float32Array(particleCount * 3);
        for (let i = 0; i < particleCount; i++) {
            particlePositions[i*3] = (Math.random() - 0.5) * 2.5;
            particlePositions[i*3+1] = (Math.random() - 0.5) * 1.2 - 0.2;
            particlePositions[i*3+2] = (Math.random() - 0.5) * 1.5 - 0.2;
        }
        particleGeo.setAttribute('position', new THREE.BufferAttribute(particlePositions, 3));
        const particleMat = new THREE.PointsMaterial({ color: 0x88aaff, size: 0.008, transparent: true, opacity: 0.4 });
        const particles = new THREE.Points(particleGeo, particleMat);
        scene.add(particles);
        
        // --- Bucle de animación y pequeña interacción dinámica ---
        let time = 0;
        function animate() {
            requestAnimationFrame(animate);
            time += 0.012;
            
            // Efecto de brillo en la articulación de muñeca
            const intensity = 0.5 + Math.sin(time * 5) * 0.2;
            wristJointMarker.material.emissiveIntensity = intensity * 0.6;
            
            // Partículas flotantes sutiles
            particles.rotation.y = time * 0.05;
            particles.rotation.x = Math.sin(time * 0.2) * 0.1;
            
            // Actualizar controles de órbita y render
            controls.update();
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
        }
        
        animate();
        
        // Manejar redimensionamiento de ventana
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        // Pequeño mensaje en consola amigable
        console.log("✅ Simulación 3D de Muñeca y Mano activa | Biomecánica interactiva");
        
        // Tooltips virtuales: guía didáctica adicional en la consola no molesta
        const styleGuide = document.createElement('style');
        styleGuide.textContent = `input[type=range] { -webkit-appearance: none; background: #3a5a6e; } input[type=range]::-webkit-slider-thumb { -webkit-appearance: none; width: 16px; height: 16px; border-radius: 50%; background: #ff8c42; cursor: pointer; }`;
        document.head.appendChild(styleGuide);
    </script>
</body>
</html>
