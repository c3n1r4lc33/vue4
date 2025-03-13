<template>
  <div class="game-container">
    <!-- Экран настроек -->
    <div v-if="screen === 'setup'" class="setup-screen">
      <h1>Настройка игры</h1>
      <div class="form-group">
        <label>Количество целей:</label>
        <input
          type="number"
          v-model.number="settings.targets"
          min="1"
          class="input-field"
        />
      </div>
      <div class="form-group">
        <label>Время игры (сек):</label>
        <input
          type="number"
          v-model.number="timeLeft"
          min="10"
          class="input-field"
        />
      </div>
      <div class="form-group">
        <label>Размер цели (пикселей):</label>
        <input
          type="number"
          v-model.number="settings.size"
          min="10"
          class="input-field"
        />
      </div>
      <button class="btn" @click="startGame">Начать игру</button>
    </div>

    <!-- Игровой экран -->
    <div v-if="screen === 'game'" class="game-screen">
      <div class="header">
        <div class="timer">Осталось времени: {{ timeLeft }}</div>
        <button class="btn secondary" @click="giveUp">Сдаться</button>
      </div>
      <div class="play-area">
        <div
          v-for="(target, index) in targets"
          :key="index"
          class="target"
          :style="{
            left: target.x + 'px',
            top: target.y + 'px',
            width: settings.size + 'px',
            height: settings.size + 'px',
          }"
          @click="hitTarget(index)"
        ></div>
      </div>
    </div>

    <!-- Экран статистики -->
    <div v-if="screen === 'stats'" class="stats-screen">
      <h1>Статистика игры</h1>
      <ul>
        <li v-for="(stat, index) in stats" :key="index">
          Целей: {{ stat.targets }}, Размер: {{ stat.size }}, Осталось времени:
          {{ stat.timeLeft }} сек.
        </li>
      </ul>
      <button class="btn" @click="resetGame">Новая игра</button>
    </div>

    <!-- Модальное окно, отображающее результат игры -->
    <div v-if="showModal" class="modal-overlay">
      <div class="modal">
        <h2>Игра окончена</h2>
        <p v-if="targets.length === 0">Поздравляем! Вы нашли все цели!</p>
        <p v-else>Время вышло! Осталось целей: {{ targets.length }}</p>
        <button class="btn" @click="closeModal">Закрыть</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      screen: "setup",
      settings: { targets: 5, speed: 1000, size: 50 },
      targets: [],
      timeLeft: 30,
      timer: null,
      stats: [],
      showModal: false,
    };
  },
  methods: {
    startGame() {
      // Инициализация целей с произвольным положением
      this.targets = Array.from({ length: this.settings.targets }, () => ({
        x: Math.random() * 300,
        y: Math.random() * 300,
      }));
      this.screen = "game";
      // Запускаем обратный отсчет с выбранным временем
      this.timer = setInterval(this.updateGame, 1000);
    },
    updateGame() {
      this.timeLeft--;
      if (this.timeLeft <= 0) {
        this.endGame();
      }
      // Перемещаем цели в случайные позиции
      this.targets.forEach((target) => {
        target.x = Math.random() * 300;
        target.y = Math.random() * 300;
      });
    },
    hitTarget(index) {
      this.targets.splice(index, 1);
      // За каждую поразившую цель добавляем 2 секунды
      this.timeLeft += 2;
      if (this.targets.length === 0) {
        this.endGame();
      }
    },
    endGame() {
      clearInterval(this.timer);
      // Сохраняем статистику игры (на случай многократных игр)
      this.stats.push({ ...this.settings, timeLeft: this.timeLeft });
      // Показываем модальное окно с результатом игры
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
      this.screen = "stats";
    },
    giveUp() {
      this.endGame();
    },
    resetGame() {
      // Возвращаемся к экрану настроек и сбрасываем значения
      this.screen = "setup";
      this.timeLeft = 30;
      this.targets = [];
    },
  },
};
</script>

<style scoped>
/* Общие стили */
.game-container {
  max-width: 600px;
  margin: 40px auto;
  padding: 20px;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  background: #f0f2f5;
}

/* Экран настроек, игры и статистики */
.setup-screen,
.game-screen,
.stats-screen {
  background: #ffffff;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.setup-screen h1,
.stats-screen h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

/* Форма настроек */
.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  font-weight: 600;
  margin-bottom: 5px;
  color: #555;
}

.input-field {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border: 2px solid #ddd;
  border-radius: 6px;
  transition: border-color 0.3s ease;
}

.input-field:focus {
  outline: none;
  border-color: #6a11cb;
}

/* Кнопки */
.btn {
  background: linear-gradient(45deg, #6a11cb, #2575fc);
  color: #fff;
  border: none;
  padding: 12px 25px;
  font-size: 16px;
  border-radius: 30px;
  cursor: pointer;
  transition: background 0.3s ease, transform 0.2s ease;
  display: block;
  margin: 0 auto;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.btn:hover {
  background: linear-gradient(45deg, #2575fc, #6a11cb);
  transform: translateY(-2px);
}

.btn.secondary {
  background: #ff4b2b;
}

.btn.secondary:hover {
  background: #ff416c;
}

/* Игровой экран */
.game-screen .header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  color: #333;
}

.timer {
  font-weight: 600;
  font-size: 18px;
}

.play-area {
  position: relative;
  width: 100%;
  height: 300px;
  border: 2px dashed #ccc;
  border-radius: 10px;
  background: #fafafa;
  overflow: hidden;
}

/* Цели */
.target {
  position: absolute;
  background: #ffcc00;
  border: 2px solid #e6b800;
  border-radius: 50%;
  cursor: pointer;
  transition: transform 0.2s ease, background 0.2s ease;
}

.target:hover {
  transform: scale(1.1);
  background: #ffc107;
}

/* Модальное окно */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: #fff;
  padding: 30px;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
}

.modal h2 {
  margin-bottom: 20px;
  color: #333;
}

.stats-screen ul {
  list-style-type: none;
  padding: 0;
}

.stats-screen li {
  background: #f7f7f7;
  padding: 10px;
  margin-bottom: 8px;
  border-radius: 6px;
  color: #555;
}
</style>
