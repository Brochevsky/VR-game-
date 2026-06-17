<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>VR Руки — Шарик</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #111;
            font-family: Arial, sans-serif;
            touch-action: none;
        }
        #video {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
            z-index: 1;
            transform: scaleX(-1);
            display: none; /* Не показываем видео, только руки в 3D */
        }
        #canvas3d {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 2;
        }
        #hint {
            position: absolute;
            bottom: 100px;
            left: 0;
            width: 100%;
            text-align: center;
            color: rgba(255, 255, 255, 0.8);
            font-size: 18px;
            z-index: 10;
            pointer-events: none;
            text-shadow: 0 0 20px rgba(0, 0, 0, 0.9);
            padding: 0 20px;
            box-sizing: border-box;
            background: rgba(0, 0, 0, 0.5);
            padding: 15px;
            border-radius: 10px;
            max-width: 90%;
            left: 50%;
            transform: translateX(-50%);
        }
        #status {
            position: absolute;
            top: 20px;
            left: 20px;
            color: #88ff88;
            font-size: 14px;
            z-index: 10;
            background: rgba(0, 0, 0, 0.7);
            padding: 8px 16px;
            border-radius: 20px;
            pointer-events: none;
        }
        @media (max-width: 600px) {
            #hint { font-size: 14px; bottom: 60px; padding: 12px; }
        }
    </style>
