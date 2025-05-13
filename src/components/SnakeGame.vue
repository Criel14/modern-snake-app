<template>
  <div class="game-container">
    <div class="language-switcher">
      <select class="language-select" v-model="currentLanguage" @change="saveLanguageSettings">
        <option value="zh">中文</option>
        <option value="en">English</option>
      </select>
    </div>
    <div class="game-controls">
      <div class="control-panel">
        <h2>{{ t('gameControls') }}</h2>
        
        <div class="game-mode">
          <h3>{{ t('gameMode') }}</h3>
          <div class="mode-buttons">
            <button 
              class="mode-button" 
              :class="{ active: gameMode === 'classic' }" 
              @click="setGameMode('classic')" 
              :disabled="gameRunning"
            >
              {{ t('classicMode') }}
            </button>
            <button 
              class="mode-button" 
              :class="{ active: gameMode === 'speed' }" 
              @click="setGameMode('speed')" 
              :disabled="gameRunning"
            >
              {{ t('speedMode') }}
            </button>
          </div>
          <div class="mode-description">
            {{ gameModeDescription }}
          </div>
        </div>
        
        <button class="start-button" @click="startGameCountdown" :disabled="gameRunning || countdownActive">
          {{ countdownActive ? `${t('countdown')}: ${countdown}` : t('startGame') }}
        </button>
        <button class="reset-button" @click="resetGame" :disabled="!gameOver && !gameRunning">{{ t('resetGame') }}</button>
        <button class="rules-button" @click="showRulesModal">{{ t('viewRules') }}</button>
        
        <div class="speed-control">
          <h3>{{ t('gameSpeed') }}</h3>
          <div class="slider-container">
            <span class="speed-label">{{ t('slow') }}</span>
            <input 
              type="range" 
              min="100" 
              max="320" 
              step="10" 
              v-model="gameSpeed" 
              :disabled="gameRunning" 
              class="custom-slider"
            />
            <span class="speed-label">{{ t('fast') }}</span>
          </div>
          <div class="speed-value">{{ 400 - gameSpeed }}ms</div>
        </div>
        
        <button class="color-settings-button" @click="showColorSettingsModal">{{ t('customizeStyle') }}</button>
      </div>
    </div>
    
    <div class="game-board-container">
      <div class="game-info">
        <div class="score">{{ t('score') }}: {{ score }}</div>
        <div class="status" v-if="gameOver">{{ t('gameOver') }}!</div>
        <div class="status" v-else-if="gamePaused">{{ t('gamePaused') }}</div>
      </div>
      
      <div 
        class="game-board" 
        ref="gameBoard"
        :style="{
          'grid-template-columns': `repeat(${boardSize}, 1fr)`,
          'grid-template-rows': `repeat(${boardSize}, 1fr)`
        }"
      >
        <div 
          v-for="(cell, index) in cells" 
          :key="index" 
          class="cell"
          :class="{
            'snake-cell': isSnakeCell(index),
            'snake-head': isSnakeHead(index),
            'apple-cell': isAppleCell(index),
            'collision-cell': collisionDelay && (isSnakeHead(index) || isSnakeCell(index)),
            'dash-cell': isDashing && (isSnakeHead(index) || isSnakeCell(index))
          }"
          :style="{
            'background-color': getCellColor(index)
          }"
        ></div>
      </div>
    </div>
    
    <div class="game-stats">
      <div class="game-metrics">
        <h2>{{ t('gameData') }}</h2>
        <div class="metric-item">
          <span class="metric-label">{{ t('gameTime') }}:</span>
          <span class="metric-value">{{ formatTime(gameTime) }}</span>
        </div>
        <div class="metric-item" v-if="score > 0">
          <span class="metric-label">{{ t('speed') }}:</span>
          <span class="metric-value">{{ timePerApple }}{{ t('secondsPerApple') }}</span>
        </div>
        <div class="metric-item" v-if="gameMode === 'speed'">
          <span class="metric-label">{{ t('target') }}:</span>
          <span class="metric-value">{{ score }}/40 {{ t('apples') }}</span>
        </div>
        <div class="progress-bar" v-if="gameMode === 'speed'">
          <div class="progress" :style="{ width: `${(score / 40) * 100}%` }"></div>
        </div>
      </div>
      
      <div class="high-scores">
        <h2>{{ gameMode === 'classic' ? t('highestScore') : t('bestTime') }}</h2>
        <div class="score-list">
          <div v-for="(record, index) in displayHighScores" :key="index" class="high-score-item">
            <span>{{ index + 1 }}.</span>
            <span v-if="gameMode === 'classic'">{{ record.score }}{{ t('points') }}</span>
            <span v-else class="score-time">{{ formatTime(record.time) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 游戏规则模态框 -->
  <div class="modal-overlay" 
       v-if="showRules" 
       @click="hideRules"
       :class="{ 'fade-out': isRulesClosing }">
    <div class="modal-content" 
         @click.stop
         :class="{ 'slide-down': isRulesClosing }">
      <div class="modal-header">
        <h2>{{ t('gameRules') }}</h2>
        <button class="close-button" @click="hideRules">&times;</button>
      </div>
      <div class="modal-body">
        <h3>{{ t('basicControls') }}</h3>
        <ul>
          <li>{{ t('controlsDescription1') }}</li>
          <li>{{ t('controlsDescription2') }}</li>
          <li>{{ t('controlsDescription3') }}</li>
        </ul>
        
        <h3>{{ t('gameFeatures') }}</h3>
        <ul>
          <li>{{ t('featuresDescription1') }}</li>
          <li>{{ t('featuresDescription2') }}</li>
          <li>{{ t('featuresDescription3') }}</li>
          <li>{{ t('featuresDescription4') }}</li>
          <li>{{ t('featuresDescription5') }}</li>
        </ul>
        
        <h3>{{ t('gameModes') }}</h3>
        <ul>
          <li><b>{{ t('classicMode') }}</b>：{{ t('classicModeDescription') }}</li>
          <li><b>{{ t('speedMode') }}</b>：{{ t('speedModeDescription') }}</li>
        </ul>
        
        <h3>{{ t('tips') }}</h3>
        <ul>
          <li>{{ t('tipsDescription1') }}</li>
          <li>{{ t('tipsDescription2') }}</li>
          <li>{{ t('tipsDescription3') }}</li>
          <li>{{ t('tipsDescription4') }}</li>
        </ul>
      </div>
    </div>
  </div>
  
  <!-- 颜色设置模态框 -->
  <div class="modal-overlay" 
       v-if="showColorSettings" 
       @click="hideColorSettings"
       :class="{ 'fade-out': isColorSettingsClosing }">
    <div class="modal-content color-settings-modal" 
         @click.stop
         :class="{ 'slide-down': isColorSettingsClosing }">
      <div class="modal-header">
        <h2>{{ t('customizeStyle') }}</h2>
        <button class="close-button" @click="hideColorSettings">&times;</button>
      </div>
      <div class="modal-body">
        <div class="color-setting-item">
          <label for="snake-color">{{ t('snakeColor') }}</label>
          <input id="snake-color" type="color" v-model="snakeColor" :disabled="gameRunning" @change="updateSnakeColor" />
        </div>
        
        <div class="color-setting-item">
          <label for="apple-color">{{ t('appleColor') }}</label>
          <input id="apple-color" type="color" v-model="appleColor" :disabled="gameRunning" @change="updateAppleColor" />
        </div>
        
        <div class="color-setting-item">
          <label for="danger-color">{{ t('dangerColor') }}</label>
          <input id="danger-color" type="color" v-model="dangerColor" :disabled="gameRunning" @change="updateDangerColor" />
        </div>
        
        <div class="color-presets">
          <h3>{{ t('presets') }}</h3>
          <div class="preset-buttons">
            <button @click="applyColorPreset('default')" :disabled="gameRunning">{{ t('defaultColor') }}</button>
            <button @click="applyColorPreset('dark')" :disabled="gameRunning">{{ t('darkMode') }}</button>
            <button @click="applyColorPreset('neon')" :disabled="gameRunning">{{ t('neonStyle') }}</button>
          </div>
        </div>
      </div>
    </div>
  </div>
  
  <!-- 游戏结束模态框 -->
  <div class="modal-overlay" 
       v-if="showGameEnd" 
       @click="hideGameEnd"
       :class="{ 'fade-out': isGameEndClosing }">
    <div class="modal-content game-end-modal" 
         @click.stop
         :class="{ 'slide-down': isGameEndClosing }">
      <div class="modal-header">
        <h2>{{ t('gameComplete') }}</h2>
        <button class="close-button" @click="hideGameEnd">&times;</button>
      </div>
      <div class="modal-body">
        <div class="game-end-message">
          <h3 v-if="gameMode === 'speed'">{{ t('speedModeComplete') }}!</h3>
          <h3 v-else>{{ t('classicModeEnd') }}!</h3>
          
          <div class="game-result">
            <div class="result-item" v-if="gameMode === 'speed'">
              <span class="result-label">{{ t('completionTime') }}:</span>
              <span class="result-value highlight">{{ formatTime(gameTime) }}</span>
            </div>
            <div class="result-item">
              <span class="result-label">{{ t('finalScore') }}:</span>
              <span class="result-value highlight">{{ score }}</span>
            </div>
            <div class="result-item" v-if="score > 0">
              <span class="result-label">{{ t('averageSpeed') }}:</span>
              <span class="result-value">{{ timePerApple }}{{ t('secondsPerApple') }}</span>
            </div>
          </div>
          
          <div class="high-score-status" v-if="isHighScore">
            <div class="high-score-icon">🏆</div>
            <div class="high-score-message">{{ t('newHighScore') }}!</div>
          </div>
        </div>
        
        <div class="game-end-actions">
          <button class="play-again-button" @click="playAgain">{{ t('playAgain') }}</button>
          <button class="change-mode-button" @click="changeGameMode">{{ t('changeMode') }}</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SnakeGame',
  data() {
    return {
      boardSize: 20,
      cells: [],
      snake: [], // 蛇的身体坐标数组
      direction: 'right', // 初始方向
      nextDirection: 'right',
      apple: null, // 苹果的坐标
      gameInterval: null,
      gameRunning: false,
      gameOver: false,
      gamePaused: false,
      score: 0,
      highScores: [],
      snakeColor: '#4ecca3', // 默认蛇的颜色
      appleColor: '#ff6b6b', // 默认苹果的颜色
      dangerColor: '#d4af37', // 默认危险提示颜色，改为偏黄的绿色
      gameSpeed: 200, // 游戏速度 (值越低越快)
      normalGameSpeed: 200, // 保存正常游戏速度
      dashSpeed: 350, // 突进速度 (毫秒，值越大越快)
      isDashing: false, // 是否在突进状态
      countdown: 3,
      countdownActive: false,
      countdownTimer: null,
      gameMode: 'classic', // 默认为经典模式
      gameTime: 0, // 游戏时间（秒）
      timePerApple: 0, // 每个苹果的平均时间
      gameTimeInterval: null, // 计时器
      appleTimestamps: [], // 记录每个苹果被吃掉的时间戳
      lastAppleTime: 0, // 上一个苹果被吃掉的时间
      collisionDelay: false, // 是否在碰撞延迟状态
      collisionTimer: null, // 碰撞延迟计时器
      showRules: false, // 控制规则模态框的显示
      showColorSettings: false, // 控制颜色设置模态框的显示
      isRulesClosing: false, // 规则模态框是否正在关闭
      isColorSettingsClosing: false, // 颜色设置模态框是否正在关闭
      showGameEnd: false, // 控制游戏结束模态框的显示
      isGameEndClosing: false, // 游戏结束模态框是否正在关闭
      isHighScore: false, // 是否创造了新的高分
      currentLanguage: 'zh', // 默认语言为中文
      translations: {
        zh: {
          gameControls: '游戏控制',
          gameMode: '游戏模式',
          classicMode: '经典模式',
          speedMode: '竞速模式',
          startGame: '开始游戏',
          resetGame: '重置游戏',
          viewRules: '查看游戏规则',
          gameSpeed: '游戏速度',
          slow: '慢',
          fast: '快',
          customizeStyle: '自定义样式',
          score: '得分',
          gameOver: '游戏结束',
          gamePaused: '游戏暂停',
          gameData: '游戏数据',
          gameTime: '游戏时间',
          speed: '速度',
          secondsPerApple: '秒/苹果',
          target: '目标',
          apples: '个苹果',
          highestScore: '最高分',
          bestTime: '最佳时间',
          points: '分',
          gameRules: '游戏规则',
          basicControls: '基本操作',
          controlsDescription1: '使用 W, A, S, D 键或方向键控制蛇的移动方向',
          controlsDescription2: '按空格键启动快速突进模式，直到突进到蛇身/苹果/墙体',
          controlsDescription3: '按 ESC 键暂停游戏',
          gameFeatures: '游戏特色',
          featuresDescription1: '蛇撞墙或撞到自身时有1秒反应时间',
          featuresDescription2: '在危险状态下按方向键可避免游戏结束',
          featuresDescription3: '成功改变方向后蛇会立即移动并退出危险状态',
          featuresDescription4: '危险状态下蛇身会闪烁发光提示',
          featuresDescription5: '突进模式下蛇会以极快速度移动，直到撞到障碍物或吃到苹果',
          gameModes: '游戏模式',
          classicModeDescription: '无时间限制，吃尽可能多的苹果，撞墙或自身则结束游戏。',
          speedModeDescription: '竞速吃够40个苹果，比拼最短完成时间。',
          tips: '小提示',
          tipsDescription1: '提前计划路线避免死路',
          tipsDescription2: '蛇撞墙后及时转向可以挽救危机',
          tipsDescription3: '合理使用空格突进可以快速抢食苹果',
          tipsDescription4: '注意观察危险提示，把握逃脱时机',
          snakeColor: '蛇的颜色',
          appleColor: '苹果颜色',
          dangerColor: '危险提示颜色',
          presets: '预设方案',
          defaultColor: '默认配色',
          darkMode: '暗黑模式',
          neonStyle: '霓虹风格',
          countdown: '倒计时',
          classicModeDesc: '无时间限制，吃尽可能多的苹果，撞墙或自身则结束游戏。',
          speedModeDesc: '竞速吃够40个苹果，比拼最短完成时间。',
          congratulations: '恭喜！您完成了竞速模式！用时',
          gameComplete: '游戏结束',
          speedModeComplete: '竞速模式完成',
          classicModeEnd: '经典模式结束',
          completionTime: '完成时间',
          finalScore: '最终得分',
          averageSpeed: '平均速度',
          newHighScore: '新的高分记录',
          playAgain: '再玩一次',
          changeMode: '切换模式'
        },
        en: {
          gameControls: 'Game Controls',
          gameMode: 'Game Mode',
          classicMode: 'Classic Mode',
          speedMode: 'Speed Mode',
          startGame: 'Start Game',
          resetGame: 'Reset Game',
          viewRules: 'View Rules',
          gameSpeed: 'Game Speed',
          slow: 'Slow',
          fast: 'Fast',
          customizeStyle: 'Customize Style',
          score: 'Score',
          gameOver: 'Game Over',
          gamePaused: 'Game Paused',
          gameData: 'Game Stats',
          gameTime: 'Game Time',
          speed: 'Speed',
          secondsPerApple: 's/apple',
          target: 'Target',
          apples: 'apples',
          highestScore: 'High Scores',
          bestTime: 'Best Time',
          points: 'pts',
          gameRules: 'Game Rules',
          basicControls: 'Basic Controls',
          controlsDescription1: 'Use W, A, S, D or arrow keys to control the snake',
          controlsDescription2: 'Press Space to dash until hitting a wall/apple/snake',
          controlsDescription3: 'Press ESC to pause the game',
          gameFeatures: 'Game Features',
          featuresDescription1: 'Snake has 1 second reaction time when hitting walls or itself',
          featuresDescription2: 'Change direction during danger state to avoid game over',
          featuresDescription3: 'Successfully changing direction moves the snake immediately',
          featuresDescription4: 'Snake will flash in danger state',
          featuresDescription5: 'Dash mode makes the snake move extremely fast until hitting an obstacle',
          gameModes: 'Game Modes',
          classicModeDescription: 'No time limit, eat as many apples as possible.',
          speedModeDescription: 'Race to eat 40 apples in the shortest time.',
          tips: 'Tips',
          tipsDescription1: 'Plan your route in advance to avoid dead ends',
          tipsDescription2: 'Quickly change direction after hitting a wall to save yourself',
          tipsDescription3: 'Use space dash wisely to quickly grab apples',
          tipsDescription4: 'Pay attention to danger indicators and react in time',
          snakeColor: 'Snake Color',
          appleColor: 'Apple Color',
          dangerColor: 'Danger Color',
          presets: 'Presets',
          defaultColor: 'Default Colors',
          darkMode: 'Dark Mode',
          neonStyle: 'Neon Style',
          countdown: 'Countdown',
          classicModeDesc: 'No time limit, eat as many apples as possible.',
          speedModeDesc: 'Race to eat 40 apples in the shortest time.',
          congratulations: 'Congratulations! You completed Speed Mode in',
          gameComplete: 'Game Complete',
          speedModeComplete: 'Speed Mode Complete',
          classicModeEnd: 'Classic Mode End',
          completionTime: 'Completion Time',
          finalScore: 'Final Score',
          averageSpeed: 'Average Speed',
          newHighScore: 'New High Score',
          playAgain: 'Play Again',
          changeMode: 'Change Mode'
        }
      }
    }
  },
  created() {
    // 初始化游戏板
    this.initializeBoard()
    
    // 加载本地存储的高分
    this.loadHighScores()
    
    // 加载颜色配置
    this.loadColorSettings()
    
    // 加载速度设置
    this.loadSpeedSettings()
    
    // 加载语言设置
    this.loadLanguageSettings()
  },
  mounted() {
    // 添加键盘事件监听
    window.addEventListener('keydown', this.handleKeyPress)
  },
  beforeUnmount() {
    // 移除事件监听
    window.removeEventListener('keydown', this.handleKeyPress)
    this.stopGame()
  },
  computed: {
    // 根据游戏模式返回描述
    gameModeDescription() {
      if (this.gameMode === 'classic') {
        return this.t('classicModeDesc');
      } else {
        return this.t('speedModeDesc');
      }
    },
    
    // 根据游戏模式显示不同的高分列表
    displayHighScores() {
      return this.highScores
    }
  },
  methods: {
    // 国际化翻译方法
    t(key) {
      return this.translations[this.currentLanguage][key] || key;
    },
    
    // 切换语言
    toggleLanguage() {
      this.currentLanguage = this.currentLanguage === 'zh' ? 'en' : 'zh';
      this.saveLanguageSettings();
    },
    
    // 保存语言设置
    saveLanguageSettings() {
      localStorage.setItem('snakeLanguage', this.currentLanguage);
    },
    
    // 加载语言设置
    loadLanguageSettings() {
      const storedLanguage = localStorage.getItem('snakeLanguage');
      if (storedLanguage) {
        this.currentLanguage = storedLanguage;
      }
    },
    initializeBoard() {
      // 创建 boardSize x boardSize 的游戏板
      this.cells = Array(this.boardSize * this.boardSize).fill(0)
      
      // 初始化蛇
      this.resetSnake()
      
      // 生成第一个苹果
      this.generateApple()
    },
    resetSnake() {
      // 蛇初始位置在游戏板中间
      const middle = Math.floor(this.boardSize / 2)
      
      // 蛇的初始长度为5
      this.snake = [
        this.getIndex(middle, middle),
        this.getIndex(middle - 1, middle),
        this.getIndex(middle - 2, middle),
        this.getIndex(middle - 3, middle),
        this.getIndex(middle - 4, middle)
      ]
      
      this.direction = 'right'
      this.nextDirection = 'right'
    },
    startGameCountdown() {
      if (this.gameRunning || this.countdownActive) return
      
      // 关闭游戏结束模态框，如果它是打开的
      if (this.showGameEnd) {
        this.hideGameEnd()
      }
      
      // 先调用resetGame重置游戏状态
      if (this.gameOver) {
        this.resetGame()
      }
      
      this.countdownActive = true
      this.countdown = 3
      
      this.countdownTimer = setInterval(() => {
        this.countdown--
        if (this.countdown <= 0) {
          clearInterval(this.countdownTimer)
          this.countdownActive = false
          this.startGame()
        }
      }, 1000)
    },
    startGame() {
      if (this.gameRunning) return
      
      // 开始游戏循环
      this.gameRunning = true
      this.gameOver = false
      this.gamePaused = false
      
      // 重置游戏时间并开始计时
      this.gameTime = 0
      this.appleTimestamps = []
      this.lastAppleTime = Date.now()
      
      this.gameTimeInterval = setInterval(() => {
        if (!this.gamePaused && !this.gameOver) {
          this.gameTime++
          this.calculateTimePerApple()
        }
      }, 1000)
      
      this.gameInterval = setInterval(this.gameLoop, 400 - this.gameSpeed)
    },
    stopGame() {
      clearInterval(this.gameInterval)
      clearInterval(this.countdownTimer)
      clearInterval(this.gameTimeInterval)
      this.gameRunning = false
      this.countdownActive = false
      this.collisionDelay = false // 确保危险状态被重置
      this.clearCollisionDelay() // 清除任何活动的碰撞延迟计时器
      this.isDashing = false // 确保突进状态被重置
    },
    resetGame() {
      this.stopGame()
      this.score = 0
      this.gameOver = false
      this.gamePaused = false
      this.collisionDelay = false // 确保危险状态被重置
      this.clearCollisionDelay() // 清除任何活动的碰撞延迟计时器
      this.isDashing = false // 确保突进状态被重置
      this.resetSnake()
      this.generateApple()
    },
    gameLoop() {
      if (this.gameOver || this.gamePaused) return;
      
      // 如果在碰撞延迟状态，不移动蛇
      if (this.collisionDelay) {
        // 如果在突进状态，停止突进
        if (this.isDashing) {
          this.stopDash();
        }
        return;
      }
      
      // 更新蛇的方向
      this.direction = this.nextDirection;
      
      // 移动蛇
      this.moveSnake();
      
      // 检查碰撞
      const hasCollision = this.checkCollision();
      
      // 如果检测到碰撞并进入了碰撞延迟状态，停止突进
      if (hasCollision && this.isDashing) {
        this.stopDash();
      }
      
      // 检查是否吃到苹果
      if (this.checkAppleCollision()) {
        this.eatApple();
        // 如果吃到苹果并且在突进状态，停止突进
        if (this.isDashing) {
          this.stopDash();
        }
      } else {
        // 如果没有吃到苹果，移除蛇尾
        this.snake.pop();
      }
    },
    moveSnake() {
      // 获取蛇头坐标
      const headIndex = this.snake[0]
      const [headX, headY] = this.getCoordinates(headIndex)
      
      let newX = headX
      let newY = headY
      
      // 根据方向移动蛇头
      switch (this.direction) {
        case 'up':
          newY--
          break
        case 'down':
          newY++
          break
        case 'left':
          newX--
          break
        case 'right':
          newX++
          break
      }
      
      // 获取新位置索引
      const newIndex = this.getIndex(newX, newY)
      
      // 检查新位置是否有效
      if (newIndex !== -1) {
        // 检查是否会与蛇身碰撞
        let collideWithSelf = false
        for (let i = 1; i < this.snake.length - 1; i++) {
          if (this.snake[i] === newIndex) {
            collideWithSelf = true
            break
          }
        }
        
        if (collideWithSelf) {
          // 如果会碰撞到自身，触发碰撞延迟
          if (!this.collisionDelay) {
            this.startCollisionDelay()
          }
          // 保持原位置
          this.snake.unshift(this.snake[0])
        } else {
          // 正常移动
          this.snake.unshift(newIndex)
        }
      } else {
        // 如果新位置无效（越界），触发碰撞延迟
        if (!this.collisionDelay) {
          this.startCollisionDelay()
        }
        // 保持原位置
        this.snake.unshift(this.snake[0])
      }
    },
    checkCollision() {
      const headIndex = this.snake[0]
      
      // 如果头部索引无效（越界），开始碰撞延迟
      if (headIndex < 0 || headIndex >= this.boardSize * this.boardSize) {
        if (!this.collisionDelay) {
          this.startCollisionDelay()
        }
        return true;
      }
      
      const [headX, headY] = this.getCoordinates(headIndex)
      
      // 检查是否撞墙
      if (headX < 0 || headX >= this.boardSize || headY < 0 || headY >= this.boardSize) {
        // 如果不在碰撞延迟状态，则开始延迟
        if (!this.collisionDelay) {
          this.startCollisionDelay()
        }
        return true;
      }
      
      // 检查是否撞到自己 (不包括尾巴，因为它会在移动时消失)
      for (let i = 1; i < this.snake.length; i++) {
        if (this.snake[i] === headIndex) {
          // 如果不在碰撞延迟状态，则开始延迟
          if (!this.collisionDelay) {
            this.startCollisionDelay()
          }
          return true;
        }
      }
      
      // 如果没有碰撞，清除碰撞状态
      if (this.collisionDelay) {
        this.clearCollisionDelay()
      }
      
      return false;
    },
    checkAppleCollision() {
      return this.snake[0] === this.apple
    },
    eatApple() {
      // 记录当前时间戳
      const now = Date.now()
      this.appleTimestamps.push(now - this.lastAppleTime)
      this.lastAppleTime = now
      
      // 增加分数
      this.score++
      
      // 在竞速模式下，检查是否已达成目标
      if (this.gameMode === 'speed' && this.score >= 40) {
        this.completeSpeedRun()
        return
      }
      
      // 重新生成苹果
      this.generateApple()
    },
    generateApple() {
      let newAppleIndex
      
      do {
        // 随机生成苹果的位置
        const x = Math.floor(Math.random() * this.boardSize)
        const y = Math.floor(Math.random() * this.boardSize)
        newAppleIndex = this.getIndex(x, y)
        
        // 确保苹果不会出现在蛇的身体上
      } while (this.snake.includes(newAppleIndex))
      
      this.apple = newAppleIndex
    },
    handleKeyPress(event) {
      // 如果游戏未开始，忽略按键
      if (!this.gameRunning && !this.gameOver) return;
      
      const key = event.key.toLowerCase();
      
      // 阻止游戏控制键的默认行为
      if (['w', 'a', 's', 'd', ' ', 'arrowup', 'arrowdown', 'arrowleft', 'arrowright', 'escape'].includes(key)) {
        event.preventDefault();
      }
      
      // 处理碰撞延迟状态下的方向更改
      if (this.collisionDelay) {
        let directionChanged = false;
        
        switch(key) {
          case 'w':
          case 'arrowup':
            if (this.direction !== 'down') {
              // 获取蛇头坐标
              const headIndex = this.snake[0];
              const [headX, headY] = this.getCoordinates(headIndex);
              
              // 判断向上移动是否可行（不会撞到自身或墙壁）
              const newY = headY - 1;
              if (newY >= 0 && !this.wouldCollideWithSelf(headX, newY)) {
                this.direction = 'up';
                this.nextDirection = 'up';
                directionChanged = true;
              }
            }
            break;
          case 's':
          case 'arrowdown':
            if (this.direction !== 'up') {
              // 获取蛇头坐标
              const headIndex = this.snake[0];
              const [headX, headY] = this.getCoordinates(headIndex);
              
              // 判断向下移动是否可行
              const newY = headY + 1;
              if (newY < this.boardSize && !this.wouldCollideWithSelf(headX, newY)) {
                this.direction = 'down';
                this.nextDirection = 'down';
                directionChanged = true;
              }
            }
            break;
          case 'a':
          case 'arrowleft':
            if (this.direction !== 'right') {
              // 获取蛇头坐标
              const headIndex = this.snake[0];
              const [headX, headY] = this.getCoordinates(headIndex);
              
              // 判断向左移动是否可行
              const newX = headX - 1;
              if (newX >= 0 && !this.wouldCollideWithSelf(newX, headY)) {
                this.direction = 'left';
                this.nextDirection = 'left';
                directionChanged = true;
              }
            }
            break;
          case 'd':
          case 'arrowright':
            if (this.direction !== 'left') {
              // 获取蛇头坐标
              const headIndex = this.snake[0];
              const [headX, headY] = this.getCoordinates(headIndex);
              
              // 判断向右移动是否可行
              const newX = headX + 1;
              if (newX < this.boardSize && !this.wouldCollideWithSelf(newX, headY)) {
                this.direction = 'right';
                this.nextDirection = 'right';
                directionChanged = true;
              }
            }
            break;
        }
        
        // 如果方向成功改变，则移除碰撞状态并立即移动一步
        if (directionChanged) {
          // 先清除碰撞状态计时器
          clearTimeout(this.collisionTimer);
          // 立即移动一步 - 这会在moveSnakeOnce中完全清除碰撞状态
          this.moveSnakeOnce();
          return;
        }
      }
      
      // 非碰撞状态下的正常按键处理
      switch (key) {
        case 'w':
        case 'arrowup':
          if (this.direction !== 'down') {
            this.nextDirection = 'up';
          }
          break;
        case 's':
        case 'arrowdown':
          if (this.direction !== 'up') {
            this.nextDirection = 'down';
          }
          break;
        case 'a':
        case 'arrowleft':
          if (this.direction !== 'right') {
            this.nextDirection = 'left';
          }
          break;
        case 'd':
        case 'arrowright':
          if (this.direction !== 'left') {
            this.nextDirection = 'right';
          }
          break;
        case 'escape':
          if (!this.gameOver) {
            this.gamePaused = !this.gamePaused;
          }
          break;
        case ' ': // 空格键 - 快速突进
          if (this.gameRunning && !this.gamePaused && !this.gameOver && !this.collisionDelay) {
            if (this.isDashing) {
              this.stopDash(); // 再次按空格键停止突进
            } else {
              this.dash(); // 开始突进
            }
          }
          break;
      }
    },
    endGame() {
      this.gameRunning = false
      this.gameOver = true
      clearInterval(this.gameInterval)
      
      // 更新高分
      this.isHighScore = this.updateHighScores()
      
      // 显示游戏结束模态框
      this.showGameEndModal()
    },
    updateHighScores() {
      const score = this.score
      const time = this.gameTime
      const mode = this.gameMode
      let isHighScore = false
      
      // 根据游戏模式确定排行榜存储键
      const storageKey = `snakeHighScores_${mode}`
      
      // 获取当前高分列表
      let highScores = JSON.parse(localStorage.getItem(storageKey) || '[]')
      
      // 将当前分数添加到高分列表
      if (mode === 'classic') {
        // 经典模式记录最高分
        highScores.push({ score, time })
        // 按分数排序
        highScores.sort((a, b) => b.score - a.score)
        
        // 检查是否是新的高分（前5名）
        isHighScore = highScores.findIndex(item => item.score === score && item.time === time) < 5
      } else {
        // 竞速模式记录完成时间
        if (score >= 40) {
          highScores.push({ score, time })
          // 按时间排序（最短时间在前）
          highScores.sort((a, b) => a.time - b.time)
          
          // 检查是否是新的高分（前5名）
          isHighScore = highScores.findIndex(item => item.score === score && item.time === time) < 5
        }
      }
      
      // 只保留前5名
      highScores = highScores.slice(0, 5)
      
      // 更新组件状态
      this.highScores = highScores
      
      // 保存到本地存储
      localStorage.setItem(storageKey, JSON.stringify(highScores))
      
      return isHighScore
    },
    // 辅助方法
    getIndex(x, y) {
      // 首先检查坐标是否在边界内，如果不在则返回-1表示无效位置
      if (x < 0 || x >= this.boardSize || y < 0 || y >= this.boardSize) {
        return -1;
      }
      return y * this.boardSize + x;
    },
    getCoordinates(index) {
      // 确保索引在有效范围内
      if (index < 0 || index >= this.boardSize * this.boardSize) {
        return [-1, -1]; // 返回无效坐标
      }
      const y = Math.floor(index / this.boardSize);
      const x = index % this.boardSize;
      return [x, y];
    },
    isSnakeCell(index) {
      return this.snake.includes(index)
    },
    isSnakeHead(index) {
      return this.snake[0] === index
    },
    isAppleCell(index) {
      return this.apple === index
    },
    getCellColor(index) {
      if (this.isSnakeHead(index)) {
        // 如果在碰撞延迟状态，蛇头闪烁
        if (this.collisionDelay) {
          return this.blinkColor(this.dangerColor, this.adjustColor(this.dangerColor, 20), 200);
        }
        // 如果在突进状态，显示亮绿色
        if (this.isDashing) {
          return this.adjustColor(this.snakeColor, 40);
        }
        // 蛇头颜色稍深
        return this.adjustColor(this.snakeColor, -20);
      } else if (this.isSnakeCell(index)) {
        // 如果在碰撞延迟状态，蛇身闪烁
        if (this.collisionDelay) {
          return this.blinkColor(this.snakeColor, this.adjustColor(this.dangerColor, 30), 300);
        }
        // 如果在突进状态，显示亮色
        if (this.isDashing) {
          return this.adjustColor(this.snakeColor, 20);
        }
        return this.snakeColor;
      } else if (this.isAppleCell(index)) {
        return this.appleColor;
      }
      return 'transparent';
    },
    adjustColor(hex, amount) {
      // 调整颜色亮度的辅助函数
      let r = parseInt(hex.substring(1, 3), 16)
      let g = parseInt(hex.substring(3, 5), 16)
      let b = parseInt(hex.substring(5, 7), 16)
      
      r = Math.max(0, Math.min(255, r + amount))
      g = Math.max(0, Math.min(255, g + amount))
      b = Math.max(0, Math.min(255, b + amount))
      
      return `#${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')}`
    },
    updateSnakeColor(event) {
      this.snakeColor = event.target.value
      this.saveColorSettings()
    },
    updateAppleColor(event) {
      this.appleColor = event.target.value
      this.saveColorSettings()
    },
    updateDangerColor(event) {
      this.dangerColor = event.target.value
      this.saveColorSettings()
    },
    setGameMode(mode) {
      this.gameMode = mode
      // 切换模式时更新高分列表
      this.loadHighScores()
      this.resetGame()
    },
    calculateTimePerApple() {
      if (this.score > 0) {
        this.timePerApple = (this.gameTime / this.score).toFixed(1)
      } else {
        this.timePerApple = 0
      }
    },
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60)
      const secs = seconds % 60
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
    },
    completeSpeedRun() {
      this.gameRunning = false
      this.gameOver = true
      clearInterval(this.gameInterval)
      clearInterval(this.gameTimeInterval)
      
      // 更新高分
      this.isHighScore = this.updateHighScores()
      
      // 显示游戏结束模态框
      this.showGameEndModal()
    },
    startCollisionDelay() {
      this.collisionDelay = true;
      
      // 清除之前的计时器
      if (this.collisionTimer) {
        clearTimeout(this.collisionTimer);
      }
      
      // 设置1秒的延迟
      this.collisionTimer = setTimeout(() => {
        // 如果1秒后仍然处于碰撞状态，则游戏结束
        if (this.collisionDelay) {
          this.endGame();
        }
      }, 1000); // 延迟时间为1秒
    },
    clearCollisionDelay() {
      // 清除碰撞延迟状态
      this.collisionDelay = false;
      
      // 清除计时器
      if (this.collisionTimer) {
        clearTimeout(this.collisionTimer);
        this.collisionTimer = null;
      }
    },
    wouldCollideWithSelf(x, y) {
      const newIndex = this.getIndex(x, y);
      if (newIndex === -1) return true; // 无效位置视为碰撞
      
      // 检查蛇身（除了尾部，因为它会在移动时消失）
      for (let i = 1; i < this.snake.length - 1; i++) {
        if (this.snake[i] === newIndex) {
          return true;
        }
      }
      
      return false;
    },
    moveSnakeOnce() {
      // 获取蛇头坐标
      const headIndex = this.snake[0];
      const [headX, headY] = this.getCoordinates(headIndex);
      
      let newX = headX;
      let newY = headY;
      
      // 根据新方向移动蛇头
      switch (this.direction) {
        case 'up':
          newY--;
          break;
        case 'down':
          newY++;
          break;
        case 'left':
          newX--;
          break;
        case 'right':
          newX++;
          break;
      }
      
      // 确保新位置有效
      const newIndex = this.getIndex(newX, newY);
      if (newIndex !== -1) {
        // 检查新位置是否与蛇身碰撞
        let collisionWithSelf = false;
        for (let i = 1; i < this.snake.length - 1; i++) {
          if (this.snake[i] === newIndex) {
            collisionWithSelf = true;
            break;
          }
        }
        
        if (!collisionWithSelf) {
          // 只有在新位置不会与蛇身碰撞时才移动
          this.snake.unshift(newIndex);
          
          // 重置碰撞状态
          this.clearCollisionDelay();
          
          // 检查是否吃到苹果
          if (newIndex === this.apple) {
            this.eatApple();
          } else {
            // 如果没有吃到苹果，移除蛇尾
            this.snake.pop();
          }
        } else {
          // 如果会与蛇身碰撞，保持碰撞状态
          this.startCollisionDelay();
        }
      } else {
        // 如果新位置无效（越界），保持碰撞状态
        this.startCollisionDelay();
      }
    },
    dash() {
      // 如果已经在突进状态，忽略此次调用
      if (this.isDashing) return;
      
      // 记住当前游戏速度
      this.normalGameSpeed = this.gameSpeed;
      
      // 设置为突进状态
      this.isDashing = true;
      
      // 清除旧的游戏循环
      clearInterval(this.gameInterval);
      
      // 以极快的速度创建新的游戏循环
      this.gameInterval = setInterval(this.gameLoop, 10); // 固定突进速度
    },
    
    // 停止突进，恢复正常速度
    stopDash() {
      if (!this.isDashing) return;
      
      // 恢复正常速度
      this.isDashing = false;
      
      // 清除快速游戏循环
      clearInterval(this.gameInterval);
      
      // 恢复原来的游戏速度
      this.gameInterval = setInterval(this.gameLoop, 400 - this.normalGameSpeed);
    },
    blinkColor(color1, color2, speed) {
      return Math.floor(Date.now() / speed) % 2 === 0 ? color1 : color2
    },
    loadHighScores() {
      // 根据当前游戏模式加载对应的高分
      const storageKey = `snakeHighScores_${this.gameMode}`
      const storedScores = localStorage.getItem(storageKey)
      
      if (storedScores) {
        this.highScores = JSON.parse(storedScores)
      } else {
        // 默认空数组
        this.highScores = []
      }
    },
    applyColorPreset(preset) {
      switch (preset) {
        case 'default':
          this.snakeColor = '#4ecca3';
          this.appleColor = '#ff6b6b';
          this.dangerColor = '#d4af37';
          break;
        case 'dark':
          this.snakeColor = '#2c3e50';
          this.appleColor = '#e74c3c';
          this.dangerColor = '#f39c12';
          break;
        case 'neon':
          this.snakeColor = '#00ffaa';
          this.appleColor = '#ff00ff';
          this.dangerColor = '#ffff00';
          break;
      }
      this.saveColorSettings();
    },
    loadColorSettings() {
      // 从本地存储加载颜色设置
      const storedColors = localStorage.getItem('snakeColorSettings');
      
      if (storedColors) {
        const colors = JSON.parse(storedColors);
        this.snakeColor = colors.snakeColor || '#4ecca3';
        this.appleColor = colors.appleColor || '#ff6b6b';
        this.dangerColor = colors.dangerColor || '#d4af37';
      }
    },
    saveColorSettings() {
      // 保存颜色设置到本地存储
      const colorSettings = {
        snakeColor: this.snakeColor,
        appleColor: this.appleColor,
        dangerColor: this.dangerColor
      };
      
      localStorage.setItem('snakeColorSettings', JSON.stringify(colorSettings));
    },
    loadSpeedSettings() {
      // 从本地存储加载速度设置
      const storedSpeed = localStorage.getItem('snakeSpeedSetting');
      
      if (storedSpeed) {
        this.gameSpeed = parseInt(storedSpeed);
        this.normalGameSpeed = this.gameSpeed;
      } else {
        // 如果没有存储的速度设置，保存默认值
        this.saveSpeedSettings();
      }
    },
    saveSpeedSettings() {
      // 保存速度设置到本地存储
      localStorage.setItem('snakeSpeedSetting', this.gameSpeed.toString());
    },
    showRulesModal() {
      this.isRulesClosing = false;
      this.showRules = true;
    },
    showColorSettingsModal() {
      this.isColorSettingsClosing = false;
      this.showColorSettings = true;
    },
    hideRules() {
      // 先设置关闭动画状态
      this.isRulesClosing = true;
      
      // 动画结束后关闭模态框
      setTimeout(() => {
        this.showRules = false;
        this.isRulesClosing = false;
      }, 300); // 与CSS动画时长匹配
    },
    hideColorSettings() {
      // 先设置关闭动画状态
      this.isColorSettingsClosing = true;
      
      // 动画结束后关闭模态框
      setTimeout(() => {
        this.showColorSettings = false;
        this.isColorSettingsClosing = false;
      }, 300); // 与CSS动画时长匹配
    },
    showGameEndModal() {
      this.isGameEndClosing = false
      this.showGameEnd = true
    },
    hideGameEnd() {
      // 先设置关闭动画状态
      this.isGameEndClosing = true
      
      // 动画结束后关闭模态框
      setTimeout(() => {
        this.showGameEnd = false
        this.isGameEndClosing = false
      }, 300) // 与CSS动画时长匹配
    },
    changeGameMode() {
      // 切换游戏模式
      this.setGameMode(this.gameMode === 'classic' ? 'speed' : 'classic')
      this.hideGameEnd()
    },
    // 添加新方法 playAgain 处理再玩一次的点击事件
    playAgain() {
      // 隐藏模态框
      this.hideGameEnd()
      
      // 确保先重置游戏
      this.resetGame()
      
      // 延迟一点点再开始游戏，确保模态框完全关闭
      setTimeout(() => {
        this.startGameCountdown()
      }, 300)
    },
  },
  watch: {
    gameSpeed(newSpeed) {
      if (this.gameRunning && !this.isDashing) {
        // 只有在不是突进状态时才更新正常游戏速度
        this.normalGameSpeed = newSpeed;
        // 更新游戏速度
        clearInterval(this.gameInterval);
        this.gameInterval = setInterval(this.gameLoop, 400 - newSpeed);
      }
      // 保存速度设置
      this.saveSpeedSettings();
    }
  }
}
</script>

