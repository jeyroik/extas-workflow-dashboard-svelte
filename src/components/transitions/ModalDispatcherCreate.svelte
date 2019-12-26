<script>
    import Btn from '../Button.svelte';
    export let dispatcherTemplate = {};
    export let schema = {};
    export let transition = {};
    export let opened = false;
    export let header = 'Modal header';
    export let buttons = [];
    export let close = {
        title: " X ",
        description: "Close"
    };
    $: openModal = opened;
</script>

{#if openModal}
    <div id="modal1" class="modal {opened?'open':'close'}" tabindex="0" style="z-index: 1003; display: block; opacity: 1; top: 10%; transform: scaleX(1) scaleY(1);">
        <div class="modal-content" style="overflow-y: scroll; overflow-x: hidden">
            <h4>{header}</h4>
            <div class="row">
                {#each dispatcherTemplate.parameters as parameter}
                    <div class="input-field col s3" title="{parameter.description}">
                        {#if parameter.name == 'field_name'}
                            <select class="validate browser-default" bind:value="{parameter.value}">
                                <option value="" disabled>Выберите поле</option>
                                {#each schema.entity_template.parameters as etParam}
                                    <option value="{etParam.name}" title="{etParam.description}">{etParam.title}</option>
                                {/each}
                            </select>
                        {:else if parameter['template@' + parameter.template]}
                            <select class="validate browser-default" bind:value="{parameter.value}">
                                <option value="" disabled>Выберите поле</option>
                                {#each parameter['template@'+parameter.template].allow as allowParamValue}
                                    <option value="{allowParamValue.name}" title="{allowParamValue.description}">{allowParamValue.title}</option>
                                {/each}
                            </select>
                        {:else}
                            <input bind:value={parameter.value} placeholder="{parameter.title}" type="text" class="validate" value="">
                        {/if}
                        <span class="helper-text">{parameter.title}</span>
                    </div>
                {/each}
            </div>
        </div>
        <div class="modal-footer">
            {#each buttons as btn}
                <Btn  color="{btn.color}" text="{btn.title}" title="{btn.description}" tooltip="top" action="{e => btn.action(e, schema, transition, dispatcherTemplate)}"/>
            {/each}
            <Btn  color="blue" text="{close.title}" title="{close.description}" tooltip="top" action="{e => {opened = !opened}}"/>
        </div>
    </div>
    <div class="modal-overlay" style="z-index: 1002; display: block; opacity: 0.5;"></div>
{/if}