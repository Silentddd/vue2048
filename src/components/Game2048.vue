<template>
  <div class="game-container">
    <el-card class="game-card">
      <div class="header">
        <div class="score-container">
          <div class="score-label">得分</div>
          <div class="score">{{ score }}</div>
        </div>
        <el-button type="primary" @click="newGame">新游戏</el-button>
      </div>
      <div class="game-grid" 
           @touchstart="handleTouchStart"
           @touchmove="handleTouchMove"
           @touchend="handleTouchEnd"
           tabindex="0"
           @keydown.prevent="handleKeydown">
        <div class="grid-row" v-for="(row, rowIndex) in board" :key="rowIndex">
          <div class="grid-cell" 
               v-for="(cell, colIndex) in row" 
               :key="colIndex"
               :class="['tile-' + cell, { 'tile-new': isNew(rowIndex, colIndex) }]">
            {{ cell || '' }}
          </div>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Game2048',
  data() {
    return {
      board: [],
      score: 0,
      newTiles: new Set(),
      touchStartX: 0,
      touchStartY: 0
    }
  },
  created() {
    this.newGame()
  },
  mounted() {
    window.addEventListener('keydown', this.handleKeydown)
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.handleKeydown)
  },
  methods: {
    newGame() {
      this.board = Array(4).fill().map(() => Array(4).fill(0))
      this.score = 0
      this.newTiles.clear()
      this.addNewTile()
      this.addNewTile()
    },
    addNewTile() {
      const emptyCells = []
      for (let i = 0; i < 4; i++) {
        for (let j = 0; j < 4; j++) {
          if (this.board[i][j] === 0) {
            emptyCells.push([i, j])
          }
        }
      }
      if (emptyCells.length > 0) {
        const [row, col] = emptyCells[Math.floor(Math.random() * emptyCells.length)]
        this.board[row][col] = Math.random() < 0.9 ? 2 : 4
        this.newTiles.add(`${row}-${col}`)
        setTimeout(() => {
          this.newTiles.delete(`${row}-${col}`)
        }, 300)
      }
    },
    isNew(row, col) {
      return this.newTiles.has(`${row}-${col}`)
    },
    move(direction) {
      const originalBoard = JSON.stringify(this.board)
      let moved = false

      // 移动和合并逻辑
      const moveLeft = () => {
        for (let i = 0; i < 4; i++) {
          let row = this.board[i].filter(cell => cell !== 0)
          for (let j = 0; j < row.length - 1; j++) {
            if (row[j] === row[j + 1]) {
              row[j] *= 2
              this.score += row[j]
              row.splice(j + 1, 1)
            }
          }
          while (row.length < 4) row.push(0)
          this.board[i] = row
        }
      }

      const rotate = () => {
        const newBoard = Array(4).fill().map(() => Array(4).fill(0))
        for (let i = 0; i < 4; i++) {
          for (let j = 0; j < 4; j++) {
            newBoard[i][j] = this.board[j][3 - i]
          }
        }
        this.board = newBoard
      }

      // 根据方向执行移动
      switch (direction) {
        case 'left':
          moveLeft()
          break
        case 'right':
          this.board.forEach(row => row.reverse())
          moveLeft()
          this.board.forEach(row => row.reverse())
          break
        case 'up':
          rotate()
          moveLeft()
          rotate()
          rotate()
          rotate()
          break
        case 'down':
          rotate()
          rotate()
          rotate()
          moveLeft()
          rotate()
          break
      }

      // 检查是否有移动
      moved = originalBoard !== JSON.stringify(this.board)
      if (moved) {
        this.addNewTile()
      }

      // 检查游戏是否结束
      if (this.isGameOver()) {
        this.$message.error('游戏结束！')
      }
    },
    isGameOver() {
      // 检查是否有空格
      for (let i = 0; i < 4; i++) {
        for (let j = 0; j < 4; j++) {
          if (this.board[i][j] === 0) return false
        }
      }

      // 检查是否可以合并
      for (let i = 0; i < 4; i++) {
        for (let j = 0; j < 3; j++) {
          if (this.board[i][j] === this.board[i][j + 1]) return false
          if (this.board[j][i] === this.board[j + 1][i]) return false
        }
      }

      return true
    },
    handleKeydown(e) {
      const keyMap = {
        'ArrowLeft': 'left',
        'ArrowRight': 'right',
        'ArrowUp': 'up',
        'ArrowDown': 'down'
      }
      if (keyMap[e.key]) {
        this.move(keyMap[e.key])
      }
    },
    handleTouchStart(e) {
      this.touchStartX = e.touches[0].clientX
      this.touchStartY = e.touches[0].clientY
    },
    handleTouchMove(e) {
      e.preventDefault()
    },
    handleTouchEnd(e) {
      const touchEndX = e.changedTouches[0].clientX
      const touchEndY = e.changedTouches[0].clientY
      const dx = touchEndX - this.touchStartX
      const dy = touchEndY - this.touchStartY
      
      if (Math.abs(dx) > Math.abs(dy)) {
        if (Math.abs(dx) > 50) {
          this.move(dx > 0 ? 'right' : 'left')
        }
      } else {
        if (Math.abs(dy) > 50) {
          this.move(dy > 0 ? 'down' : 'up')
        }
      }
    }
  }
}
</script>