<style scoped>
.game-container {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  gap: 30px;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  height: calc(100vh - 120px); /* 减去头部标题的高度 */
  overflow: hidden;
  box-sizing: border-box;
  padding: 0 20px;
  position: relative;
}

.game-board-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 20px;
}

.game-controls, .game-stats {
  width: 260px;
  display: flex;
  flex-direction: column;
  gap: 15px; /* 减小间距 */
  max-height: 100%; /* 限制最大高度 */
  overflow-y: auto; /* 如果内容过多，允许侧栏滚动 */
}

/* 调整游戏数据部分的高度 */
.game-metrics, .high-scores {
  flex: 1;
  min-height: 220px;
}

.control-panel, .instructions, .high-scores, .game-tips, .game-metrics {
  background-color: rgba(30, 30, 60, 0.7);
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  margin-bottom: 10px;
}

h2, h3 {
  margin-top: 0;
  color: #4ecca3;
  font-weight: 400;
}

button {
  width: 100%;
  padding: 10px;
  margin: 5px 0;
  border: none;
  border-radius: 5px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s;
}

.start-button {
  background-color: #4ecca3;
  color: #1a1a2e;
}

.start-button:hover:not(:disabled) {
  background-color: #3dbb92;
}

.reset-button {
  background-color: #ff6b6b;
  color: white;
}

