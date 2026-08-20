# Word-Craftsman<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Craft Edit</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: sans-serif; touch-action: none; }
        #crosshair {
            position: absolute; top: 50%; left: 50%;
            width: 10px; height: 10px;
            background: rgba(255, 255, 255, 0.8);
            transform: translate(-50%, -50%);
            border-radius: 50%; pointer-events: none;
        }
        #ui {
            position: absolute; bottom: 20px; left: 50%;
            transform: translateX(-50%);
            display: flex; gap: 10px;
        }
        button {
            padding: 12px 20px; font-size: 16px; font-weight: bold;
            background: rgba(0, 0, 0, 0.6); color: white;
            border: 1px solid white; border-radius: 8px;
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.min.js"></script>
</head>
<body>

<div id="crosshair"></div>
<div id="ui">
    <button id="btn-build">Tambah Blok</button>
</div>

<script>
    // Setup Scene, Camera, Renderer
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x87ceeb); // Warna langit

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: false }); // Antialias off agar ringan
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Pencahayaan
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(5, 10, 7.5).normalize();
    scene.add(light);
    scene.add(new THREE.AmbientLight(0x404040));

    // Buat Lantai Rumput (Dunia Sederhana)
    const blocks = [];
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshLambertMaterial({ color: 0x4caf50 });

    for (let x = -5; x <= 5; x++) {
        for (let z = -5; z <= 5; z++) {
            const cube = new THREE.Mesh(geometry, material);
            cube.position.set(x, 0, z);
            scene.add(cube);
            blocks.push(cube);
        }
    }

    camera.position.set(0, 2, 5);

    // Bintang/Kursor Pemilih Blok
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2(0, 0); // Pusat layar

    // Tombol Pasang Blok
    document.getElementById('btn-build').addEventListener('click', () => {
        raycaster.setFromCamera(mouse, camera);
        const intersects = raycaster.intersectObjects(blocks);

        if (intersects.length > 0) {
            const intersect = intersects[0];
            const newCube = new THREE.Mesh(geometry, new THREE.MeshLambertMaterial({ color: 0x8b5a2b }));
            newCube.position.copy(intersect.point).add(intersect.face.normal).floor().addScalar(0.5);
            scene.add(newCube);
            blocks.push(newCube);
        }
    });

    // Kontrol Sentuh HP untuk Memutar Kamera
    let isDragging = false;
    let previousTouchPosition = { x: 0, y: 0 };

    window.addEventListener('touchstart', (e) => {
        if (e.target.tagName !== 'BUTTON') {
            isDragging = true;
            previousTouchPosition = { x: e.touches[0].clientX, y: e.touches[0].clientY };
        }
    });

    window.addEventListener('touchmove', (e) => {
        if (!isDragging) return;
        const deltaMove = {
            x: e.touches[0].clientX - previousTouchPosition.x,
            y: e.touches[0].clientY - previousTouchPosition.y
        };

        camera.rotation.y -= deltaMove.x * 0.005;
        camera.rotation.x -= deltaMove.y * 0.005;

        previousTouchPosition = { x: e.touches[0].clientX, y: e.touches[0].clientY };
    });

    window.addEventListener('touchend', () => { isDragging = false; });

    // Loop Animasi Game
    function animate() {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
    }
    animate();

    // Penyesuaian Ukuran Layar HP
    window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    });
</script>
</body>
</html>
