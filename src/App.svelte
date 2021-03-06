<script>
  import { onMount } from "svelte";
  import { fade } from "svelte/transition";

  const defaultLink =
    "https://raw.githubusercontent.com/coolbaluk/the-matrix/master/src/App.svelte";
  const defaultChars = "ﾊﾐﾋｰｳｼﾅﾓﾆｻﾜﾂｵﾘｱﾎﾃﾏｹﾒｴｶｷﾑﾕﾗｾﾈｽﾀﾇﾍ";

  const fontSize = 16;
  const color = "#0f0";
  const interval = 33;

  let canvas;
  let windowWidth = 300;
  let windowHeight = 150;
  let link = defaultLink;
  let chars = defaultChars;

  let focused = false;
  let visible = true;
  let fetching = false;

  const getRandomInt = max => {
    return Math.floor(Math.random() * max);
  };

  $: columns = Math.round(windowWidth / fontSize);
  $: drops = Array(columns).fill(windowHeight);
  $: startingCharacterIndices = Array(columns).fill(0);

  const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

  const handleClick = async () => {
    if (link.trim() === "") return;

    fetching = true;
    chars = defaultChars;
    drops = drops.map(() => 0);

    await sleep((windowHeight / 2 / fontSize - 2) * interval);

    visible = false;

    await sleep((windowHeight / 2 / fontSize + 10) * interval);
    const response = await fetch(link);
    if (!response.ok) {
      chars = "Mr. Anderson ";
      return;
    }
    chars = await response.text();
  };

  onMount(async () => {
    [windowWidth, windowHeight] = await [window.innerWidth, window.innerHeight];
    const ctx = canvas.getContext("2d");
    ctx.canvas.width = window.innerWidth;
    ctx.canvas.height = window.innerHeight;

    while (true) {
      ctx.fillStyle = "rgba(0, 0, 0, 0.05)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = color;
      ctx.font = `${fontSize}px arial`;
      for (let i = 0; i < columns; i++) {
        const charIndex =
          (startingCharacterIndices[i] + drops[i]) % chars.length;
        const text = chars[charIndex];
        ctx.fillText(text, i * fontSize, drops[i] * fontSize);

        const reachedEndOfScreen =
          drops[i] * fontSize > canvas.height && Math.random() > 0.975;
        if (reachedEndOfScreen) {
          drops[i] = getRandomInt(columns);
          startingCharacterIndices[i] = getRandomInt(chars.length);
        }
        drops[i]++;
      }
      await sleep(interval);
    }
  });
</script>

<svelte:window bind:innerWidth={windowWidth} bind:innerHeight={windowHeight} />

{#if visible}
  <div class="wrapper" transition:fade>
    <div
      class="enter {focused && 'enter-focused'}"
      on:mouseenter={() => (focused = true)}
      on:mouseleave={() => (focused = false)}>
      <input
        bind:value={link}
        onfocus="this.select();"
        onmouseup="return false;" />
      <button on:click={handleClick} disabled={fetching}>Enter</button>
    </div>

    <div class="text" transition:fade>
      Paste a (raw github) link or use the default
      <span on:click={() => (link = defaultLink)}>(this website)</span>
      to see it matrixified
    </div>
  </div>
{/if}

<canvas bind:this={canvas} width={windowWidth} height={windowHeight} />