.reset-button:hover:not(:disabled) {
  background-color: #ff5252;
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.game-mode {
  margin-bottom: 15px;
}

.mode-buttons {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  margin: 10px 0;
}

.mode-button {
  flex: 1;
  background-color: #2d3748;
  color: white;
  transition: all 0.3s;
}

.mode-button.active {
  background-color: #4ecca3;
  color: #1a1a2e;
}

.mode-description {
  font-size: 0.85rem;
  color: #ddd;
  margin-top: 8px;
  min-height: 40px;
}

.speed-control, .color-control {
  margin-top: 15px;
}

.slider-container {
  display: flex;
  align-items: center;
  gap: 10px;
}

.speed-label {
  color: #ddd;
  font-size: 0.85rem;
  width: 30px;
}

.custom-slider {
  -webkit-appearance: none;
  width: 100%;
  height: 8px;
  border-radius: 4px;
  background: rgba(60, 60, 100, 0.5);
  outline: none;
  margin: 15px 0;
}

.custom-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4ecca3;
  cursor: pointer;
  box-shadow: 0 0 5px rgba(78, 204, 163, 0.5);
  transition: all 0.2s;
}

.custom-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4ecca3;
  cursor: pointer;
  box-shadow: 0 0 5px rgba(78, 204, 163, 0.5);
  transition: all 0.2s;
  border: none;
}

