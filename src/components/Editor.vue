<template>
  <div class="editor">
    <editor
      :api-key="api_key"
      :init="config"
    />

    <div class="modal_container"
         v-if="(typeEditor === 1) || (typeEditor === 2)"
    >
      <div class="editor_modal"
           :class="(idOpenInputContent === inputContent.id)?('editor_modal__active'):('')"
           :id="'editor_modal_' + idEditor + '_' + inputContent.id"
           v-for="(inputContent, i) in inputsContent"
           :key="'inputs_Content_' + idEditor + '_' + i"
           v-show="idOpenInputContent === inputContent.id"
      >
        <!-- v-show="idOpenInputContent === inputContent.id"-->
        <div class="editor_item"
             v-for="(item, j) in inputContent.answers"
             :key="'inputs_Content_' + idEditor + '_' + i + '_' + j"
        >
          <div class="item_block"
               v-if="typeEditor === 1"
          >
            <input type="text"
                   class="modal__input"
                   v-model="item.text"
                   placeholder="Введите ответ"
                   @blur="updateTitleCheckbox(i, j)"
            />
          </div>
          <div class="item_block"
               v-else-if="typeEditor === 2"
          >
            <input type="radio"
                   class="custom-radio"
                   :id="'editor_elem_radio_' + idEditor + '_' + i + '_' + j"
                   :value="item.id"
                   v-model="inputContent.pickedAnswer"
                   @change="setSelectedAnswer(inputContent.pickedAnswer, i, j)"
            >
            <label :for="'editor_elem_radio_' + idEditor + '_' + i + '_' + j">
              <input type="text"
                     class="modal__input"
                     v-model="item.text"
                     placeholder="Введите ответ"
                     @blur="updateTitleCheckbox(i, j)"
              />
            </label>
          </div>

          <div class="item_delete"
               :class="(((typeEditor === 1) && (inputContent.answers.length <= 1)) || ((typeEditor === 2) && (inputContent.answers.length <= 2)))?('item_delete__none'):('')"
               @click="deleteAnswer(i, j)"
          >
            <svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M12.9376 3.75L8.99954 7.6875L5.06238 3.75L3.75 5.0625L7.68715 9L3.75 12.9375L5.06238 14.25L8.99954 10.3125L12.9376 14.25L14.25 12.9375L10.3128 9L14.25 5.0625L12.9376 3.75Z" fill="#C5C5C9"/>
            </svg>
          </div>
        </div>

        <div class="modal_button"
             @click="addNewAnswer(i)"
        >
          Добавить вариант ответа
        </div>
      </div>
    </div>

    <div class="test_buttons">
      <button @click="getContent()">get content</button>
      <button @click="addClass()">add class</button>
      <button @click="getScroll()">get scroll</button>
      <button @click="getSize()">get size</button>
    </div>
  </div>
</template>

<script>
/* global tinymce */
import Editor from '@tinymce/tinymce-vue';

const api_key = "dpqfmt24nf7cf4qxmcfnnbs6sjuee9gsybwxbu3wwni4tfmh";

