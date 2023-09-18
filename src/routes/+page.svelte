<script lang="ts">
    import { onMount } from "svelte";
    import classes from './classes.json';

    // June 13, 2024 3:20:00 PM
    const LAST_DAY = new Date(2024, 5, 13, 15, 20, 0, 0);

    let timeLeft = LAST_DAY.getTime() - Date.now();
    let timeData = getTimeData(timeLeft);
    let prevTimeData = { ... timeData };

    $: blocks = getBlocks();


    // TODO, update live, fix timing
    $: getBlocks = () => {
        const counts: { [key: string]: number } = {};
        for (const name of ['1', '2', '3', '4', '5', '6', '7', 'Lun', 'C&C', 'As', 'LCoCu', 'ECoCu', 'PMCoCu']) {
            let sum = 0;
            for (const { start, end } of (classes as any)[name]) {
                const startTime = start * 1000 + 7 * 60 * 60 * 1000;
                const endTime = end * 1000 + 7 * 60 * 60 * 1000;
                if (Date.now() < startTime) {
                    sum += 1;
                } else if (Date.now() < endTime) {
                    sum += 1 - Math.max((Date.now() - startTime) / (endTime - startTime), 0);
                } else {

                }
            }
            counts[name] = sum;
        }
        counts['CoCu'] = counts['LCoCu'] + counts['ECoCu'] + counts['PMCoCu'];
        return counts;
    }

    function getTimeData(timeLeft: number) {
        // use actual date
        const now = LAST_DAY.getTime() - timeLeft;
        const date = new Date(now);
        
        // months
        const monthsLeft = date.getFullYear() == LAST_DAY.getFullYear() ? LAST_DAY.getMonth() - date.getMonth() : LAST_DAY.getMonth() + 11 - date.getMonth();
        // if june 2024 set to last day
        const endOfMonth = monthsLeft === 0 ? LAST_DAY.getTime() : new Date(date.getFullYear(), date.getMonth() + 1, 0).getTime();
        
        console.log(endOfMonth);

        const timeToEnd = endOfMonth - now;
        const weeks = Math.max(Math.floor(timeToEnd / (1000 * 60 * 60 * 24 * 7)), 0);
        const days = Math.max(Math.floor(timeToEnd / (1000 * 60 * 60 * 24) % 7), 0);

        return {
            seconds: Math.max(Math.floor((timeLeft / 1000) % 60), 0),
            totalSeconds: Math.max(timeLeft / 1000, 0),

            minutes: Math.max(Math.floor((timeLeft / (1000 * 60)) % 60)),
            totalMinutes: Math.max(timeLeft / (1000 * 60), 0),

            hours: Math.max(Math.floor((timeLeft / (1000 * 60 * 60)) % 24), 0),
            totalHours: Math.max(timeLeft / (1000 * 60 * 60), 0),

            days,
            totalDays: Math.max(timeLeft / (1000 * 60 * 60 * 24), 0),

            weeks,
            totalWeeks: Math.max(timeLeft / (1000 * 60 * 60 * 24 * 7), 0),

            months: monthsLeft,
            totalMonths: Math.max(timeLeft / (1000 * 60 * 60 * 24 * 30), 0),

            years: Math.max(Math.floor(timeLeft / (1000 * 60 * 60 * 24 * 365)), 0),
            totalYears: Math.max(timeLeft / (1000 * 60 * 60 * 24 * 365), 0),
        };
    }
    
    function tick() {
        prevTimeData = { ... timeData };
        timeLeft = LAST_DAY.getTime() - Date.now();
        timeData = getTimeData(timeLeft);
        blocks = getBlocks();
    }

    $: isEvenSecond = () => {
        return Math.floor(timeData.totalSeconds) % 2 === 0;
    }

    $: isZero = (name: string) => {
        return (timeData as any)[name] === 0 && (prevTimeData as any)[name] !== 0;
    }

    function roundOrFixed(num: number) {
        if (num % 1 === 0) return num.toFixed(0);
        return num.toFixed(4);
    }

    let interval: any = null;
    onMount(() => {
        if (interval) clearInterval(interval);
        interval = setInterval(tick, 1000);
    });
    
