<script context="module">
	export async function preload({ params, query }) {
        const res0 = await this.fetch('process.JSON_RPC__ENDPOINT', {
          method: 'POST',
          headers: {
              "x-extas-expand": "schema.transitions,schema.entity"
          },
          body: JSON.stringify({method: "schema.index"})
        });
        const res1 = await this.fetch('process.JSON_RPC__ENDPOINT', {
          method: 'POST',
          body: JSON.stringify({method: "transition.index"})
        });

        const res2 = await this.fetch('process.JSON_RPC__ENDPOINT', {
            method: 'POST',
            body: JSON.stringify({method: "entity.template.index"})
        });

        const res3 = await this.fetch('process.JSON_RPC__ENDPOINT', {
             method: 'POST',
             body: JSON.stringify({method: "transition.dispatcher.template.index"})
        });

        const schemas = await res0.json().then(r => r.result.items);
        const transitions = await res1.json().then(r => r.result.items);
        const templates = await res2.json().then(r => r.result.items);
        const dispatchersTemplates = await res3.json().then(r => r.result.items);

        return { schemas, transitions, templates, dispatchersTemplates };
	}
</script>

<script>
import Btn from '../../components/Button.svelte';
import { jsonRpc } from '../../components/JsonRpc.svelte';
import Modal from '../../components/transitions/ModalDispatcherCreate.svelte';

export let schemas;
export let transitions;
export let templates;
export let dispatchersTemplates;

let lastSelectedItem;

function add() {
    schemas.unshift({
        tmp: true,
        name: '',
        title: '',
        description: '',
        transitions: [],
        entity_template: {
            name: '',
            title: '',
            description: ''
        }
    });

    schemas = schemas;
}

function addTransition(schema, transition) {
    if (schema.tmp) {
        schema.transitions.push(transition);
        schemas = schemas;
    } else {
        jsonRpc(
            {
                method: "schema.transition.add",
                schema_name: schema.name,
                transition_name: transition.name,
                dispatchers: []
            },
            () => {
                M.toast({
                    html: 'Переход "' + transition.title + '" добавлен',
                    classes: 'green'
                });
                schemas.forEach(schemaItem => {
                    if (schemaItem.name == schema.name) {
                        schemaItem.transitions.push(transition);
                    }
                });
                schemas = schemas;
                schemasMissedTransitions[schema.name].forEach((missedTransition, i) => {
                    console.log('[i] = ' + i);
                    if (missedTransition.name == transition.name) {
                        console.log('i = ' + i);
                        schemasMissedTransitions[schema.name].splice(i, 1);
                    }
                });
                schemasMissedTransitions = schemasMissedTransitions;
            }
        );
    }
}

function removeTransition(schema, transition) {
    if (schema.tmp) {
        schema.transitions.forEach((t, i) => {
            if (t.name == transition.name) {
                schema.transitions.splice(i, 1);
            }
        });
        schemas = schemas;
    } else {
        jsonRpc({
            method: "schema.transition.remove",
            schema_name: schema.name,
            transition_name: transition.name
        },
        () => {
            M.toast({
                html: 'Переход "' + transition.title + '" удалён',
                classes: 'green'
            });
            schemas.forEach(schemaItem => {
                if (schemaItem.name == schema.name) {
                    schemaItem.transitions.forEach((t, i) => {
                        if (t.name == transition.name) {
                            schemaItem.transitions.splice(i, 1);
                        }
                    })
                }
            });
            schemas = schemas;

            schemasMissedTransitions[schema.name].unshift(transition);
            schemasMissedTransitions = schemasMissedTransitions;
        });
    }
}

function settings(schema, transition) {
    schemas.forEach(s => {
        if (s.name == schema.name) {
            s.transitions.forEach(t => {
                if (t.name == transition.name) {
                    // вот тут надо сбегать на апишку и подтащить список диспетчеров
                    jsonRpc(
                        {
                            method: "transition.dispatcher.index",
                            filter: {
                                schema_name: {"$equal" : s.name},
                                transition_name: {"$equal": t.name}
                            }
                        },
                        result => {
                            t.dispatchers = result.items;
                            t.setting = !t.setting;
                            t.opened = t.opened ? t.opened : {};
                            schemas = schemas;
                        }
                    );
                }
            })
        }
    });
}