export default {
  props: {
    idEditor: {  // номер данного текстового редактора
      type: Number,
      default: () => {
        return 0
      }
    },
    typeEditor: { // тип данного текстового редактора / 0 - стандартный / 1 - с блоками пропусков (список правидьных ответов) / 2 - с блоками пропусков (выбор из списка)
      type: Number,
      default: () => {
        return 0
      }
    }
  },
  name: 'HelloWorld',
  data() {
    return {
      api_key,
      config: {
        height: 500,
        menubar: false,
        statusbar: false,
        language_url: '/ru.js',
        language: 'ru',
        content_css: '/style.css',
        plugins: [
          'advlist autolink lists link image charmap print preview anchor',
          'searchreplace visualblocks code fullscreen',
          'insertdatetime media table paste code help wordcount'
        ],
        toolbar:
          'bold italic underline strikethrough | forecolor  backcolor | \
          alignleft aligncenter alignright alignjustify | \
          bullist numlist | removeformat | pasteInput',
        setup: (editor) => {
          editor.on('init', () => {
            editor.setContent('<p class="pad">Hello world!</p>');

            console.log('setTimeout')
            if ((this.typeEditor === 1) || (this.typeEditor === 2)) {
              editor.dom.setAttrib('tinymce', 'class', 'mce-content-body  custom__line_height');
            }
          });
          editor.on('Paste input click', () => {   // Change disable
            this.checkActiveModal();
           // console.log('the content ', editor.getContent());
          });
          editor.on('blur', () => {
            console.log('on blur content ', editor.getContent());
          });
          editor.on('BeforeAddUndo', function() {
            return false;
          });
          editor.on('ScrollContent', () => {
            this.checkActiveModal();
          });
          editor.ui.registry.addButton('pasteInput', {
            text: 'Добавить пропуск',
            onAction: () => {
              this.setTinyContent(editor);
            }
          });
        }
      },

      idOpenInputContent: undefined, // id открытого блока с ответами
      enableCloseInputContent: false, // разрешить закрыть открытре модальное окно
      inputsContent: [
      /*  {
          id: 0,
          pickedAnswer: undefined, // выбранный ответ, используется если typeEditor === 2
          answers: [
            {
              id: 0,
              text: 'Юрий Долгорукий'
            }
          ]
        }  */
      ]
    }
  },

  created() {

  },

  mounted() {
    console.log('idEditor = ' + this.idEditor)
  },

  beforeDestroy() {

  },

  watch: {

  },

  methods: {
    // добавить новый вариант ответа
    addNewAnswer(i) {
      this.inputsContent[i].answers.push({
        id: this.getMaxAnswerId(i) + 1,
        title: ''
      });
    },

    // удалить вариант ответа
    deleteAnswer(i, j) {
      if (((this.typeEditor === 1) && (this.inputsContent[i].answers.length > 1)) || ((this.typeEditor === 2) && (this.inputsContent[i].answers.length > 2))) {

        if ((this.typeEditor === 2) && (this.inputsContent[i].answers[j].id === this.inputsContent[i].pickedAnswer)) {
          this.inputsContent[i].pickedAnswer = undefined;
        }

        this.inputsContent[i].answers.splice(j, 1);
      }
    },

    // отправить на бек новое наименование чекбакса ответа
    updateTitleCheckbox(i, j) {
      console.log('j = ' + j + ' and i = ' + i)
    },

    // установить / снять правильный вариант ответа
    setSelectedAnswer(pickedAnswers, i, j) {
      console.log('pickedAnswers = ' + pickedAnswers);
      console.log('i = ' + i);
      console.log('j = ' + j)
    },

    setTinyContent(editor) {
      editor.focus();
      let idContent = this.getMaxId + 1;
      let content = `<input type="text" class="custom_input" id="custom_input_${this.idEditor}_${idContent}" placeholder="Добавьте ответ" ref="submitBtn" value=""/>`;
    //  let content = `<div contenteditable="true" id="custom_input_${this.counter}" ref="submitBtn">Добавьте ответ</div>`;
      tinymce.activeEditor.insertContent(content);

      let th = this;
      tinymce.activeEditor.dom.select('#custom_input_' + this.idEditor + '_' + idContent)[0].contentEditable = 'false';
      this.addEvent(tinymce.activeEditor.dom.select('#custom_input_' + this.idEditor + '_' + idContent)[0], 'click', th.visibleContentModal);
      this.addEvent(tinymce.activeEditor.dom.select('#custom_input_' + this.idEditor + '_' + idContent)[0], 'DOMNodeRemoved', th.removedContentModal);

      let answers = [];

      if (this.typeEditor === 1) {
        Object.assign(answers, [
          {
            id: 0,
            text: ''
          }
        ])
      } else if (this.typeEditor === 2) {
        Object.assign(answers, [
          {
            id: 0,
            text: ''
          },
          {
            id: 1,
            text: ''
          }
        ])
      }

      this.inputsContent.push({
        id: idContent,
        pickedAnswer: undefined, // выбранный ответ, используется если typeEditor === 2
        answers: answers
      });

      console.log(this.inputsContent)
    },

    // функция добавления события на элемент tinymce редактора
    addEvent(node, eventName, func) {
      if ("undefined" == typeof node || null == node) {
        console.log('undefined nodes')
      } else {
        if (!node.ownerDocument.addEventListener && node.ownerDocument.attachEvent) {
          node.attachEvent('on' + eventName, func);
        } else node.addEventListener(eventName, func, false);
      }
    },

    // переключить видимость модального окна с массивом ответов
    visibleContentModal($event) {
      this.enableCloseInputContent = false;

      let idElem = $event.target.id;
      let pos = tinymce.activeEditor.dom.getPos(idElem);

      let elemInput = tinymce.activeEditor.dom.get(idElem);
      let elemInputHeight = elemInput.offsetHeight;

      let distanceToTop = Number(elemInput.getBoundingClientRect().top);
      let el = document.getElementsByClassName('tox-editor-header')[0];

      let idText = 'custom_input_' + this.idEditor + '_';
      let idContent = Number(idElem.slice(idText.length, idElem.length));
      let notificationElem = document.getElementById('editor_modal_' + this.idEditor + '_' + idContent);

      notificationElem.style.top = (distanceToTop + el.offsetHeight + elemInputHeight) + 'px';
      notificationElem.style.left = pos.x + 'px';

      if (this.idOpenInputContent === idContent) {   // закрыть модальное окно
        this.idOpenInputContent = undefined;
        tinymce.activeEditor.dom.setAttrib(idElem, 'class', 'custom_input');
      } else {        // открыть модальное окно
        tinymce.activeEditor.dom.setAttrib(idElem, 'class', 'custom_input custom_input__active');

        this.idOpenInputContent = idContent;
        setTimeout(() => {
          this.enableCloseInputContent = true;
        }, 100)
      }

      // tinymce.activeEditor.dom.setAttrib('custom_input_0', 'value', 'myvalue');   // изменение значения по id

      //tinymce.activeEditor.execCommand('mceReplaceContent',false, `<input type="text" id="${$event.target.id}" placeholder="Добавьте ответ" ref="submitBtn" value="000"/>`);  // робит
    },

    // проверка на открытые модальные окна
    checkActiveModal() {
      if ((this.idOpenInputContent !== undefined) && this.enableCloseInputContent) { // если открыто, закрыть
        let idElem = 'custom_input_' + this.idEditor + '_' + this.idOpenInputContent;
        tinymce.activeEditor.dom.setAttrib(idElem, 'class', 'custom_input');
        this.idOpenInputContent = undefined;
        this.enableCloseInputContent = false;
      }
    },

    // удаление блока с ошибками
    removedContentModal($event) {
      console.log('removedFunc')
      console.log($event)
      console.log($event.target.id);
    //  let nameId = $event.target.id;

    },

    getContent() {
      console.log(tinymce.activeEditor.getContent());
    },

    addClass() {
      tinymce.activeEditor.dom.setAttrib('custom_input_0', 'class', 'custom_input custom_input__active');   // изменение класса
    },

    getScroll() {
      let elemDoc = tinymce.activeEditor.dom.get('body');
      console.log(elemDoc);

      console.log(tinymce.activeEditor.getWin().pageYOffset())
    },

    getSize() {
      let elemInput0 = tinymce.activeEditor.dom.get('custom_input_0');
      console.log('elemInput0.offsetTop = ' + elemInput0.offsetTop);
      console.log('elemInput0.clientHeight  = ' + elemInput0.clientHeight);
      var distanceToTop = elemInput0.getBoundingClientRect().top;

      console.log('distanceToTop = ' + distanceToTop);

      let pos = tinymce.activeEditor.dom.getPos('custom_input_0');
      console.log('pos');
      console.log(pos)
    },

    // возраващает максимальный id из массива ответов inputsContent
    getMaxAnswerId(k) {
      let maxId = -Infinity;

      if (this.inputsContent.length > 0) {
        this.inputsContent[k].answers.forEach((item) => {
          if (item.id > maxId) {
            maxId = item.id;
          }
        });
      }

      if (maxId === -Infinity) {
        maxId = -1;
      }

      return maxId
    }
  },

  computed: {
    // возраващает максимальный id из массива inputsContent
    getMaxId() {
      let maxId = -Infinity;

      this.inputsContent.forEach((item) => {
        if (item.id > maxId) {
          maxId = item.id;
        }
      });

      if (maxId === -Infinity) {
        maxId = -1;
      }

      return maxId
    }
  },

  directives: {
    clickOutside: {
      bind(el, binding) {
        el.addEventListener('click', e => e.stopPropagation());
        document.body.addEventListener('click', binding.value);
      },
      unbind(el, binding) {
        document.body.removeEventListener('click', binding.value);
      }
    }
  },

  components: {
    Editor
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
  @import "../styles_fonts.scss";

  $font_Geometria: Geometria, 'Geometria';

  $color_background: #F6F6FB;
  $color_Gray_1: #C5C5C9;
  $color_Gray_2: #ABABB2;
  $color_Gray_3: #8D909F;
  $color_Gray_4: #EBEBEB;
  $color_Gray_5: #F2F2F2;
  $color_Gray_6: #8D909F;
  $color_Gray_7: #F9F9F9;
  $color_black: #333333;
  $color_purple: #8C71FE;
  $color_purple_2: #F2F1FF;
  $color_purple_3: #E2E2F6;
  $color_purple_4: #7B61EB;
  $color_purple_5: #6F57D6;
  $color_red: #FF6597;
  $color_green: #89CE54;
  $color_grey: #AEBCD6;
  $color_cyan: #C0CADC;
  $color_cyan_2: #AEBCD6;
  $color_cyan_3: #D1DBED;

  $color_whiteHover: #fafafa;

  $transition_medium: 0.5s ease;
  $transition_fast: 0.25s ease;

  $path_img_checkbox: '/lk/2021/img/save.svg';
  $path_img_checkbox_focus: '/image/checkImageCheckbox.svg';

  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */ {
    opacity: 0;
  }

  .fadeFast-enter-active, .fadeFast-leave-active {
    transition: opacity .25s ease-in;
  }
  .fadeFast-enter, .fadeFast-leave-to /* .fade-leave-active до версии 2.1.8 */ {
    opacity: 0;
  }

  .editor {
    position: relative;
    display: block;
    width: auto;
    height: auto;
    font-family: $font_Geometria;
    color: $color_black;

    .tox-tbtn__select-label {
      font-family: $font_Geometria;
    }

    .tox-tinymce {
      font-family: $font_Geometria;
    }
  }

  .editor_modal {
    display: flex;
    flex-direction: column;
    position: absolute;
    left: 0;
    top: 0;
    width: 360px;
    height: auto;
    background-color: white;
    z-index: 5;
    border: 1px solid $color_Gray_4;
    box-shadow: 0px 6px 8px -6px rgba(24, 39, 75, 0.12), 0px 8px 16px -6px rgba(24, 39, 75, 0.08);
    border-radius: 3px;
    transition: $transition_medium;
  }

  .editor_item {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    height: 56px;
    border-bottom: 1px solid rgba(0, 0, 0, 0.08);
    padding: 0 12px 0 16px;
  }

  .item_block {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    margin-right: 8px;
    width: 100%;
  }

  .item_delete {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    cursor: pointer;

    svg {
      transition: $transition_medium;
    }

    svg path {
      transition: $transition_medium;
    }

    &:hover {
      svg path {
        fill: $color_red;
      }
    }
  }

  .item_delete__none {
    cursor: default;

    svg path {
      fill: rgba(0, 0, 0, 0.08);
    }

    &:hover {
      svg path {
        fill: rgba(0, 0, 0, 0.08);
      }
    }
  }

  .tox .tox-tbtn {
    cursor: pointer;

    .tox-tbtn__select-label {
      cursor: pointer;
    }
  }

  .custom-checkbox, .custom-radio {
    position: absolute;
    z-index: -1;
    opacity: 0;
  }

  .custom-checkbox+label, .custom-radio+label {
    display: inline-flex;
    align-items: center;
    user-select: none;
    width: 100%;
  }
  .custom-radio+label::before {
    content: '';
    display: inline-block;
    width: 22px;
    height: 22px;
    flex-shrink: 0;
    flex-grow: 0;
    border: 1px solid rgba(0, 0, 0, 0.08);
    border-radius: 50%;
    margin-right: 10px;
  //  background-repeat: no-repeat;
    background-position: center center;
    background-size: 16px 16px;
    transition: $transition_medium;
    cursor: pointer;

    &:hover {
      border-color: $color_Gray_2;
    }
  }

  .custom-checkbox+label::before {
    content: '';
    display: inline-block;
    width: 22px;
    height: 22px;
    flex-shrink: 0;
    flex-grow: 0;
    border: 1px solid rgba(0, 0, 0, 0.08);
    border-radius: 4px;
    margin-right: 10px;
    background-repeat: no-repeat;
    background-position: center center;
    background-size: 16px 16px;
    transition: $transition_medium;
    cursor: pointer;

    &:hover {
      border-color: $color_Gray_2;
    }
  }


  .custom-checkbox:checked+label::before {
    border-color: $color_purple;
    background-color: $color_purple;
    background-image: url($path_img_checkbox_focus);
  }

  .custom-radio:checked+label::before {
    border-color: $color_purple;
    background-color: $color_purple;
    background-image: url($path_img_checkbox_focus);
  }

  .modal__input {
    height: 28px;
    outline: none;
    border: 1px solid rgba(0, 0, 0, 0.08);
    border-radius: 3px;
    padding: 0 28px 0 8px;
    transition: $transition_medium;
    background-repeat: no-repeat;
    background-position: right 5px center;
    font-size: 15px;
    line-height: 19px;
    font-family: $font_Geometria;
    cursor: text;
    width: 100%;

    &:hover {
      border-color: $color_Gray_2;
    }

    &:focus {
      border-color: $color_purple;
    }
  }

  .modal_button {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    margin: 14px 16px;
    cursor: pointer;
    border: 1px solid $color_cyan_3;
    box-sizing: border-box;
    border-radius: 3px;
    height: 40px;
    font-weight: 500;
    font-size: 15px;
    line-height: 19px;
    color: $color_cyan;
    transition: $transition_medium;

    &:hover {
      color: $color_purple;
      border-color: $color_purple;
    }
  }

</style>