.custom-slider::-webkit-slider-thumb:hover {
  background: #3dbb92;
  box-shadow: 0 0 8px rgba(78, 204, 163, 0.7);
  transform: scale(1.1);
}

.custom-slider::-moz-range-thumb:hover {
  background: #3dbb92;
  box-shadow: 0 0 8px rgba(78, 204, 163, 0.7);
  transform: scale(1.1);
}

.custom-slider:disabled {
  opacity: 0.5;
}

.custom-slider:disabled::-webkit-slider-thumb {
  background: #999;
  cursor: not-allowed;
}

.custom-slider:disabled::-moz-range-thumb {
  background: #999;
  cursor: not-allowed;
}

input[type="color"] {
  width: 100%;
  height: 30px;
  border: none;
  border-radius: 5px;
  background: none;
  margin: 5px 0;
  cursor: pointer;
}

.speed-value {
  text-align: center;
  font-size: 0.9rem;
  color: #ddd;
  font-weight: bold;
}

.game-info {
  display: flex;
  justify-content: space-between;
  width: 100%;
  margin-bottom: 10px;
}

.score, .status {
  font-size: 1.2rem;
  font-weight: bold;
}

.status {
  color: #ff6b6b;
}

.collision-warning {
  color: #ff0000;
  font-weight: bold;
  animation: blink 0.3s infinite;
}