function openTransitionDispatcherTemplate(schema, transition, template) {
    schemas.forEach(s => {
        if (s.name == schema.name) {
            s.transitions.forEach(t => {
                if (t.name == transition.name) {
                    t.opened[template.name] = !t.opened[template.name];
                    console.log('open template, transition:',t);
                }
            })
        }
    });
    schemas = schemas;
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

async function create(schema) {
    let curSchemaTransitions = [];
    schema.transitions.forEach(t => {
        curSchemaTransitions.push(t.name);
    });
    jsonRpc(
        {
          method: 'schema.create',
          data: {
              name: schema.name,
              title: schema.title,
              description: schema.description,
              transitions: curSchemaTransitions,
              entity_template: schema.entity_template.name
          }
        },
        function (result) {
            M.toast({
                html: 'Схема "' + result.title + '" создана',
                classes: 'green'
            });
            schema.tmp = null;
            schemas = schemas.filter(t => t.name);
            schemasMissedTransitions[schema.name] = transitions.filter(t => {
                schema.transitions.forEach(schemaTransition => {
                    if (schemaTransition.name == t.name) {
                        return false;
                    }
                });
                return true;
            });
        }
    );
}

function removeDispatcher(schema, transition, dispatcher)
{
    jsonRpc(
        {
            method: 'transition.dispatcher.delete',
            name: dispatcher.name
        },
        () => {
             M.toast({
                html: 'Обработчик удалён',
                classes: 'green'
            });
            schemas.forEach(s => {
                if (s.name == schema.name) {
                    s.transitions.forEach(t => {
                        if (t.name == transition.name) {
                            t.dispatchers.forEach((d,i) => {
                                if (d.name == dispatcher.name) {
                                    t.dispatchers.splice(i, 1);
                                    schemas = schemas;
                                }
                            });
                        }
                    });
                }
            });
        }
    );
}

async function save(schema) {
    jsonRpc(
        {
          method: 'schema.update',
          data: {
              name: schema.name,
              title: schema.title,
              description: schema.description,
              entity_template: schema.entity_template.name
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

let schemasMissedTransitions = {};
schemas.forEach(schema => {
    schemasMissedTransitions[schema.name] = transitions.filter(t => {
        schema.transitions.forEach(schemaTransition => {
            if (schemaTransition.name == t.name) {
                return false;
            }
        });
        return true;
    });
});

let buttons = [
    {
        title: "Добавить",
        description: "Добавить данный обработчик для текущего перехода",
        color: "green",
        action: (e, curSchema, curTransition, curDispatcherTemplate) => {
            const index = curTransition.dispatchers ? curTransition.dispatchers.length : 0;
            const newDispatcher = {
                  name: curSchema.name + '__' + curTransition.name + '__' + curDispatcherTemplate.name + '__' + index,
                  template: curDispatcherTemplate.name,
                  schema_name: curSchema.name,
                  transition_name: curTransition.name,
                  type: curDispatcherTemplate.type,
                  parameters: curDispatcherTemplate.parameters
            };
            jsonRpc(
                {
                    method: 'transition.dispatcher.create',
                    data: newDispatcher
                },
                () => {
                    M.toast({
                        html: 'Обработчик добавлен',
                        classes: 'green'
                    });
                    schemas.forEach(s => {
                        if (s.name == curSchema.name) {
                            s.transitions.forEach(t => {
                                if (t.name == curTransition.name) {
                                    t.dispatchers = t.dispatchers ? t.dispatchers : [];
                                    t.dispatchers.push(newDispatcher);
                                    schemas = schemas;
                                }
                            });
                        }
                    });
                }
            );
        }
    }
];
let opened = false;
let modalBody = '';
let modalDT = {};
let modalSchema = {};
let modalTransition = {};
</script>


<svelte:head>
	<title>Workflow - Схемы</title>
</svelte:head>

<h1>Список схем</h1>

<div class="">
    <Btn icon="add" title="Создать" action="{add}" round="round" tooltip="right"></Btn>
</div>

<Modal bind:schema={modalSchema} bind:transition={modalTransition} bind:dispatcherTemplate={modalDT} bind:opened={opened} header="Настройки перехода" buttons="{buttons}"></Modal>
<ul class="collection">
{#each schemas as schema}
    <li class="collection-item animated bounceInLeft">
        <div class="row">
            <div class="input-field col s3">
                <input readonly="{!schema.tmp}" bind:value={schema.name} placeholder="Имя" type="text" value="{schema.name}">
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
                    <option value="" disabled>Выберите шаблон сущности</option>
                    <option selected value="{schema.entity_template.name}" title="{schema.entity_template.description}">{schema.entity_template.title}</option>
                    {#each templates as template}
                        {#if schema.entity_template.name != template.name}
                            <option value="{template}" title="{template.description}">{template.title}</option>
                        {/if}
                    {/each}
                </select>
                <span class="helper-text">Выберите Шаблон сущности</span>
            </div>
        </div>
        {#each schema.transitions as transition}
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
                    <Btn name="" tooltip="top" color="red" icon="close" title="Удалить из схемы" action="{e => removeTransition(schema, transition)}" round=""></Btn>
                    <Btn tooltip="top" icon="settings" title="Настройки" action="{e => settings(schema, transition)}" round=""></Btn>
                </div>
            </div>
            {#if transition.setting}
                <div class="row animated bounceInLeft">
                    <ul class="collection with-header">
                        {#if transition.dispatchers}
                            <li class="collection-item active">Добавленные обработчики</li>
                            {#each transition.dispatchers as dispatcher}
                                {#each dispatchersTemplates as dt}
                                    {#if dt.name == dispatcher.template}
                                        <li class="collection-item">
                                            <div class="row">{dt.title} <span class="secondary-content">{dt.description}</span> </div>
                                            <div class="row">
                                                <ul class="collection">
                                                {#each dispatcher.parameters as param}
                                                    {#if param.name == 'field_name'}
                                                        {#each schema.entity_template.parameters as etParam}
                                                            {#if etParam.name == param.value}
                                                                <li class="collection-item" title="{param.description}">{param.title}: {etParam.title}</li>
                                                            {/if}
                                                        {/each}
                                                    {:else if param['template@' + param.template]}
                                                        {#each param['template@' + param.template].allow as allowParam}
                                                            {#if allowParam.name == param.value}
                                                                <li class="collection-item" title="{param.description}">{param.title}: {allowParam.title}</li>
                                                            {/if}
                                                        {/each}
                                                    {:else}
                                                        <li class="collection-item" title="{param.description}">{param.title}: {param.value}</li>
                                                    {/if}
                                                {/each}
                                                </ul>
                                            </div>
                                            <div class="row">
                                                <span class="secondary-content"><Btn icon="delete" color="red" text="Х" title="Удалить" tooltip="top" action="{e => removeDispatcher(schema, transition, dispatcher)}"></Btn></span>
                                            </div>
                                        </li>
                                    {/if}
                                {/each}
                            {/each}
                        {/if}
                        <li class="collection-item active">Добавить новый обработчик</li>
                        {#each dispatchersTemplates as dispatcherTemplate}
                            <li class="collection-item li-button">
                                <div on:click="{e => {modalDT = dispatcherTemplate; modalSchema = schema; modalTransition = transition; opened = !opened;}}">
                                    {dispatcherTemplate.title}<span class="secondary-content">{dispatcherTemplate.description}</span>
                                </div>
                                {#if transition.opened[dispatcherTemplate.name]}
                                    <div>
                                        <ul class="collection with-header">
                                        {#each dispatcherTemplate.parameters as parameter}
                                            <li class="collection-item">

                                                    <input placeholder="{parameter.name}" type="text" class="validate" value="">
                                                    <span class="helper-text">{parameter.name}</span>

                                            </li>
                                        {/each}
                                        </ul>
                                    </div>
                                {/if}
                            </li>
                        {/each}
                    </ul>
                </div>
            {/if}
        {/each}

        <div class="row">
            <div class="input-field col s12">
                <select bind:value={lastSelectedItem} style="display: initial" on:change="{e => addTransition(schema, lastSelectedItem)}" title="Выберите переход">
                    <option value="" disabled selected>Выберите переход</option>
                    {#if schema.tmp}
                        {#each transitions as transition}
                            <option value="{transition}">{transition.title} [{transition.name}]</option>
                        {/each}
                    {:else}
                        {#each schemasMissedTransitions[schema.name] as transition}
                            <option value="{transition}">{transition.title} [{transition.name}]</option>
                        {/each}
                    {/if}
                </select>
                <span class="helper-text">Добавить переход</span>
              </div>
        </div>

        <div class="row">
            {#if schema.tmp}
                <Btn color="green" text="Создать" title="Создать" tooltip="top" name="{schema.name}" action="{e => create(schema)}"></Btn>
            {:else}
                <Btn color="green" text="Сохранить" title="Сохранить" tooltip="top" name="{schema.name}" action="{e => save(schema)}"></Btn>
            {/if}
            <Btn color="red" text="Х" title="Удалить" tooltip="top" name="{schema.name}" action="{clear}"></Btn>
        </div>
	</li>
{/each}
</ul>

<style>
    .li-button {
        cursor: pointer;
    }
</style>