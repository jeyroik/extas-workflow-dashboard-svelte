<script context="module">
	export function preload({ params, query }) {
        return this.fetch(`process.JSON_RPC__ENDPOINT`, {
            method: 'POST',
            body: JSON.stringify({method: "transition.index"})
        }).then(r => r.json()).then(transitions => {
            transitions = transitions.result.items;
            return { transitions };
        });
	}
</script>

<script>
import Btn from '../../components/Button.svelte';
import { jsonRpc } from '../../components/JsonRpc.svelte';

export let transitions;

function add() {
    transitions.unshift({ tmp: true, name: '', title: '', description: '', state_from: '', state_to: ''});
    // force reactive
    transitions = transitions;
}

function clear(e) {
    transitions.forEach(async function(el) {
        if (el.name == e.toElement.name) {
            jsonRpc(
                {
                    method: 'transition.delete',
                    name: e.toElement.name
                },
                function (result) {
                    el.name = '';
                    let deleted = result.pop();
                    M.toast({
                        html: 'Переход "' + deleted.title + '" удалён',
                        classes: 'green'
                    });
                    transitions = transitions.filter(t => t.name);
                }
            );
        }
    });
}

function create (e) {
    transitions.forEach(function(transition) {
        if (transition.name == e.toElement.name) {
            jsonRpc(
                {
                  method: 'transition.create',
                  data: {
                      name: transition.name,
                      title: transition.title,
                      description: transition.description,
                      state_from: transition.state_from,
                      state_to: transition.state_to
                  }
                },
                function (result) {
                    M.toast({
                        html: 'Переход "' + result.title + '" создан',
                        classes: 'green'
                    });
                    transition.tmp = null;
                    transitions = transitions.filter(t => t.name);
                }
            );
        }
    });
}

function save(e) {
    let transition;
    transitions.forEach(function(el) {
        if (el.name == e.toElement.name) {
            transition = el;
        }
    });

    if (transition) {
        jsonRpc(
            {
              method: 'transition.update',
              data: {
                  name: transition.name,
                  title: transition.title,
                  description: transition.description,
                  state_from: transition.state_from,
                  state_to: transition.state_to
              }
            },
            function (result) {
                M.toast({
                    html: 'Переход "' + result.pop().title + '" обновлён',
                    classes: 'green'
                });
                transitions = transitions.filter(t => t.name);
            }
        );
    }
}
</script>


<svelte:head>
	<title>Workflow - Переходы</title>
</svelte:head>

<h1>Список переходов</h1>

<div class="">
    <Btn tooltip="right" icon="add" title="Создать" action="{add}" round="round"></Btn>
</div>

<ul class="collection">
{#each transitions as transition}
    <li class="collection-item animated bounceInLeft">
        <div class="row">
            <div class="input-field col s3">
                <input readonly="{!transition.tmp}" bind:value={transition.name} placeholder="Имя" type="text" class="validate" value="{transition.name}">
                <span class="helper-text">Имя</span>
            </div>
            <div class="input-field col s3">
                <input bind:value={transition.title} placeholder="Название" type="text" class="validate" value="{transition.title}">
                <span class="helper-text">Название</span>
            </div>
            <div class="input-field col s6">
                <input bind:value={transition.description} placeholder="Описание" type="text" class="validate" value="{transition.description}">
                <span class="helper-text">Описание</span>
            </div>
        </div>
        <div class="row">
            <div class="input-field col s6">
                <input bind:value={transition.state_from} placeholder="Описание" type="text" class="validate" value="{transition.state_from}">
                <span class="helper-text">Из состояния</span>
            </div>
            <div class="input-field col s6">
                <input bind:value={transition.state_to} placeholder="Описание" type="text" class="validate" value="{transition.state_to}">
                <span class="helper-text">В состояние</span>
            </div>
        </div>
        <div class="row">
            {#if transition.tmp}
                <Btn color="green" text="Создать" title="Создать" tooltip="top" name="{transition.name}" action="{create}"></Btn>
            {:else}
                <Btn color="green" text="Сохранить" title="Сохранить" tooltip="top" name="{transition.name}" action="{save}"></Btn>
            {/if}
            <Btn color="red" text="Х" title="Удалить" tooltip="top" name="{transition.name}" action="{clear}"></Btn>
        </div>
	</li>
{/each}
</ul>