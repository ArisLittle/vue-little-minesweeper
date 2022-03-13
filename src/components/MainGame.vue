<template>
  <div>
    <div class="row" v-for="(row, y) in blocks" :key="y">
      <div class="col" v-for="(block, x) in row" :key="x">
        <button
          class="box"
          :class="{
            'box-boom': block.boom && block.revealed,
            [`box-num-${block.neighborBooms}`]: true,
            revealed: block.revealed
          }"
          @click="onClick(block)"
          @contextmenu.prevent="onRightClick(block)"
        >
          <span v-if="block.flagged" class="flag">🏳️</span>
          <span v-else-if="!block.revealed && !isOver" class="trans-num"
            >0</span
          >
          <span v-else-if="block.boom" class="boom"> 💥 </span>
          <span v-else :class="{ 'trans-num': block.neighborBooms === 0 }">
            {{ block.neighborBooms }}
          </span>
        </button>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { defineComponent } from 'vue';
import { ref, reactive } from 'vue';

interface BlockState {
  flagged: boolean;
  revealed: boolean;
  boom: boolean;
  neighborBooms: number;
  col: number;
  row: number;
  index: number;
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const WIDTH = ref(10);
    const HEIGHT = ref(10);
    const isOver = ref(false);
    let blocks: BlockState[][] = [];

    function initBlocks() {
      blocks = reactive(
        Array.from({ length: HEIGHT.value }, (_, rowIndex) => {
          return Array.from({ length: WIDTH.value }, (_, colIndex) => {
            return {
              index: rowIndex * 10 + (colIndex + 1),
              row: rowIndex,
              col: colIndex,
              flagged: false,
              revealed: false,
              boom: isBoom(),
              neighborBooms: 0
            };
          });
        })
      );
    }

    function isBoom() {
      return Math.random() < 0.1;
    }

    const directions = [
      [-1, -1],
      [-1, 0],
      [-1, 1],
      [1, 1],
      [1, 0],
      [1, -1],
      [0, 1],
      [0, -1]
    ];

    function getNeighbors(block: BlockState) {
      return directions
        .map(([dRow, dCol]) => {
          const newRow = block.row + dRow;
          const newCol = block.col + dCol;
          if (
            newRow < 0 ||
            newRow >= HEIGHT.value ||
            newCol < 0 ||
            newCol >= WIDTH.value
          ) {
            return undefined;
          }
          return blocks[newRow][newCol];
        })
        .filter((item) => item) as BlockState[];
    }

    function caclNeighborBooms() {
      blocks.forEach((row) => {
        row.forEach((block) => {
          let booms = 0;
          getNeighbors(block).forEach((item) => {
            if (item.boom) {
              booms++;
            }
          });
          block.neighborBooms = booms;
        });
      });
    }

    initBlocks();
    caclNeighborBooms();

    function reveal(block: BlockState) {
      if (block.revealed) {
        return;
      }
      block.revealed = true;
      block.flagged = false;
      if (block.neighborBooms === 0) {
        getNeighbors(block).forEach((n) => {
          if (n.neighborBooms === 0) {
            reveal(n);
          }
        });
      }
      checkGameStatus();
    }

    function checkGameStatus() {
      const flatBlocks = blocks.flat();
      const isAllChecked = flatBlocks.every((item) => {
        return item.flagged || item.revealed;
      });
      if (isAllChecked) {
        const lose = flatBlocks.some((item) => item.flagged && !item.boom);
        if (lose) {
          showLose();
        } else {
          showWin();
        }
      }
    }

    function showWin() {
      alert('你胜利了~');
      isOver.value = true;
    }

    function showLose() {
      alert('爆炸了~');
      isOver.value = true;
    }

    return {
      blocks,
      isOver,
      reveal,
      showLose,
      checkGameStatus
    };
  },
  methods: {
    onClick(block: BlockState) {
      this.reveal(block);
      if (block.boom) {
        this.showLose();
      }
    },
    onRightClick(block: BlockState) {
      block.flagged = true;
      this.checkGameStatus();
    }
  }
});
</script>
<style scoped>
.col {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  padding: 0.5px;
}
.box {
  width: 40px;
  height: 40px;
  border-radius: 0;
  border: none;
  line-height: 1;
  background-color: rgba(66, 66, 66, 0.4);
  font-weight: 800;
  font-size: 16px;
  cursor: pointer;
}
.box-num-1 {
  color: darkorchid;
}
.box-num-2 {
  color: darkkhaki;
}
.box-num-3 {
  color: orange;
}
.box-num-4 {
  color: lightgreen;
}
.box-num-5 {
  color: lightpink;
}
.box-num-6 {
  color: purple;
}
.box-num-7 {
  color: brown;
}
.box-num-8 {
  color: teal;
}
.box:hover:not(.revealed) {
  background-color: rgba(66, 66, 66, 0.2);
}
.revealed {
  background: rgba(66, 66, 66, 0.1);
  cursor: default;
}
.box-boom {
  background: rgba(255, 0, 0, 0.3);
}
.trans-num {
  color: transparent;
}
</style>