.dash-indicator {
  color: #4ecca3;
  font-weight: bold;
  animation: dash-pulse 0.2s infinite alternate;
}

@keyframes dash-pulse {
  from { opacity: 0.7; transform: scale(1); }
  to { opacity: 1; transform: scale(1.05); }
}

@keyframes blink {
  0% { opacity: 1; }
  50% { opacity: 0.3; }
  100% { opacity: 1; }
}

@keyframes pulse {
  from {
    box-shadow: 0 0 10px v-bind('adjustColor(dangerColor, 20)'), 0 0 15px v-bind('dangerColor');
    transform: scale(1);
  }
  to {
    box-shadow: 0 0 15px v-bind('adjustColor(dangerColor, 20)'), 0 0 25px v-bind('dangerColor');
    transform: scale(1.05);
  }
}

.game-board {
  display: grid;
  width: 500px;
  height: 500px;
  max-width: 90vmin; /* 响应式调整 */
  max-height: 90vmin; /* 响应式调整 */
  background-color: #232741;
  border: 2px solid #333a56;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
}

.cell {
  width: 100%;
  height: 100%;
  border: 1px solid rgba(80, 80, 120, 0.2);
  box-sizing: border-box;
  transition: background-color 0.1s;
}

.snake-cell {
  border-radius: 4px;
}

