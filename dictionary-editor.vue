<!-- Компонент для админок справочников -->

<template>
  <div class="row">
    <div class="col-md-4">

      <div class="panel panel-default">
        <div class="panel-body">

          <slot name="above-sidebar"></slot>

          <div class="form-group" v-if="allowedAdd">
            <button type="button" class="btn btn-block btn-default" @click="add">
              <i class="fa fa-fw fa-plus"></i>
              Добавить элемент
            </button>
          </div>

          <div class="form-group" v-if="enableSearch">
            <input type="text" class="form-control" placeholder="Найти..." v-model="search">
          </div>

          <div class="panel panel-default">
            <div class="panel-heading bg-grey-salsa">
              {{ title }}
            </div>
            <div class="list-group dictionary-list-group">
              <div v-if="loading" class="list-group-item list-group-item-warning">
                Загрузка элементов...
              </div>
              <div v-else-if="!haveItems" class="list-group-item list-group-item-warning">
                Нет элементов
              </div>
              <button type="button" v-else v-for="(item, index) in filteredList" :key="item[idProp]" :class="['list-group-item', isActive(index) ? 'active' : '']" @click="select(item)" title="Выбрать элемент для редактирования">
                <slot name="sidebar-item" v-bind="item">
                  {{ item[nameProp] }}
                </slot>
              </button>
            </div>
          </div>

          <button type="button" class="btn btn-default btn-block" @click="loadDefault" v-if="defaultItemsUrl">
            Загрузить элементы по умолчанию
          </button>

          <slot name="under-sidebar"></slot>

        </div>
      </div>

    </div>
    <div class="col-md-8">

      <div v-if="!selected" class="alert alert-info">
        Выберите элемент в списке для редактирования
        <a href="#" class="new" @click.prevent="add" v-if="allowedAdd">или добавьте новый</a>
      </div>
      <div v-else class="panel panel-default">
        <div class="panel-body">

          <slot name="header" :selected="selected"></slot>

          <vue-form-generator :schema="schema" :model="selected" :options="formOptions" @validated="formValidated"></vue-form-generator>

          <slot name="footer" :selected="selected"></slot>

          <div class="btn-toolbar" role="toolbar">
            <div class="btn-group" role="group">
              <button type="button" class="btn btn-success" @click="save">
                Сохранить изменения
                <i class="fa fa-fw fa-save"></i>
              </button>
            </div>
            <div class="btn-group" role="group">
              <button type="button" class="btn btn-danger" @click="remove">
                Удалить элемент
                <i class="fa fa-fw fa-times"></i>
              </button>
            </div>
          </div>

        </div>
      </div>

    </div>
  </div>
</template>

<script>
// Используемые плагины
import axios from '~/plugins/axios.js';
import alertify from '~/plugins/alertify.js';
import VueFormGenerator from 'vue-form-generator/dist/vfg-core.js';
import 'vue-form-generator/dist/vfg-core.css';

