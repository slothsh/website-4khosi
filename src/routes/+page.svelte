<script lang="ts">
    import { onMount } from "svelte";
    import { slide } from "svelte/transition";
    import { quintInOut, quintOut } from "svelte/easing";
    import anime from "animejs";
    import Arrow from "$lib/assets/arrow.png"
    import Coin from "$lib/assets/coin.png"

    import PoemsData from "$lib/assets/poems.json?raw"
    import Heart from "$lib/assets/heart.png"
    import Ladybug from "$lib/assets/ladybug.png"
    import Tiara from "$lib/assets/tiara.png"
    import Afro from "$lib/assets/afro.png"
    import Melody from "$lib/assets/melody.png"
    import Crash from "$lib/assets/crash.png"
    import Couple from "$lib/assets/couple.png"
    import Saxophone from "$lib/assets/saxophone.png"
    import Languages from "$lib/assets/languages.png"
    import Bibimbap from "$lib/assets/bibimbap.png"
    import CrossMark from "$lib/assets/cross.png"
    import CheckMark from "$lib/assets/heart.png"

    enum CycleDirection {
        Left = -1,
        Right = 1,
    }

    enum Answer {
        No = 0,
        Yes,
    }

    interface Poem {
        title: string,
        poem: string,
        icon: string,
        alt: string,
        answer: boolean,
        claimed: boolean
    }

    const shuffle = <T>(array: T[]) => { 
        for (let i = array.length - 1; i > 0; i--) { 
            const j = Math.floor(Math.random() * (i + 1)); 
            [array[i], array[j]] = [array[j], array[i]]; 
        } 
        return array; 
    }; 

    const poems: Poem[]  = shuffle(JSON.parse(PoemsData));
    const answeredPoems: [Poem, Answer][] = [];
    poems.forEach((poem) => {
        poem.claimed = false;
    });

    const totalCorrectPoems = poems
        .map((e): number => { if (e.answer) { return 1; } return 0; })
        .reduce((previous, current) => { return previous + current; });

    const maxMoola = 150.0;
    const correctPoemMoola = (maxMoola / totalCorrectPoems);
    let totalMoola = 0.00;
    let totalCoins = 0;
    let currentPoem = 0;

    let showIcon = false;
    const toggleTitle = (to: number) => {
        if (to !== currentPoem) {
            anime({
                targets: ".icon-anim",
                translateY: 64,
                duration: 0,
                opacity: 0,
                easing: "linear",
            });
            anime({
                targets: ".icon-anim",
                translateY: 0,
                duration: 150,
                opacity: 1,
                easing: "easeInSine",
            });

            anime({
                targets: ".toggle-anim",
                translateY: -24,
                duration: 0,
                opacity: 0,
                easing: "linear",
            });
            anime({
                targets: ".toggle-anim",
                translateY: 0,
                duration: 300,
                opacity: 1,
                easing: "easeInSine",
            })

            anime({
                targets: ".toggle-anim-lines",
                translateY: -24,
                duration: 0,
                opacity: 0,
                easing: "easeInSine",
            })
            anime({
                targets: ".toggle-anim-lines",
                duration: 300,
                translateY: 0,
                opacity: 1,
                delay: 250,
                easing: "easeInSine",
            })
        }
    }

    const cyclePoem = (direction: CycleDirection) => {
        const tmp = currentPoem;
        currentPoem = Math.max(0, Math.min(currentPoem + direction, poems.length - 1));
        toggleTitle(tmp);
    }

    const resolveIcon = (name: string) => {
        switch (name) {
            case "heart.png": return Heart; break;
            case "ladybug.png": return Ladybug; break;
            case "tiara.png": return Tiara; break;
            case "afro.png": return Afro; break;
            case "bibimbap.png": return Bibimbap; break;
            case "languages.png": return Languages; break;
            case "couple.png": return Couple; break;
            case "melody.png": return Melody; break;
            case "crash.png": return Crash; break;
            case "saxophone.png": return Saxophone; break;
            case "cross.png": return CrossMark; break;
            case "check.png": return Heart; break;
            default: return Heart;
        }

        return Heart;
    }

    const checkResult = (answer: Answer) => {
        if (answer === Answer.Yes && poems[currentPoem].answer && !poems[currentPoem].claimed) {
            totalMoola = Math.max(0, Math.min(totalMoola + correctPoemMoola, maxMoola));
            totalCoins = Math.floor(totalMoola / (maxMoola / 5));
        }

        if (!poems[currentPoem].claimed) {
            answeredPoems.push([poems[currentPoem], answer]);
        }

        poems[currentPoem].claimed = true;
        console.log(answeredPoems.length);
    }

    const resetResults = () => {
        totalCoins = 0;
        totalMoola = 0;
        poems.forEach((poem) => {
            poem.claimed = false;
        });
    }

    onMount(() => {
        showIcon = true;
        anime({
            targets: ".icon-anim",
            loop: true,
            scale: 1.3,
            duration: 3000,
            easing: "easeInOutSine",
            direction: "alternate",
        });
    });
</script>

