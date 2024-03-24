    <script>
        import { wordList } from '../lib/app.js';
        import { onMount } from 'svelte';
        import { fly, fade, slide } from 'svelte/transition';
        import { quintOut } from 'svelte/easing';
        import Two from 'two.js';
        import '../app.css'

        let playerInput = ""; // bound to player input
        let word = " "; // the korean word shown to the player
        let englishWord = "" // the english word associated with the korean word
        let wordSpliced = []; // contains all generated words split into blocks, used to make the word blocks fly across the screen when a game ends
        let learnedVocabulary = []; // contains all the words the player has learned, is shown when a game ends
        let difficulty = 0.05; // rate at which timer decreases

        let inputField; // html element reference
        let duration = null; // used in the timer to display the # of periods
        let timer = null; // seconds
        let timeLeft = 100; // a percentage
        let gameStarted = false;
        let gameOver = false;
        let countdown;
        let timerBar = "........"
        let score = 0;

        const handleInput = (event) => {}
        const styles = {
        size: 128,
        weight: 900
        };

        const restartGame = () => {
            playerInput = "";
            word = " ";
            wordSpliced = [];
            inputField;
            duration = null;
            timer = null; // seconds
            timeLeft = 100; // a percentage
            gameStarted = false;
            gameOver = false;
            countdown = null;
            score = 0;
            englishWord = "";
            learnedVocabulary = [];
        }

        const generateWord = () => {
            return wordList[Math.floor(Math.random() * wordList.length)];
        }

        const updateTimerBar = () => {
            const percentage = (timer / duration) * 100; // Calculate the remaining percentage
            const numDots = Math.floor(percentage / 10); // Calculate the number of dots based on the remaining percentage
            timerBar = ".".repeat(numDots < 0 ? 0 : numDots); // Generate the timer bar string
        }

        $: {
            // isComposing <- A lot of IMEs for non-latin languages have this property, and it's sort of buggy so I had to do this terribleness
            if (playerInput === word) {
                setTimeout(() => {
                    gameStarted = true;
                    let split = word.split("");
                    for (let block of split) {
                        wordSpliced.push(block);
                    }
                    playerInput = "";
                    let englishAndKorean = generateWord();
                    learnedVocabulary.push(englishAndKorean);
                    word = englishAndKorean["kr"];
                    englishWord = englishAndKorean["en"];
                    // inputField.blur();
                    inputField.focus();
                    resetTimer();
                    score++;
                }, 1);

                setTimeout(() => {
                    playerInput = "";
                }, 2);
            } 
        }

        const resetTimer = () => {  
            clearInterval(countdown);
            duration = (word.length * 0.65) + 1.5;
            console.log(`dur: ${duration}, len:${word.length}`)
            timer = duration;
            countdown = setInterval(() => {
                if (timer > 0) {
                    timer-=difficulty;
                    updateTimerBar();
                } else {
                    clearInterval(countdown);
                    gameOver = true;
                    renderWordBlocks();
                }
            }, 100);
        }

        let engine;
        let render;
        let two;
        let characters;

        onMount(() => {
            inputField.focus();
            // Initialize Matter.js engin
            two = new Two({
                type: Two.Types.svg,
                fullscreen: true,
                autostart: true
                }).appendTo(document.body);
            two.renderer.domElement.style.background = '#e8e1da';
            // Ensure Two.js canvas is positioned absolutely
            two.renderer.domElement.style.position = 'absolute';
            two.renderer.domElement.style.left = '0';
            two.renderer.domElement.style.top = '0';

            // Adjust z-index to make sure it's behind other elements
            two.renderer.domElement.style.zIndex = '-1';

            characters = [];

// Set the gravity of the world.
// Try changing the x and y components
// to change the "falling" animation
const gravity = new Two.Vector(0, 0.16);




// Add callbacks to events that Two.js triggers.
two.bind('resize', resize);
two.bind('update', update);


// Set the position of the directions
// to the center of the page.
function resize() {
  
}

// An update callback that is called every time
// the scene is rendered. Think of this as an
// animation loop.
function update() {

  // Iterate through all letters in the `characters`
  // array, apply gravity to their velocity, and
  // update their respective position.
  for (let i = 0; i < characters.length; i++) {

    const text = characters[i];
    
    text.translation.addSelf(text.velocity);
    text.rotation += text.velocity.r;
    text.velocity.add(gravity);

    // If the text's velocity is greater than 0
    // and the position is off the page, then
    // remove the text.
    if (text.velocity.y > 200 && text.translation.y > two.height) {
      two.scene.remove(text);
      two.release(text);
      characters.splice(i, 1);
    }
  }
}
});

