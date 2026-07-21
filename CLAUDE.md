# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es esto

Tetris clásico en JavaScript vanilla (HTML5 Canvas + CSS), sin dependencias, sin bundler ni transpilador, sin `package.json`. Todo el proyecto son 3 archivos: `index.html`, `style.css`, `game.js` (~300 líneas).

## Ejecutar / probar cambios

No hay build ni tests automatizados. Para probar cambios, sirve el directorio y abre en el navegador:

```bash
python3 -m http.server 8000   # o: npx serve .
```

Luego `http://localhost:8000`. También se puede abrir `index.html` directamente con `open index.html`.

## Arquitectura (`game.js`)

Todo el estado y la lógica viven en variables globales de módulo y funciones puras/casi-puras que operan sobre ellas — no hay clases, no hay módulos ES, no hay frameworks.

- **Tablero**: matriz `ROWS × COLS` (20×10) donde cada celda es `0` (vacía) o un índice 1–7 que mapea a `COLORS`/`PIECES` (identifica también el color de la pieza que la ocupó).
- **Piezas**: definidas como matrices cuadradas en `PIECES`. La rotación (`rotateCW`) es una transposición + inversión de filas, no rotaciones precomputadas por pieza.
- **Colisión** (`collide`): única función que valida límites del tablero y solapamiento; todo movimiento/rotación/spawn pasa por ella antes de aplicarse.
- **Wall kicks** (`tryRotate`): al rotar, prueba desplazamientos `[0, -1, 1, -2, 2]` columnas hasta encontrar uno sin colisión, si no descarta el giro.
- **Loop de juego** (`loop`): un único `requestAnimationFrame` acumula `dt` en `dropAccum` y baja la pieza cuando supera `dropInterval`; pausa/reanudación cancela/reinicia este frame con `cancelAnimationFrame`/`requestAnimationFrame` (no hay múltiples loops corriendo a la vez).
- **Fijado y limpieza** (`lockPiece` → `merge` + `clearLines`): al fijar, `clearLines` recorre de abajo hacia arriba, y al eliminar una fila re-evalúa el mismo índice (`r++` dentro del for que decrementa) porque las filas de arriba bajan una posición.
- **Puntuación/nivel**: tabla `LINE_SCORES` multiplicada por `level`; el nivel sube cada 10 líneas y `dropInterval` se recalcula como `max(100, 1000 - (level-1)*90)`.
- **Ghost piece** (`ghostY`): proyecta hacia abajo la posición final de la pieza actual con el mismo `collide`, se dibuja con `globalAlpha = 0.2`.
- **Renderizado**: `draw()` limpia y redibuja todo el canvas cada frame (grid + tablero fijado + ghost + pieza actual); no hay diffing ni dirty-rects. El canvas de "siguiente pieza" (`drawNext`) es independiente y solo se redibuja al hacer `spawn()`.

Si se cambian `COLS`, `ROWS` o `BLOCK` en `game.js`, hay que ajustar también `width`/`height` del `<canvas id="board">` en `index.html` para que coincidan (`COLS × BLOCK`, `ROWS × BLOCK`).
