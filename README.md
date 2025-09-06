<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width,initial-scale=1" />
        <title>Lattice — abstract strategy game</title>
        <style>
            :root {
                --bg: #0e0f12;
                --panel: #171a21;
                --line: #2b2f3a;
                --muted: #8b93a7;
                --accent: #6ae3ff;
                --green: #2ecc71;
                --red: #ff5b5b;
                --white: #f4f6ff;
                --shadow: 0 10px 24px rgba(0, 0, 0, 0.28);
            }
            html,
            body {
                height: 100%;
            }
            body {
                margin: 0;
                background: radial-gradient(
                    1200px 700px at 70% 10%,
                    #19202a 0%,
                    var(--bg) 55%
                );
                color: var(--white);
                font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI,
                    Roboto, 'Helvetica Neue', Arial, 'Noto Sans',
                    'Apple Color Emoji', 'Segoe UI Emoji';
                display: flex;
                gap: 22px;
                padding: 22px;
                box-sizing: border-box;
            }
            #left,
            #right {
                background: var(--panel);
                border: 1px solid #202532;
                border-radius: 16px;
                box-shadow: var(--shadow);
            }
            #left {
                flex: 1;
                min-width: 280px;
                display: flex;
                align-items: center;
                justify-content: center;
                position: relative;
                padding: 14px;
            }
            #right {
                width: 360px;
                padding: 18px;
                display: flex;
                flex-direction: column;
                gap: 14px;
            }
            header {
                display: flex;
                align-items: center;
                justify-content: space-between;
                gap: 10px;
            }
            .badge {
                font-size: 12px;
                letter-spacing: 0.3px;
                text-transform: uppercase;
                padding: 6px 10px;
                border-radius: 999px;
                border: 1px solid var(--line);
                color: var(--muted);
            }
            .row {
                display: flex;
                gap: 10px;
                align-items: center;
                flex-wrap: wrap;
            }
            .pill {
                background: #0f1218;
                border: 1px solid #222a36;
                padding: 10px 12px;
                border-radius: 12px;
                display: flex;
                align-items: center;
                gap: 8px;
            }
            .turn {
                font-weight: 700;
                letter-spacing: 0.2px;
            }
            button {
                background: #0f1218;
                border: 1px solid #222a36;
                color: var(--white);
                padding: 10px 12px;
                border-radius: 12px;
                cursor: pointer;
                transition: 0.15s transform, 0.15s background, 0.15s opacity;
            }
            button:hover {
                transform: translateY(-1px);
                background: #121620;
            }
            button:disabled {
                opacity: 0.45;
                cursor: not-allowed;
                transform: none;
            }
            .danger {
                border-color: #3a2020;
                background: #1a1111;
            }
            .success {
                border-color: #1e3827;
                background: #102017;
            }
            .tiny {
                font-size: 12px;
                padding: 6px 8px;
            }
            .label {
                font-size: 12px;
                color: var(--muted);
            }
            .token {
                padding: 6px 10px;
                border-radius: 999px;
                border: 1px dashed #2e3645;
                font-size: 12px;
                color: #cbd6ff;
                background: #0e1320;
            }
            details {
                border: 1px solid #263047;
                border-radius: 12px;
                padding: 10px 12px;
                background: #0f1524;
            }
            details > summary {
                cursor: pointer;
                font-weight: 600;
            }
            .legend {
                display: flex;
                gap: 10px;
                flex-wrap: wrap;
            }
            .dot {
                width: 14px;
                height: 14px;
                border-radius: 50%;
            }
            .dot.black {
                background: #0a0a0a;
                border: 1px solid #000;
            }
            .dot.white {
                background: #fff;
                border: 1px solid #000;
            }
            .swatch {
                width: 18px;
                height: 3px;
                border-radius: 2px;
            }
            .swatch.green {
                background: var(--green);
            }
            .swatch.red {
                background: var(--red);
            }
            canvas {
                max-width: min(92vw, 1100px);
                width: 100%;
                height: auto;
                display: block;
            }
            .footer {
                margin-top: auto;
                font-size: 12px;
                color: var(--muted);
            }
            .kbd {
                font-family: ui-monospace, SFMono-Regular, Menlo, Monaco,
                    Consolas, 'Liberation Mono', monospace;
                font-size: 12px;
                padding: 2px 6px;
                border: 1px solid #2d3340;
                border-radius: 6px;
                background: #0d1117;
                color: #cfe3ff;
            }
            .gridTip {
                position: absolute;
                left: 14px;
                top: 12px;
                font-size: 12px;
                color: var(--muted);
            }
            .toast {
                position: fixed;
                bottom: 18px;
                left: 50%;
                transform: translateX(-50%);
                background: #0f1524;
                border: 1px solid #263047;
                padding: 10px 14px;
                border-radius: 10px;
                color: #dbe5ff;
                box-shadow: var(--shadow);
                opacity: 0;
                pointer-events: none;
                transition: 0.35s;
            }
            .toast.show {
                opacity: 1;
            }
        </style>
    </head>
    <body>
        <div id="left">
            <div class="gridTip">
                Tip: click a hex to place; pick an edge (two adjacent stones) to
                <span class="kbd">Reinforce</span> or
                <span class="kbd">Sever</span>.
            </div>
            <canvas
                id="board"
                width="1200"
                height="1100"
                aria-label="Lattice board"
            ></canvas>
            <div id="toast" class="toast"></div>
        </div>

        <div id="right">
            <header>
                <div class="row">
                    <div class="badge">Lattice</div>
                    <div class="label">7×7 board • two players</div>
                </div>
                <button id="newGame" class="tiny">New game</button>
            </header>

            <div class="pill" role="status" aria-live="polite">
                <div
                    class="dot black"
                    style="box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.06)"
                ></div>
                <div class="turn" id="turnLabel">Black to move</div>
            </div>

            <div class="row">
                <button id="placeBtn" class="success">Place marker</button>
                <button id="reinforceBtn">
                    Use Reinforce
                    <span id="reinLeft" class="token">B:3 • W:3</span>
                </button>
                <button id="severBtn" class="danger">
                    Use Sever <span id="sevLeft" class="token">B:3 • W:3</span>
                </button>
            </div>

            <div class="row">
                <button id="undoBtn" class="tiny">Undo</button>
                <button id="passBtn" class="tiny">Pass</button>
            </div>

            <div>
                <div class="label">Score (stable control)</div>
                <div class="row">
                    <div class="pill">
                        <div class="dot black"></div>
                        <span id="scoreB">0</span>
                    </div>
                    <div class="pill">
                        <div class="dot white"></div>
                        <span id="scoreW">0</span>
                    </div>
                    <div class="pill">
                        <span class="label">Empty controlled:</span>
                        <span id="controlledInfo">—</span>
                    </div>
                </div>
            </div>

            <details open>
                <summary>Legend & rules</summary>
                <div class="legend" style="margin-top: 8px">
                    <div class="pill">
                        <div class="dot black"></div>
                        Black stone
                    </div>
                    <div class="pill">
                        <div class="dot white"></div>
                        White stone
                    </div>
                    <div class="pill">
                        <div class="swatch green"></div>
                        Reinforced edge
                    </div>
                    <div class="pill">
                        <div class="swatch red"></div>
                        Severed edge
                    </div>
                </div>
                <p style="color: var(--muted); line-height: 1.5">
                    Stones that touch edge-to-edge form a <b>cluster</b>. You
                    may either place a stone, reinforce one of your cluster
                    edges, or sever one edge in your opponent’s cluster (each
                    player has 3 Reinforce and 3 Sever tokens). A cluster is
                    <b>stable</b> if it contains a cycle (i.e., its number of
                    internal edges ≥ number of stones). Stable clusters
                    permanently control their adjacent empty hexes (unless
                    contested by both colors). Highest stable control wins when
                    both players pass or the board is full.
                </p>
            </details>

            <div class="footer">
                Keyboard: <span class="kbd">1</span> Place •
                <span class="kbd">2</span> Reinforce •
                <span class="kbd">3</span> Sever •
                <span class="kbd">Z</span> Undo •
                <span class="kbd">P</span> Pass
            </div>
        </div>

        <script>
            /* --------------------------- Geometry & drawing --------------------------- */
            const canvas = document.getElementById('board');
            const ctx = canvas.getContext('2d');
            const W = canvas.width,
                H = canvas.height;

            // Hex layout (odd-q offset)
            const COLS = 7,
                ROWS = 7;
            const R = 46; // hex radius
            const HEX_W = R * 2;
            const HEX_H = Math.sqrt(3) * R;
            const XOFF = 120,
                YOFF = 110; // canvas margin

            function hexToPixel(col, row) {
                const x = XOFF + (3 / 2) * R * col;
                const y = YOFF + HEX_H * (row + 0.5 * (col & 1));
                return [x, y];
            }

            function polygon(x, y, r = R) {
                const pts = [];
                for (let i = 0; i < 6; i++) {
                    const a = (Math.PI / 180) * (60 * i - 30);
                    pts.push([x + r * Math.cos(a), y + r * Math.sin(a)]);
                }
                return pts;
            }

            function drawHex(col, row, fill, stroke = '#2b2f3a') {
                const [x, y] = hexToPixel(col, row),
                    pts = polygon(x, y, R - 2);
                ctx.beginPath();
                pts.forEach(([px, py], i) =>
                    i ? ctx.lineTo(px, py) : ctx.moveTo(px, py)
                );
                ctx.closePath();
                ctx.fillStyle = fill;
                ctx.strokeStyle = stroke;
                ctx.lineWidth = 1.2;
                ctx.fill();
                ctx.stroke();
            }

            function drawStone(col, row, color) {
                const [x, y] = hexToPixel(col, row);
                ctx.beginPath();
                ctx.arc(x, y, R * 0.47, 0, Math.PI * 2);
                ctx.fillStyle = color === 'B' ? '#0a0a0a' : '#ffffff';
                ctx.fill();
                ctx.lineWidth = 2;
                ctx.strokeStyle = '#000000';
                if (color === 'W') ctx.stroke();
            }

            function drawEdge(a, b, opts = {}) {
                const [x1, y1] = hexToPixel(a.col, a.row);
                const [x2, y2] = hexToPixel(b.col, b.row);
                ctx.beginPath();
                ctx.moveTo(x1, y1);
                ctx.lineTo(x2, y2);
                ctx.lineWidth = opts.width || 5;
                ctx.strokeStyle = opts.color || '#888';
                if (opts.dash) {
                    ctx.setLineDash(opts.dash);
                } else {
                    ctx.setLineDash([]);
                }
                ctx.stroke();
                ctx.setLineDash([]);
            }

            /* ------------------------------ Game state -------------------------------- */
            const emptyBoard = () => ({
                cells: Array.from({ length: COLS }, (_, c) =>
                    Array.from({ length: ROWS }, (_, r) => ({
                        col: c,
                        row: r,
                        stone: null,
                    }))
                ),
                // connections keyed by "c1,r1|c2,r2" with canonical order
                edges: {}, // value: {owner:'B'|'W', status:'normal'|'reinforced'|'severed'}
                turn: 'B',
                passes: 0,
                tokens: {
                    B: { reinforce: 3, sever: 3 },
                    W: { reinforce: 3, sever: 3 },
                },
                mode: 'place', // 'place' | 'reinforce' | 'sever'
            });

            let state = emptyBoard();
            const history = [];

            /* ------------------------------- Utilities -------------------------------- */
            function inBounds(c, r) {
                return c >= 0 && c < COLS && r >= 0 && r < ROWS;
            }
            const neighDirs = [
                [+1, 0],
                [0, +1],
                [-1, +1],
                [-1, 0],
                [0, -1],
                [+1, -1],
            ]; // axial-equivalent on odd-q

            function neighbors(col, row) {
                // odd-q offset
                const odd = col & 1;
                const deltas = [
                    [+1, 0],
                    [0, +1],
                    [-1, +1],
                    [-1, 0],
                    [0, -1],
                    [+1, -1],
                ];
                // convert to odd-q vertical adjustments
                const adj = [
                    [+1, 0],
                    [0, +1],
                    [-1, +1],
                    [-1, 0],
                    [0, -1],
                    [+1, -1],
                ];
                // For odd columns, +1/-1 rows are relative; our grid already matches the offsets above.
                const list = [];
                for (const [dc, dr] of adj) {
                    const nr =
                        row +
                        dr +
                        (dc === +1 || dc === -1
                            ? dc === +1
                                ? row & 1
                                : 0
                            : 0);
                    const c2 = col + dc,
                        r2 =
                            dc === +1 || dc === -1
                                ? row +
                                  (dc === +1
                                      ? col & 1
                                          ? 0
                                          : 0
                                      : col & 1
                                      ? 0
                                      : 0) +
                                  dr
                                : row + dr;
                    // Actually our chosen offsets already work for odd-q if we build by hand below:
                }
                // we'll implement directly based on col parity:
                const n = [];
                const dirs = [
                    [+1, 0],
                    [-1, 0],
                    [0, +1],
                    [0, -1],
                    [+1, col % 2 ? -1 : 0],
                    [-1, col % 2 ? -1 : 0],
                    [+1, col % 2 ? 0 : +1],
                    [-1, col % 2 ? 0 : +1],
                ];
                // remove duplicate pairs by hand:
                const uniq = [
                    [+1, 0],
                    [-1, 0],
                    [0, +1],
                    [0, -1],
                    [+1, col % 2 ? -1 : 0],
                    [-1, col % 2 ? -1 : 0],
                ];
                // Simpler: we know a clean odd-q neighbors set:
                const del =
                    col % 2
                        ? [
                              [+1, 0],
                              [+1, -1],
                              [0, -1],
                              [-1, -1],
                              [-1, 0],
                              [0, +1],
                          ]
                        : [
                              [+1, +1],
                              [+1, 0],
                              [0, -1],
                              [-1, 0],
                              [-1, +1],
                              [0, +1],
                          ];
                for (const [dc, dr] of del) {
                    const c = col + dc,
                        r = row + dr;
                    if (inBounds(c, r)) n.push({ col: c, row: r });
                }
                return n;
            }

            function edgeKey(a, b) {
                const s1 = `${a.col},${a.row}`,
                    s2 = `${b.col},${b.row}`;
                return s1 < s2 ? `${s1}|${s2}` : `${s2}|${s1}`;
            }

            function getCell(c, r) {
                return state.cells[c][r];
            }

            function edgeBetween(a, b) {
                const key = edgeKey(a, b);
                return state.edges[key] || null;
            }

            function setEdge(a, b, owner, status) {
                const key = edgeKey(a, b);
                state.edges[key] = { owner, status };
            }

            function removeEdge(a, b) {
                delete state.edges[edgeKey(a, b)];
            }

            /* ------------------------------ Game logic -------------------------------- */
            function snapshot() {
                return JSON.parse(JSON.stringify(state));
            }
            function pushHistory() {
                history.push(snapshot());
                if (history.length > 400) history.shift();
            }

            function switchTurn() {
                state.turn = state.turn === 'B' ? 'W' : 'B';
                updateUI();
            }

            function placeStone(col, row) {
                const cell = getCell(col, row);
                if (cell.stone) return toast('That hex is occupied.');
                pushHistory();
                cell.stone = state.turn;
                // create edges to same-col neighbors unless severed explicitly later
                const neigh = neighbors(col, row);
                for (const n of neigh) {
                    const c2 = getCell(n.col, n.row);
                    if (c2.stone === state.turn) {
                        const existing = edgeBetween(cell, c2);
                        if (!existing || existing.status === 'severed') {
                            // if severed, don't reconnect
                            setEdge(cell, c2, state.turn, 'normal');
                        }
                    }
                }
                state.passes = 0;
                switchTurn();
                recomputeScore();
                draw();
            }

            function canReinforceEdgesFor(player) {
                const pairs = [];
                for (const key in state.edges) {
                    const e = state.edges[key];
                    if (e.owner === player && e.status !== 'reinforced') {
                        const [a, b] = key.split('|').map((s) => {
                            const [c, r] = s.split(',').map(Number);
                            return getCell(c, r);
                        });
                        // must still both exist
                        if (a.stone === player && b.stone === player) {
                            // and adjacent
                            pairs.push([a, b]);
                        }
                    }
                }
                return pairs;
            }

            function canSeverEdgesFor(player) {
                const opponent = player === 'B' ? 'W' : 'B';
                const pairs = [];
                for (const key in state.edges) {
                    const e = state.edges[key];
                    if (
                        e.owner === opponent &&
                        e.status !== 'reinforced' &&
                        e.status !== 'severed'
                    ) {
                        const [a, b] = key.split('|').map((s) => {
                            const [c, r] = s.split(',').map(Number);
                            return getCell(c, r);
                        });
                        if (a.stone === opponent && b.stone === opponent) {
                            pairs.push([a, b]);
                        }
                    }
                }
                return pairs;
            }

            let edgeSelect = null; // when reinforcing/severing: store first-clicked stone

            function handleEdgeAction(targetCell) {
                const me = state.turn;
                const mode = state.mode;
                const opponent = me === 'B' ? 'W' : 'B';

                // pick a valid adjacency based on mode
                if (!edgeSelect) {
                    if (mode === 'reinforce' && targetCell.stone !== me) {
                        toast('Select one of your stones first.');
                        return;
                    }
                    if (mode === 'sever' && targetCell.stone !== opponent) {
                        toast('Select an opponent stone first.');
                        return;
                    }
                    edgeSelect = targetCell;
                    draw();
                    return;
                } else {
                    // second click must be adjacent stone of correct color
                    const adj = neighbors(edgeSelect.col, edgeSelect.row)
                        .map((n) => getCell(n.col, n.row))
                        .find(
                            (c) =>
                                c.col === targetCell.col &&
                                c.row === targetCell.row
                        );
                    if (!adj) {
                        toast('Not adjacent. Pick an adjacent stone.');
                        return;
                    }

                    if (mode === 'reinforce') {
                        if (targetCell.stone !== me) {
                            toast('Second stone must be yours.');
                            return;
                        }
                        const e = edgeBetween(edgeSelect, targetCell);
                        if (!e || e.owner !== me || e.status === 'reinforced') {
                            toast("That edge can't be reinforced.");
                            return;
                        }
                        if (state.tokens[me].reinforce <= 0) {
                            toast('No Reinforce tokens left.');
                            return;
                        }
                        pushHistory();
                        e.status = 'reinforced';
                        state.tokens[me].reinforce--;
                        state.passes = 0;
                        edgeSelect = null;
                        switchTurn();
                        recomputeScore();
                        draw();
                    } else if (mode === 'sever') {
                        if (targetCell.stone !== opponent) {
                            toast("Second stone must be opponent's.");
                            return;
                        }
                        const e = edgeBetween(edgeSelect, targetCell);
                        if (
                            !e ||
                            e.owner !== opponent ||
                            e.status !== 'normal'
                        ) {
                            toast("That edge can't be severed.");
                            return;
                        }
                        if (state.tokens[me].sever <= 0) {
                            toast('No Sever tokens left.');
                            return;
                        }
                        pushHistory();
                        e.status = 'severed';
                        state.tokens[me].sever--;
                        state.passes = 0;
                        edgeSelect = null;
                        switchTurn();
                        recomputeScore();
                        draw();
                    }
                }
            }

            function passTurn() {
                pushHistory();
                state.passes++;
                if (state.passes >= 2 || isBoardFull()) {
                    recomputeScore();
                    const winner =
                        scores.B > scores.W
                            ? 'Black wins!'
                            : scores.W > scores.B
                            ? 'White wins!'
                            : 'Draw!';
                    toast('Game over — ' + winner);
                } else {
                    switchTurn();
                }
            }

            function isBoardFull() {
                for (let c = 0; c < COLS; c++)
                    for (let r = 0; r < ROWS; r++)
                        if (!state.cells[c][r].stone) return false;
                return true;
            }

            /* ----------------------- Clusters, stability, control ---------------------- */
            function clusterGraph(owner) {
                // Build adjacency among stones of owner using edges that are owner + (normal|reinforced)
                const nodes = [];
                const idx = {};
                for (let c = 0; c < COLS; c++)
                    for (let r = 0; r < ROWS; r++) {
                        const cell = getCell(c, r);
                        if (cell.stone === owner) {
                            idx[`${c},${r}`] = nodes.length;
                            nodes.push(cell);
                        }
                    }
                const adj = Array.from(
                    { length: nodes.length },
                    () => new Set()
                );
                let edgeCount = 0;
                for (const key in state.edges) {
                    const e = state.edges[key];
                    if (e.owner !== owner) continue;
                    if (e.status === 'severed') continue;
                    const [s1, s2] = key.split('|');
                    if (!(s1 in idx) || !(s2 in idx)) continue;
                    const a = idx[s1],
                        b = idx[s2];
                    adj[a].add(b);
                    adj[b].add(a);
                    edgeCount++;
                }
                return { nodes, adj };
            }

            function components(graph) {
                const { nodes, adj } = graph;
                const seen = new Array(nodes.length).fill(false);
                const comps = [];
                for (let i = 0; i < nodes.length; i++) {
                    if (seen[i]) continue;
                    const stack = [i];
                    seen[i] = true;
                    const verts = [];
                    let edges = 0;
                    while (stack.length) {
                        const v = stack.pop();
                        verts.push(nodes[v]);
                        edges += adj[v].size;
                        for (const w of adj[v]) {
                            if (!seen[w]) {
                                seen[w] = true;
                                stack.push(w);
                            }
                        }
                    }
                    // each undirected edge counted twice
                    comps.push({ verts, edges: edges / 2 });
                }
                return comps;
            }

            function hasCycle(comp) {
                // Undirected connected graph contains a cycle iff edges >= vertices
                return comp.edges >= comp.verts.length && comp.verts.length > 0;
            }

            function pressuredBy(owner) {
                // Map of empty cells -> set('B'|'W') that pressure it
                const pressure = new Map();
                for (let c = 0; c < COLS; c++)
                    for (let r = 0; r < ROWS; r++) {
                        const cell = getCell(c, r);
                        if (cell.stone === owner) {
                            for (const n of neighbors(c, r)) {
                                const nb = getCell(n.col, n.row);
                                if (!nb.stone) {
                                    const k = `${n.col},${n.row}`;
                                    if (!pressure.has(k))
                                        pressure.set(k, new Set());
                                    pressure.get(k).add(owner);
                                }
                            }
                        }
                    }
                return pressure;
            }

            let scores = { B: 0, W: 0 };

            function recomputeScore() {
                const pB = pressuredBy('B'),
                    pW = pressuredBy('W');
                // Stable clusters of each color
                function stableControl(owner) {
                    const g = clusterGraph(owner);
                    const compsList = components(g);
                    const stableVerts = new Set();
                    compsList.forEach((comp) => {
                        if (hasCycle(comp))
                            comp.verts.forEach((v) =>
                                stableVerts.add(`${v.col},${v.row}`)
                            );
                    });
                    // An empty hex is permanently controlled by owner if it's adjacent to ANY stone in a stable component
                    const ctrl = new Set();
                    for (const key of owner === 'B' ? pB.keys() : pW.keys()) {
                        const [c, r] = key.split(',').map(Number);
                        const cell = getCell(c, r);
                        // check adjacency to at least one stable stone of owner
                        const near = neighbors(c, r).some((n) =>
                            stableVerts.has(`${n.col},${n.row}`)
                        );
                        if (near) ctrl.add(key);
                    }
                    return ctrl;
                }

                const ctrlB = stableControl('B');
                const ctrlW = stableControl('W');

                // contested empties are removed from both
                const finalB = new Set();
                const finalW = new Set();
                for (const k of ctrlB) {
                    if (!ctrlW.has(k)) finalB.add(k);
                }
                for (const k of ctrlW) {
                    if (!ctrlB.has(k)) finalW.add(k);
                }

                scores = { B: finalB.size, W: finalW.size };
                document.getElementById('scoreB').textContent = scores.B;
                document.getElementById('scoreW').textContent = scores.W;
                document.getElementById('controlledInfo').textContent = `B:${
                    finalB.size
                } • W:${finalW.size} ${
                    isBoardFull() || state.passes >= 2 ? '• (final)' : ''
                }`;

                // also store for rendering
                state.control = { B: finalB, W: finalW };
            }

            /* --------------------------------- Render --------------------------------- */
            function draw() {
                ctx.clearRect(0, 0, W, H);

                // subtle board glow
                ctx.save();
                ctx.shadowColor = 'rgba(0,0,0,0.45)';
                ctx.shadowBlur = 12;

                // grid
                for (let c = 0; c < COLS; c++) {
                    for (let r = 0; r < ROWS; r++) {
                        drawHex(
                            c,
                            r,
                            (c + r) % 2 ? '#141821' : '#10151e',
                            '#283041'
                        );
                    }
                }
                ctx.restore();

                // draw controlled empties (underlay)
                function drawControl(set, color) {
                    ctx.save();
                    ctx.globalAlpha = 0.2;
                    for (const k of set) {
                        const [c, r] = k.split(',').map(Number);
                        drawHex(c, r, color, 'transparent');
                    }
                    ctx.restore();
                }
                if (state.control) {
                    drawControl(state.control.B, '#2aeeff');
                    drawControl(state.control.W, '#ffd76a');
                }

                // edges
                for (const key in state.edges) {
                    const e = state.edges[key];
                    const [s1, s2] = key.split('|').map((s) => {
                        const [c, r] = s.split(',').map(Number);
                        return getCell(c, r);
                    });
                    if (!s1.stone || !s2.stone) continue;
                    if (e.status === 'severed')
                        drawEdge(s1, s2, {
                            color: '#ff5b5b',
                            width: 4,
                            dash: [8, 8],
                        });
                    else if (e.status === 'reinforced')
                        drawEdge(s1, s2, { color: '#2ecc71', width: 6 });
                    else
                        drawEdge(s1, s2, {
                            color: 'rgba(200,200,200,.35)',
                            width: 4,
                        });
                }

                // stones
                for (let c = 0; c < COLS; c++)
                    for (let r = 0; r < ROWS; r++) {
                        const cell = getCell(c, r);
                        if (cell.stone) drawStone(c, r, cell.stone);
                    }

                // edge selection hints
                if (edgeSelect) {
                    ctx.save();
                    const [x, y] = hexToPixel(edgeSelect.col, edgeSelect.row);
                    ctx.beginPath();
                    ctx.arc(x, y, R * 0.56, 0, Math.PI * 2);
                    ctx.lineWidth = 3;
                    ctx.strokeStyle =
                        state.mode === 'reinforce' ? '#2ecc71' : '#ff5b5b';
                    ctx.setLineDash([6, 6]);
                    ctx.stroke();
                    ctx.restore();
                }
            }

            /* ------------------------------ Event wiring ------------------------------ */
            function canvasToCell(x, y) {
                // try all cells and check point-in-hex via winding
                for (let c = 0; c < COLS; c++)
                    for (let r = 0; r < ROWS; r++) {
                        const [cx, cy] = hexToPixel(c, r);
                        // quick circle hit
                        if ((x - cx) ** 2 + (y - cy) ** 2 > (R * 1.05) ** 2)
                            continue;
                        // polygon hit test
                        const pts = polygon(cx, cy, R - 2);
                        let inside = false;
                        for (
                            let i = 0, j = pts.length - 1;
                            i < pts.length;
                            j = i++
                        ) {
                            const xi = pts[i][0],
                                yi = pts[i][1],
                                xj = pts[j][0],
                                yj = pts[j][1];
                            const intersect =
                                yi > y !== yj > y &&
                                x < ((xj - xi) * (y - yi)) / (yj - yi) + xi;
                            if (intersect) inside = !inside;
                        }
                        if (inside) return getCell(c, r);
                    }
                return null;
            }

            canvas.addEventListener('click', (e) => {
                const rect = canvas.getBoundingClientRect();
                const x = (e.clientX - rect.left) * (canvas.width / rect.width);
                const y =
                    (e.clientY - rect.top) * (canvas.height / rect.height);
                const cell = canvasToCell(x, y);
                if (!cell) return;

                if (state.mode === 'place') {
                    placeStone(cell.col, cell.row);
                } else {
                    handleEdgeAction(cell);
                }
            });

            window.addEventListener('keydown', (e) => {
                if (e.key === '1') {
                    setMode('place');
                }
                if (e.key === '2') {
                    setMode('reinforce');
                }
                if (e.key === '3') {
                    setMode('sever');
                }
                if (e.key.toLowerCase() === 'z') {
                    undo();
                }
                if (e.key.toLowerCase() === 'p') {
                    passTurn();
                }
            });

            /* --------------------------------- UI bits -------------------------------- */
            function updateUI() {
                document.getElementById('turnLabel').textContent =
                    (state.turn === 'B' ? 'Black' : 'White') + ' to move';
                document.getElementById(
                    'reinLeft'
                ).textContent = `B:${state.tokens.B.reinforce} • W:${state.tokens.W.reinforce}`;
                document.getElementById(
                    'sevLeft'
                ).textContent = `B:${state.tokens.B.sever} • W:${state.tokens.W.sever}`;
                document
                    .getElementById('placeBtn')
                    .classList.toggle('success', state.mode === 'place');
                document
                    .getElementById('reinforceBtn')
                    .classList.toggle('success', state.mode === 'reinforce');
                document
                    .getElementById('severBtn')
                    .classList.toggle('danger', state.mode === 'sever');
            }

            function setMode(m) {
                state.mode = m;
                edgeSelect = null;
                updateUI();
                draw();
            }

            function undo() {
                if (!history.length) {
                    toast('Nothing to undo.');
                    return;
                }
                state = history.pop();
                updateUI();
                recomputeScore();
                draw();
            }

            function newGame() {
                state = emptyBoard();
                history.length = 0;
                edgeSelect = null;
                recomputeScore();
                updateUI();
                draw();
                toast('New game. Black to move.');
            }

            function toast(msg) {
                const el = document.getElementById('toast');
                el.textContent = msg;
                el.classList.add('show');
                clearTimeout(toast._t);
                toast._t = setTimeout(() => el.classList.remove('show'), 1800);
            }

            document.getElementById('placeBtn').onclick = () =>
                setMode('place');
            document.getElementById('reinforceBtn').onclick = () =>
                setMode('reinforce');
            document.getElementById('severBtn').onclick = () =>
                setMode('sever');
            document.getElementById('undoBtn').onclick = undo;
            document.getElementById('passBtn').onclick = passTurn;
            document.getElementById('newGame').onclick = newGame;

            /* --------------------------------- Start! --------------------------------- */
            newGame();
        </script>
    </body>
</html>
