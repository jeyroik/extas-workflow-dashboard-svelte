<script>
    export let action;
    export let title;
    export let text = '';
    export let tooltip = 'top';
    export let name = '';
    export let color = '';
    export let round = '';
    export let icon = '';
    export let size = 'btn-large';
    export let floating = 'btn-floating';
</script>

{#if round == 'round'}
    <span data-position="{tooltip}" data-tooltip="{title}" on:click="{action}" class="{color} tooltipped {floating} {size} waves-effect waves-light">
    {#if icon}
        <i class="material-icons" data-name="{name}">{icon}</i>
    {:else}
        {text}
    {/if}
    </span>
{:else}
    <button class="btn {color} waves-effect waves-light tooltipped" name="{name}" on:click="{action}" data-position="{tooltip}" data-tooltip="{title}">
    {#if icon}
        <i class="material-icons" data-name="{name}">{icon}</i>
    {:else}
        {text}
    {/if}
    </button>
{/if}