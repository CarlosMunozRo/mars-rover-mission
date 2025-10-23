<script setup lang="ts">
  import { ref } from 'vue';

  const REGEX_COMMANDS = /^[FLR]+$/i;

  const DIMENSIONS = {
    x: 10,
    y: 10,
  };

  var roverPosition = ref({
    x: clamp(10, 1, DIMENSIONS.x),
    y: clamp(5, 1, DIMENSIONS.y),
  });

  const OBSTACLES = [
    { x: 3, y: 4 },
    { x: 7, y: 8 },
    { x: 6, y: 2 },
  ];

  var loading = ref(false);

  var roverDirection = ref('N');

  var commandText = ref("");

  var commandLog = ref<string[]>([]);
  
  var shouldContinueSequence = ref(true);
  var currentTimeouts: number[] = [];
  
  function sendCommand(command: string) {

    if (!REGEX_COMMANDS.test(command)) {
      alert("Comando inválido. Solo se aceptan los siguientes comandos F, L, R.");
      commandText.value = "";
      return;
    }

    currentTimeouts.forEach(timeout => clearTimeout(timeout));
    currentTimeouts = [];
    
    commandLog.value.push(command);
    commandText.value = "";
    loading.value = true;
    shouldContinueSequence.value = true;

    for (let i = 0; i < command.length; i++) {
      const char = command[i];
      if (!char) return;
      
      const timeoutId = setTimeout(() => {
        if (shouldContinueSequence.value) {
          const collisionDetected = moveRover(char);
          if (collisionDetected) {
            shouldContinueSequence.value = false;
            loading.value = false;
          }
        }
      }, i * 500);
      
      currentTimeouts.push(timeoutId);
    }

    setTimeout(() => {
      loading.value = false;
    }, command.length * 500);
  }

  const DIRECTION_MOVES = {
    'N': { axis: 'x' as const, delta: -1 },
    'S': { axis: 'x' as const, delta: 1 },
    'E': { axis: 'y' as const, delta: 1 },
    'W': { axis: 'y' as const, delta: -1 }
  };

  const ROTATION_MAP = {
    'L': { 'N': 'W', 'W': 'S', 'S': 'E', 'E': 'N' },
    'R': { 'N': 'E', 'E': 'S', 'S': 'W', 'W': 'N' }
  } as const;

  function clamp(value: number, min: number, max: number): number {
    return Math.min(Math.max(value, min), max);
  }

  function isObstacle(x: number, y: number): boolean {
    return OBSTACLES.some(obstacle => obstacle.x === x && obstacle.y === y);
  }

  function moveRover(direction: string): boolean {
    const cmd = direction.toUpperCase();
    
    switch (cmd) {
      case 'F':
      case 'B': {
        const move = DIRECTION_MOVES[roverDirection.value as keyof typeof DIRECTION_MOVES];
        if (move) {
          const maxValue = move.axis === 'x' ? DIMENSIONS.x : DIMENSIONS.y;
          const delta = cmd === 'F' ? move.delta : -move.delta;
          
          const newValue = clamp(
            roverPosition.value[move.axis] + delta, 
            1, 
            maxValue
          );
          
          const newPosition = { ...roverPosition.value };
          newPosition[move.axis] = newValue;
          
          if (isObstacle(newPosition.x, newPosition.y)) {
            alert(`Secuencia abortada. Obstáculo detectado en {${newPosition.x}, ${newPosition.y}}`);
            return true;
          }
          
          roverPosition.value[move.axis] = newValue;
        }
        break;
      }
      case 'L':
      case 'R': {
        const rotations = ROTATION_MAP[cmd];
        const newDirection = rotations[roverDirection.value as keyof typeof rotations];
        if (newDirection) {
          roverDirection.value = newDirection;
        }
        break;
      }
    }
    return false;
  }

</script>

