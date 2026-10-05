<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from './assets/vite.svg'
  import heroImg from './assets/hero.png'
  import Game from './components/Game.svelte';

  let game: Game;
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
    
    <div class="flex flex-row justify-center" >
      <button class="p-2 px-3 mt-2 -mb-2 mx-1 rounded-md text-white bg-blue-500 hover:bg-blue-400" on:click={() => game.solve()} >Solve</button>
      <button class="p-2 px-3 mt-2 -mb-2 mx-1 rounded-md text-white bg-red-500 hover:bg-red-400" on:click={() => game.reset()} >Reset</button>
    </div>    
  </div>
  
  <Game bind:this={game} />
</section>
