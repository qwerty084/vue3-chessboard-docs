<script setup>
import EngineExample from "./EngineExample.vue";
</script>

# Engines

## Play vs Stockfish

You play vs Stockfish as white. Make your first move!

<EngineExample />

In this example we can play vs Stockfish using [stockfish.js](https://github.com/lichess-org/stockfish.js), which uses either a webassembly or javascript implementation depending on the browser.

There is also [stockfish.wasm](https://github.com/lichess-org/stockfish.wasm) which is a stronger version, but it requires setting some HTTP headers which is currently not possible with GitHub Pages.

## Setup

vue3-chessboard does not ship with engine support out of the box, but its pretty easy to add to the board, since there are methods for drawing/hinting moves (`drawMove`) and also to make moves programatically (`move`).

In this example [stockfish.js](https://github.com/lichess-org/stockfish.js) by [Lichess](https://lichess.org) is used.

It uses a Wasm compiled version of stockfish if the browser supports it and a JavaScript implementation as a fallback for legacy browsers.
To keep the UI responsive we are running Stockfish in a [Worker](https://developer.mozilla.org/en-US/docs/Web/API/Worker).

### Install stockfish.js

::: code-group

```bash [npm]
npm i stockfish.js
```

```bash [yarn]
yarn add stockfish.js
```

```bash [pnpm]
pnpm install stockfish.js
```

:::

### Move stockfish files into public folder

::: code-group

```bash
cp node_modules/stockfish.js/stockfish.* public
```

```Powershell
Copy-Item -Path .\node_modules\stockfish.js\stockfish.* -Destination .\public\
```

:::

### Engine Communication

Now we create a class for reading the engine output and forwarding the info like the best move to the board.

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
        Uint8Array.of(0x0, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00)
      );

    this.stockfish = new Worker(
      wasmSupported ? 'stockfish.wasm.js' : 'stockfish.js'
    );

    this.setupListeners();

    this.stockfish.postMessage('uci');
  }

  private setupListeners() {
    this.stockfish.addEventListener('message', (data) =>
      this.handleEngineStdout(data)
    );

    this.stockfish.addEventListener('error', (err) => console.error(err));

    this.stockfish.addEventListener('messageerror', (err) =>
      console.error(err)
    );
  }

  private handleEngineStdout(data) {
    const uciStringSplitted = (data.data).split(' ');

    if (uciStringSplitted[0] === 'uciok') {
      this.setOption('UCI_AnalyseMode', 'true');
      this.setOption('Analysis Contempt', 'Off');

      this.stockfish.postMessage('ucinewgame');
      this.stockfish.postMessage('isready');
      return;
    }

    if (uciStringSplitted[0] === 'readyok') {
      this.stockfish.postMessage('go movetime 1500');
      return;
    }

    if (uciStringSplitted[0] === 'bestmove' && uciStringSplitted[1]) {
      if (uciStringSplitted[1] !== this.bestMove) {
        this.bestMove = uciStringSplitted[1];
        if (this.boardApi.getTurnColor() === 'black') {
          this.boardApi.move({
            from: this.bestMove.slice(0, 2),
            to: this.bestMove.slice(2, 4),
          });
        }
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
import { type BoardApi } from 'vue3-chessboard';
import { SquareKey } from 'vue3-chessboard';

export class Engine {
  private stockfish: Worker | undefined;
  private boardApi: BoardApi | undefined;
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
    this.stockfish.addEventListener('message', (data) =>
      this.handleEngineStdout(data),
    );

    this.stockfish.addEventListener('error', (err) => console.error(err));

    this.stockfish.addEventListener('messageerror', (err) =>
      console.error(err),
    );
  }

  private handleEngineStdout(data: MessageEvent<unknown>) {
    const uciStringSplitted = (data.data as string).split(' ');

    if (uciStringSplitted[0] === 'uciok') {
      this.setOption('UCI_AnalyseMode', 'true');
      this.setOption('Analysis Contempt', 'Off');

      this.stockfish.postMessage('ucinewgame');
      this.stockfish.postMessage('isready');
      return;
    }

    if (uciStringSplitted[0] === 'readyok') {
      this.stockfish.postMessage('go movetime 1500');
      return;
    }

    if (uciStringSplitted[0] === 'bestmove' && uciStringSplitted[1]) {
      if (uciStringSplitted[1] !== this.bestMove) {
        this.bestMove = uciStringSplitted[1];
        if (this.boardApi.getTurnColor() === 'black') {
          this.boardApi.move({
            from: this.bestMove.slice(0, 2) as SquareKey,
            to: this.bestMove.slice(2, 4) as SquareKey,
          });
        }
      }
    }
  }

  private setOption(name: string, value: string): void {
    this.stockfish.postMessage(`setoption name ${name} value ${value}`);
  }

  public sendPosition(position: string) {
    this.stockfish.postMessage(`position startpos moves ${position}`);
    this.stockfish.postMessage('go movetime 2000');
  }
}
```

:::

### Vue Component

Now we create a vue component and import our new class.

::: code-group

```vue [JavaScript]
<script setup>
import { TheChessboard } from 'vue3-chessboard';
import { Engine } from './Engine';

let boardAPI;
let engine;

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
    @board-created="handleBoardCreated"
    @move="handleMove"
    :player-color="'white'"
  />
</template>
```

```vue [TypeScript]
<script setup lang="ts">
import { TheChessboard, type BoardApi } from 'vue3-chessboard';
import { Engine } from './Engine';

let boardAPI: BoardApi | undefined;
let engine: Engine | undefined;

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
    @board-created="handleBoardCreated"
    @move="handleMove"
    :player-color="'white'"
  />
</template>
```

:::

Thats it. You can now play vs Stockfish. <br>
For a more advanced setup take a look at [ceval](https://github.com/lichess-org/lila/tree/master/ui/ceval) from lichess.