export default {
  name: 'DictionaryEditor',
  components: {
    'vue-form-generator': VueFormGenerator.component
  },
  props: {
    // REST API URL
    restUrl: { type: String, required: true },

    // Название свойства с кодом элемента
    idProp: { type: String, required: true },

    // Название свойства с названием элемента
    nameProp: { type: String, required: true },

    // Название свойства под которым приходит список элементов
    // Если не указано, или указана пустая строка - используем весь Response
    responseProp: { type: String, default: '' },

    // Название свойства добавленного элемента (но ещё несохранённого)
    addedProp: { type: String, default: 'isAdded' },

    // Заголовок над списком
    title: { type: String, required: true },

    // Схема для генерации полей редактирования
    schema: { type: Object, required: true },

    // Добавлять новые элементы в начало массива (по умолчанию в конец)
    addToStart: { type: Boolean, default: false },

    // Возможность добавлять новые элементы
    allowedAdd: { type: Boolean, default: true },

    // URL на список стандартных элементов
    defaultItemsUrl: { type: String },

    // Название свойства стандартных элементов в response
    defaultItemProp: { type: String },

    // Флаг доступности поиска по списку
    enableSearch: { type: Boolean, default: false }
  },
  data: () => ({
    // Список элементов
    list: [],

    // Флаг что выполняется загрузка списка
    loading: true,

    // Выбранный элемент списка для редактирования
    selected: '',

    // Индекс выбранного элемента
    // Необходим для определения выбранности пустых только добавленных элементов, у которых нет ID
    selectedIndex: null,

    // Строка поиска
    search: '',

    // Флаг валидности формы
    validated: false,

    // Опции vue-form-generator
    // Для валидации формы после загрузки / после каждого изменения
    formOptions: {
      validateAfterLoad: true,
      validateAfterChanged: true
    }
  }),
  computed: {
    // Флаг наличия элементов в списке
    haveItems() {
      return this.list.length > 0;
    },

    // Список отфильтрованных по поиску элементов
    filteredList() {
      if (!this.enableSearch || !this.search) {
        return this.list;
      }

      const searchLowerCase = this.search.toLowerCase();

      // Поиск по всем свойствам элементов списка
      return this.list.filter(item => {
        return Object.keys(item).some(key => {
          const keyLowerCase = item[key]
            ? item[key].toLowerCase()
            : '';

          return keyLowerCase.includes(searchLowerCase);
        });
      });
    }
  },
  methods: {
    // Выбор элемента для отображения формы редактирования
    select(item) {
      this.selected = item;
      this.selectedIndex = this.list.findIndex(x => x === item);
    },

    // Сброс выбранного элемента
    unselect() {
      this.selected = '';
      this.selectedIndex = null;
    },

    // Обработка события валидации формы
    // меняем флаг по которому определяем заполненность полей
    formValidated(isValid, errors) {
      this.validated = isValid;
    },

    // Флаг выбранного элемента в списке
    isActive(index) {
      return this.selectedIndex === index;
    },

    // Добавление нового элемента в список
    add() {
      if (!this.allowedAdd) {
        console.error('Попытка добавить элемент в список, когда allowedAdd = false')
        return;
      }

      axios.get(`${this.restUrl}/new`)
        .then((response) => response.data)
        .then((response) => {
          // Проверяем по наличию поля с ID, что в ответе возвращается сразу объект
          if (!Object.prototype.hasOwnProperty.call(response, this.idProp)) {
            console.error('Response не содержит поле ID! Возможна лишняя вложенность', response);
            return;
          }

          // Пустая модель объекта с сервера
          const emptyModel = response;

          // Объект с значениями по умолчанию из переданной схемы
          const defaults = VueFormGenerator.schema.createDefaultObject(this.schema);

          // Комбиринуем итоговый элемент, добавляя свойство-флаг что элемент добавленный и ещё несохранённый в базе
          const model = Object.assign(emptyModel, defaults, { [this.addedProp]: true });

          // Добавляем элемент в список и делаем его выбранным
          if (this.addToStart) {
            this.list.unshift(model);
          }
          else {
            this.list.push(model);
          }

          this.select(model);
        })
        .catch((error) => alertify.alert(`Произошла ошибка при создании элемента: ${error}`));
    },

    // Извлекаем данные из указанного свойства Response
    // если не указано - используем весь Response в качестве источника данных
    getData(response) {
      return this.responseProp.length
        ? response[this.responseProp]
        : response;
    },

    // Извлекаем данные по умолчанию из указанного свойства Response
    // если не указано - используем весь Response в качестве источника данных
    getDefaultData(response) {
      return this.defaultItemProp.length
        ? response[this.defaultItemProp]
        : response;
    },

    // Загрузка списка элементов
    loadData() {
      axios.get(this.restUrl)
        .then((response) => response.data)
        .then((response) => this.getData(response))
        .then((response) => {
          if (!Array.isArray(response)) {
            console.error('Response возвращает не массив', response);
            return;
          }

          this.list = response;
          this.loading = false;
        })
        .catch((error) => alertify.alert(`Произошла ошибка при загрузке списка: ${error}`));
    },

    // Загрузка элементов по умолчанию
    loadDefault() {
      // Пустой, так как метод не ждёт аргументов, но надо добавить
      // чтобы использовать параметр конфигурации
      const data = {};

      const config = {
        validateStatus: (status) => status >= 200 && status < 300 || status === 422
      }

      axios.post(this.defaultItemsUrl, data, config)
        .then((response) => {
          // Если статус 422, значит элементы уже добавлялись
          if (response.status === 422) {
            alertify.alert('Элементы были добавлены ранее!');
            return;
          }

          // Получаем список заявок
          let defaultItems = this.getDefaultData(response.data);

          // Добавляем к существующим заявкам загруженные
          this.list = this.list.concat(defaultItems);
        })
        .catch((error) => alertify.alert(`Произошла ошибка при загрузке элементов по умолчанию: ${error}`));
    },

    // Сохранение выбранного элемента
    save() {
      if (!this.validated) {
        alertify.error('Заполните обязательные поля формы');
        return;
      }

      // Флаг что элемент добавленный и ещё несохранённый в базе
      const isAdded = this.selected[this.addedProp] === true;

      // eslint-disable-next-line no-unused-expressions
      isAdded
        ? this.saveNew()
        : this.saveExist();
    },

    // Сохранение нового элемента (несохранённого в базе)
    saveNew() {
      const idProp = this.idProp;

      axios.post(`${this.restUrl}`, this.selected)
        .then((response) => response.data)
        .then((response) => {
          // Проверяем что в ответе возвращается элемент с присвоенным ID
          if (!response) {
            console.error('При сохранении нового элемента в ответе должен возвращаться он же, но с заполненным ID');
            return;
          }

          // Проверяем наличие поля с ID в ответе, должен возвращаться сразу объект
          if (!Object.prototype.hasOwnProperty.call(response, this.idProp)) {
            console.error('В возвращаемом объекте нет поля ID! Возможна лишняя вложенность', response);
            return;
          }

          // Подставляем назначенный элементу ID из ответа
          this.selected[idProp] = response[idProp];

          // Сбрасываем флаг у элемента, что он добавленный (но несохранённый в базе)
          this.selected[this.addedProp] = false;

          alertify.success('Успешно сохранено');
        })
        .catch((error) => alertify.alert(`Произошла ошибка при сохранении изменения: ${error}`));
    },

    // Сохранение уже существовавшего элемента (сохранённого в базе)
    saveExist() {
      axios.patch(`${this.restUrl}/${this.selected[this.idProp]}`, this.selected)
        .then(() => alertify.success('Успешно сохранено'))
        .catch((error) => alertify.alert(`Произошла ошибка при сохранении изменения: ${error}`));
    },

    // Удаление выбранного элемента из списка
    remove() {
      alertify.confirm('Удалить элемент?', () => {

        // ID удаляемого элемента
        const id = this.selected[this.idProp];

        // Если удаляется новый элемент, не сохраненный в БД
        // то не нужен запрос на сервер, удаляем из списка
        if (this.selected[this.addedProp] === true) {
          this.removeItem(id);
          return;
        }

        axios.delete(`${this.restUrl}/${id}`)
          .then(() => this.removeItem(id))
          .catch((error) => alertify.alert(`Произошла ошибка при удалении: ${error}`));
      });
    },

    // Удаление элемента из локального списка
    removeItem(id) {
      this.unselect();
      this.list = this.list.filter((item) => item[this.idProp] !== id);
      alertify.success('Успешно удалено');
    }
  },
  mounted() {
    this.loadData();
  }
};
</script>

<style>
/* Список словарей (скролл с небольшим количеством видимых элементов и переносы длинных названий) */

.dictionary-list-group {
  max-height: 300px;
  overflow-y: auto;
  word-wrap: break-word;
}
</style>
