<script setup>
import EngineArrowExample from "./EngineArrowExample.vue";
</script>

# Displaying Engine Moves

::: warning
This section is WIP, some examples may not be fully functional at this time.
:::

In this example you can see a hint for the best move provided by the engine.

If you have not read "Play vs Stockfish" please do so to get started, with installing and setting up the requirements.

<EngineArrowExample />

## Engine Communication

We will now again create a class which checks for the best move and then uses the
`drawMove` method by the board.

::: code-group

```js [Engine.js]
export class Engine {
  private stockfish;
  private boardApi;
  public bestMove;
  public engineName;

  constructor(boardApi) {
    this.boardApi = boardApi;
    const wasmSupported =
      typeof WebAssembly === 'object' &&
      WebAssembly.validate(
        Uint8Array.of(0x0, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00),
      );

    this.stockfish = new Worker(
      wasmSupported ? 'stockfish.wasm.js' : 'stockfish.js',
    );

    this.setupListeners();

    this.stockfish.postMessage('uci');
  }

  private setupListeners() {
    this.stockfish.addEventListener('message', (data) =>
      this.handleEngineStdout(data),
    );

    this.stockfish.addEventListener('error', (err) => console.error(err));

    this.stockfish.addEventListener('messageerror', (err) =>
      console.error(err),
    );
  }

  private handleEngineStdout(data: MessageEvent<unknown>) {
    const uciStringSplitted = data.data.split(' ');

    if (uciStringSplitted[0] === 'uciok') {
      this.setOption('UCI_AnalyseMode', 'true');
      this.setOption('Analysis Contempt', 'Off');

      this.stockfish.postMessage('ucinewgame');
      this.stockfish.postMessage('isready');
      return;
    }

    if (uciStringSplitted[0] === 'readyok') {
      this.stockfish.postMessage('go movetime 1000');
      return;
    }

    if (uciStringSplitted[0] === 'bestmove' && uciStringSplitted[1]) {
      if (uciStringSplitted[1] !== this.bestMove) {
        this.bestMove = uciStringSplitted[1];
        const orig = this.bestMove.slice(0, 2);
        const dest = this.bestMove.slice(2, 4);
        this.boardApi.drawMove(orig, dest, 'paleBlue');
      }
    }
  }

  private setOption(name, value) {
    this.stockfish.postMessage(`setoption name ${name} value ${value}`);
  }

  public sendPosition(position) {
    this.stockfish.postMessage(`position startpos moves ${position}`);
    this.stockfish.postMessage('go movetime 2000');
  }
}
```

```ts [Engine.ts]
import { type BoardApi, type SquareKey } from 'vue3-chessboard';

export class Engine {
  private stockfish: Worker | undefined;
  private boardApi: BoardApi;
  public bestMove: string | null;
  public engineName: string | null;

  constructor(boardApi: BoardApi) {
    this.boardApi = boardApi;
    const wasmSupported =
      typeof WebAssembly === 'object' &&
      WebAssembly.validate(
        Uint8Array.of(0x0, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00),
      );

    this.stockfish = new Worker(
      wasmSupported ? 'stockfish.wasm.js' : 'stockfish.js',
    );

    this.setupListeners();

    this.stockfish.postMessage('uci');
  }

  private setupListeners(): void {
    this.stockfish?.addEventListener('message', (data) =>
      this.handleEngineStdout(data),
    );

    this.stockfish?.addEventListener('error', (err) => console.error(err));

    this.stockfish?.addEventListener('messageerror', (err) =>
      console.error(err),
    );
  }

  private handleEngineStdout(data: MessageEvent<unknown>) {
    const uciStringSplitted = (data.data as string).split(' ');

    if (uciStringSplitted[0] === 'uciok') {
      this.setOption('UCI_AnalyseMode', 'true');
      this.setOption('Analysis Contempt', 'Off');

      this.stockfish?.postMessage('ucinewgame');
      this.stockfish?.postMessage('isready');
      return;
    }

    if (uciStringSplitted[0] === 'readyok') {
      this.stockfish?.postMessage('go movetime 1000');
      return;
    }

    if (uciStringSplitted[0] === 'bestmove' && uciStringSplitted[1]) {
      if (uciStringSplitted[1] !== this.bestMove) {
        this.bestMove = uciStringSplitted[1];
        const orig = this.bestMove.slice(0, 2) as SquareKey;
        const dest = this.bestMove.slice(2, 4) as SquareKey;
        this.boardApi.drawMove(orig, dest, 'paleBlue');
      }
    }
  }

  private setOption(name: string, value: string): void {
    this.stockfish?.postMessage(`setoption name ${name} value ${value}`);
  }

  public sendPosition(position: string) {
    this.stockfish?.postMessage(`position startpos moves ${position}`);
    this.stockfish?.postMessage('go movetime 2000');
  }
}
```

:::

### Vue Component

Now we create a vue component and import the class.

::: code-group

```vue [JavaScript]
<script setup>
import { TheChessboard } from 'vue3-chessboard';
import { Engine } from './Engine';

let boardAPI;
let engine;
let boardConfig = {
  events: {
    select: () => {
      if (engine?.bestMove) {
        boardAPI?.drawMove(
          engine.bestMove.slice(0, 2),
          engine.bestMove.slice(2, 4),
          'paleBlue',
        );
      }
    },
    move: () => {
      boardAPI?.hideMoves();
    },
  },
};

function handleBoardCreated(boardApi) {
  boardAPI = boardApi;

  engine = new Engine(boardApi);
}

function handleMove() {
  const history = boardAPI?.getHistory(true);

  const moves = history?.map((move) => {
    if (typeof move === 'object') {
      return move.lan;
    } else {
      return move;
    }
  });

  if (moves) {
    engine?.sendPosition(moves.join(' '));
  }
}
</script>

<template>
  <TheChessboard
    :board-config="boardConfig"
    @board-created="handleBoardCreated"
    @move="handleMove"
  />
</template>
```

```vue [TypeScript]
<script setup lang="ts">
import { TheChessboard, type BoardApi, type SquareKey } from 'vue3-chessboard';
import { Engine } from './Engine';

let boardAPI: BoardApi | undefined;
let engine: Engine | undefined;
let boardConfig = {
  events: {
    select: () => {
      if (engine?.bestMove) {
        boardAPI?.drawMove(
          engine.bestMove.slice(0, 2) as SquareKey,
          engine.bestMove.slice(2, 4) as SquareKey,
          'paleBlue',
        );
      }
    },
    move: () => {
      boardAPI?.hideMoves();
    },
  },
};

function handleBoardCreated(boardApi: BoardApi) {
  boardAPI = boardApi;

  engine = new Engine(boardApi);
}

function handleMove() {
  const history = boardAPI?.getHistory(true);

  const moves = history?.map((move) => {
    if (typeof move === 'object') {
      return move.lan;
    } else {
      return move;
    }
  });

  if (moves) {
    engine?.sendPosition(moves.join(' '));
  }
}
</script>

<template>
  <TheChessboard
    :board-config="boardConfig"
    @board-created="handleBoardCreated"
    @move="handleMove"
  />
</template>
```

:::

Thats it. Now once stockfish has calculated the best move an arrow will be drawn on the board.
