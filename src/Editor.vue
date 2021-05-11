<template>
  <div>
    <!-- Editor Mode -->
    <div class="medium-editor-container" v-if="!readOnly">
      <insert-embed
        v-if="editor"
        :uploadUrl="options.uploadUrl"
        :uploadUrlHeader="options.uploadUrlHeader"
        :file_input_name="options.file_input_name"
        :imgur_bool="options.imgur"
        :onChange="triggerChange"
        :editorRef="$refs.editor"
        :editor="editor"
        v-on:uploaded="uploadedCallback"
        v-on:upload-error="onUploadError"
        v-on:onLarge="onLarge"
        v-on:onSmall="onSmall"
        v-on:onRemove="onRemove"
      ></insert-embed>
      <list-handler
        v-if="editor"
        :editor="editor"
        :onChange="triggerChange"
      ></list-handler>
      <div
        class="editor"
        v-bind:class="editorClass"
        v-html="prefill"
        ref="editor"
      ></div>
    </div>
    <!-- Read Only Mode -->
    <read-mode v-if="readOnly" :content="prefill"></read-mode>
  </div>
</template>

<script>
import MediumEditor from "medium-editor";
import InsertEmbed from "./libs/InsertEmbed";
import ListHandler from "./libs/ListHandler";
import ReadMode from "./libs/ReadMode";
import _ from "underscore";
import rangy from "rangy";
import "rangy/lib/rangy-classapplier";

export default {
  name: "medium-editor",
  data() {
    const MediumEditor = require("medium-editor");
    const mediumEditorColorButtons = require("medium-editor-colorpicker-buttons-jp").get(
      MediumEditor
    );
    const TextColorButtonClass = mediumEditorColorButtons.TextColorButtonClass;

    var SmallButton = MediumEditor.extensions.button.extend({
      name: "small",

      tagNames: ["small"], // nodeName which indicates the button should be 'active' when isAlreadyApplied() is called
      contentDefault: "<b>キャプション</b>", // default innerHTML of the button
      contentFA: "<small>キャプション</small>", // innerHTML of button when 'fontawesome' is being used
      aria: "Caption", // used as both aria-label and title attributes
      action: "caption", // used as the data-action attribute of the button

      init: function() {
        MediumEditor.extensions.button.prototype.init.call(this);

        this.classApplier = rangy.createClassApplier("caption", {
          elementTagName: "small",
          normalize: true
        });
      },

      handleClick: function(event) {
        this.classApplier.toggleSelection();
        this.base.checkContentChanged();
      }
    });

    return {
      editor: null,
      defaultOptions: {
        forcePlainText: false,
        placeholder: {
          text: "Write something great!!"
        },
        uploadUrl: "https://api.imgur.com/3/image",
        uploadUrlHeader: { Authorization: "Client-ID db856b43cc7f441" },
        file_input_name: "image",
        imgur: true,
        toolbar: {
          buttons: [
            "bold",
            "italic",
            "quote",
            "h1",
            "h2",
            "h3",
            "h4",
            "h5",
            "textcolor",
            "small"
          ]
        },
        extensions: {
          textcolor: new TextColorButtonClass(/* options? */),
          small: new SmallButton()
        }
      },
      hasContent: false
    };
  },
  props: ["options", "onChange", "prefill", "readOnly"],
  computed: {
    editorOptions() {
      return _.extend(this.defaultOptions, this.options);
    },
    editorClass() {
      return {
        "has-content": this.hasContent
      };
    }
  },
  components: {
    InsertEmbed,
    ListHandler,
    ReadMode
  },
  mounted() {
    if (!this.readOnly) {
      this.createElm();
    }
  },
  methods: {
    createElm() {
      this.editor = new MediumEditor(this.$refs.editor, this.editorOptions);
      if (this.prefill) {
        if (/<[a-z][\s\S]*>/i.test(this.prefill)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
        this.$refs.editor.focus();
      }
      this.editor.subscribe("editableInput", this.triggerChange);
    },
    destroyElm() {
      this.editor.destroy();
    },
    triggerChange() {
      const content = this.editor.getContent();
      setTimeout(() => {
        if (/<[a-z][\s\S]*>/i.test(content)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
      }, 0);
      this.$emit("input", content);
      if (this.onChange) {
        this.onChange(content);
      }
    },
    uploadedCallback(url) {
      this.$emit("uploaded", url);
    },
    onUploadError(error) {
      this.$emit("upload-error", error);
    },
    onLarge(handler) {
      handler.currentLine.className = handler.currentLine.className.replace(
        "editor-image-small",
        ""
      );
    },
    onSmall(handler) {
      handler.currentLine.className = handler.currentLine.className.replace(
        "editor-image-small",
        ""
      );
      handler.currentLine.className = handler.currentLine.className.replace(
        "editor-image",
        "editor-image editor-image-small"
      );
    },
    onRemove(handler) {
      handler.currentLine.remove();
    }
  },
  destroyed() {
    this.destroyElm();
  }
};
</script>