.snake-head {
  border-radius: 6px;
}

.apple-cell {
  border-radius: 50%;
  box-shadow: 0 0 5px rgba(255, 107, 107, 0.8);
}

.collision-cell {
  box-shadow: 0 0 10px v-bind('adjustColor(dangerColor, 20)'), 0 0 15px v-bind('dangerColor');
  z-index: 10;
  animation: pulse 0.3s infinite alternate;
  border: 2px solid v-bind('dangerColor') !important;
}

.dash-cell {
  box-shadow: 0 0 8px v-bind('adjustColor(snakeColor, 40)'), 0 0 12px v-bind('snakeColor');
  z-index: 5;
  animation: dash-glow 0.2s infinite alternate;
}

@keyframes dash-glow {
  from {
    box-shadow: 0 0 8px v-bind('adjustColor(snakeColor, 40)'), 0 0 12px v-bind('snakeColor');
  }
  to {
    box-shadow: 0 0 12px v-bind('adjustColor(snakeColor, 40)'), 0 0 18px v-bind('snakeColor');
  }
}

.game-metrics {
  margin-bottom: 20px;
}

.metric-item {
  display: flex;
  justify-content: space-between;
  margin: 10px 0;
  font-size: 0.95rem;
}

.metric-label {
  color: #ddd;
}