<template>
  <div class="mission-wrapper">
    <div class="mars">
      <div v-for="dim_x in DIMENSIONS.x" class="mars-row">
        <div v-for="dim_y in DIMENSIONS.y" class="mars-cell" :key="`cell-${dim_x}-${dim_y}`">

          <span v-if="roverPosition.x === dim_x && roverPosition.y === dim_y" :class="`rover-direction-${roverDirection.toLowerCase()}`">🚀</span>

          <span v-else-if="isObstacle(dim_x, dim_y)" class="obstacle">🪨</span>
        </div>
      </div>
    </div>
    <div class="console-center">
      <div class="console-log-wrapper">
        <div class="console-log-title">Historial de comandos</div>
        <div class="console-log">
          <div v-for="logEntry in commandLog" class="console-log-entry">
            {{ logEntry.toUpperCase() }}
          </div>
        </div>
      </div>
      <form class="console-input" @submit.prevent="sendCommand(commandText)" :class="{ 'console-input--disabled': loading }">
        <input v-model="commandText" placeholder="FFLFRF"/>
        <button type="submit"><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#e3e3e3"><path d="M120-160v-640l760 320-760 320Zm80-120 474-200-474-200v140l240 60-240 60v140Zm0 0v-400 400Z"/></svg></button>
      </form>
    </div>
  </div>
</template>

<style lang ="scss">

  body {
    margin: 0;
    padding: 0;
    // background-color: red;
    background-color: #282828;
    font-family: system-ui, sans-serif;
  }

  .mission-wrapper {
    display: flex;
    gap: 20px;    
    height: 100vh;
  }

  .mars {
    position: relative;
    height: fit-content;
    flex: 1;
    display: flex;
    flex-direction: column;
    aspect-ratio: 1 / 1;
    border: 1px solid white;
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 0 20px rgba(0,0,0,0.5);
    margin-left: 20px;
    margin-top: 20px;
    max-height: 80vh;

    .mars-row {
      display: flex;
      align-items: stretch;
      flex: 1;

      &:first-child {
        .mars-cell {
          border-top: none;
        }
      }

      &:last-child {
        .mars-cell {
          border-bottom: none;
        }
      }

      .mars-cell {
        position: relative;
        border: 1px solid white;
        background-color: #4e4e4e;
        flex: 1;

        &:first-child {
          border-left: none;
        }

        &:last-child {
          border-right: none;
        }

        >span {
          position: absolute;
          left: 50%;
          top: 50%;
          translate: -50% -50%;
          font-size: 2.5rem;
          z-index: 1;
          filter: drop-shadow(2px 2px 4px rgba(0,0,0,0.5));


          &:before {
            content: '';
            position: absolute;
            left: 10%;
            top: 0%;
            translate: -50% -50%;
            height:0;
            width:0;
            border-bottom:10px solid yellow;
            border-left:10px solid transparent;
            border-right:10px solid transparent;
          }

          &.rover-direction-n:before {
            rotate: 0deg;
          }

          &.rover-direction-w:before {
            rotate: -90deg;
          }

          &.rover-direction-e:before {
            rotate: 90deg;
          }

          &.rover-direction-s:before {
            rotate: 180deg;
          }

          &.obstacle:before {
            display: none;
          }

        }
      }
    }
  }

  .console-center {
    display: flex;
    flex:1;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: #1f1f1f;
    border-left: 2px solid white;
    min-width: 100px;
    max-width: 400px;

    .console-log-wrapper {
      flex: 1;
      width: 100%;
      overflow: auto;
    }

    .console-log-title {
      color: white;
      font-size: 1.2rem;
      font-weight: bold;
      padding: 10px 20px;
      border-bottom: 1px solid #4e4e4e;
      text-align: center;
    }

    .console-log {
      color: white;

      .console-log-entry {
        padding: 10px 20px;
        border-bottom: 1px solid #4e4e4e;
      }
    }

    .console-input {
      width: calc(100% - 20px);
      padding: 20px 10px;
      display: flex;
      gap: 10px;      

      &--disabled {
        opacity: 0.5;
        pointer-events: none;
      }

      input {
        width: 100%;
        outline: none;
        border: none;
        margin: 0;
        padding: 10px 10px;
        border-radius: 5px;
        background-color: #4e4e4e;
        color: white;
      }

      button {
        background-color: transparent;
        border: none;
        cursor: pointer;
      }
    }
  }
</style>