// Utility function to render word blocks every 50ms
function renderWordBlocks() {
    let index = 0; // Initialize index counter
  
    const renderInterval = setInterval(() => {
      // Check if all word blocks have been rendered
      if (index >= wordSpliced.length) {
        clearInterval(renderInterval); // Stop the rendering interval
        return;
      }
  
      const block = wordSpliced[index]; // Get the current word block
      console.log(two)
      const x = (Math.random() * two.width) / 2 + two.width / 4;
      const y = two.height * 1.25;
  
      const text = two.makeText(block, x, y, styles); // Create text object
      text.size = 128;
  
      text.velocity = new Two.Vector();
      text.velocity.x = 10 * (Math.random() - 0.5);
      text.velocity.y = -(8 * Math.random() + 10);
      text.velocity.r = (Math.random() * Math.PI) / 200 * (Math.random() > 0.5 ? 1 : -1);
  
      characters.push(text); // Add text object to characters array
      
  
      index++; // Increment index counter
    }, 200);
}


    </script>

    <svelte:window on:keydown={handleInput}/>
    <div class="container">
        {#if !gameStarted}
            <h1>Koretype</h1>
        {/if}
        {#if gameStarted}
            {#key timerBar}
            <p id="bar" class={timerBar.length <= 3 ? "red" : ""}>{timerBar.length === 0 ? "!" : timerBar}</p>
            {/key}
            {#key word}
            <p id="word">{word}</p>
            <p id="english">{englishWord}</p>
            {/key}
        {/if}
        
        {#if !gameOver}
        <div id="inputs">
            <input id="player-input" type="text" placeholder={gameStarted ? "" : "Press space here to start."} bind:value={playerInput} bind:this={inputField}/>
            {#if !gameStarted}
            <label for="difficulty">Difficulty</label>
            <input name="difficulty" type="text" placeholder="Default: 0.05" bind:value={difficulty}/>
            {/if}
        </div>
            {/if}
        
        {#if gameOver}
            <div class="game-over">
                <p id="game-over-text">Game Over</p>
                <p id="score">Score: {score - 1}</p>
                <button on:click={restartGame}>Restart</button>
                {#if learnedVocabulary.length > 0}
                <div>
                    {#each learnedVocabulary as word}
                        <p class="learned">{word["kr"]} - {word["en"]}</p>
                    {/each}
                </div>
                {/if}
            </div>
        {/if}
    </div>

    <style>
        h1 {
            margin-top: 3rem;
            font-size: 4rem;
        }
        * {
            text-align: center;
        }
        #player-input {
            margin-top: 2.5rem;
        }

        #inputs {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #english {
            font-size: 1.5rem;
        }
        #score {
            font-size: 3rem;    
            margin-top: -1rem;
        }
        /* .container {
            display: flex;
            flex-direction: column;
        } */

        #word {
            font-size: 8rem;
            font-weight: 900;
            animation: fade-in 0.5s;
        }
        #bar {
            font-size: 5rem;
            animation: fade-in-smoother  0.25s;
        }
        
        .red {
            color: rgb(209, 31, 0)        }

        @keyframes fade-in-smoother {
            0% {
                opacity: 0.5;
                transform: translateY(-5px);
            }
            100% {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @keyframes fade-in {
            0% {
                opacity: 0;
                transform: translateY(-20px);
            }
            100% {
                opacity: 1;
                transform: translateY(0);
            }
        }

        #matter-container {
            float: right;
            width: 100%;
        }

        .bar {
            height: 100%; /* Full height */
            height: 4rem;
            background-color: #4caf50; /* Green color */
            transition: width 0.1s linear; /* Smooth width transition */
        }

        input {
            text-align: center;
            margin-bottom: 2rem;
        }

        #game-over-text {
            font-size: 5rem;
        }

        .learned {
            font-size: 1rem;
            color: rgb(201, 205, 209)        }

        .game-over * {
            animation: fade-in 0.5s;
        }

        .game-over button {
            border: 1px solid #222222;
  border-radius: 8px;
  box-sizing: border-box;
  color: #222222;
  cursor: pointer;
  display: inline-block;

  font-size: 16px;
  font-weight: 600;
  line-height: 20px;
  margin: 0 0 1rem 0;
  outline: none;
  padding: 13px 23px;
  position: relative;
  text-align: center;
  text-decoration: none;
  touch-action: manipulation;
  background-color: white;
}

.game-over button:focus {
  
}

.game-over button:hover {
  
  transform: translateY(-2px);
}

.game-over button:active {
  
  transform: translateY(2px);
}

        input {
                border: 2px solid #5551ff;
                border-radius: 8px;
                height: 50px;
                line-height: normal;
                color: #282828;
                /* display: block; */
                /* width: 100%; */
                box-sizing: border-box;
                user-select: auto;
                font-size: 16px;
                padding: 0 6px;
                padding-left: 12px;
        }

        input:focus {
            border: 1px solid #5551ff;
        }
    </style>