</script>

<main>
    <div class="top">
        <div class="title">
            <p class:upped={!isEvenSecond()}>LAST</p>
            <p class:upped={isEvenSecond()}>DAY</p>
        </div>
    </div>
    <div class="grid">
        <div class="gridItem timeCounter" class:zero={isZero('years')}>
            <p class="timeTitle">YEARS</p>
            <p class="timeValue">{timeData.years}</p>
            <p class="timeTotal">{timeData.totalYears.toFixed(8)}</p>
        </div>
        <div class="gridItem timeCounter" class:zero={isZero('months')}>
            <p class="timeTitle">MONTHS</p>
            <p class="timeValue">{timeData.months}</p>
            <p class="timeTotal">{timeData.totalMonths.toFixed(7)}</p>
        </div>
        <div class="gridItem timeCounter" class:zero={isZero('weeks')}>
            <p class="timeTitle">WEEKS</p>
            <p class="timeValue">{timeData.weeks}</p>
            <p class="timeTotal">{timeData.totalWeeks.toFixed(6)}</p>
        </div>
        <div class="gridItem timeCounter" class:zero={isZero('days')}>
            <p class="timeTitle">DAYS</p>
            <p class="timeValue">{timeData.days}</p>
            <p class="timeTotal">{timeData.totalDays.toFixed(5)}</p>
        </div>
        <div class="gridItem timeCounter" class:zero={isZero('hours')}>
            <p class="timeTitle">HOURS</p>
            <p class="timeValue">{timeData.hours.toString().padStart(2, '0')}</p>
            <p class="timeTotal">{timeData.totalHours.toFixed(4)}</p>
        </div>
        <div class="gridItem timeCounter" class:zero={isZero('minutes')}>
            <p class="timeTitle">MINUTES</p>
            <p class="timeValue">{timeData.minutes.toString().padStart(2, '0')}</p>
            <p class="timeTotal">{timeData.totalMinutes.toFixed(2)}</p>
        </div>
        <div class="gridItem timeCounter" class:zero={isZero('seconds')}>
            <p class="timeTitle">SECONDS</p>
            <p class="timeValue">{timeData.seconds.toString().padStart(2, '0')}</p>
            <p class="timeTotal">{timeData.totalSeconds.toFixed(0)}</p>
        </div>
        <div class="gridItem messageContainer">
            <p class="message">UNTIL THE END OF SCHOOL</p>
        </div>

        {#each new Array(7).fill(0).map((_, i) => i + 1) as index}
            <div class={`gridItem classCounter block${index}`}>
                <p class="timeTitle">BLOCK {index}</p>
                <p class="classValue">{roundOrFixed(blocks[`${index}`])}</p>
                <p class="timeTitle">REMAINING</p>
            </div>
        {/each}
        
        <div class="otherBlocks">
            <div class="other">
                <p class="otherTitle">LUNCHES</p>
                <p class="otherValue">{roundOrFixed(blocks['Lun'])}</p>
            </div>
            <div class="other">
                <p class="otherTitle">C&CS</p>
                <p class="otherValue">{roundOrFixed(blocks['C&C'])}</p>
            </div>
            <div class="other">
                <p class="otherTitle">ASSEMBLIES</p>
                <p class="otherValue">{roundOrFixed(blocks['As'])}</p>
            </div>
            <div class="other">
                <p class="otherTitle">COCUS</p>
                <p class="otherValue">{roundOrFixed(blocks['CoCu'])}</p>
            </div>
        </div>
    </div>
</main>

<style>
    main {
        display: flex;

        flex-direction: column;
        align-items: center;

        width: 100vw;
        height: 100%;
        min-height: 100vh;

        margin: 0;
        padding: 0;
        overflow-x: hidden;
    }

    .top {
        width: 100%;
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
    }

    .title {
        font-size: 3rem;
        font-weight: 700;
        color: #fff;
        
        display: flex;
        flex-direction: row;
        line-height: 1;
    }

    .title > p {
        margin: 0.5rem;
        padding-top: 0.1rem;
        transition: transform 0.5s cubic-bezier(0.16, 1, 0.3, 1);
    }

    .upped {
        transform: translateY(-0.2rem);
    }

    .grid {
        display: grid;
        grid-auto-flow: row;

        height: max-content;
        flex-grow: 1;
        margin-bottom: 1rem;
        width: calc(100% - 1rem);
        margin-left: 0.5rem;
        margin-right: 0.5rem;
        gap: 0.5rem;
    }

    @media (min-width: 512px) {
        .grid {
            grid-template-columns: repeat(4, minmax(0, 1fr));
            grid-template-rows: repeat(4, minmax(0, 1fr));
        }
    }

    @media (min-width: 768px) {
        .grid {
            grid-template-columns: repeat(8, minmax(0, 1fr));
            grid-template-rows: repeat(2, minmax(0, 1fr));
        }
    }

    .gridItem {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        width: 100%;
        height: 100%;
        max-width: 100%;
        text-align: center;

        min-height: 12rem;
    }

    

    .timeCounter {
        position: relative;
        
    }

    .timeCounter::before {
        content: "";
        position: absolute;
        inset: 0;
        padding: 0.1rem; 
        background-position: 0% 0%;
        background-size: 2000% 2000%;
        background-image: linear-gradient(#ffff, #ffff, #fff0, #ffff, #ffff); 
        -webkit-mask: 
            linear-gradient(#fff 0 0) content-box, 
            linear-gradient(#fff 0 0);
        -webkit-mask-composite: xor;
                mask-composite: exclude; 
    }

    @keyframes shine {
        0% {
            background-position: 0% 0%;
        }
        100% {
            background-position: 100% 100%;
        }
    }

    .timeCounter.zero::before {
        animation: shine 2s infinite;
    }

    .classCounter {
        box-sizing: border-box;
        border: 0.1rem solid;
    }

    .classCounter > p {
        margin: 0;
        padding: 0.25rem;
    }

    .block1 {
        /* yellow */
        border-color: #ff0;
        color: #ff0;
        background-color: #ff01;
    }

    .block2 {
        /* purple */
        border-color: #80f;
        color: #80f;
        background-color: #80f1;
    }

    .block3 {
        /* orange */
        border-color: #f80;
        color: #f80;
        background-color: #f801;
    }

    .block4 {
        /* blue */
        border-color: #08f;
        color: #08f;
        background-color: #08f1;
    }

    .block5 {
        /* green */
        border-color: #0f0;
        color: #0f0;
        background-color: #0f01;
    }

    .block6 {
        /* red */
        border-color: #f00;
        color: #f00;
        background-color: #f001;
    }

    .block7 {
        /* pink */
        border-color: #f0f;
        color: #f0f;
        background-color: #f0f1;
    }

    .other {
        /* gray */
        border-color: #888;
        color: #888;
        background-color: #8881;
    }

    .otherTitle {
        font-weight: 700;
        font-size: 0.75rem;
        line-height: 0;
        margin: 0;
        padding: 0.25rem;
    }

    .otherValue {
        font-weight: 700;
        font-size: 2rem;
    }

    .otherBlocks {
        display: grid;
        grid-template-rows: repeat(4, minmax(0, 1fr));
        grid-auto-flow: column;
        row-gap: 0.5rem;
    }

    .otherBlocks > div {
        box-sizing: border-box;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-width: 100%;
        width: 100%;
        height: 100%;
        max-width: 100%;
        text-align: center;
        border: 0.1rem solid;
    }

    .otherBlocks > div > p {
        margin: 0;
        line-height: 1;
        font-weight: 700;
    }


    .timeCounter > p {
        margin: 0;
        padding: 0.25rem;
    }

    .timeTitle {
        font-weight: 700;
        font-size: 1rem;
    }

    .timeValue {
        font-weight: 700;
        font-size: 5rem;
    }

    .classValue {
        font-weight: 700;
        font-size: 2rem;
    }

    .timeTotal {
        color: #fff6;
        font-weight: 700;
        font-size: 1rem;
    }

    .message {
        font-weight: 700;
        font-size: 1.5rem;
        color: #fff;
        text-align: center;
    }
</style>