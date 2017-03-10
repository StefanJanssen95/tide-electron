<template>
  <div class="fill">
    <div v-if="isTbinfo">
      <div class="tbinfo-header">
        <span class="tbinfo-header-title">{{tbinfoTitle}}</span>
        <div class="tbinfo-tab-container">
          <span
              class="tbinfo-tab clickable"
              v-bind:class="{'selected': showView===0}"
              v-on:click="showUiView"
          >
            UI view
          </span>
          <span
              class="tbinfo-tab clickable"
              v-bind:class="{'selected': showView===1}"
              v-on:click="showCodeView"
          >
            Code view
          </span>
        </div>
      </div>
      <div class="tbinfo-seperator"></div>
      <div class="tbinfo-editor fill" v-show="showView===0"></div>
      <div class="tbinfo-editor fill" id="editor" v-show="editorVisible"></div>
    </div>
    <div v-else>
      <div class="fill" id="editor" v-show="editorVisible"></div>
      <div class="fill" id="image-viewer" v-if="!editorVisible">
        <img :src="file.path" />
      </div>
    </div>
  </div>
</template>

<style>
  .tbinfo-header{
    display: flex;
    justify-content: space-between;
    height: 65px;
    background-color: #1E1E1E;
  }

  .tbinfo-header-title{
    display: inline-block;
    height: 65px;
    line-height: 65px;
    margin-left: 25px;

    color: #FFFFFF;
    font-size: 16px;
  }

  .tbinfo-tab-container{
    display: flex;
    align-items: flex-end;
    height: 100%;
    margin-right: 10px;
  }

  .tbinfo-tab{
    display: inline-block;
    padding: 10px 15px;

    color: rgba(255,255,255,0.5);
    font-size: 14px;
  }

  .tbinfo-tab.selected{
    background-color: #272822;

  }

  .tbinfo-seperator{
    width: 100%;
    height: 12px;
    background-color: #272822;
  }

  .tbinfo-editor.fill{
    position: absolute;
    left: 0;
    right: 0;
    top: 77px;
    bottom: 0;
  }
</style>

<script>
  import ace from 'brace';
  import 'brace/mode/python';
  import 'brace/theme/monokai';
  import { isValidUTF8 } from '../utils/utils';

  var editor = {};

  export default {
    data: function () {
      return {
        file: {type: 'file'},
        document: new ace.EditSession("Something went wrong :S","ace/mode/python"),
        showView: 0,
      };
    },
    events: {
      openFile: function(file){
        console.log("Opening editor on " + file.path);
        this.file = file;

        if (this.editorVisible) {
          this.document = this.file.editSession;
        }
      },
      saveFile: function(){
        console.log("Saving file: "+ this.file.path);
        this.file.saveTo(this.file.path);
      },
      resize: function () {
        editor.resize();
      }
    },
    computed: {
      editorVisible: function () {
        // always use the code editor when the file type is code or text
        if (this.file.type == 'code' || this.file.type == 'text') {
          return true;
        }
        // never use the editor when type is image
        if (this.file.type == 'image') {
          return false;
        }

        // otherwise, opportunistically use it if can round-trip file->editor->file
        // without any data loss
        if (this.file.readSync) {
          const fileData = this.file.readSync();
          return isValidUTF8(fileData);
        } else {
          // can't read the file
          return false;
        }
      },
      isTbinfo: function () {
          return this.file.name === 'app.tbinfo';
      },
      tbinfoTitle: function(){
          return this.showView === 0 ? "Tingbot app info" : "app.tbinfo";
      }
    },
    watch: {
      'document': function(val){
        editor = ace.edit("editor");
        editor.setTheme("ace/theme/monokai");
        editor.getSession().setMode("ace/mode/python");

        editor.setShowPrintMargin(false);
        editor.$blockScrolling = Infinity;

        editor.focus();
        editor.setSession(val);
      }
    },
    methods:{
      showUiView: function(){
        this.showView = 0;
      },
      showCodeView: function(){
        this.showView = 1;
      }
    }
 };


</script>
