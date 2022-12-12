<script lang="ts">
    import { fdp, steps } from "../store";
    import { scrollToBottom } from "../utils";
    export let handler: any;
    export let step:string;
    export let actionText;
    export let disabled = false;
</script>

{#if (!$fdp || !$steps[step]) }
<button {disabled} on:click={async ()=>{
    await handler();
    setTimeout(scrollToBottom, 500);
}}>Step {step} : {actionText}!</button>
{:else}
<button disabled >Step {step} : {actionText} - done ✓ </button>
{/if}