<!-- <svelte:window on:click={cyclePoem}></svelte:window> -->
<div id="backgroundMain" class="fill-screen"></div>
<div id="moolaCounter">
    <!-- <a id="moolaValue" href="">R{totalMoola.toPrecision(3)}</a> -->
    <a id="moolaValue" href="">Coupons:</a>
    <div class="moola-pouch">
    {#each { length: totalCoins } as i}
        <img class="moola-coin" src={Coin} alt="coin">
    {/each}
    </div>
</div>
<div id="siteTitle">
    <a href="">Stevey x Khosi</a>
</div>
<div id="answeredPoemsContainer">
{#each answeredPoems as [poem, answer]}
    <div class="answer-poem-item">
        <p>{poem.title}</p>
        <img src={(answer === Answer.Yes && poem.answer) ? CheckMark : CrossMark } alt="check mark">
    </div>
{/each}
</div>
<section id="sectionMain" class="fill-screen">
    <div id="iconContainer">
    {#if showIcon}
        <div id="poemIconContainer" class="icon-anim">
            <img transition:slide={{easing: quintInOut, axis: "x"}}
                 class="poem-icon"
                 src={resolveIcon(poems[currentPoem].icon)}
                 alt={poems[currentPoem].alt}>
        </div>
    {/if}
    </div>
    <div class="divider"></div>
    <div id="textContainer">
        <h2 class="anim-01 text-item">4 Khosi</h2>
        <p class="anim-01 text-item">Stevey x Khosi</p>
    </div>
    <div id="poemNavigator">
        <img on:click|preventDefault={() => cyclePoem(CycleDirection.Left)} id="leftPoemArrow" src={Arrow} alt="arrow" class="poem-arrow flip">
        <div id="sectionPoem">
            <h3 class="toggle-anim">{poems[currentPoem].title}</h3>
            {#each poems[currentPoem].poem.split("\n") as line}
                <p class="toggle-anim-lines">{line}</p>
            {/each}
        </div>
        <img on:click|preventDefault={() => cyclePoem(CycleDirection.Right)} id="rightPoemArrow" src={Arrow} alt="arrow" class="poem-arrow">
    </div>
    <div id="buttonContainer">
        <button on:click|preventDefault={() => checkResult(Answer.Yes)}>Yeah, it's AI</button>
        <button on:click|preventDefault={() => checkResult(Answer.No)}>Nah, Stevey wrote that</button>
    </div>
</section>


<style scoped>
    :global(*) {
        font-family: 'Single Day', 'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif;
        color: #181818;
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    #backgroundMain {
        /* background-color: #decccc; */
        /* background-color: #FF8B94; */
        background-color: #deaaaa;
        z-index: -1;
    }

    #sectionMain {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }

    #sectionPoem {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        margin-top: 2rem;
        width: 20rem;
    }

    #sectionPoem > h3 {
        margin-bottom: 1rem;
    }

    #textContainer {
        display: flex;
        justify-content: start;
        align-items: center;
        flex-direction: column;
        margin-top: 1rem;
    }

    #iconContainer {
        width: 6rem;
        height: 6rem;
        max-height: 100%;
        margin-bottom: 1rem;
        overflow: hidden;
    }

    #siteTitle {
        display: flex;
        flex-direction: row-reverse;
        justify-content: start;
        align-items: center;
        position: absolute;
        top: 1rem;
        right: 2rem;
        gap: 1rem;
        z-index: 10;
    }

    #siteTitle > a {
        text-decoration: none;
        border-radius: 6px;
        padding: 6px;
        transition: all 300ms ease-in-out;
    }

    #siteTitle > a:hover {
        box-shadow: 0px 0px 10px #18181833;
    }

    #moolaCounter {
        display: flex;
        flex-direction: row;
        justify-content: start;
        align-items: center;
        position: absolute;
        top: 1rem;
        left: 2rem;
    }

    #moolaValue {
        text-decoration: none;
        margin-right: 8px;
    }

    #poemNavigator {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        gap: 3rem;
    }

    #poemIconContainer {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
    }

    #buttonContainer {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        margin-top: 3rem;
        gap: 1rem;
    }

    #answeredPoemsContainer {
        position: absolute;
        left: 0;
        top: 12.5%;
        margin-left: 2rem;
    }
    
    .flip {
        transform: scaleX(-1) scaleY(-1);
    }

    .poem-arrow {
        width: 32px;
        border-radius: 32px;
        padding: 8px;
        transition: all 300ms ease-in-out;
    }

    .poem-arrow:hover {
        background-color: #FFAAAA;
        transform: scale(1.1);
        box-shadow: 0px 0px 20px #00000033;
    }

    .poem-arrow.flip:hover {
        transform: scaleX(-1) scaleY(-1) scale(1.1);
    }

    .moola-coin {
        width: 16px;
    }

    .moola-pouch {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 4px;
    }

    .poem-icon {
        /* position: relative; */
        /* top: 50%; */
        /* left: 50%; */
        /* transform: translateX(-50%) translateY(-50%); */
        width: 100%;
    }

    .divider {
        background-color: #181818;
        height: 2px;
        max-height: 2px;
        width: 8rem;
    }

    .text-item {
        height: 1em;
    }

    .fill-screen {
        position: absolute;
        width: 100vw;
        height: 100vh;
        top: 0;
        left: 0;
    }

    .answer-poem-item {
        display: flex;
        flex-direction: row;
        justify-content: start;
        align-items: center;
    }

    button {
        font-size: 1.5rem;
        background-color: #E6A71E;
        border: none;
        border-radius: 4px;
        padding: 0 1rem 0 1rem;
        transition: all 300ms ease-in-out;
    }

    button:hover {
        transform: scale(1.2);
        margin: 0 2rem 0 2rem;
        background-color: #FFCA59;
        box-shadow: 0px 0px 20px #00000033;
    }
</style>
