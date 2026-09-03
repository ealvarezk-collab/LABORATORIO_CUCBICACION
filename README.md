<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Juego de Cubicación – Primera Evaluación</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <div id="app">
        <!-- Pantalla de carga -->
        <div id="loading-screen" class="screen active">
            <div class="loading-spinner"></div>
            <p>Cargando motor...</p>
        </div>

        <!-- Modal de nombre (heredado de tu maqueta) -->
        <div id="name-modal" class="modal-overlay" style="display:flex;">
            <div class="modal-card">
                <h2>🏗️ Juego de Cubicación</h2>
                <p style="color:var(--text-sub); margin-top:8px; font-size:14px;">Preparación Primera Evaluación<br>Ingresa tu nombre para comenzar.</p>
                <input type="text" id="student-name" placeholder="Nombre completo..." maxlength="50">
                <button id="start-game-btn" class="btn primary" style="width:100%;">Comenzar</button>
            </div>
        </div>

        <!-- Menú principal -->
        <div id="main-menu" class="screen hidden">
            <header class="game-header">
                <h1>📐 Cubicación – Niveles</h1>
                <div class="player-info">
                    <span id="player-name-display"></span>
                    <span id="total-points-display">0 pts</span>
                    <span id="total-stars-display">★ 0</span>
                    <span id="total-time-display">00:00</span>
                </div>
            </header>
            <div class="level-grid" id="level-grid">
                <!-- Se genera dinámicamente -->
            </div>
            <button id="final-boss-btn" class="btn boss-btn">Final Boss (opcional)</button>
        </div>

        <!-- Pantalla de nivel -->
        <div id="level-screen" class="screen hidden">
            <header class="game-header">
                <button id="back-to-menu" class="btn small secondary">← Menú</button>
                <h2 id="level-title"></h2>
                <div class="level-stats">
                    <span id="level-points"></span>
                    <span id="level-stars"></span>
                </div>
            </header>
            <div id="challenge-list" class="challenge-grid"></div>
        </div>

        <!-- Pantalla de desafío -->
        <div id="challenge-screen" class="screen hidden">
            <header class="game-header challenge-header">
                <button id="back-to-level" class="btn small secondary">← Nivel</button>
                <div>
                    <h2 id="challenge-title"></h2>
                    <p id="challenge-description"></p>
                </div>
                <div class="challenge-status">
                    <span id="challenge-points">0 pts</span>
                    <span id="challenge-attempts">Intentos: 0</span>
                </div>
            </header>
            <div id="challenge-content" class="challenge-content"></div>
            <div class="pista-area">
                <button id="show-hint-btn" class="btn secondary">💡 Pista</button>
                <div id="hint-text" class="hint-text hidden"></div>
            </div>
            <div id="feedback-area" class="feedback hidden"></div>
        </div>

        <!-- Modal de resultado -->
        <div id="result-modal" class="modal-overlay" style="display:none;">
            <div class="modal-card result-content">
                <h2 id="result-title"></h2>
                <p id="result-message"></p>
                <div id="result-stars" class="stars-display"></div>
                <p id="result-points"></p>
                <button id="continue-btn" class="btn primary">Continuar</button>
            </div>
        </div>

        <!-- Final Boss placeholder -->
        <div id="boss-screen" class="screen hidden">
            <header class="game-header">
                <button id="back-to-menu-boss" class="btn small secondary">← Menú</button>
                <h2>JEFE DE CUBICACIÓN</h2>
            </header>
            <div class="boss-placeholder">
                <p>Misión integradora opcional (en desarrollo)</p>
                <p>Combina itemizado, lectura de plano, conversión, fórmula y cubicación.</p>
            </div>
        </div>
    </div>

    <!-- Scripts -->
    <script src="data/exercises.js"></script>
    <script src="data/plans.js"></script>
    <script src="js/storage.js"></script>
    <script src="js/scoring.js"></script>
    <script src="js/hints.js"></script>
    <script src="js/evaluator.js"></script>
    <script src="js/levels.js"></script>
    <script src="js/exercises.js"></script>
    <script src="js/game.js"></script>
</body>
</html>
