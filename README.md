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
    transform: scaleX(-1); /* mirror preview for natural feel */
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

  #frames {
    display: flex;
    justify-content: center;
    margin-top: 12px;
    gap: 8px;
  }

  .frame {
    width: 18px;
    height: 18px;
    border-radius: 4px;
    border: 2px solid #b99a5b;
    background: transparent;
  }

  .frame.filled {
    background: #b99a5b;
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

<div id="frames">
  <div class="frame"></div>
  <div class="frame"></div>
  <div class="frame"></div>
  <div class="frame"></div>
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
const frameBoxes = document.querySelectorAll(".frame");

const photoCtx = photoCanvas.getContext("2d");
const stripCtx = stripCanvas.getContext("2d");

const photos = [];
const PHOTO_COUNT = 4;

// Load overlay frame (replace with your own frame PNG with transparent center)
const overlay = new Image();
overlay.src = "https://i.imgur.com/Oz6yAIl.png";

// Camera access
navigator.mediaDevices.getUserMedia({ video: true })
  .then(stream => video.srcObject = stream)
  .catch(() => alert("Camera access required"));

// Take photo function
async function takePhoto() {
  const photoWidth = 480;
  const photoHeight = 600;

  photoCanvas.width = photoWidth;
  photoCanvas.height = photoHeight;

  // Draw video flipped horizontally for correct final image
  photoCtx.save();
  photoCtx.scale(-1, 1);
  photoCtx.drawImage(video, -photoWidth, 0, photoWidth, photoHeight);
  photoCtx.restore();

  // Apply black & white filter
  photoCtx.filter = "grayscale(100%) contrast(1.2) brightness(1.05)";
  photoCtx.drawImage(photoCanvas, 0, 0);

  // Draw overlay frame
  photoCtx.drawImage(overlay, 0, 0, photoWidth, photoHeight);

  // Save photo
  photos.push(photoCanvas.toDataURL("image/png"));

  // Fill next progress frame
  frameBoxes[photos.length - 1].classList.add("filled");
}

// Countdown before each shot
async function countdownShot() {
  for (let i = 3; i > 0; i--) {
    countdown.innerText = i;
    countdown.style.display = "block";
    await new Promise(r => setTimeout(r, 1000));
  }
  countdown.style.display = "none";
  await takePhoto();
}

// Create final photostrip
function createStrip() {
  const photoWidth = 480;
  const photoHeight = 600;
  const gap = 20;
  const sidePadding = 20;

  stripCanvas.width = photoWidth + sidePadding * 2;
  stripCanvas.height = photoHeight * PHOTO_COUNT + gap * (PHOTO_COUNT - 1) + sidePadding * 2;

  stripCtx.fillStyle = "#fff";
  stripCtx.fillRect(0, 0, stripCanvas.width, stripCanvas.height);

  photos.forEach((src, i) => {
    const img = new Image();
    img.src = src;
    img.onload = () => {
      const x = sidePadding;
      const y = sidePadding + i * (photoHeight + gap);
      stripCtx.drawImage(img, x, y, photoWidth, photoHeight);

      // Draw overlay for each photo
      stripCtx.drawImage(overlay, x, y, photoWidth, photoHeight);

      // Add bottom text after last photo
      if (i === photos.length - 1) {
        stripCtx.fillStyle = "#000";
        stripCtx.textAlign = "center";
        stripCtx.font = "36px serif";
        stripCtx.fillText("Val & Dan", stripCanvas.width / 2, stripCanvas.height - 50);
        stripCtx.font = "24px serif";
        stripCtx.fillText("Vow Renewal", stripCanvas.width / 2, stripCanvas.height - 15);

        const finalImage = stripCanvas.toDataURL("image/png");
        download.href = finalImage;
        download.download = "Val-Dan-Vow-Renewal-Photostrip.png";
        download.style.display = "inline-block";
      }
    };
  });
}

// Start photobooth
startBtn.addEventListener("click", async () => {
  photos.length = 0;
  download.style.display = "none";
  frameBoxes.forEach(box => box.classList.remove("filled"));

  for (let i = 0; i < PHOTO_COUNT; i++) {
    await countdownShot();
    await new Promise(r => setTimeout(r, 700));
  }

  createStrip();
});
</script>

</body>
</html>

