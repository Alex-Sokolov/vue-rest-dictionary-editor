# vue-rest-dictionary-editor

Vue-компонент для быстрой реализации страничек администрирования различных списков, когда работа должна осуществляться по REST.

## Зависимости
- axios (для запросов)
- alertify (для уведомлений)
- vue-form-generator (для генерации формы по схеме)

## Использование

``` vue
<template>
	<dictionary-editor title="Список уведомлений" :rest-url="restUrl" :id-prop="idProp" :name-prop="nameProp" :response-prop="responseProp" :schema="schema" :add-to-start="true">
	</dictionary-editor>
</template>

<script>
import DictionaryEditor from 'components/dictionary-editor.vue';

export default {
	components: {
		DictionaryEditor
	},
	data: () => ({
		// REST URL
		restUrl: location.href,

		// Имя свойства c ID элементов
		idProp: 'id',

		// Имя свойства с названием элементов
		nameProp: 'name',

		// Имя свойства в котором приходит список элементов
		responseProp: 'list',

		// Список полей, отображаемых при редактировании элемента
		schema: {
			fields: [
				{
					type: 'input',
					inputType: 'text',
					label: 'Название элемента',
					model: 'summary',
					default: 'Название нового элемента'
				},
				{
					type: 'input',
					inputType: 'text',
					label: 'Дата создания',
					model: 'created',
					readonly: true
				}
			]
		}
	})
}
</script>

```

## Опции

**restUrl**
URL REST-endpoint, относительно которой строятся другие адреса

**idProp**
Название свойства, в котором будет храниться уникальный ID элемента

**nameProp**
Название свойства, в котором будет храниться название элемента

**responseProp**
Название свойства, в котором будет храниться список элементов при получении списка с сервера
Если не указан, список будет сохраняться из самого response

**addedProp**
Название свойства, в котором будет храниться флаг, что элемент добавлен, но ещё не сохранялся (по умолчанию `isAdded`)

**title**
Заголовок, отображаемый над списком элементов

**schema**
Схема для vue-form-generator для генерации формы редактирования элементов.
https://icebob.gitbooks.io/vueformgenerator/content/schema.html

**addToStart**
Флаг, добавлять ли новые элементы в начало массива (по умолчанию `false`)

## Дополнительные слоты

### sidebar-item
Можно использовать для переопределения шаблона отображения элемента в списке. Слоту передаётся входными параметрами элемент:
``` vue
<dictionary-editor title="Список сотрудников" :rest-url="restUrl" :id-prop="idProp" :name-prop="nameProp" :response-prop="responseProp" :schema="schema" :add-to-start="true">
	<template slot="sidebar-item" scope="props">
		{{ props.last_name }} {{ props.name }} {{ props.middle_name }}
	</template>
</dictionary-editor>
```

### above-sidebar
Можно использовать например для добавления дополнительных кнопок над списком

### under-sidebar
Можно использовать например для добавления дополнительных кнопок под списком

### header
Слот для добавления дополнительной функциональности над формой редактирования элемента. Слоту передаётся входными параметрами элемент

#### footer
Слот для добавления дополнительной функциональности над формой редактирования элемента. Слоту передаётся входными параметрами элемент

