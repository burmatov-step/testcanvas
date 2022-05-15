<script>
	import { writable } from 'svelte/store';
	import { onDestroy, setContext } from 'svelte';
	import Canvas from './Canvas.svelte'
	import CanvasCore from './CanvasCore.svelte'
	import Btn from './Btn.svelte'


	const draw = writable(false);
	const stateCanvas = writable(false)


setContext('draw', draw)
setContext('stateCanvas', stateCanvas)

</script>
<Btn />
<CanvasCore />


<style>

</style>