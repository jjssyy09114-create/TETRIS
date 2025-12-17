<!DOCTYPE html>
<html>
<head>
    <title>Simple Tetris</title>
    <style>
        body { background: #202028; color: #fff; font-family: sans-serif; display: flex; flex-direction: column; align-items: center; }
        canvas { border: solid 2px #fff; background-color: #000; }
    </style>
</head>
<body>
    <h1>Simple Tetris</h1>
    <div id="score">Score: 0</div>
    <canvas id="tetris" width="240" height="400"></canvas>

    <script>
        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');

        context.scale(20, 20); // 블록 크기 확대

        // 1. 블록 모양 정의
        function createPiece(type) {
            if (type === 'T') return [[0, 1, 0], [1, 1, 1], [0, 0, 0]];
            if (type === 'O') return [[2, 2], [2, 2]];
            if (type === 'L') return [[0, 3, 0], [0, 3, 0], [0, 3, 3]];
            if (type === 'J') return [[0, 4, 0], [0, 4, 0], [4, 4, 0]];
            if (type === 'I') return [[0, 5, 0, 0], [0, 5, 0, 0], [0, 5, 0, 0], [0, 5, 0, 0]];
            if (type === 'S') return [[0, 6, 6], [6, 6, 0], [0, 0, 0]];
            if (type === 'Z') return [[7, 7, 0], [0, 7, 7], [0, 0, 0]];
        }

        // 2. 충돌 감지
        function collide(arena, player) {
            const [m, o] = [player.matrix, player.pos];
            for (let y = 0; y < m.length; ++y) {
                for (let x = 0; x < m[y].length; ++x) {
                    if (m[y][x] !== 0 && (arena[y + o.y] && arena[y + o.y][x + o.x]) !== 0) return true;
                }
            }
            return false;
        }

        // 3. 보드(Arena) 생성
        function createMatrix(w, h) {
            const matrix = [];
            while (h--) matrix.push(new Array(w).fill(0));
            return matrix;
        }

        // 4. 그리기 함수
        function draw() {
            context.fillStyle = '#000';
            context.fillRect(0, 0, canvas.width, canvas.height);
            drawMatrix(arena, {x: 0, y: 0});
            drawMatrix(player.matrix, player.pos);
        }

        function drawMatrix(matrix, offset) {
            const colors = [null, '#FF0D72', '#0DC2FF', '#0DFF72', '#F538FF', '#FF8E0D', '#FFE138', '#3877FF'];
            matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) {
                        context.fillStyle = colors[value];
                        context.fillRect(x + offset.x, y + offset.y, 1, 1);
                    }
                });
            });
        }

        // 5. 블록 합치기 및 줄 삭제
        function arenaSweep() {
            let rowCount = 1;
            outer: for (let y = arena.length - 1; y > 0; --y) {
                for (let x = 0; x < arena[y].length; ++x) {
                    if (arena[y][x] === 0) continue outer;
                }
                const row = arena.splice(y, 1)[0].fill(0);
                arena.unshift(row);
                ++y;
                player.score += rowCount * 10;
                rowCount *= 2;
            }
            updateScore();
        }

        function merge(arena, player) {
            player.matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) arena[y + player.pos.y][x + player.pos.x] = value;
                });
            });
        }

        // 6. 플레이어 동작
        function playerDrop() {
            player.pos.y++;
            if (collide(arena, player)) {
                player.pos.y--;
                merge(arena, player);
                playerReset();
                arenaSweep();
            }
            dropCounter = 0;
        }

        function playerMove(dir) {
            player.pos.x += dir;
            if (collide(arena, player)) player.pos.x -= dir;
        }

        function playerReset() {
            const pieces = 'ILJOTSZ';
            player.matrix = createPiece(pieces[pieces.length * Math.random() | 0]);
            player.pos.y = 0;
            player.pos.x = (arena[0].length / 2 | 0) - (player.matrix[0].length / 2 | 0);
            if (collide(arena, player)) {
                arena.forEach(row => row.fill(0));
                player.score = 0;
                updateScore();
            }
        }

        function playerRotate(dir) {
            const pos = player.pos.x;
            let offset = 1;
            rotate(player.matrix, dir);
            while (collide(arena, player)) {
                player.pos.x += offset;
                offset = -(offset + (offset > 0 ? 1 : -1));
                if (offset > player.matrix[0].length) {
                    rotate(player.matrix, -dir);
                    player.pos.x = pos;
                    return;
                }
            }
        }

        function rotate(matrix, dir) {
            for (let y = 0; y < matrix.length; ++y) {
                for (let x = 0; x < y; ++x) {
                    [matrix[x][y], matrix[y][x]] = [matrix[y][x], matrix[x][y]];
                }
            }
            if (dir > 0) matrix.forEach(row => row.reverse());
            else matrix.reverse();
        }

        function updateScore() {
            scoreElement.innerText = `Score: ${player.score}`;
        }

        let dropCounter = 0;
        let dropInterval = 1000;
        let lastTime = 0;

        function update(time = 0) {
            const deltaTime = time - lastTime;
            lastTime = time;
            dropCounter += deltaTime;
            if (dropCounter > dropInterval) playerDrop();
            draw();
            requestAnimationFrame(update);
        }

        const arena = createMatrix(12, 20);
        const player = { pos: {x: 0, y: 0}, matrix: null, score: 0 };

        document.addEventListener('keydown', event => {
            if (event.keyCode === 37) playerMove(-1);
            else if (event.keyCode === 39) playerMove(1);
            else if (event.keyCode === 40) playerDrop();
            else if (event.keyCode === 81) playerRotate(-1); // Q
            else if (event.keyCode === 87) playerRotate(1);  // W
        });

        playerReset();
        updateScore();
        update();
    </script>
</body>
</html>
