<script>
  import {setContext} from 'svelte';
  
  let context = {};
  setContext("canvas", context);
</script>

<!-- <canvas bind.this={canvas} on:mousedown={$draw.mousedown.bind($draw)} on:mousemove={$draw.mousemove.bind($draw)} on:mouseup={$draw.mouseup.bind($draw)}  width={window.innerWidth} height={window.innerHeight}/> -->
<canvas bind:this={context.canvas} width={window.innerWidth} height={window.innerHeight}/>

{#if context.canvas}
	<slot />
{/if}
<style>

</style>