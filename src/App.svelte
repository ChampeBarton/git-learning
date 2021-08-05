
<script>
    import { tweened } from 'svelte/motion';
    import { cubicOut } from 'svelte/easing';
	import { scaleLinear, scaleBand } from 'd3-scale';
    import { max, ticks } from 'd3-array';
    import { format } from 'd3-format';
	import data from './data.tsv';
    
    // This is a change to this file. Register it, git overlords!

    let margin = {top: 50, right: 20, bottom: 30, left: 40}
    let width = 960;
	let height = 500;

    let xScale = scaleBand().rangeRound([margin.left, width-margin.right]).padding(0.1);
    let yScale = scaleLinear().rangeRound([height-margin.bottom, margin.top]);
    
    data.forEach((d) => {
		d.frequency = +d.frequency;
	});
    
    // $: xSortKey = toggle ? "letter" : "frequency";
    $: xScaleDomain = toggle ? data.sort((a, b) => (a.letter > b.letter) ? 1 : -1).map(function(d) { return d.letter; }) : 
        data.sort((a, b) => (a.frequency > b.frequency) ? -1 : 1).map(function(d) { return d.letter; });
    $: xScale = xScale.domain(xScaleDomain);
    yScale.domain([0, max(data, function(d) { return d.frequency; })]);

    let letterDict = {"A": 1, "B": 2, "C": 3, "D": 4, "E": 5, "F": 6, "G": 7,
                    "H": 8, "I": 9, "J": 10, "K": 11, "L": 12, "M": 13, "N": 14,
                    "O": 15, "P": 16, "Q": 17, "R": 18, "S": 19, "T": 20, "U": 21,
                    "V": 22, "W": 23, "X": 24, "Y": 25, "Z": 26};

    let idDict = {1: "A", 2: "B", 3: "C", 4: "D", 5: "E", 6: "F", 7: "G",
                    8: "H", 9: "I", 10: "J", 11: "K", 12: "L", 13: "M", 14: "N",
                    15: "O", 16: "P", 17: "Q", 18: "R", 19: "S", 20: "T", 21: "U",
                    22: "V", 23: "W", 24: "X", 25: "Y", 26: "Z"};

    $: bars = data.map(d => {
        var x = xScale(d.letter);
        var y = yScale(d.frequency);
        var h = height - margin.bottom - y;
        var id = letterDict[d.letter];
        // var l = d.letter;
        return {x, y, h, id};
    });

    // change # 2
    
    $: xTicks = data.map(d => {
        var id = letterDict[d.letter];
        var xPos = xScale(d.letter) + xScale.bandwidth()/2;
        return {id, xPos}
    });
    let yTicks = ticks(yScale.domain()[0], yScale.domain()[1]+0.003, 13);

	const tweenedBars = tweened(bars, {
		delay: 0,
		duration: 750,
		easing: cubicOut
	});

    const tweenedTicks = tweened(xTicks, {
        delay: 0,
        duration: 750,
        easing: cubicOut
    })

    let toggle = true;

    // We're tweening positionally, so this makes sure each letter tweens 
    // to its future position rather than to the new letter that will be 
    // at its current position.
    $: bars = bars.sort((a, b) => (a.id > b.id) ? 1 : -1);
    $: xTicks = xTicks.sort((a, b) => (a.id > b.id) ? 1 : -1);

    $: tweenedBars.set(bars);
    $: tweenedTicks.set(xTicks);

    // $: console.log(xTicks)

</script>

<style>
    rect {
      fill: rgb(99, 172, 190);
    }
    
    rect:hover {
      fill: rgb(68, 100, 49);
    }

    text {
        font-family:'Courier New', Courier, monospace;
        font-size: small;
    }

    button {
        position: absolute;
        top: 100px;
        left: 900px;
        z-index: 10;
        line-height: 1.2;
		padding: 0.5em 2.5em 0.5em 2em;
		margin: 0 0 0.5em 0;
		border-radius: 2px;
		user-select: none;
		border: 1px solid hsl(240, 8%, 70%);
		background-color:hsl(240, 8%, 93%);
		color: #333;
    }

    button:active {
        background: #e5e5e5;
        -webkit-box-shadow: inset 0px 0px 5px #c1c1c1;
        -moz-box-shadow: inset 0px 0px 5px #c1c1c1;
        box-shadow: inset 0px 0px 5px #c1c1c1;
        outline: none;
    }

    .container {
        position: relative;
    }

    .svg {
        position: relative;
    }
    
</style>


<div class="container">

    <button on:click={() => toggle = !toggle}> SORT </button>

    <svg class="svg" {width} {height} transform="translate(100,50)">

        <!-- ---------- y axis ---------- -->
        <g transform="translate(0, {margin.top/3})">
            <text font-style="italic">↑ Frequency</text>
        </g>

        <g transform="translate({margin.left+20}, 0)"> 
            {#each yTicks as y} 
                <g class="tick" opacity="1" transform="translate(0,{yScale(y)})">
                    <line stroke="black" x2="-5" />
                    <text dy="0.32em" fill="black" x="-{margin.left}">
                        {format("0.0%")(y)}
                    </text>
                </g>
            {/each}
        </g>

        <!-- ---------- x axis ---------- -->
        <g transform = "translate(20, {height})">
            <line y1={-margin.bottom} y2={-margin.bottom} x1={margin.left} x2={width-margin.right} stroke="black" fill="none"></line>
            {#each $tweenedTicks as x(x.id)} 
                <g class="tick" opacity="1" transform="translate(0,{-margin.bottom})">
                    <line stroke="black" x1 = {x.xPos} x2 = {x.xPos} y2="6" />
                    <text fill="black" y="9" dy="0.71em" x={x.xPos} text-anchor="middle">
                        {idDict[x.id]}
                    </text>
                </g>
            {/each}

            {#each $tweenedBars as bar(bar.id)}
                <rect 
                    x = {bar.x}
                    y = {bar.y}
                    width = {xScale.bandwidth()}
                    height = {bar.h} 
                    transform="translate(0, {-height})"
                /> 
            {/each}
        </g>

    </svg>
</div>





