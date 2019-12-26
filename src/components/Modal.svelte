<script>
    export let opened = false;
    export let header = 'Modal header';
    export let body = 'Modal body';
    export let buttons = [];
    export let close = {
        title: " X ",
        description: "Close"
    };
    $: openModal = opened;
</script>

{#if openModal}
    <div id="modal1" class="modal {opened?'open':'close'}" tabindex="0" style="z-index: 1003; display: block; opacity: 1; top: 10%; transform: scaleX(1) scaleY(1);">
        <div class="modal-content">
            <h4>{header}</h4>
            <p>{body}</p>
        </div>
        <div class="modal-footer">
            {#each buttons as btn}
                <span on:click="{btn.action}" class="modal-close waves-effect waves-green btn-flat" title="{btn.description}">{btn.title}</span>
            {/each}
            <span on:click="{e => {opened = !opened}}" class="modal-close waves-effect waves-green btn-flat" title="{close.description}">{close.title}</span>
        </div>
    </div>
    <div class="modal-overlay" style="z-index: 1002; display: block; opacity: 0.5;"></div>
{/if}