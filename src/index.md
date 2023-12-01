<script setup>
import { ref, watch, onMounted } from 'vue';
import { useData } from 'vitepress';
import { TheChessboard } from 'vue3-chessboard';

const theme = useData();
const svgPath = ref(null);
const versionNumber = ref("");

function setSvgPath() {
  if (theme.isDark.value) {
    svgPath.value = '/github-mark-white.svg';
  } else {
    svgPath.value = '/github-mark.svg';
  }
}

onMounted(async () => {
  setSvgPath();
  try {
    const res = await fetch('https://registry.npmjs.org/vue3-chessboard/latest');
    const data = await res.json();
    versionNumber.value = data.version ?? "";
  } catch {}
});

watch(theme.isDark, () => {
  setSvgPath();
});

let boardAPI;

function handleCheckmate(isMated) {
  if (isMated === 'w') {
    alert('Black wins!');
  } else {
    alert('White wins!');
  }
  boardAPI?.resetBoard();
}

function handleCheck(isChecked) {
  console.log(isChecked)
  if (isChecked === 'w') {
    alert('White is in check!');
  } else {
    alert('Black is in check!');
  }
}

</script>

# vue3-chessboard

## A vue.js component library

<div class="chessboard">
  <div class="buttons">
    <button @click="boardAPI.toggleOrientation">
      Toggle orientation
    </button>
    <button @click="boardAPI.resetBoard">
      Reset Board
    </button>
    <button @click="boardAPI.undoLastMove">
      Undo last move
    </button>
    <button @click="boardAPI.toggleMoves">
      Toggle possible moves/captures
    </button>
  </div>
  <TheChessboard
    @board-created="(api) => (boardAPI = api)"
    @checkmate="handleCheckmate"
    @check="handleCheck"
  />
</div>

<div class="svg-container">
  <a v-show="theme.isDark" href="https://github.com/qwerty084/vue3-chessboard" target="_blank" rel="noreferrer">
    <img :src="svgPath" alt="github repository" title="GitHub Repository" />
  </a>
  <a href="https://www.npmjs.com/package/vue3-chessboard" target="_blank" rel="noreferrer" >
    <img src="/npm.svg" alt="NPM Package" title="NPM Package" class="npm-svg" />
  </a>
</div>

<p class="version-number">{{ versionNumber }}</p>
