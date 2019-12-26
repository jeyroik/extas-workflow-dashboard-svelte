<script context="module">
	export function preload({ params, query }) {
		return this.fetch('process.JSON_RPC__ENDPOINT', {
		    method: 'POST',
		    body: JSON.stringify({method: "entity.template.index"})
		}).then(r => r.json()).then(entityTemplates => {
		    entityTemplates = entityTemplates.result.items;
			return { entityTemplates };
		});
	}
</script>

<script>
import Btn from '../../components/Button.svelte';
import { jsonRpc } from '../../components/JsonRpc.svelte';

export let entityTemplates;

function add() {
    entityTemplates.unshift({
        tmp: true,
        name: '',
        title: '',
        description: '',
        class: 'extas\\components\\workflows\\entities\\WorkflowEntity',
        parameters: []
    });
    // force reactive
    entityTemplates = entityTemplates;
}

function addParam(entityTemplate) {
    entityTemplate.parameters.unshift({
        name: '',
        title: '',
        description: ''
    });

    entityTemplates = entityTemplates;
}

function clear(entityTemplate) {
    jsonRpc(
        {
            method: 'entity.template.delete',
            name: entityTemplate.name
        },
        result => {
            el.name = '';
            let deleted = result.pop();
            M.toast({
                html: 'Шаблон "' + deleted.title + '" удалён',
                classes: 'green'
            });
            entityTemplates = entityTemplates.filter(t => t.name);
        }
    );
}

function clearParam(entityTemplate, param) {
    entityTemplate.parameters.forEach(p => {
        if (p.name == param.name) {
            p.deleted = true;
        }
    })
    entityTemplate.parameters = entityTemplate.parameters.filter(t => !t.deleted);
    save(entityTemplate);
    //entityTemplates = entityTemplates;
}

function create (e) {
    entityTemplates.forEach(function(entityTemplate) {
        if (entityTemplate.name == e.toElement.name) {
            jsonRpc(
                {
                  method: 'entity.template.create',
                  data: {
                      name: entityTemplate.name,
                      title: entityTemplate.title,
                      description: entityTemplate.description,
                      class: entityTemplate.class,
                      parameters: entityTemplate.parameters
                  }
                },
                result => {
                    M.toast({
                        html: 'Шаблон "' + result.title + '" создан',
                        classes: 'green'
                    });
                    entityTemplate.tmp = null;
                    entityTemplates = entityTemplates.filter(t => t.name);
                }
            );
        }
    });
}

function save(entityTemplate) {
    jsonRpc(
        {
          method: 'entity.template.update',
          data: {
              name: entityTemplate.name,
              title: entityTemplate.title,
              description: entityTemplate.description,
              class: entityTemplate.class,
              parameters: entityTemplate.parameters
          }
        },
        result => {
            let tmpl = result.pop();
            M.toast({
                html: 'Шаблон "' + tmpl.title + '" обновлён',
                classes: 'green'
            });
            entityTemplates = entityTemplates.filter(t => t.name);
        }
    );
}
</script>

<svelte:head>
	<title>Workflow - Шаблоны сущностей</title>
</svelte:head>

<h1>Список шаблонов сущностей</h1>

<div class="">
    <Btn tooltip="right" icon="add" title="Создать" action="{add}" round="round"></Btn>
</div>

<ul class="collection">
{#each entityTemplates as entityTemplate}
    <li class="collection-item animated bounceInLeft">
        <div class="row">
            <div class="input-field col s6">
                <input readonly="{!entityTemplate.tmp}" bind:value={entityTemplate.name} placeholder="Имя" type="text" class="validate" value="{entityTemplate.name}">
                <span class="helper-text">Имя</span>
            </div>
            <div class="input-field col s6">
                <input bind:value={entityTemplate.title} placeholder="Название" type="text" class="validate" value="{entityTemplate.title}">
                <span class="helper-text">Название</span>
            </div>

        </div>
        <div class="row">
            <div class="input-field col s6">
                <input bind:value={entityTemplate.description} placeholder="Описание" type="text" class="validate" value="{entityTemplate.description}">
                <span class="helper-text">Описание</span>
            </div>
            <div class="input-field col s6">
                <input bind:value={entityTemplate.class} placeholder="Класс" type="text" class="validate" value="{entityTemplate.class}">
                <span class="helper-text">Описание</span>
            </div>
        </div>

        <div class="row">
            <Btn color="blue" text="Создать параметр" title="Создать параметр" tooltip="top" action="{e => addParam(entityTemplate)}"></Btn>
        </div>

        {#each entityTemplate.parameters as param}
            <div class="row">
                <div class="input-field col s3">
                    <input bind:value={param.name} placeholder="Имя" type="text" class="validate" value="{param.name}">
                    <span class="helper-text">Имя</span>
                </div>
                <div class="input-field col s3">
                    <input bind:value={param.title} placeholder="Название" type="text" class="validate" value="{param.title}">
                    <span class="helper-text">Название</span>
                </div>
                <div class="input-field col s3">
                    <input bind:value={param.description} placeholder="Описание" type="text" class="validate" value="{param.description}">
                    <span class="helper-text">Описание</span>
                </div>
                <div class="input-field col s3">
                    <Btn color="red" text="Х" title="Удалить" tooltip="top" action="{e => clearParam(entityTemplate, param)}"></Btn>
                </div>
            </div>
        {/each}

        <div class="row">
            {#if entityTemplate.tmp}
                <Btn color="green" text="Создать" title="Создать" tooltip="top" name="{entityTemplate.name}" action="{create}"></Btn>
            {:else}
                <Btn color="green" text="Сохранить" title="Сохранить" tooltip="top" name="{entityTemplate.name}" action="{e => save(entityTemplate)}"></Btn>
            {/if}
            <Btn color="red" text="Х" title="Удалить шаблон сущности" tooltip="top" name="{entityTemplate.name}" action="{e => clear(entityTemplate)}"></Btn>
        </div>
	</li>
{/each}
</ul>