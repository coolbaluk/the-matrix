<script>
  import { onMount } from "svelte";

  let chars = "ﾊﾐﾋｰｳｼﾅﾓﾆｻﾜﾂｵﾘｱﾎﾃﾏｹﾒｴｶｷﾑﾕﾗｾﾈｽﾀﾇﾍ".split("");
  const noInitialDrop = true;
  const fontSize = 16;
  const color = "#0f0";
  const interval = 1000;

  let canvas;
  let windowWidth = 300;
  let windowHeight = 150;
  let focused = false;
  let defaultLink =
    "https://raw.githubusercontent.com/coolbaluk/autohotkey-scripts/master/volume-control-scroll.ahk";
  let link = defaultLink;

  $: columns = Math.round(windowWidth / fontSize);
  $: drops = Array(columns).fill(noInitialDrop ? windowHeight : 0);

  const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

  const handleClick = async () => {
    const response = await fetch(link);
    chars = await response.text();
  };

  const handleFocus = () => {
    link = "";
  };

  const handleEnter = () => {
    focused = true;
  };

  const handleLeave = () => {
    focused = false;
  };

  onMount(async () => {
    console.log(drops);
    [windowWidth, windowHeight] = await [window.innerWidth, window.innerHeight];
    console.log(windowHeight);
    const ctx = canvas.getContext("2d");
    ctx.canvas.width = window.innerWidth;
    ctx.canvas.height = window.innerHeight;

    while (true) {
      ctx.fillStyle = "rgba(0, 0, 0, 0.05)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = color;
      ctx.font = `${fontSize}px arial`;
      for (let i = 0; i < columns; i++) {
        // Displays a random character from chars
        // const randomChar = Math.floor(Math.random() * chars.length);
        const randomChar = drops[i];
        console.log(randomChar, drops.length, columns);
        const text = chars[randomChar];
        ctx.fillText(text, i * fontSize, drops[i] * fontSize);
        const reachedEndOfScreen =
          drops[i] * fontSize > canvas.height && Math.random() > 0.975;
        // console.log(drops[i] * fontSize > canvas.height);
        // if (reachedEndOfScreen) drops[i] = 0
        if (reachedEndOfScreen) drops[i] = Math.floor(Math.random() * columns);
        drops[i]++;
      }
      await sleep(interval);
    }
  });
</script>

<svelte:window bind:innerWidth={windowWidth} bind:innerHeight={windowHeight} />

<div
  class="enter {focused && 'enter-focused'}"
  on:mouseenter={handleEnter}
  on:mouseleave={handleLeave}>
  <input bind:value={link} on:focus={handleFocus} />
  <button on:click={handleClick}>Enter</button>
</div>
<canvas bind:this={canvas} width={windowWidth} height={windowHeight} />
