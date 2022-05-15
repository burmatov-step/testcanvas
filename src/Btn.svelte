<script>
import { getContext } from 'svelte';
const draw = getContext('draw')

$: console.log('draw', $draw)

</script>

<button on:click={$draw.undo_last.bind($draw)}>Undo</button>