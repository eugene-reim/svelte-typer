<script lang="ts">
    import {onMount} from 'svelte'
    import {tweened} from 'svelte/motion'
    import {blur} from 'svelte/transition'
    type Game = 'waiting for input' | 'in progress' | 'game over';
    type Word = string;

    let game: Game = 'waiting for input'
    let typedLetter = ''

    let words: Word[] = []

    let wordIndex = 0
    let letterIndex = 0
    let correctLetters = 0
    let typedLeters = 0
    let seconds = 30
    let toggleReset = false


    let wordsPerMinute = tweened(0, { delay: 300, duration: 1000})
    let accuracy = tweened(0, { delay: 1300, duration: 1000})


    let wordsEl: HTMLDivElement
    let letterEl: HTMLSpanElement
    let inputEl: HTMLInputElement
    let caretEl: HTMLDivElement

    function resetGame(){
        toggleReset = !toggleReset

        setGameState('waiting for input')
        getWords(100)
        setTimeout(focusInput, 500)
        seconds = 30
        typedLetter = ''
        wordIndex = 0
        letterIndex = 0
        correctLetters = 0
        typedLeters = 0

        $wordsPerMinute = 0
        $accuracy = 0
    }

    function updateGameState() {
        setLetter()
        checkLetter()
        nextLetter()
        resetLetter()
        updateLine()
        moveCaret()
    }

    function getWordsPerMinute(){
        const word = 5
        const minutes = 0.5
        return Math.floor(correctLetters / word / minutes)
    }

    function getResults(){
        $wordsPerMinute = getWordsPerMinute()
        $accuracy = getAccuracy()
    }

    function getAccuracy(){
        return Math.floor((correctLetters/typedLeters)*100)
    }

    function setLetter() {
        const isWordCompleted = letterIndex > words[wordIndex].length - 1

        if (!isWordCompleted) {
            letterEl = wordsEl.children[wordIndex].children[letterIndex] as HTMLSpanElement
        }
    }

    function checkLetter() {
        const currentLetter = words[wordIndex][letterIndex]
            typedLeters += 1
            if (typedLetter === currentLetter) {
                letterEl.dataset.letter = 'correct'
                increaseScore()
            }
            
            if (typedLetter !== currentLetter) {
                letterEl.dataset.letter = 'incorrect'
            }
    }

    function updateLine(){
        const wordEl = wordsEl.children[wordIndex]
        const wordsY = wordsEl.getBoundingClientRect().y
        const wordY = wordEl.getBoundingClientRect().y

        if (wordY > wordsY) {
            wordsEl.scrollIntoView({block: 'center'})
        }
    }

    function increaseScore(){
        correctLetters += 1
    }

    function nextLetter(){
        letterIndex += 1
    }

    function resetLetter(){
        typedLetter = ''
    }

    function nextWord(){
        const isNotFirstLetter = letterIndex !== 0
        const isOneLetterWord = words[wordIndex].length === 1

        if (isNotFirstLetter || isOneLetterWord) {
            wordIndex += 1
            letterIndex = 0
            moveCaret()
        }
    }
    
    function moveCaret(){
        const offset = 4
        caretEl.style.top = `${letterEl.offsetTop + offset}px`
        caretEl.style.left = `${letterEl.offsetLeft + letterEl.offsetWidth}px`
    }

    function startGame() {
        setGameState('in progress')
        setGameTimer()
    }

    function setGameState(state: Game){
        game = state
    }

    function setGameTimer(){
        function gameTimer(){
            if (seconds > 0){
                seconds -= 1
            }
            if (game === 'waiting for input' || seconds === 0){
                clearInterval(interval)
            }
            if (seconds === 0){
                setGameState('game over')
                getResults()
            }
        }

        const interval = setInterval(gameTimer, 1000)
    }

    async function getWords(limit: number){
        const response = await fetch(`/api/words?limit=${limit}`)
        words = await response.json() 
    }

    function handleKeyDown(event: KeyboardEvent) {
        if (event.code === 'Space') {
            event.preventDefault()
            if (game === 'in progress') {
                nextWord()
            }
        }

        if (game === 'waiting for input') {
            startGame()
        }

    }
    function focusInput(){
        inputEl.focus()
    }
    onMount(async ()=>{
        focusInput()
        getWords(100)
    })

</script>

{#if game === 'game over'}
    <div in:blur class='results'>
        <div>
            <p class="title">wpm</p>
            <p class="score">{Math.trunc($wordsPerMinute)}</p>
        </div>

        <div>
            <p class='title'>accuracy</p>
            <p class='score'>{Math.trunc($accuracy)}%</p>
        </div>
        <button class='play' on:click={resetGame}>play again</button>
    </div>    
{/if}


{#if game !=='game over'}
<div class="game" data-game={game}>
    <input 
        bind:this={inputEl}
        bind:value={typedLetter}
        on:input={updateGameState}
        on:keydown={handleKeyDown}
        class="input"
        type="text"
    />

    <div class="time">{seconds}</div>
    {#key toggleReset}
    <div in:blur|local bind:this={wordsEl} class="words">
        {#each words as word}
            <span class="word">
                {#each word as letter}
                    <span class="letter">{letter}</span>
                {/each}
            </span>
        {/each}
        <div class="caret" bind:this={caretEl}></div>
    </div>
    {/key}
    <div class="reset">
        <button on:click={resetGame} aria-label="reset">
        ↩️ restart
        </button>
    </div>
</div>
{/if}
<style lang="scss">
    .results {
        .title {
            font-size: 2rem;
            color: var(--fg-200);
        }

        .score {
            font-size: 4rem;
            color: var(--primary);
        }

        .play {
            margin-top: 1rem;
        }
    }
    .game {
        position: relative;

        .time {
            position: absolute;
            top: -48px;
            font-size: 1.5rem;
            color: var(--primary);
            opacity: 0;
            transition: all 0.3s ease;
        }
        &[data-game='in progress'] .time {
            opacity: 1;
        }
        .reset {
            width: 100%;
            display: grid;
            justify-content: center;
            margin-top: 2rem;
        }
        .input {
            position: absolute;
            opacity: 0;
        }
    }
    .words {
        --line-height: 1em;
        --lines: 3;

        width: 100%;
        max-height: calc(var(--line-height) * var(--lines) * 1.42);
        display: flex;
        flex-wrap: wrap;
        gap: 0.6em;
        position: relative;
        font-size: 1.5rem;
        line-height: var(--line-height);
        overflow: hidden;
        user-select: none;

        .letter {
            opacity: 0.4;
            transition: all 0.3s ease;

            &:global(.letter[data-letter='correct']){
                opacity: 0.8;
            }

            &:global(.letter[data-letter='incorrect']){
                color: var(--primary);
                opacity: 1;
            }
        }
        
        .game {
            &[data-game='in progress'] .caret {
                animation: none;
            }
        }

        .caret {
            position: absolute;
            height: 1.5rem;
            top: 0;
            border-right: 2px solid var(--primary);
            animation: caret 1s infinite;
            transition: all 0.2s ease;

            @keyframes caret {
                0%,
                to {
                    opacity: 0;
                }
                50% {
                    opacity: 1;
                }
            }
        }
    }
</style>