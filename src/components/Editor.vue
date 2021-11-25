<template>
  <div class="editor">
    <h2>It's new editor</h2>

    <button @click="getContent()">get content</button>
    <button @click="addClass()">add class</button>

    <div id="editor_container_id" class="editor_container">
      <editor
        :api-key="api_key"
        :init="config"
      />

      <div id="elem_1" class="element"></div>

    </div>
  </div>
</template>

<script>
/* global tinymce */
import Editor from '@tinymce/tinymce-vue';

const api_key = "dpqfmt24nf7cf4qxmcfnnbs6sjuee9gsybwxbu3wwni4tfmh";

export default {
  props: {
    numEditor: {  // номер данного текстового редактора
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
          'undo redo | bold italic underline strikethrough | forecolor  backcolor | \
          alignleft aligncenter alignright alignjustify | \
          bullist numlist | removeformat | pasteInput',
        setup: (editor) => {
          editor.on('init', function () {
            editor.setContent('<p class="pad">Hello world!</p>');
          });
          editor.on('Paste Change input Undo Redo', function () {

           // console.log('the content ', editor.getContent());
          });
          editor.on('blur', function () {
            console.log('on blur content ', editor.getContent());
            // console.log('the content ', editor.getContent());
          });
          editor.ui.registry.addButton('pasteInput', {
            text: 'Добавить пропуск',
            onAction: () => {
              this.setTinyContent();
            }
          });
        }
      },
      counterInputId: 0,

    }
  },

  created() {

  },

  mounted() {
    console.log('numEditor = ' + this.numEditor)
  },

  beforeDestroy() {

  },

  watch: {

  },

  methods: {
    setTinyContent() {
      // <pre id=test>Hello from Tiny!</pre>
      let content = `<input type="text" class="custom_input" id="custom_input_${this.counterInputId}" placeholder="Добавьте ответ" ref="submitBtn" value="123"/>`;
      //let content = `<div contenteditable="true" id="custom_input_${this.counter}" ref="submitBtn">Добавьте ответ</div>`;
      tinymce.activeEditor.insertContent(content);

      let addEvent = function(node, eventName, func){

        if ("undefined" == typeof node || null == node) {
          console.log('undefined nodes')
        } else {
          if (!node.ownerDocument.addEventListener && node.ownerDocument.attachEvent) {
            node.attachEvent('on' + eventName, func);
          } else node.addEventListener(eventName, func, false);
        }
      };

      let th = this;
      tinymce.activeEditor.dom.select('#custom_input_' + this.counterInputId)[0].contentEditable = 'false';
      addEvent(tinymce.activeEditor.dom.select('#custom_input_' + this.counterInputId)[0], 'click', th.testFunc);
      addEvent(tinymce.activeEditor.dom.select('#custom_input_' + this.counterInputId)[0], 'DOMNodeRemoved', th.removedFunc);
      this.counterInputId++;
    },

    testFunc($event) {
      console.log('click on element')
      console.log($event)
      console.log($event.target.id)

      let elem = document.getElementById($event.target.id);
      console.log('elem')
      console.log(elem)

      tinymce.activeEditor.dom.setAttrib('custom_input_0', 'value', 'myvalue');   // изменение значения по id
      let pos = tinymce.activeEditor.dom.getPos('custom_input_0')
      console.log('pos')
      console.log(pos)

      let elemInput0 = tinymce.activeEditor.dom.get('custom_input_0');
      let elemInput0_height = elemInput0.offsetHeight;
      console.log('elemInput0')
      console.log(elemInput0)

      let el = document.getElementsByClassName('tox-editor-header')[0];
      let notificationElem = document.getElementById('elem_1');
      notificationElem.style.top = (pos.y + el.offsetHeight + elemInput0_height) + 'px';
      notificationElem.style.left = pos.x + 'px';

      //tinymce.activeEditor.execCommand('mceReplaceContent',false, `<input type="text" id="${$event.target.id}" placeholder="Добавьте ответ" ref="submitBtn" value="000"/>`);  // робит
    },

    removedFunc($event) {
      console.log('removedFunc')
      console.log($event)
      console.log($event.target.id)
    },

    getContent() {
      console.log(tinymce.activeEditor.getContent());
    },

    addClass() {
      tinymce.activeEditor.dom.setAttrib('custom_input_0', 'class', 'custom_input custom_input__active');   // изменение класса
    }
  },

  components: {
    Editor
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
  .editor_container {
    position: relative;
    display: block;
    width: auto;
    height: auto;
  }

  .element {
    position: absolute;
    left: 0;
    top: 0;
    width: 200px;
    height: 50px;
    background-color: #cfe8ff;
    display: flex;
  }

  .custom_input {
    width: 160px;
    height: 36px;
    outline: 0;
    border: 1px solid #337dff;
    border-radius: 5px;
    padding: 10px 40px;
    transition: 0.5s ease;
    font-size: 16px;
  }

  .pad {
    font-size: 20px;
    color: red;
  }

</style>