<style scoped>
.game-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  box-sizing: border-box;
}

.game-card {
  width: 500px;
  background-color: #bbada0 !important;
  border-radius: 6px;
  padding: 15px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.score-container {
  background: #8f7a66;
  padding: 15px 25px;
  border-radius: 6px;
  color: white;
  text-align: center;
}

.score-label {
  font-size: 13px;
  color: #eee4da;
}

.score {
  font-size: 25px;
  font-weight: bold;
}

.game-grid {
  display: grid;
  grid-template-rows: repeat(4, 1fr);
  gap: 15px;
  background-color: #bbada0;
  border-radius: 6px;
  padding: 15px;
  margin: 0 auto;
}

.grid-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
}

.grid-cell {
  width: 100px;
  height: 100px;
  background-color: rgba(238, 228, 218, 0.35);
  border-radius: 3px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 55px;
  font-weight: bold;
  color: #776e65;
  transition: all 0.15s ease;
}

.tile-2 { background-color: #eee4da; }
.tile-4 { background-color: #ede0c8; }
.tile-8 { background-color: #f2b179; color: #f9f6f2; }
.tile-16 { background-color: #f59563; color: #f9f6f2; }
.tile-32 { background-color: #f67c5f; color: #f9f6f2; }
.tile-64 { background-color: #f65e3b; color: #f9f6f2; }
.tile-128 { 
  background-color: #edcf72; 
  color: #f9f6f2;
  font-size: 45px;
}
.tile-256 { 
  background-color: #edcc61; 
  color: #f9f6f2;
  font-size: 45px;
}
.tile-512 { 
  background-color: #edc850; 
  color: #f9f6f2;
  font-size: 45px;
}
.tile-1024 { 
  background-color: #edc53f; 
  color: #f9f6f2;
  font-size: 35px;
}
.tile-2048 { 
  background-color: #edc22e; 
  color: #f9f6f2;
  font-size: 35px;
}

.tile-new {
  animation: appear 0.3s ease-in-out;
}

@keyframes appear {
  0% {
    opacity: 0;
    transform: scale(0);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

/* 响应式设计 */
@media screen and (max-width: 520px) {
  .game-card {
    width: 100%;
    margin: 0 10px;
  }

  .grid-cell {
    width: auto;
    height: auto;
    aspect-ratio: 1;
    font-size: 35px;
  }

  .tile-128,
  .tile-256,
  .tile-512 {
    font-size: 30px;
  }

  .tile-1024,
  .tile-2048 {
    font-size: 25px;
  }
}
</style> 