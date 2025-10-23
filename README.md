# Mars Rover Mission

This project simulates a Mars rover mission using Vue.js. It provides an interactive interface to control and monitor the rover's activities on the Martian surface.

## Project Setup

### Install the dependencies

```sh
npm install
```

### Run the development server

```sh
npm run dev
```

## Features

- Interactive Mars rover control panel with real-time visualization
- Command sequence execution with visual feedback
- Obstacle detection and collision avoidance
- Command history logging
- Responsive grid-based Mars surface representation

## Controls

Send commands to the rover using the following characters:
- **F**: Move forward
- **L**: Turn left (90°)
- **R**: Turn right (90°)

Example commands: `FFLR`, `FFFRRR`, `LLLF`

## Variables

```ts
// This variable defines the size of the Mars surface grid. It is set to 10, creating a 10x10 grid for the rover to navigate.
const DIMENSIONS = {
    x: 10,
    y: 10,
};
```
---
```ts
// Is an object that defines how the rover's orientation changes when it turns left or right.
const ROTATION_MAP = {
    'L': { 'N': 'W', 'W': 'S', 'S': 'E', 'E': 'N' },
    'R': { 'N': 'E', 'E': 'S', 'S': 'W', 'W': 'N' }
};
```
---
```ts
// This array holds the coordinates of obstacles on the Mars surface grid.
const OBSTACLES = [
    { x: 3, y: 4 },
    { x: 7, y: 8 },
    { x: 6, y: 2 },
];
```
---
```ts
// This object tracks the current position and orientation of the rover.
var roverPosition = ref({
    x: 10,
    y: 5,
});
```
---
```ts
// This variable holds the current direction the rover is facing (N, E, S, W).
var roverDirection = ref('N');
```

## Functions

### isObstacle
This function checks if there is an obstacle at a given coordinate on the Mars surface grid. It takes the x and y coordinates as parameters and returns a boolean indicating whether an obstacle is present.

### clamp
This function ensures that a given value stays within specified minimum and maximum bounds. It takes three parameters: the value to be clamped, the minimum bound, and the maximum bound. The function returns the clamped value.

### moveRover
This function handles the movement of the rover based on the provided command sequence. It processes each command, updates the rover's position and orientation, checks for obstacles, and logs the command history. The function takes a string of commands as input and executes them sequentially.

## Technologies Used
- **Vue.js 3**
- **TypeScript**
- **SCSS**