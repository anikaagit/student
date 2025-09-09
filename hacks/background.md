---
layout: default 
title: Background with Object
description: Use JavaScript to have an in motion background.
sprite: images/platformer/sprites/frog-lilypad_1308-24303.png
background: images/platformer/backgrounds/pond.jpg
permalink: /background
---

<!-- The canvas element where all the drawing happens -->
<canvas id="world"></canvas> 

<script>
   /* ===============================
     CANVAS + IMAGE SETUP SECTION
     ===============================
     - We grab the <canvas> element
     - Create a 2D drawing context
     - Load two images (background + UFO)
  */
  const canvas = document.getElementById("world"); 
  const ctx = canvas.getContext('2d');
  const backgroundImg = new Image(); 
  const spriteImg = new Image();  
  backgroundImg.src = '{{page.background}}';
  spriteImg.src = '{{page.sprite}}'; //jekyll code 
   
  /* ===============================
     IMAGE LOADING CONTROL
     ===============================
     - We must wait until both images are
       fully loaded before starting the game.
     - imagesLoaded counts how many are ready.
  */

  let imagesLoaded = 0; 
  backgroundImg.onload = function() { //
    imagesLoaded++;
    startGameWorld(); // Attempt to start game
  };
  spriteImg.onload = function() {
    imagesLoaded++;
    startGameWorld(); // Attempt to start game
  };

 /* ===============================
     MAIN GAME INITIALIZER
     ===============================
     - startGameWorld() only runs once both
       background and sprite images are loaded.
     - It defines our classes and starts the loop.
  */

  function startGameWorld() { //game world won't start until everything is loaded 
    if (imagesLoaded < 2) return;
     /* -------------------------------
       BASE CLASS: GameObject
       -------------------------------
       - All other objects inherit this.
       - Stores position, size, image, speed.
       - Has default draw() method.
    */
    class GameObject {
      constructor(image, width, height, x = 0, y = 0, speedRatio = 0) {
        this.image = image;
        this.width = width;
        this.height = height;
        this.x = x;
        this.y = y;
        this.speedRatio = speedRatio;
        this.speed = GameWorld.gameSpeed * this.speedRatio;
      }
      update() {} // Placeholder, overridden by subclasses
      draw(ctx) {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
      }
    }

     /* -------------------------------
       BACKGROUND CLASS
       -------------------------------
       - Inherits from GameObject
       - Scrolls infinitely to the left
         to create a looping effect
       - Uses modulus to wrap position
    */

    class Background extends GameObject { //background IS a gameobject
      constructor(image, gameWorld) {
        // Fill entire canvas
        super(image, gameWorld.width, gameWorld.height, 0, 0, 0.1);
      }
      update() {
        this.x = (this.x - this.speed) % this.width;
      }
      draw(ctx) {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
        ctx.drawImage(this.image, this.x + this.width, this.y, this.width, this.height);
      }
    }

    /* -------------------------------
       PLAYER CLASS (UFO)
       -------------------------------
       - Shrinks sprite to half size
       - Centers UFO in the screen
       - Animates up and down using sine wave
    */

    class Player extends GameObject { //player object (the UFO)
      constructor(image, gameWorld) {
        const width = image.naturalWidth / 2;
        const height = image.naturalHeight / 2;
        const x = (gameWorld.width - width) / 2;
        const y = (gameWorld.height - height) / 2;
        super(image, width, height, x, y);
        this.baseY = y;
        this.frame = 0;
      }
      update() {
        this.y = this.baseY + Math.sin(this.frame * 0.05) * 20;
        this.frame++;
      }
    }

     /* -------------------------------
       GAME WORLD CLASS
       -------------------------------
       - Controls the entire canvas
       - Holds background + player objects
       - Runs the main game loop
    */

    class GameWorld { //creating a game world!!
      static gameSpeed = 5; // Shared speed for all objects
      constructor(backgroundImg, spriteImg) {
        this.canvas = document.getElementById("world");
        this.ctx = this.canvas.getContext('2d');
        this.width = window.innerWidth;   // Resize canvas to match window
        this.height = window.innerHeight;
        this.canvas.width = this.width;
        this.canvas.height = this.height;
        this.canvas.style.width = `${this.width}px`;  // Style canvas for fullscreen placement
        this.canvas.style.height = `${this.height}px`;
        this.canvas.style.position = 'absolute';
        this.canvas.style.left = `0px`;
        this.canvas.style.top = `${(window.innerHeight - this.height) / 2}px`;

          // Add objects (background + player UFO)
        this.objects = [
         new Background(backgroundImg, this),
         new Player(spriteImg, this)
        ];
      }

         /* -------------------------------
         GAME LOOP
         -------------------------------
         - Clears the canvas
         - Updates and redraws all objects
         - Uses requestAnimationFrame
           for smooth 60fps animation
      */

      gameLoop() {
        this.ctx.clearRect(0, 0, this.width, this.height);
        for (const obj of this.objects) {
          obj.update(); //isa
          obj.draw(this.ctx); //hasa
        }
        requestAnimationFrame(this.gameLoop.bind(this));
      }
      start() {
        this.gameLoop(); // Begin looping
      }
    }
      
      /* ===============================
       CREATE + START GAME INSTANCE
       =============================== */
    

    const world = new GameWorld(backgroundImg, spriteImg); //gameworld object
    world.start();
  }
