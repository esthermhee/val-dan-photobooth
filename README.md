# val-dan-photobooth
A web-based photobooth for guests to take and download photo strips.
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Val & Dan’s Vow Renewal Photobooth</title>

<style>
  body {
    font-family: "Georgia", serif;
    background: #f6f2ec;
    text-align: center;
    padding: 20px;
  }

  h1 {
    margin-bottom: 5px;
  }

  #video {
    width: 300px;
    height: 400px;
    border-radius: 12px;
    box-shadow: 0 12px 25px rgba(0,0,0,0.2);
  }

  #countdown {
    font-size: 72px;
    font-weight: bold;
    position: absolute;
    left: 50%;
    top: 45%;
    transform: translate(-50%, -50%);
    color: white;
    text-shadow: 0 0 10px black;
    display: none;
  }

  #container {
    position: relative;
    display: inline-block;
  }

  button, a {
    padding: 12px 20px;
    font-size: 16px;
    border-radius: 10px;
    border: none;
    margin-top: 15px;
    cursor: pointer;
    text-decoration: none;
  }

  button {
    background: #2c2c2c;
    color: white;
  }

  a {
    background: #b99a5b;
    color: white;
    display: none;
  }

  canvas {
    display: none;
  }
</style>
</head>

<body>

<h1>Val & Dan’s Vow Renewal</h1>
<p>Tap start, pose, and download your photostrip 🤍</p>

<div id="container">
  <video id="video" autoplay playsinline></video>
  <div id="countdown">3</div>
</div>

<br>
<button id="start">Start Photobooth</button>
<br>
<a id="download">Download Photostrip</a>

<canvas id="photoCanvas"></canvas>
<canvas id="stripCanvas"></canvas>

<script>
const video = document.getElementById("video");
const photoCanvas = document.getElementById("photoCanvas");
const stripCanvas = document.getElementById("stripCanvas");
const startBtn = document.getElementById("start");
const countdown = document.getElementById("countdown");
const download = document.getElementById("download");

const photoCtx = photoCanvas.getContext("2d");
const stripCtx = stripCanvas.getContext("2d");

const photos = [];
const PHOTO_COUNT = 4;

// Camera access
navigator.mediaDevices.getUserMedia({ video: true })
  .then(stream => video.srcObject = stream)
  .catch(() => alert("Camera access required"));

async function takePhoto() {
  photoCanvas.width = 600;
  photoCanvas.height = 800;

  photoCtx.filter = "sepia(30%) contrast(1.1) brightness(1.05)";
  photoCtx.drawImage(video, 0, 0, photoCanvas.width, photoCanvas.height);

  photos.push(photoCanvas.toDataURL("image/png"));
}

async function countdownShot() {
  for (let i = 3; i > 0; i--) {
    countdown.innerText = i;
    countdown.style.display = "block";
    await new Promise(r => setTimeout(r, 1000));
  }
  countdown.style.display = "none";
  await takePhoto();
}

function createStrip() {
  stripCanvas.width = 600;
  stripCanvas.height = 1600;
  stripCtx.fillStyle = "#fff";
  stripCtx.fillRect(0, 0, stripCanvas.width, stripCanvas.height);

  photos.forEach((src, i) => {
    const img = new Image();
    img.src = src;
    img.onload = () => {
      stripCtx.drawImage(img, 50, i * 350 + 50, 500, 300);

      if (i === photos.length - 1) {
        stripCtx.fillStyle = "#000";
        stripCtx.textAlign = "center";
        stripCtx.font = "36px serif";
        stripCtx.fillText("Val & Dan", 300, 1450);
        stripCtx.font = "24px serif";
        stripCtx.fillText("Vow Renewal", 300, 1500);

        const finalImage = stripCanvas.toDataURL("image/png");
        download.href = finalImage;
        download.download = "Val-Dan-Vow-Renewal-Photostrip.png";
        download.style.display = "inline-block";
      }
    };
  });
}

startBtn.addEventListener("click", async () => {
  photos.length = 0;
  download.style.display = "none";

  for (let i = 0; i < PHOTO_COUNT; i++) {
    await countdownShot();
    await new Promise(r => setTimeout(r, 700));
  }

  createStrip();
});
</script>

</body>
</html>
