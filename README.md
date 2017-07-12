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
					label: 'Текст уведомления',
					model: 'summary',
					default: 'Текст нового уведомления'
				},
				{
					type: 'input',
					inputType: 'text',
					label: 'Создано',
					model: 'created_at',
					readonly: true
				},
				{
					type: 'multiselect',
					label: 'Выбранные аккаунты',
					model: 'accounts',
					idProp: 'id',
					titleProp: 'email'
				}
			]
		}
	})
}
</script>

```

## Опции

## Дополнительные слоты

- above-sidebar
- under-sidebar
- header
- footer
