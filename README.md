<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sürüş Simülasyonu</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: sans-serif; }
        #loading-screen {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            background: #000;
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 5rem;
            z-index: 10;
            transition: opacity 1s ease-in-out;
        }
        #game-container {
            display: none; /* Oyun başladığında görünür olacak */
        }
        #game-canvas {
            display: block;
        }
    </style>
</head>
<body>

    <div id="loading-screen">
        ASF game
    </div>

    <div id="game-container">
        <canvas id="game-canvas"></canvas>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js"></script>

    <script src="app.js"></script>

</body>
</html>
// Temel Three.js kurulumu
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ canvas: document.getElementById('game-canvas') });
renderer.setSize(window.innerWidth, window.innerHeight);

// Işıklandırma
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);
const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
directionalLight.position.set(5, 10, 7.5);
scene.add(directionalLight);

// Yol oluşturma
const roadGeometry = new THREE.PlaneGeometry(200, 10, 1, 1);
const roadMaterial = new THREE.MeshLambertMaterial({ color: 0x555555 });
const road = new THREE.Mesh(roadGeometry, roadMaterial);
road.rotation.x = -Math.PI / 2;
scene.add(road);

// GLTFLoader'ı başlatın ve araba modelini yükleyin
const loader = new THREE.GLTFLoader();
let car; // Arabayı global olarak tanımla

loader.load(
    'car_model.gltf', // Model dosyasının yolu
    function (gltf) {
        car = gltf.scene;
        // İndirdiğiniz modele göre boyutunu, pozisyonunu ve rotasyonunu ayarlayın.
        // Bu değerler, indirdiğiniz modelin yapısına göre değişebilir.
        // Örneğin, 1990'ların sonundaki meşhur Japon spor arabalarına benzeyen bir model
        // için aerodinamik bir gövde ve arka kısımda belirgin bir spoyler olabilir.
        car.scale.set(0.5, 0.5, 0.5); // Modeli küçült
        car.position.y = 0;           // Yükseklik ayarı
        scene.add(car);

        // Kamera ayarını model yüklendikten sonra yapın
        camera.position.z = car.position.z + 8;
        camera.position.y = car.position.y + 3;
        camera.lookAt(car.position);
    },
    // Yükleme sırasında ilerlemeyi görmek için (isteğe bağlı)
    function (xhr) {
        console.log((xhr.loaded / xhr.total * 100) + '% yüklendi');
    },
    // Hata durumunda
    function (error) {
        console.error('Bir hata oluştu:', error);
    }
);

// Pencere boyutu değiştiğinde renderer'ı güncelle
window.addEventListener('resize', () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
});

// Animasyon döngüsü
function animate() {
    requestAnimationFrame(animate);

    if (car) { // Eğer araba modeli yüklendiyse
        // Arabanın ve kameranın pozisyonunu güncelleyerek ileri hareket efekti ver
        car.position.z -= 0.1;
        camera.position.z -= 0.1;
    }

    renderer.render(scene, camera);
}

// Yükleme ekranını gizle ve oyunu başlat
window.addEventListener('load', () => {
    const loadingScreen = document.getElementById('loading-screen');
    const gameContainer = document.getElementById('game-container');

    loadingScreen.style.opacity = '0';
    setTimeout(() => {
        loadingScreen.style.display = 'none';
        gameContainer.style.display = 'block';
        animate();
    }, 1000); // Geçiş süresi
});

