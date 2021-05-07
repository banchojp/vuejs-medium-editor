<template>
  <file-upload
    ref="upload"
    class="btn-toggle"
    :class="toggleClass"
    extensions="gif,jpg,jpeg,png,webp"
    accept="image/png,image/gif,image/jpeg,image/webp"
    :post-action="uploadUrl"
    :headers="uploadUrlHeader"
    :name="file_input_name"
    :multiple="true"
    :size="1024 * 1024 * 10"
    :data="extraData"
    v-model="insert.files"
    @input-filter="inputFilter"
    @input-file="inputFile"
  >
    写真を追加
  </file-upload>
</template>

<script>
import VueUploadComponent from "vue-upload-component";
import _ from "underscore";
export default {
  props: [
    "editor",
    "insert",
    "uploadUrl",
    "uploadUrlHeader",
    "file_input_name",
    "imgur_bool",
    "editorRef",
    "handler"
  ],
  components: {
    "file-upload": VueUploadComponent
  },
  data() {
    return {
      currentLine: null,
      currentImg: null,
      currentSize: "is-normal",
      position: {
        top: "0"
      },
      isShow: false,
      uploadedImageId: null,
      extraData: {}
    };
  },
  computed: {
    toggleClass: function() {
      if (this.insert.isToggle) {
        return "btn-toggle-show";
      } else {
        return null;
      }
    }
  },
  methods: {
    initializeExisting() {
      const handlerVm = this;
      setTimeout(() => {
        const editorImages = document
          .querySelector(".editor")
          .getElementsByClassName("editor-image");
        _.map(editorImages, elm => {
          // Set Onclick to Show Image Size Handler
          elm.onclick = function() {
            setTimeout(() => {
              handlerVm.sizingHandler(this);
            });
          };
        });
      });
    },
    addImage(url) {
      this.$emit("uploaded", url);
      if (this.insert.isToggle) {
        const handlerVm = this;
        this.editorRef.focus();
        this.editor.selectElement(this.insert.focusLine);
        this.editor.pasteHTML(
          `<div class="editor-image">
                        <img src="${url}" />
                    </div>
                    <div class="editor-image-description"><br></div>
                    <br />`,
          { cleanAttrs: [], cleanTags: [], unwrapTags: [] }
        );
        this.currentLine = this.editor.getSelectedParentElement().previousElementSibling.previousElementSibling;
        this.currentImg = this.currentLine.querySelector("img");
        const currentPos = this.currentImg.getBoundingClientRect();
        this.window.scrollTo(0, currentPos.top - currentPos.x);
        this.currentLine.onclick = function() {
          setTimeout(() => {
            handlerVm.sizingHandler(this);
          });
        };
        this.insert.isShow = true;
      }
    },
    sizingHandler(elm) {
      const className = elm.className;
      if (className.includes("is-focus")) {
        // do nothing
      } else {
        elm.className = className.replace(/ +$/g, "") + " is-focus";
      }
      if (className.indexOf("expand") > -1) {
        this.currentSize = "is-expand";
      } else if (className.indexOf("full") > -1) {
        this.currentSize = "is-full";
      } else {
        this.currentSize = "is-normal";
      }
      const img = elm.querySelector("img");
      this.currentLine = elm;
      this.isShow = true;
      const currentPos = img.getBoundingClientRect();
      this.position.top = currentPos.top + "px";
      this.$emit("imageClick", {
        position: this.position,
        currentLine: this.currentLine,
        isShow: this.isShow,
        currentSize: this.currentSize
      });
    },
    makeid(length) {
      var result = "";
      var characters =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
      var charactersLength = characters.length;
      for (var i = 0; i < length; i++) {
        result += characters.charAt(
          Math.floor(Math.random() * charactersLength)
        );
      }
      return result;
    },
    inputFilter(newFile, oldFile, prevent) {
      if (newFile && !oldFile) {
        if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.name)) {
          return prevent();
        }
        if (/\.(php5?|html?|jsx?)$/i.test(newFile.name)) {
          return prevent();
        }
      }
    },
    inputFile(newFile, oldFile) {
      if (newFile && !oldFile) {
        this.$refs.upload.active = true;

        const reader = new FileReader();
        reader.onload = e => {
          this.editorRef.focus();
          this.editor.selectElement(this.insert.focusLine);
          this.editor.pasteHTML(
            '<div class="editor-image"><img class="uploading-overlay" id="' +
              this.uploadedImageId +
              '" src="' +
              e.target.result +
              '" /></div><br/>',
            {
              cleanAttrs: [],
              cleanTags: [],
              unwrapTags: []
            }
          );
          const currentPos = document
            .getElementById(this.uploadedImageId)
            .getBoundingClientRect();
          this.window.scrollTo(0, currentPos.bottom);
          this.insert.isShow = true;

          this.uploadedImageId = this.makeid(5);
          this.extraData = { image_id: this.uploadedImageId };
        };
        reader.readAsDataURL(newFile.file);
      }

      // Image Upload Successful
      if (newFile && newFile.success) {
        if (this.imgur_bool) {
          document.getElementById(newFile.response.data.image_id).src =
            newFile.response.data.link;
          document
            .getElementById(newFile.response.data.image_id)
            .classList.remove("uploading-overlay");
          this.$emit("uploaded", newFile.response.data.link);
          // Add ImagePosition handler
          this.currentLine = document.getElementById(
            newFile.response.data.image_id
          ).parentElement;
          const handlerVm = this;
          this.currentImg = this.currentLine.querySelector("img");
          this.currentLine.onclick = function() {
            setTimeout(() => {
              handlerVm.sizingHandler(this);
            });
          };
        } else {
          this.uploadedImage = newFile.response.url;
          this.$emit("uploaded", newFile.response.url);
        }
      } else if (newFile && newFile.active && newFile.response.error) {
        this.$emit("upload-error", newFile.response.data.message);
        var imgElem = document.getElementById(newFile.response.data.image_id);
        if (imgElem && imgElem.parentNode) {
          imgElem.parentNode.parentNode.removeChild(imgElem.parentNode);
        }
      }
    }
  },
  created() {
    this.uploadedImageId = this.makeid(5);
    this.extraData = { image_id: this.uploadedImageId };
  },
  mounted() {
    this.initializeExisting();
  },
  beforeMount() {
    this.window = window;
    window.addEventListener("scroll", this.handleScroll);
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.handleScroll);
  }
};
</script>

<style>
.uploading-overlay {
  opacity: 0.75;
}
</style>