</head>
<body>

    <div id="status">⏳ Загрузка...</div>
    <div id="hint">👐 Покажи руки камере<br>Сведи указательный и большой пальцы → схватить шарик</div>

    <!-- Видео с камеры (скрыто) -->
    <video id="video" playsinline></video>

    <!-- 3D сцена -->
    <div id="canvas3d"></div>

    <!-- Подключаем библиотеки -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>

    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/",
                "cannon-es": "https://cdn.jsdelivr.net/npm/cannon-es@0.20.0/dist/cannon-es.js"
            }
        }
    </script>

    <script type="module">
        // --- ИМПОРТЫ ---
        import * as THREE from 'three';
        import * as CANNON from 'cannon-es';

        // --- ЭЛЕМЕНТЫ DOM ---
        const container = document.getElementById('canvas3d');
        const video = document.getElementById('video');
        const statusEl = document.getElementById('status');
        const hintEl = document.getElementById('hint');

        // --- НАСТРОЙКИ ---
        const CONFIG = {
            eyeSep: 0.06,
            scale: 1.0,
            grabDistance: 0.5, // дистанция захвата в метрах (виртуальных)
            lerpSpeed: 0.15,
        };

        // --- СЦЕНА THREE ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x111122);

        // --- КАМЕРЫ (стерео) ---
        const aspect = window.innerWidth / window.innerHeight;
        const cameraL = new THREE.PerspectiveCamera(70, aspect * 0.5, 0.1, 20);
        const cameraR = new THREE.PerspectiveCamera(70, aspect * 0.5, 0.1, 20);
        cameraL.position.set(-CONFIG.eyeSep / 2, 0, 0);
        cameraR.position.set(CONFIG.eyeSep / 2, 0, 0);
        cameraL.lookAt(0, 0, -5);
        cameraR.lookAt(0, 0, -5);

        // --- РЕНДЕР ---
        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        container.appendChild(renderer.domElement);

        // --- СВЕТ ---
        const ambient = new THREE.AmbientLight(0x404066);
        scene.add(ambient);

        const dirLight = new THREE.DirectionalLight(0xffeedd, 1.2);
        dirLight.position.set(2, 5, 3);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 512;
        dirLight.shadow.mapSize.height = 512;
        const d = 5;
        dirLight.shadow.camera.left = -d;
        dirLight.shadow.camera.right = d;
        dirLight.shadow.camera.top = d;
        dirLight.shadow.camera.bottom = -d;
        dirLight.shadow.camera.near = 1;
        dirLight.shadow.camera.far = 10;
        scene.add(dirLight);

        const fillLight = new THREE.DirectionalLight(0x4488ff, 0.4);
        fillLight.position.set(-2, 1, -2);
        scene.add(fillLight);

        // --- ПОЛ ---
        const floorMat = new THREE.MeshStandardMaterial({
            color: 0x2a2a4a,
            roughness: 0.7,
            metalness: 0.1,
            transparent: true,
            opacity: 0.9,
        });
        const floor = new THREE.Mesh(new THREE.PlaneGeometry(8, 8), floorMat);
        floor.rotation.x = -Math.PI / 2;
        floor.position.y = -0.5;
        floor.receiveShadow = true;
        scene.add(floor);

        const gridHelper = new THREE.GridHelper(8, 20, 0x88ccff, 0x445588);
        gridHelper.position.y = -0.48;
        scene.add(gridHelper);

        // --- ШАРИК ---
        const ballRadius = 0.3;
        const ballGeo = new THREE.SphereGeometry(ballRadius, 32, 32);
        const ballMat = new THREE.MeshStandardMaterial({
            color: 0xff5533,
            roughness: 0.2,
            metalness: 0.6,
            emissive: 0xff2200,
            emissiveIntensity: 0.15,
        });
        const ballMesh = new THREE.Mesh(ballGeo, ballMat);
        ballMesh.castShadow = true;
        ballMesh.receiveShadow = true;
        ballMesh.position.set(0, 0.3, -2);
        scene.add(ballMesh);

        // --- РУКИ (визуал в VR) ---
        const handMat = new THREE.MeshStandardMaterial({
            color: 0x00ccff,
            emissive: 0x0088ff,
            emissiveIntensity: 0.3,
            roughness: 0.2,
            metalness: 0.1,
            transparent: true,
            opacity: 0.8,
        });
        const handGeo = new THREE.SphereGeometry(0.08, 16, 16);

        const handL = new THREE.Mesh(handGeo, handMat);
        const handR = new THREE.Mesh(handGeo, handMat);
        handL.position.set(-0.5, 0.5, -1.5);
        handR.position.set(0.5, 0.5, -1.5);
        scene.add(handL);
        scene.add(handR);

        // Пальцы (указательный и большой)
        const fingerMat = new THREE.MeshStandardMaterial({
            color: 0x88ddff,
            emissive: 0x4488ff,
            emissiveIntensity: 0.1,
        });
        const fingerGeo = new THREE.SphereGeometry(0.04, 8, 8);

        // Для левой руки
        const thumbL = new THREE.Mesh(fingerGeo, fingerMat);
        const indexL = new THREE.Mesh(fingerGeo, fingerMat);
        thumbL.position.set(-0.5, 0.5, -1.5);
        indexL.position.set(-0.5, 0.5, -1.5);
        scene.add(thumbL);
        scene.add(indexL);

        // Для правой руки
        const thumbR = new THREE.Mesh(fingerGeo, fingerMat);
        const indexR = new THREE.Mesh(fingerGeo, fingerMat);
        thumbR.position.set(0.5, 0.5, -1.5);
        indexR.position.set(0.5, 0.5, -1.5);
        scene.add(thumbR);
        scene.add(indexR);

        // Линии от камеры к рукам
        const lineMat = new THREE.LineBasicMaterial({ color: 0x44aaff, transparent: true, opacity: 0.15 });
        const createLine = () => {
            const geo = new THREE.BufferGeometry().setFromPoints([
                new THREE.Vector3(0, 0, -0.3),
                new THREE.Vector3(0, 0, -1.5)
            ]);
            return new THREE.Line(geo, lineMat);
        };
        const lineL = createLine();
        const lineR = createLine();
        scene.add(lineL);
        scene.add(lineR);

        // --- ФИЗИКА (Cannon) ---
        const world = new CANNON.World();
        world.gravity.set(0, -9.82, 0);
        world.broadphase = new CANNON.SAPBroadphase(world);
        world.defaultContactMaterial.restitution = 0.6;
        world.defaultContactMaterial.friction = 0.3;

        const floorBody = new CANNON.Body({ mass: 0 });
        floorBody.addShape(new CANNON.Plane());
        floorBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), -Math.PI / 2);
        floorBody.position.y = -0.5;
        world.addBody(floorBody);

        const ballMatPhys = new CANNON.Material('ball');
        const ballBody = new CANNON.Body({ mass: 1, material: ballMatPhys });
        ballBody.addShape(new CANNON.Sphere(ballRadius));
        ballBody.position.set(0, 0.3, -2);
        ballBody.linearDamping = 0.05;
        ballBody.angularDamping = 0.1;
        world.addBody(ballBody);

        const contactMat = new CANNON.ContactMaterial(
            ballMatPhys,
            world.defaultMaterial,
            { restitution: 0.6, friction: 0.3 }
        );
        world.addContactMaterial(contactMat);

        // --- ГИРОСКОП (поворот головы) ---
        let beta = 0,
            gamma = 0;
        if (window.DeviceOrientationEvent) {
            window.addEventListener('deviceorientation', (e) => {
                beta = e.beta || 0;
                gamma = e.gamma || 0;
            }, true);
        }

        // --- СОСТОЯНИЕ ЗАХВАТА ---
        let isGrabbing = false;
        let grabOffset = new THREE.Vector3();
        let handWorldPos = new THREE.Vector3();

        // --- ПОЛУЧАЕМ ДАННЫЕ О РУКАХ ИЗ MEDIAPIPE ---
        let handsData = {
            left: { thumb: null, index: null, wrist: null },
            right: { thumb: null, index: null, wrist: null }
        };
        let handsDetected = false;

        // Функция обновления рук из MediaPipe
        window.updateHands = function(landmarks, handedness) {
            handsDetected = true;
            const isLeft = handedness === 'Left';
            const hand = isLeft ? handsData.left : handsData.right;

            // Берем ключевые точки
            const wrist = landmarks[0];
            const thumbTip = landmarks[4];
            const indexTip = landmarks[8];

            // Переводим в 3D координаты (масштабируем под сцену)
            const scale = 2.5;
            const offsetX = 0;
            const offsetY = 0.2;
            const offsetZ = -2.5;

            hand.wrist = new THREE.Vector3(
                (wrist.x - 0.5) * scale + offsetX,
                (0.5 - wrist.y) * scale + offsetY,
                wrist.z * scale + offsetZ
            );
            hand.thumb = new THREE.Vector3(
                (thumbTip.x - 0.5) * scale + offsetX,
                (0.5 - thumbTip.y) * scale + offsetY,
                thumbTip.z * scale + offsetZ
            );
            hand.index = new THREE.Vector3(
                (indexTip.x - 0.5) * scale + offsetX,
                (0.5 - indexTip.y) * scale + offsetY,
                indexTip.z * scale + offsetZ
            );

            // Если есть обе руки — проверяем жест "щипок"
            if (handsData.left.thumb && handsData.right.thumb) {
                checkPinch();
            }
        };

        // --- ПРОВЕРКА ЖЕСТА "ЩИПОК" ---
        function checkPinch() {
            const leftThumb = handsData.left.thumb;
            const leftIndex = handsData.left.index;
            const rightThumb = handsData.right.thumb;
            const rightIndex = handsData.right.index;

            if (!leftThumb || !leftIndex || !rightThumb || !rightIndex) return;

            // Расстояние между большим и указательным на левой руке
            const distL = leftThumb.distanceTo(leftIndex);
            // Расстояние между большим и указательным на правой руке
            const distR = rightThumb.distanceTo(rightIndex);

            const pinchThreshold = 0.08; // порог срабатывания

            // Если обе руки "щипают" — схватить
            if (distL < pinchThreshold && distR < pinchThreshold) {
                if (!isGrabbing) {
                    // Проверяем, рядом ли шарик
                    const handPos = new THREE.Vector3();
                    handPos.copy(leftThumb).add(rightThumb).multiplyScalar(0.5);
                    const distToBall = handPos.distanceTo(ballMesh.position);
                    if (distToBall < CONFIG.grabDistance) {
                        isGrabbing = true;
                        grabOffset.copy(ballMesh.position).sub(handPos);
                        ballBody.velocity.set(0, 0, 0);
                        ballBody.angularVelocity.set(0, 0, 0);
                        ballMat.color.setHex(0x44ff88);
                        ballMat.emissive.setHex(0x00ff44);
                        hintEl.style.opacity = '0';
                        statusEl.textContent = '🟢 ЗАХВАТ!';
                    }
                }
            } else {
                // Разжали пальцы — отпустить
                if (isGrabbing) {
                    isGrabbing = false;
                    ballMat.color.setHex(0xff5533);
                    ballMat.emissive.setHex(0xff2200);
                    statusEl.textContent = '🔄 ОТПУСТИЛ';
                    setTimeout(() => {
                        statusEl.textContent = '👋 Жду жест';
                    }, 1000);
                }
            }
        }

        // --- ЗАПУСК MEDIAPIPE ---
        async function initMediaPipe() {
            try {
                const hands = new Hands({
                    locateFile: (file) => {
                        return `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`;
                    }
                });

                hands.setOptions({
                    maxNumHands: 2,
                    modelComplexity: 1,
                    minDetectionConfidence: 0.7,
                    minTrackingConfidence: 0.6,
                });

                hands.onResults((results) => {
                    if (results.multiHandLandmarks && results.multiHandedness) {
                        for (let i = 0; i < results.multiHandLandmarks.length; i++) {
                            const landmarks = results.multiHandLandmarks[i];
                            const handedness = results.multiHandedness[i].label;
                            window.updateHands(landmarks, handedness);
                        }
                    }
                });

                // Запускаем камеру
                const camera = new Camera(video, {
                    onFrame: async () => {
                        await hands.send({ image: video });
                    },
                    width: 320,
                    height: 240,
                });
                await camera.start();

                statusEl.textContent = '✅ Камера работает! Покажи руки';
                statusEl.style.color = '#88ff88';
                console.log('✅ MediaPipe запущен');

                // Скрываем подсказку через 10 сек
                setTimeout(() => {
                    hintEl.style.opacity = '0.3';
                }, 10000);

            } catch (err) {
                console.error('❌ Ошибка MediaPipe:', err);
                statusEl.textContent = '❌ Ошибка камеры! Разреши доступ';
                statusEl.style.color = '#ff6644';
                hintEl.textContent = '⚠️ Разреши доступ к камере в браузере';
            }
        }

        // --- АНИМАЦИЯ ---
        function animate() {
            requestAnimationFrame(animate);

            // 1. Обновляем позиции рук из данных MediaPipe
            if (handsData.left.thumb && handsData.left.index) {
                // Левая рука
                const lPos = handsData.left.wrist || handsData.left.thumb;
                if (lPos) {
                    handL.position.lerp(lPos, CONFIG.lerpSpeed);
                    thumbL.position.lerp(handsData.left.thumb, CONFIG.lerpSpeed);
                    indexL.position.lerp(handsData.left.index, CONFIG.lerpSpeed);
                }

                // Правая рука
                const rPos = handsData.right.wrist || handsData.right.thumb;
                if (rPos) {
                    handR.position.lerp(rPos, CONFIG.lerpSpeed);
                    thumbR.position.lerp(handsData.right.thumb, CONFIG.lerpSpeed);
                    indexR.position.lerp(handsData.right.index, CONFIG.lerpSpeed);
                }

                // Линии к рукам
                lineL.position.copy(handL.position);
                lineR.position.copy(handR.position);
            }

            // 2. Камеры (поворот головы)
            const camTargetX = -gamma * 0.02;
            const camTargetY = -beta * 0.02 + 0.1;

            cameraL.position.x += (camTargetX - cameraL.position.x) * 0.1;
            cameraL.position.y += (camTargetY - cameraL.position.y) * 0.1;
            cameraL.lookAt(0, 0, -5);

            cameraR.position.x += (camTargetX - cameraR.position.x) * 0.1;
            cameraR.position.y += (camTargetY - cameraR.position.y) * 0.1;
            cameraR.lookAt(0, 0, -5);

            // 3. Физика
            world.step(1 / 60, 1 / 60, 3);
            ballMesh.position.copy(ballBody.position);
            ballMesh.quaternion.copy(ballBody.quaternion);

            // 4. Захват
            if (isGrabbing) {
                // Центр между руками
                const handPos = new THREE.Vector3();
                const lPos = handsData.left.thumb || handsData.left.wrist;
                const rPos = handsData.right.thumb || handsData.right.wrist;
                if (lPos && rPos) {
                    handPos.copy(lPos).add(rPos).multiplyScalar(0.5);
                    const targetPos = handPos.clone().add(grabOffset);
                    ballBody.position.x += (targetPos.x - ballBody.position.x) * 0.3;
                    ballBody.position.y += (targetPos.y - ballBody.position.y) * 0.3;
                    ballBody.position.z += (targetPos.z - ballBody.position.z) * 0.3;
                    ballBody.velocity.set(0, 0, 0);
                    ballBody.angularVelocity.set(0, 0, 0);
                }
            }

            // 5. Стерео-рендеринг
            const w = window.innerWidth;
            const h = window.innerHeight;
            renderer.setViewport(0, 0, w / 2, h);
            renderer.setScissor(0, 0, w / 2, h);
            renderer.setScissorTest(true);
            renderer.render(scene, cameraL);

            renderer.setViewport(w / 2, 0, w / 2, h);
            renderer.setScissor(w / 2, 0, w / 2, h);
            renderer.render(scene, cameraR);

            renderer.setScissorTest(false);
        }

        animate();

        // --- ЗАПУСК ---
        initMediaPipe();

        // --- АДАПТАЦИЯ ---
        window.addEventListener('resize', () => {
            const w = window.innerWidth;
            const h = window.innerHeight;
            renderer.setSize(w, h);
            const aspect = w / h;
            cameraL.aspect = aspect * 0.5;
            cameraR.aspect = aspect * 0.5;
            cameraL.updateProjectionMatrix();
            cameraR.updateProjectionMatrix();
        });

        document.addEventListener('touchmove', (e) => e.preventDefault(), { passive: false });

        console.log('🚀 VR с руками запущен! Покажи руки камере.');
    </script>
</body>
</html>
