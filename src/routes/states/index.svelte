<script context="module">
	export function preload({ params, query }) {
		return this.fetch('process.JSON_RPC__ENDPOINT', {
		    method: 'POST',
		    body: JSON.stringify({method: "state.index"})
		}).then(r => r.json()).then(states => {
		    states = states.result.items;
			return { states };
		});
	}
</script>

<script>
import Btn from '../../components/Button.svelte';
import { jsonRpc } from '../../components/JsonRpc.svelte';

export let states;

function add() {
    states.unshift({ tmp: true, name: '', title: '', description: ''});
    // force reactive
    states = states;
}

function clear(e) {
    states.forEach(async function(el) {
        if (el.name == e.toElement.name) {
            jsonRpc(
                {
                    method: 'state.delete',
                    name: e.toElement.name
                },
                function (result) {
                    el.name = '';
                    let deleted = result.pop();
                    M.toast({
                        html: 'Состояние "' + deleted.title + '" удалено',
                        classes: 'green'
                    });
                    states = states.filter(t => t.name);
                }
            );
        }
    });
}

async function create(e) {
    states.forEach(function(state) {
        if (state.name == e.toElement.name) {
            jsonRpc(
                {
                  method: 'state.create',
                  data: {
                      name: state.name,
                      title: state.title,
                      description: state.description,
                  }
                },
                function (result) {
                    M.toast({
                        html: 'Состояние "' + result.title + '" создано',
                        classes: 'green'
                    });
                    state.tmp = null;
                    states = states.filter(t => t.name);
                }
            );
        }
    });
}

async function save(e) {
    let state;
    states.forEach(function(el) {
        if (el.name == e.toElement.name) {
            state = el;
        }
    });

    if (state) {
        jsonRpc(
            {
              method: 'state.update',
              data: {
                  name: state.name,
                  title: state.title,
                  description: state.description,
              }
            },
            function (result) {
                M.toast({
                    html: 'Состояние "' + result.pop().title + '" сохранено',
                    classes: 'green'
                });
                states = states.filter(t => t.name);
            }
        );
    }
}
</script>


<svelte:head>
	<title>Workflow - Состояния</title>
</svelte:head>

<h1>Список состояний</h1>

<div class="">
    <Btn tooltip="right" icon="add" title="Создать" action="{add}" round="round"></Btn>
</div>

<ul class="collection">
{#each states as state}
    <li class="collection-item animated bounceInLeft">
        <div class="row">
            <div class="input-field col s3">
                <input readonly="{!state.tmp}" bind:value={state.name} placeholder="Имя" type="text" class="validate" value="{state.name}">
                <span class="helper-text">Имя</span>
            </div>
            <div class="input-field col s3">
                <input bind:value={state.title} placeholder="Название" type="text" class="validate" value="{state.title}">
                <span class="helper-text">Название</span>
            </div>
            <div class="input-field col s6">
                <input bind:value={state.description} placeholder="Описание" type="text" class="validate" value="{state.description}">
                <span class="helper-text">Описание</span>
            </div>
        </div>
        <div class="row">
            {#if state.tmp}
                <Btn color="green" text="Создать" title="Создать" tooltip="top" name="{state.name}" action="{create}"></Btn>
            {:else}
                <Btn color="green" text="Сохранить" title="Сохранить" tooltip="top" name="{state.name}" action="{save}"></Btn>
            {/if}
            <Btn color="red" text="Х" title="Удалить" tooltip="top" name="{state.name}" action="{clear}"></Btn>
        </div>
	</li>
{/each}
</ul>