.metric-value {
  font-weight: bold;
  color: #4ecca3;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background-color: rgba(60, 60, 100, 0.5);
  border-radius: 4px;
  overflow: hidden;
  margin-top: 10px;
}

.progress {
  height: 100%;
  background-color: #4ecca3;
  transition: width 0.3s ease;
}

.high-scores {
  flex: 1;
}

.score-list {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.high-score-item {
  padding: 5px;
  background-color: rgba(40, 40, 80, 0.5);
  border-radius: 5px;
  font-size: 1rem;
  display: flex;
  justify-content: space-between;
}

.high-score-item .score-time {
  color: #4ecca3;
  font-weight: bold;
}

.game-tips p {
  margin: 5px 0;
  font-size: 0.85rem;
  color: #ddd;
  line-height: 1.3;
}

.instructions p {
  margin: 5px 0;
  font-size: 0.85rem;
  line-height: 1.3;
}

@media (max-width: 1100px) {
  .game-container {
    flex-direction: column;
    align-items: center;
  }
  
  .game-controls, .game-stats {
    flex-direction: row;
    width: 100%;
    max-width: 500px;
    margin-bottom: 20px;
  }
  
  .control-panel, .game-metrics, .high-scores {
    flex: 1;
    min-height: 0;
  }
  
  .game-board {
    width: 400px;
    height: 400px;
  }
}

@media (max-width: 500px) {
  .game-container {
    padding: 0 10px;
  }
  
  .game-controls, .game-stats {
    flex-direction: column;
  }
  
  .game-board {
    width: 300px;
    height: 300px;
  }
}

/* 模态框样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(3px);
  animation: fadeIn 0.3s ease;
}

.modal-overlay.fade-out {
  animation: fadeOut 0.3s ease forwards;
}

.modal-content {
  background-color: #1e1e3e;
  border-radius: 10px;
  width: 90%;
  max-width: 600px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 5px 30px rgba(0, 0, 0, 0.3);
  color: #fff;
  animation: slideUp 0.4s ease;
  transform-origin: bottom center;
}

.modal-content.slide-down {
  animation: slideDown 0.3s ease forwards;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to { opacity: 0; }
}

@keyframes slideUp {
  from { 
    transform: translateY(50px);
    opacity: 0; 
  }
  to { 
    transform: translateY(0);
    opacity: 1; 
  }
}

@keyframes slideDown {
  from { 
    transform: translateY(0);
    opacity: 1; 
  }
  to { 
    transform: translateY(50px);
    opacity: 0; 
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.modal-header h2 {
  margin: 0;
  color: #4ecca3;
}

.close-button {
  background: none;
  border: none;
  font-size: 24px;
  color: #fff;
  cursor: pointer;
  width: auto;
  margin: 0;
  padding: 0 10px;
}

.modal-body {
  padding: 20px;
}

.modal-body h3 {
  color: #4ecca3;
  margin-top: 20px;
  margin-bottom: 10px;
}

.modal-body ul {
  padding-left: 25px;
  margin: 10px 0;
}

.modal-body li {
  margin-bottom: 8px;
  line-height: 1.4;
  text-align: left;
}

.rules-button {
  background-color: #3b82f6;
  color: white;
  margin-top: 10px;
}

.rules-button:hover:not(:disabled) {
  background-color: #2563eb;
}

.color-settings-button {
  background-color: #8e44ad;
  color: white;
  margin-top: 15px;
}

.color-settings-button:hover:not(:disabled) {
  background-color: #9b59b6;
}

.color-settings-modal {
  width: 90%;
  max-width: 450px;
}

.color-setting-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 15px;
  padding-bottom: 15px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.color-setting-item label {
  font-size: 1rem;
  color: #fff;
  flex: 1;
}

.color-setting-item input[type="color"] {
  width: 120px;
  height: 35px;
  border-radius: 5px;
}

.color-setting-item:last-of-type {
  border-bottom: none;
}

.color-presets {
  margin-top: 25px;
  padding-top: 10px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.preset-buttons {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 10px;
}

.preset-buttons button {
  flex: 1;
  min-width: 90px;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s;
  background-color: #2d3748;
  color: white;
}

.preset-buttons button:hover:not(:disabled) {
  background-color: #4ecca3;
  color: #1a1a2e;
}

.color-setting-item h3 {
  margin-bottom: 10px;
}

.language-switcher {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
}

.language-select {
  background-color: rgba(30, 30, 60, 0.9);
  color: #4ecca3;
  padding: 8px 15px;
  border: 1px solid #4ecca3;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background-color 0.2s;
  width: auto;
  outline: none;
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  background-image: url("data:image/svg+xml;utf8,<svg fill='%234ecca3' height='24' viewBox='0 0 24 24' width='24' xmlns='http://www.w3.org/2000/svg'><path d='M7 10l5 5 5-5z'/><path d='M0 0h24v24H0z' fill='none'/></svg>");
  background-repeat: no-repeat;
  background-position: right 8px center;
  padding-right: 30px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
}

.language-select:hover {
  background-color: rgba(40, 40, 80, 0.9);
}

.language-select option {
  background-color: #1e1e3e;
  color: #4ecca3;
  padding: 10px;
  font-size: 0.95rem;
}

/* 游戏结束模态框 */
.game-end-modal {
  width: 90%;
  max-width: 500px;
}

.game-end-message {
  text-align: center;
  padding: 10px 0 25px 0;
}

.game-end-message h3 {
  font-size: 1.5rem;
  margin-bottom: 20px;
  color: #4ecca3;
}

.game-result {
  margin: 25px 0;
  background-color: rgba(40, 40, 80, 0.5);
  padding: 15px;
  border-radius: 8px;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.2);
}

.result-item {
  display: flex;
  justify-content: space-between;
  margin: 12px 0;
  font-size: 1.1rem;
}

.result-value {
  font-weight: bold;
  color: #ddd;
}

.result-value.highlight {
  color: #4ecca3;
  font-size: 1.3rem;
}

.high-score-status {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  animation: pulse 1.5s infinite alternate;
}

.high-score-icon {
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.high-score-message {
  font-size: 1.2rem;
  font-weight: bold;
  color: gold;
}

.game-end-actions {
  display: flex;
  justify-content: space-between;
  gap: 15px;
  margin-top: 20px;
}

.play-again-button, .change-mode-button {
  flex: 1;
  padding: 12px;
  font-size: 1rem;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s;
  border: none;
}

.play-again-button {
  background-color: #4ecca3;
  color: #1a1a2e;
}

.play-again-button:hover {
  background-color: #3dbb92;
}

.change-mode-button {
  background-color: #3b82f6;
  color: white;
}

.change-mode-button:hover {
  background-color: #2563eb;
}

@keyframes pulse {
  from { transform: scale(1); opacity: 0.9; }
  to { transform: scale(1.05); opacity: 1; }
}
</style> 