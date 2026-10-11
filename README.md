# game-3d-vy

<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vy 3D Adventure</title>

<style>
* { box-sizing: border-box; }
body {
  margin: 0;
  overflow: hidden;
  font-family: Arial, sans-serif;
  background: #87ceeb;
}
canvas { display: block; }
#hud {
  position: fixed;
  top: 15px;
  left: 15px;
  color: white;
  background: #142b42cc;
  padding: 14px 18px;
  border-radius: 14px;
  line-height: 1.8;
  z-index: 2;
}
#message {
  position: fixed;
  top: 18px;
  left: 50%;
  transform: translateX(-50%);
  color: white;
  background: #142b42cc;
  padding: 10px 16px;
  border-radius: 12px;
  z-index: 2;
  text-align: center;
  white-space: nowrap;
}
#help {
  position: fixed;
  bottom: 15px;
  left: 50%;
  transform: translateX(-50%);
  background: #142b42cc;
  color: white;
  border-radius: 12px;
  padding: 10px 14px;
  text-align: center;
  width: max-content;
  max-width: 94vw;
  font-size: 13px;
  z-index: 2;
}
#restart {
  position: fixed;
  right: 15px;
  top: 15px;
  z-index: 3;
  padding: 12px 16px;
  border: 0;
  border-radius: 12px;
  background: #ffca28;
  font-weight: bold;
  cursor: pointer;
}
</style>

<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.186.0/build/three.module.js"
  }
}
</script>
</head>

<body>
<div id="hud">
  🌟 VY 3D ADVENTURE<br>
  🪙 Điểm: <span id="score">0</span><br>
  📍 Khu vực: Đảo xanh
</div>

<div id="message">Chào mừng đến với thế giới 3D!</div>

<button id="restart">🔄 Chơi lại</button>

<div id="help">
  Di chuyển: WASD / phím mũi tên
  · Nhảy: Space
  · Thu thập xu vàng!
</div>

<script type="module">
import * as THREE from 'three';

// 1. TẠO THẾ GIỚI 3D
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);
scene.fog = new THREE.Fog(0x87ceeb, 35, 100);

const camera = new THREE.PerspectiveCamera(
  70, window.innerWidth / window.innerHeight, 0.1, 200
);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

// 2. ÁNH SÁNG
scene.add(new THREE.HemisphereLight(0xffffff, 0x557744, 2));

const sun = new THREE.DirectionalLight(0xffffff, 2.5);
sun.position.set(10, 20, 10);
sun.castShadow = true;
scene.add(sun);

// 3. MẶT ĐẤT
const grass = new THREE.Mesh(
  new THREE.BoxGeometry(100, 1, 100),
  new THREE.MeshStandardMaterial({ color: 0x52b84b })
);
grass.position.y = -0.5;
grass.receiveShadow = true;
scene.add(grass);

// Tạo khối hộp 3D
function makeBox(x, y, z, sx, sy, sz, color) {
  const mesh = new THREE.Mesh(
    new THREE.BoxGeometry(sx, sy, sz),
    new THREE.MeshStandardMaterial({ color })
  );
  mesh.position.set(x, y, z);
  mesh.castShadow = true;
  mesh.receiveShadow = true;
  scene.add(mesh);
  return mesh;
}

// 4. CÁC KHỐI TRANG TRÍ VÀ NỀN TẢNG
for (let i = 0; i < 18; i++) {
  const x = (i % 6) * 12 - 30;
  const z = Math.floor(i / 6) * 14 - 15;

  makeBox(x, 1, z, 4, 2, 4, [
    0xff7675, 0x74b9ff, 0xa29bfe, 0xfdcb6e
  ][i % 4]);
}

// Các cây đơn giản
for (let i = 0; i < 35; i++) {
  const x = Math.random() * 80 - 40;
  const z = Math.random() * 80 - 40;

  if (Math.abs(x) < 8 && Math.abs(z) < 8) continue;

  makeBox(x, 1.2, z, 0.7, 2.4, 0.7, 0x8d5524);

  const leaves = new THREE.Mesh(
    new THREE.ConeGeometry(1.8, 4, 7),
    new THREE.MeshStandardMaterial({ color: 0x168b45 })
  );
  leaves.position.set(x, 4.2, z);
  leaves.castShadow = true;
  scene.add(leaves);
}

// 5. NHÂN VẬT
const player = new THREE.Group();
scene.add(player);

// Thân
const body = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1.2, 0.6),
  new THREE.MeshStandardMaterial({ color: 0x1677ff })
);
body.position.y = 1.3;
body.castShadow = true;
player.add(body);

// Đầu
const head = new THREE.Mesh(
  new THREE.BoxGeometry(0.8, 0.8, 0.8),
  new THREE.MeshStandardMaterial({ color: 0xffd0a1 })
);
head.position.y = 2.3;
head.castShadow = true;
player.add(head);

