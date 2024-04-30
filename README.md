<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kaçış Zamanı</title>
    <style>
/*Oyun alanı çerçevesi için Css stilleri */
        canvas {
            border: 1px solid black;
        }
/* Oyuncu karakterinin CSS stilleri */
        .player {
            width: 50px;
            height: 50px;
            background-color: red;
            clip-path: polygon(50% 0%, 61% 35%, 98% 35%, 68% 57%, 79% 91%, 50% 70%, 21% 91%, 32% 57%, 2% 35%, 39% 35%);
        }
/* kare karakterlerin CSS stilleri */
        .square {
            width: 30px;
            height: 30px;
            background-color: black;
        }
/* Yuvarlak karakterin belirleyen CSS */
        .circle {
            width: 40px;
            height: 40px;
            background-color: green;
            border-radius: 50%;
        }
/* Oyun mesaj kutusu */
        .message {
            position: absolute;
            padding: 10px;
            background-color: rgba(255, 255, 255, 0.8);
            border: 2px solid black;
            border-radius: 10px;
            display: none;
        }
/* Oyun talimatı butonunun CSS */
        .instruction-button {
            margin: 10px;
            padding: 5px 10px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
/* Oyun talimatı metninin CSS */
        .instruction-text {
            position: absolute;
            padding: 20px;
            background-color: rgba(255, 255, 255, 0.8);
            border: 2px solid black;
            border-radius: 10px;
            display: none;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    <button id="pauseButton">Duraklat</button>
    <button id="resumeButton">Devam Et</button>
    <button id="restartButton" style="display: none;">Yeniden Başlat</button>
    <div id="messageBox" class="message"></div>
    <button id="instructionButton" class="instruction-button">Oyun Talimatları</button>
    <div id="instructionText" class="instruction-text">
        <p><strong>Kaçış Zamanı</strong> adlı bu oyunun amacı, kırmızı yıldız karakterini kontrol ederek karelerden kaçmak ve yuvarlakları yemektir. Kareleri yediğinizde -10 puanınız azalırken, yuvarlakları yediğinizde +10 puanınız artar. Amacınız, puanınızı mümkün olduğunca yüksek tutmak ve mümkün olduğunca uzun süre hayatta kalmaktır.</p>
        <p><strong>Kontroller:</strong></p>
        <ul>
            <li>Yukarı Hareket: ↑ tuşu veya W tuşu</li>
            <li>Aşağı Hareket: ↓ tuşu veya S tuşu</li>
            <li>Sol Hareket: ← tuşu veya A tuşu</li>
            <li>Sağ Hareket: → tuşu veya D tuşu</li>
        </ul>
        <p>Oyunu duraklatmak veya devam ettirmek için "Duraklat" ve "Devam Et" butonlarını kullanabilirsiniz. Oyun bittiğinde "Yeniden Başlat" butonuyla oyunu yeniden başlatabilirsiniz.</p>
    </div>

    <script>
        // Oyun alanı
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        // Karakter özellikleri
        const player = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            width: 50,
            height: 50,
            speed: 5,
            score: 0
        };

        // Kare karakterler
        let squares = [];

        // Yuvarlak karakterler
        let circles = [];

        // Oyun durumu
        let paused = false;
        let gameOver = false;

        // Kontrol etme biçimleri
        const keys = {};

        document.addEventListener("keydown", function(event) {
            keys[event.key] = true;
        });

        document.addEventListener("keyup", function(event) {
            keys[event.key] = false;
        });

        // Kare karakter ekleme
        function addSquare() {
            squares.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                speedX: Math.random() * 5 - 2.5,
                speedY: Math.random() * 5 - 2.5
            });
        }

        // Yuvarlak karakter ekleme
        function addCircle() {
            circles.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                speedX: Math.random() * 5 - 2.5,
                speedY: Math.random() * 5 - 2.5
            });
        }

        // Oyun duraklatma
        document.getElementById("pauseButton").addEventListener("click", function() {
            paused = true;
        });

        // Oyun devam ettirme
        document.getElementById("resumeButton").addEventListener("click", function() {
            paused = false;
            update();
        });

        // Oyunu yeniden başlatma
        document.getElementById("restartButton").addEventListener("click", function() {
            restartGame();
        });

        // Oyunu başlatma veya yeniden başlatma
        function restartGame() {
            player.x = canvas.width / 2;
            player.y = canvas.height / 2;
            player.score = 0;
            squares = [];
            circles = [];
            gameOver = false;
            document.getElementById("restartButton").style.display = "none";
            update();
        }

        // Mesaj gösterme
        function showMessage(message, x, y) {
            const messageBox = document.getElementById("messageBox");
            messageBox.innerText = message;
            messageBox.style.left = x + "px";
            messageBox.style.top = y + "px";
            messageBox.style.display = "block";
            setTimeout(function() {
                messageBox.style.display = "none";
            }, 300);
        }

        // Gösterilen metni gizleme
        function hideInstructionText() {
            document.getElementById("instructionText").style.display = "none";
        }

        // Gösterilen metni gösterme
        function showInstructionText() {
            document.getElementById("instructionText").style.display = "block";
        }

        // Ana oyun döngüsü
        function update() {
            if (paused || gameOver) return;

            if (keys["ArrowUp"] || keys["w"]) {
                player.y -= player.speed;
            }
            if (keys["ArrowDown"] || keys["s"]) {
                player.y += player.speed;
            }
            if (keys["ArrowLeft"] || keys["a"]) {
                player.x -= player.speed;
            }
            if (keys["ArrowRight"] || keys["d"]) {
                player.x += player.speed;
            }

            // Kare karakterlerin hareketi
            squares.forEach(function(square) {
                square.x += square.speedX;
                square.y += square.speedY;

                if (square.x < 0 || square.x > canvas.width) {
                    square.speedX *= -1;
                }

                if (square.y < 0 || square.y > canvas.height) {
                    square.speedY *= -1;
                }
            });

            // Yuvarlak karakterlerin hareketi
            circles.forEach(function(circle) {
                circle.x += circle.speedX;
                circle.y += circle.speedY;

                if (circle.x < 0 || circle.x > canvas.width) {
                    circle.speedX *= -1;
                }

                if (circle.y < 0 || circle.y > canvas.height) {
                    circle.speedY *= -1;
                }
            });

            // Etkileşim kontrolü
            squares.forEach(function(square, index) {
                if (player.x < square.x + 30 &&
                    player.x + player.width > square.x &&
                    player.y < square.y + 30 &&
                    player.y + player.height > square.y) {
                    player.score -= 10;
                    squares.splice(index, 1);
                    showMessage("-10 Puan!", square.x, square.y);
                }
            });

            circles.forEach(function(circle, index) {
                if (player.x < circle.x + 40 &&
                    player.x + player.width > circle.x &&
                    player.y < circle.y + 40 &&
                    player.y + player.height > circle.y) {
                    player.score += 10;
                    circles.splice(index, 1);
                    showMessage("+10 Puan!", circle.x, circle.y);
                }
            });

            // Oyun durumu kontrolü
            if (player.score < 0) {
                gameOver = true;
                document.getElementById("restartButton").style.display = "block";
                showMessage("Kaybettiniz!", canvas.width / 2, canvas.height / 2);
            }

            // Oyun sahnesi
            draw();
            
            requestAnimationFrame(update);
        }

        // Oyun sahnesini çizme
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Kare karakterleri çizme
            ctx.fillStyle = "black";
            squares.forEach(function(square) {
                ctx.fillRect(square.x, square.y, 30, 30);
            });

            // Yuvarlak karakterleri çizme
            ctx.fillStyle = "green";
            circles.forEach(function(circle) {
                ctx.beginPath();
                ctx.arc(circle.x, circle.y, 20, 0, Math.PI * 2);
                ctx.fill();
            });

            // Oyuncuyu çizme
            ctx.fillStyle = "red";
            ctx.beginPath();
            ctx.moveTo(player.x + player.width / 2, player.y);
            ctx.lineTo(player.x + player.width * 0.61, player.y + player.height * 0.35);
            ctx.lineTo(player.x + player.width * 0.98, player.y + player.height * 0.35);
            ctx.lineTo(player.x + player.width * 0.68, player.y + player.height * 0.57);
            ctx.lineTo(player.x + player.width * 0.79, player.y + player.height * 0.91);
            ctx.lineTo(player.x + player.width / 2, player.y + player.height * 0.7);
            ctx.lineTo(player.x + player.width * 0.21, player.y + player.height * 0.91);
            ctx.lineTo(player.x + player.width * 0.32, player.y + player.height * 0.57);
            ctx.lineTo(player.x + player.width * 0.02, player.y + player.height * 0.35);
            ctx.lineTo(player.x + player.width * 0.39, player.y + player.height * 0.35);
            ctx.closePath();
            ctx.fill();

            // Puanı çizme
            ctx.fillStyle = "black";
            ctx.font = "24px Arial";
            ctx.fillText("Puan: " + player.score, 10, 30);
        }

        // Oyun başlatma
        setInterval(addSquare, 1000);
        setInterval(addCircle, 1500);
        update();

        // Oyun talimatları butonunu gösterme ve gizleme
        document.getElementById("instructionButton").addEventListener("click", function() {
            const instructionText = document.getElementById("instructionText");
            if (instructionText.style.display === "none") {
                showInstructionText();
            } else {
                hideInstructionText();
            }
        });
    </script>
</body>
</html>
