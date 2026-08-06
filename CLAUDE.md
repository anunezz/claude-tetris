# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris clásico implementado en JavaScript vanilla puro, sin dependencias, sin build, sin `package.json`. Tres archivos: `index.html`, `style.css`, `game.js`.

## Running / testing

No hay build ni test suite. Para probar cambios, servir el directorio y abrir en navegador:

```bash
python3 -m http.server 8000   # o: npx serve .   /   php -S localhost:8000
```

Luego abrir `http://localhost:8000`. Verificar manualmente en navegador tras cambios (no hay tests automatizados).

## Architecture

Todo el estado y lógica del juego vive en `game.js` como funciones y variables globales de módulo (no hay clases, no hay framework):

- **Tablero**: matriz `board[ROWS][COLS]` (20×10) donde cada celda es `0` (vacía) o índice 1–7 que mapea a `COLORS`/`PIECES`.
- **Piezas**: `PIECES` define las 7 formas estándar como matrices cuadradas. Rotación vía `rotateCW` (transposición + reverso de filas), no vía tablas de rotación SRS.
- **Wall kicks** (`tryRotate`): intenta desplazamientos `[0, -1, 1, -2, 2]` en columnas tras rotar; si ninguno libera colisión, descarta el giro.
- **Colisión** (`collide(shape, ox, oy)`): única función que valida límites de tablero y solapamiento; toda lógica de movimiento/rotación/drop pasa por ella.
- **Game loop** (`loop`, vía `requestAnimationFrame`): acumula `dt` en `dropAccum`; al superar `dropInterval` baja la pieza o dispara `lockPiece()` (merge al tablero + `clearLines()` + `spawn()` de la siguiente).
- **Puntuación/nivel**: tabla `LINE_SCORES = [0,100,300,500,800]` × `level`; nivel sube cada 10 líneas (`Math.floor(lines/10)+1`); `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` proyecta la posición final hacia abajo; se dibuja con `globalAlpha=0.2` en `draw()`.
- **Render**: dos canvas independientes — `#board` (tablero + pieza + ghost, vía `draw()`) y `#next-canvas` (preview de siguiente pieza, vía `drawNext()`). Todo el dibujo pasa por `drawBlock()`.
- **Input**: un único listener `keydown` global despacha por `e.code` (flechas, `KeyX` rotar, `Space` hard drop, `KeyP` pausa). No hay debounce/repeat handling más allá del nativo del navegador.
- **Fin de juego/pausa**: overlay compartido (`#overlay`) reutilizado para GAME OVER y PAUSA, alternando texto vía `overlayTitle`/`overlayScore`.

Flujo: `init()` → crea tablero, genera `next`, llama `spawn()` (mueve `next`→`current`, genera nuevo `next`) → arranca `loop()`. Si `spawn()` detecta colisión inmediata, dispara `endGame()`.

## Customización rápida (constantes en `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval` inicial. Si se cambia `COLS`/`ROWS`/`BLOCK`, ajustar también `width`/`height` de `<canvas id="board">` en `index.html` (debe ser `COLS×BLOCK` por `ROWS×BLOCK`).
