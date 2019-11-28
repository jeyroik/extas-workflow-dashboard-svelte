<script context="module">
	export function preload({ params, query }) {
		return this.fetch('process.JSON_RPC__ENDPOINT', {
		    method: 'POST',
		    body: JSON.stringify({method: "schema.index"})
		}).then(r => r.json()).then(schemas => {
		    schemas = schemas.result.items;
		    return this.fetch('process.JSON_RPC__ENDPOINT', {
            		    method: 'POST',
            		    body: JSON.stringify({method: "transition.index"})
            		}).then(r => r.json()).then(transitions => {
            		    transitions = transitions.result.items;
            		    return { schemas, transitions };
            		});
		});
	}
</script>

<script>
import Btn from '../../components/Button.svelte';
import { jsonRpc } from '../../components/JsonRpc.svelte';

export let schemas;
export let transitions;

function add() {
    schemas.unshift({ tmp: true, name: '', title: '', description: '', transitions: [], entity_template: ''});
    // force reactive
    schemas = schemas;
}

function addTransition(e) {
    let names = e.toElement.dataset.name.split('/');
    let schemaName = names[0];
    let transitionName = names[1];

    schemas.forEach(function (schema) {
        if (schema.name == schemaName) {
            schema.transitions.push(transitionName);
            jsonRpc(
                {
                    method: 'schema.update',
                    data: schema
                },
                function (result) {
                    el.name = '';
                    let updated = result.pop();
                    M.toast({
                        html: 'Схема "' + updated.title + '" обновлена',
                        classes: 'green'
                    });
                }
            );
        }
    });
    schemas = schemas;
}

function settings() {

}


function clear(e) {
    schemas.forEach(async function(el) {
        if (el.name == e.toElement.name) {
            jsonRpc(
                {
                    method: 'schema.delete',
                    name: e.toElement.name
                },
                function (result) {
                    let deleted = result.pop();
                    M.toast({
                        html: 'Схема "' + deleted.title + '" удалена',
                        classes: 'green'
                    });
                    schemas = schemas.filter(t => t.name);
                }
            );
        }
    });
}

function removeTransition(e) {
    let names = e.toElement.dataset.name.split('/');
    let schemaName = names[0];
    let transitionName = names[1];

    schemas.forEach(function (schema) {
        if (schema.name == schemaName) {
            schema.transitions.forEach(function (transition, index) {
                if (transition == transitionName) {
                    schema.transitions.splice(index,1);
                    jsonRpc(
                        {
                            method: 'schema.update',
                            data: schema
                        },
                        function (result) {
                            let updated = result.pop();
                            M.toast({
                                html: 'Схема "' + updated.title + '" обновлена',
                                classes: 'green'
                            });
                        }
                    );
                }
            });
        }
    });
    schemas = schemas;
}

async function create(e) {
    schemas.forEach(function(schema) {
        if (schema.name == e.toElement.name) {
            jsonRpc(
                {
                  method: 'schema.create',
                  data: {
                      name: schema.name,
                      title: schema.title,
                      description: schema.description,
                      transitions: schema.transitions,
                      entity_template: schema.entity_template
                  }
                },
                function (result) {
                    M.toast({
                        html: 'Схема "' + result.title + '" создана',
                        classes: 'green'
                    });
                    schema.tmp = null;
                    schemas = schemas.filter(t => t.name);
                }
            );
        }
    });
}

async function save(e) {
    let schema;
    schemas.forEach(function(el) {
        if (el.name == e.toElement.name) {
            schema = el;
        }
    });

    if (schema) {
        jsonRpc(
            {
              method: 'schema.update',
              data: {
                  name: schema.name,
                  title: schema.title,
                  description: schema.description,
                  transitions: schema.transitions,
                  entity_template: schema.entity_template
              }
            },
            function (result) {
                M.toast({
                    html: 'Схема "' + result.pop().title + '" сохранена',
                    classes: 'green'
                });
                schemas = schemas.filter(t => t.name);
            }
        );
    }
}

</script>


<svelte:head>
	<title>Workflow - Схемы</title>
</svelte:head>

<h1>Список схем</h1>

<div class="">
    <Btn icon="add" title="Создать" action="{add}" round="round" tooltip="right"></Btn>
</div>

<ul class="collection">
{#each schemas as schema}
    <li class="collection-item animated bounceInLeft">
        <div class="row">
            <div class="input-field col s3">
                <input readonly="{!schema.tmp}" bind:value={schema.name} placeholder="Имя" type="text" class="validate" value="{schema.name}">
                <span class="helper-text">Имя</span>
            </div>
            <div class="input-field col s3">
                <input bind:value={schema.title} placeholder="Название" type="text" class="validate" value="{schema.title}">
                <span class="helper-text">Название</span>
            </div>
            <div class="input-field col s6">
                <input bind:value={schema.description} placeholder="Описание" type="text" class="validate" value="{schema.description}">
                <span class="helper-text">Описание</span>
            </div>
        </div>
        <div class="row">
            <div class="input-field col s6">
                <select class="validate browser-default" bind:value="{schema.entity_template}">
                    <option value="" disabled selected>Выберите шаблон сущности</option>

                </select>
                <span class="helper-text">Выберите Шаблон сущности</span>
            </div>
        </div>
        {#each transitions as transition}
            {#if schema.transitions.indexOf(transition.name) >= 0}
                <div class="row">
                    <div class="input-field col s3">
                        <input readonly bind:value={transition.title} placeholder="Название" type="text" class="validate" value="{transition.title}">
                        <span class="helper-text">Название</span>
                    </div>
                    <div class="input-field col s6">
                        <input readonly bind:value={transition.description} placeholder="Описание" type="text" class="validate" value="{transition.description}">
                        <span class="helper-text">Описание</span>
                    </div>
                    <div class="input-field col s3">
                        <Btn name="{schema.name + '/' + transition.name}" tooltip="top" color="red" icon="close" title="Удалить из схемы" action="{removeTransition}" round=""></Btn>
                        <Btn tooltip="top" icon="settings" title="Настройки" action="{settings}" round=""></Btn>
                    </div>
                </div>
            {/if}
        {/each}
        {#each transitions as transition}
            {#if schema.transitions.indexOf(transition.name) < 0}
                <div class="row">
                    <div class="input-field col s3">
                        <input readonly bind:value={transition.title} placeholder="Название" type="text" class="validate" value="{transition.title}">
                        <span class="helper-text">Название</span>
                    </div>
                    <div class="input-field col s6">
                        <input readonly bind:value={transition.description} placeholder="Описание" type="text" class="validate" value="{transition.description}">
                        <span class="helper-text">Описание</span>
                    </div>
                    <div class="input-field col s3">
                        <Btn name="{schema.name + '/' + transition.name}" tooltip="top" color="green" icon="add" title="Добавить в схему" action="{addTransition}" round=""></Btn>
                    </div>
                </div>
            {/if}
        {/each}
        <div class="row">
            {#if schema.tmp}
                <Btn color="green" text="Создать" title="Создать" tooltip="top" name="{schema.name}" action="{create}"></Btn>
            {:else}
                <Btn color="green" text="Сохранить" title="Сохранить" tooltip="top" name="{schema.name}" action="{save}"></Btn>
            {/if}
            <Btn color="red" text="Х" title="Удалить" tooltip="top" name="{schema.name}" action="{clear}"></Btn>
        </div>
	</li>
{/each}
</ul>