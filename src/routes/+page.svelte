<script lang="ts">
  import "$lib/main.css";
  import { onMount } from "svelte";

  type Pos = [number, number];

  const board_w = 40;
  const board_h = 30;
  const initial_dir = "right";

  let el_board: HTMLDivElement;
  let current_dir: string | null = initial_dir;
  let interval: ReturnType<typeof setInterval>;

  const empty_board: Array<string> = new Array(board_w * board_h).fill("<div>·</div>");
  let board: Array<string> = $state(empty_board);
  let score: number = $state(0);

  const dir: Record<string, Pos> = {
    up: [0, -1],
    right: [1, 0],
    down: [0, 1],
    left: [-1, 0],
  };

  const cx = Math.floor(board_w / 2);
  const cy = Math.floor(board_h / 2);

  let snake: Array<Pos> = $state([
    [cx, cy],
    [cx - 1, cy],
    [cx - 2, cy],
    [cx - 3, cy],
  ]);

  let fruit: Pos = [Math.floor(board_w * Math.random()), Math.floor(board_h * Math.random())];

  onMount(() => {
    el_board.style.width = `${board_w * 1}rem`;
    el_board.style.height = `${board_h * 1}rem`;

    interval = setInterval(() => {
      move();
    }, 50);
  });

  function move() {
    // Calcule la nouvelle position du serpent.
    snake.unshift(new_pos(snake[0], dir[current_dir || initial_dir]));
    snake.pop();
    snake[0] = [(snake[0][0] + board_w) % board_w, (snake[0][1] + board_h) % board_h];

    // Test de collision de la tête du serpent avec le corps.
    for (let i = 1; i < snake.length; i++) {
      if (snake[0][0] === snake[i][0] && snake[0][1] === snake[i][1]) {
        stop();
        alert("Body collision");
      }
    }

    // Test : le serpent mange le fruit ?
    if (snake[0][0] === fruit[0] && snake[0][1] === fruit[1]) {
      score = score + 5;
      // Détermine la direction (vecteur) dans laquelle la queue du serpent doit s'allonger, et effectue l'allongement.
      let tail_grow_dir = [
        snake[snake.length - 1][0] - snake[snake.length - 2][0],
        snake[snake.length - 1][1] - snake[snake.length - 2][1],
      ];
      for (let i = 0; i < 5; i++) {
        snake.push([
          snake[snake.length - 1][0] + tail_grow_dir[0],
          snake[snake.length - 1][1] + tail_grow_dir[1],
        ]);
      }
      // On place un nouveau fruit.
      fruit = [Math.floor(board_w * Math.random()), Math.floor(board_h * Math.random())];
    }

    //--- Dessin du plateau ---
    board = empty_board;
    board[pos_to_i(fruit)] = `<div class="f">◆</div>`;
    for (let i = 0; i < snake.length; i++) {
      board[pos_to_i(snake[i])] = `<div class="s">⬤</div>`;
    }

    function stop() {
      clearInterval(interval);
    }

    function new_pos(s1: Pos, s2: Pos): Pos {
      return [s1[0] + s2[0], s1[1] + s2[1]];
    }

    function pos_to_i(pos: Pos) {
      return pos[1] * board_w + pos[0];
    }
  }

  function keypress(e: KeyboardEvent) {
    if (e.key === "ArrowUp" && current_dir !== "down") current_dir = "up";
    if (e.key === "ArrowRight" && current_dir !== "left") current_dir = "right";
    if (e.key === "ArrowDown" && current_dir !== "up") current_dir = "down";
    if (e.key === "ArrowLeft" && current_dir !== "right") current_dir = "left";
  }
</script>

<svelte:head>
  <title>Snake</title>
</svelte:head>

<svelte:window onkeydown={keypress} />

<div class="game-container">
  <div bind:this={el_board} class="board">{@html board.join("")}</div>
  <div class="score">{score}</div>
</div>