// Hai mắt
for (const x of [-0.2, 0.2]) {
  const eye = new THREE.Mesh(
    new THREE.SphereGeometry(0.07, 12, 12),
    new THREE.MeshBasicMaterial({ color: 0x111111 })
  );
  eye.position.set(x, 2.38, 0.405);
  player.add(eye);
}

// Chân
const legs = [];
for (const x of [-0.25, 0.25]) {
  const leg = new THREE.Mesh(
    new THREE.BoxGeometry(0.3, 0.7, 0.35),
    new THREE.MeshStandardMaterial({ color: 0x263b75 })
  );
  leg.position.set(x, 0.35, 0);
  leg.castShadow = true;
  player.add(leg);
  legs.push(leg);
}

player.position.set(0, 0, 0);

// 6. TẠO ĐỒNG XU
const coins = [];

for (let i = 0; i < 15; i++) {
  const coin = new THREE.Mesh(
    new THREE.TorusGeometry(0.42, 0.13, 10, 20),
    new THREE.MeshStandardMaterial({
      color: 0xffd700,
      metalness: 0.65,
      roughness: 0.25,
      emissive: 0x553300
    })
  );

  const angle = i * 2.4;
  coin.position.set(
    Math.sin(angle) * (5 + (i % 4) * 4),
    1.5,
    Math.cos(angle) * (5 + (i % 4) * 4)
  );

  coin.castShadow = true;
  scene.add(coin);
  coins.push(coin);
}

// 7. ĐIỀU KHIỂN
const keys = {};
let velocityY = 0;
let grounded = true;
let score = 0;
let won = false;

window.addEventListener('keydown', (e) => {
  keys[e.code] = true;

  if ([
    'Space', 'ArrowUp', 'ArrowDown',
    'ArrowLeft', 'ArrowRight'
  ].includes(e.code)) e.preventDefault();

  if (e.code === 'Space' && grounded && !won) {
    velocityY = 8;
    grounded = false;
  }
});

window.addEventListener('keyup', (e) => {
  keys[e.code] = false;
});

function resetGame() {
  player.position.set(0, 0, 0);
  velocityY = 0;
  grounded = true;
  score = 0;
  won = false;

  document.getElementById('score').textContent = score;
  document.getElementById('message').textContent =
    'Hãy đi tìm những đồng xu vàng!';

  coins.forEach(coin => {
    coin.visible = true;
  });
}

document.getElementById('restart').addEventListener(
  'click', resetGame
);

// 8. VÒNG LẶP GAME
const clock = new THREE.Clock();
const cameraOffset = new THREE.Vector3(0, 7, 10);

function animate() {
  requestAnimationFrame(animate);

  const dt = Math.min(clock.getDelta(), 0.05);
  const t = clock.elapsedTime;

  coins.forEach((coin, i) => {
    if (!coin.visible) return;

    coin.rotation.y += dt * 2;
    coin.position.y = 1.5 + Math.sin(t * 2 + i) * 0.2;

    if (player.position.distanceTo(coin.position) < 1.4) {
      coin.visible = false;
      score += 10;
      document.getElementById('score').textContent = score;

      document.getElementById('message').textContent =
        'Tuyệt vời! Bạn nhận được 10 điểm!';
    }
  });

  if (!won) {
    let dx = 0;
    let dz = 0;

    if (keys.KeyW || keys.ArrowUp) dz -= 1;
    if (keys.KeyS || keys.ArrowDown) dz += 1;
    if (keys.KeyA || keys.ArrowLeft) dx -= 1;
    if (keys.KeyD || keys.ArrowRight) dx += 1;

    const length = Math.hypot(dx, dz);

    if (length > 0) {
      dx /= length;
      dz /= length;

      player.position.x += dx * 7 * dt;
      player.position.z += dz * 7 * dt;

      player.rotation.y = Math.atan2(dx, dz);

      legs[0].rotation.x = Math.sin(t * 12) * 0.6;
      legs[1].rotation.x = -Math.sin(t * 12) * 0.6;
    } else {
      legs.forEach(leg => leg.rotation.x = 0);
    }

    // Giữ nhân vật trong bản đồ
    player.position.x = THREE.MathUtils.clamp(
      player.position.x, -47, 47
    );
    player.position.z = THREE.MathUtils.clamp(
      player.position.z, -47, 47
    );

    // Trọng lực và nhảy
    velocityY -= 20 * dt;
    player.position.y += velocityY * dt;

    if (player.position.y <= 0) {
      player.position.y = 0;
      velocityY = 0;
      grounded = true;
    }

    if (score === 150) {
      won = true;
      document.getElementById('message').textContent =
        '🏆 CHIẾN THẮNG! Bạn đã thu thập tất cả xu!';
    }
  }

  // Camera theo sau nhân vật
  const desiredCamera = player.position.clone().add(cameraOffset);
  camera.position.lerp(desiredCamera, 1 - Math.exp(-4 * dt));

  camera.lookAt(
    player.position.x,
    player.position.y + 1.2,
    player.position.z
  );

  renderer.render(scene, camera);
}

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

resetGame();
animate();
</script>
</body>
</html>
