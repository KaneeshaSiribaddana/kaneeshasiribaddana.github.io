<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Upgraded Car Game</title>
    <!-- Bootstrap CSS Link -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #87CEEB; /* Sky Blue */
            transition: background-color 0.5s ease-in-out;
        }

        #game-container {
            position: relative;
            width: 80vw;
            height: 100vh;
            overflow: hidden;
        }

        .track {
            position: absolute;
            width: 20%;
            height: 100%;
            background-color: #2E2E2E; /* Road color */
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .lane {
            width: 2px;
            height: 100%;
            background-color: white;
            margin-top: 10%;
            border-radius: 1px;
            position: relative;
        }

        #car {
            position: absolute;
            bottom: 10%;
            left: 40%;
            transform: translateX(-50%);
            width: 20%;
            height: 80px;
            background-color: red;
            transition: left 0.5s ease-in-out;
            z-index: 2;
        }

        .obstacle {
            position: absolute;
            width: 20%;
            height: 30px;
            background-color: #00FF00; /* Green */
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div class="track" style="left: 0;">
            <div class="lane"></div>
            <div class="lane"></div>
            <div class="lane"></div>
        </div>
        <div class="track" style="left: 20%;">
            <div class="lane"></div>
            <div class="lane"></div>
            <div class="lane"></div>
        </div>
        <div class="track" style="left: 40%;">
            <div class="lane"></div>
            <div class="lane"></div>
            <div class="lane"></div>
        </div>
        <div class="track" style="left: 60%;">
            <div class="lane"></div>
            <div class="lane"></div>
            <div class="lane"></div>
        </div>

        <div id="car"></div>
    </div>

    <!-- Bootstrap JS and Popper.js (required for Bootstrap) -->
    <script src="https://cdn.jsdelivr.net/npm/popper.js@2.10.2/dist/umd/popper.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.min.js"></script>

    <script>
        const car = document.getElementById('car');
        const tracks = document.querySelectorAll('.track');
        let carPosition = 2; // initial position of the car (percentage)
        let trackWidth = 20; // track width in percentage
        let speed = 0;
        let isRaceOver = false;
        let objectCount = 0;
        let maxObstacles = 5;

        document.addEventListener('keydown', function(event) {
            if (!isRaceOver) {
                // Accelerate the car
                if (event.key === 'ArrowUp' && speed < 10) {
                    speed += 2;
                }

                // Decelerate the car
                if (event.key === 'ArrowDown' && speed > 0) {
                    speed -= 2;
                }

                // Move the car left
                if (event.key === 'ArrowLeft' && carPosition > 0) {
                    carPosition -= 1;
                }

                // Move the car right
                if (event.key === 'ArrowRight' && carPosition < 3) {
                    carPosition += 1;
                }

                updateCarPosition();
                updateBackgroundColor();
            }
        });

        // Add touch event handling
        let touchStartX = 0;

        document.addEventListener('touchstart', function(event) {
            touchStartX = event.touches[0].clientX;
        });

        document.addEventListener('touchmove', function(event) {
            const touchX = event.touches[0].clientX;
            const deltaX = touchX - touchStartX;

            // Adjust car position based on touch movement
            if (!isRaceOver) {
                carPosition += deltaX / window.innerWidth * 3; // Adjust sensitivity
                if (carPosition < 0) carPosition = 0;
                if (carPosition > 3) carPosition = 3;

                touchStartX = touchX;
                updateCarPosition();
                updateBackgroundColor();
            }
        });

        function updateCarPosition() {
            car.style.left = carPosition * trackWidth + '%';
            car.style.transition = 'left ' + (0.5 - speed / 20) + 's ease-in-out'; // Adjust transition based on speed
        }

        function updateBackgroundColor() {
            // Change background color based on speed
            const newColor = `rgb(135, ${(255 - speed * 10)}, 235)`;
            document.body.style.backgroundColor = newColor;
        }

        function moveCar() {
            // Move the car based on speed
            carPosition += speed / 2;
            if (carPosition < 0) carPosition = 0;
            if (carPosition > 3) carPosition = 3;

            updateCarPosition();
            updateBackgroundColor();
            checkObstacles();
            checkWin();
            if (!isRaceOver) {
                requestAnimationFrame(moveCar);
            }
        }

        function checkObstacles() {
            // Create obstacles randomly if the count is less than the maximum
            if (Math.random() < 0.02 && document.querySelectorAll('.obstacle').length < maxObstacles) {
                createObstacle();
            }

            // Move and check collision with obstacles
            const obstacles = document.querySelectorAll('.obstacle');
            obstacles.forEach((obstacle) => {
                moveObstacle(obstacle);
                if (checkCollision(obstacle)) {
                    endGame();
                }
            });
        }

        function createObstacle() {
            const trackIndex = Math.floor(Math.random() * tracks.length);
            const track = tracks[trackIndex];
            const obstacle = document.createElement('div');
            obstacle.className = 'obstacle';
            obstacle.style.left = Math.random() * (track.offsetWidth - 30) + 'px'; // Random position within the track
            obstacle.style.top = '0';
            track.appendChild(obstacle);
        }

        function moveObstacle(obstacle) {
            const obstacleSpeed = 5;
            const obstacleTop = parseFloat(obstacle.style.top) || 0;
            obstacle.style.top = obstacleTop + obstacleSpeed + 'px';
            if (obstacleTop > window.innerHeight) {
                // Remove obstacle when it goes off the screen
                obstacle.remove();
                objectCount++;
            }
        }

        function checkCollision(obstacle) {
            // Check if the car has collided with the obstacle
            const carRect = car.getBoundingClientRect();
            const obstacleRect = obstacle.getBoundingClientRect();

            return (
                carRect.bottom >= obstacleRect.top &&
                carRect.top <= obstacleRect.bottom &&
                carRect.left <= obstacleRect.right &&
                carRect.right >= obstacleRect.left
            );
        }

        function endGame() {
            isRaceOver = true;
            showModal(`Game Over! You hit an obstacle. Objects Passed: ${objectCount}`);
        }

        function checkWin() {
            if (objectCount >= 10) {
                isRaceOver = true;
                showModal('Congratulations! You won the game by passing 10 objects.');
            }
        }

        // Show Bootstrap Modal
        function showModal(message) {
            const modalContent = document.createElement('div');
            modalContent.innerHTML = `
                <div class="modal fade" id="gameModal" tabindex="-1" aria-labelledby="gameModalLabel" aria-hidden="true">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title" id="gameModalLabel">Game Result</h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </div>
                            <div class="modal-body">
                                <p>${message}</p>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                            </div>
                        </div>
                    </div>
                </div>
            `;
            document.body.appendChild(modalContent);

            // Activate the modal
            const gameModal = new bootstrap.Modal(document.getElementById('gameModal'), {
                backdrop: 'static', // Disable closing by clicking outside the modal
                keyboard: false     // Disable closing with the keyboard
            });
            gameModal.show();
        }

        // Start the game loop
        requestAnimationFrame(moveCar);
    </script>
</body>
</html>
