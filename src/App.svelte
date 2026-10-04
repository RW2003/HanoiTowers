<script>
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from './assets/vite.svg'
  import heroImg from './assets/hero.png'
  import Game from './components/Game.svelte';
</script>

<section id="center">
  <div class="hero">
    <img src={heroImg} class="base" width="170" height="179" alt="" />
    <!--
      <img src={svelteLogo} class="framework" alt="Svelte logo" />
      <img src={viteLogo} class="vite" alt="Vite logo" />
    -->
  </div>
  <div>
    <h1>Tower of Hanoi</h1>
    <p>See <a class="text-red-400" href="https://en.wikipedia.org/wiki/Tower_of_Hanoi">Wikipedia/Tower_of_Hanoi</a> for Description and Solution</p>
  </div>
  
  <Game />
</section>
