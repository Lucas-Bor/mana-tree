<template>
  <div>
    <main>
      <h1 class="main-title">Arbre du mana vivant (v1.0)</h1>
      <ul class="class-list">
        <class-list
          v-for="classType in data.classes"
          v-bind:classType="classType"
          v-bind:key="classType.id"
          v-on:change-class="data.currentClass = classType.id"
          v-bind:constants="data.constants"
          v-bind:currentClass="data.currentClass"
        ></class-list>
      </ul>
      <div class="talent-toolbar">
        <div class="talent-info">
          <p class="talent-info-stat">Niveau de classe requis : {{ requiredLevel }}</p>
        </div>
      </div>
      <class-panel
        v-bind:class-type="data.classes[data.currentClass]"
        v-bind:constants="data.constants"
      ></class-panel>
      <talent-path v-bind:currentClass="data.classes[data.currentClass]"></talent-path>
    </main>
    <div class="centerx">
      <vs-popup
        class="popup"
        :background-color-popup="colorx"
        title="Importa Talenti"
        :active.sync="popupImportActive"
      >
        <textarea-autosize
          placeholder="Incolla il contenuto da importare..."
          v-model="importMessage"
          :min-height="50"
          :max-height="350"
        ></textarea-autosize>
        <button class="button" @click="popupImportMessage">
          <i class="material-icons md-18">cloud_download</i> Importé
        </button>
      </vs-popup>
      <vs-popup
        class="popup"
        :background-color-popup="colorx"
        title="Esporta Talenti"
        :active.sync="popupExportActive"
      >
        <textarea-autosize
          ref="textToexport"
          v-model="exportMessage"
          :min-height="30"
          :max-height="350"
        ></textarea-autosize>
        <button class="button" @click="copyTextArea">
          <i class="material-icons" style="vertical-align: middle">assignment</i> Copia negli Appunti
        </button>
      </vs-popup>
      <vs-popup
        class="popup"
        :background-color-popup="colorx"
        title="Esporta Talenti"
        :active.sync="popupExportChoiceActive"
      >
        <div class="popup-btns">
          <button class="button" @click="popupExportMessageSingle">Classe Corrente</button>
          <button class="button" @click="popupExportMessageAll">Tutte le Classi</button>
        </div>
      </vs-popup>
    </div>
  </div>
</template>
<script>
import talentData from "../assets/data/talent-data.json";
import classList from "./ClassList";
import classPanel from "./ClassPanel";
import talentPath from "./TalentPath";
let impexpjson = require("jsonpack/main");
let base64 = require("js-base64").Base64;

export default {
  components: {
    classList,
    classPanel,
    talentPath
  },
  data: function() {
    return {
      data: talentData,
      colorx: "#4a5153",
      popupImportActive: false,
      popupExportActive: false,
      popupExportChoiceActive: false,
      importMessage: "",
      exportMessage: ""
    };
  },
  mounted: function() {
    let firstDotIndex = window.location.hostname.indexOf(".");
    document.domain = window.location.hostname.substring(firstDotIndex + 1);
    window.localStorage.removeItem("talentsDataImported");
    let localDataSaved = window.localStorage.getItem("talentsDataSaved");
    if (localDataSaved != null) {
      this.data = JSON.parse(localDataSaved);
    }
  },
  computed: {
    skillPoints: function() {
      return this.data.classes[this.data.currentClass].availableSkillPoints;
    },
    requiredLevel: function() {
      return this.data.classes[this.data.currentClass].requiredLevel;
    },
  },
  methods: {
    popupImportMessage: function() {
      let importedData = impexpjson.unpack(base64.decode(this.importMessage));
      let joinedData = this.data;
      this.popupImportActive = false;
      this.importMessage = "";
      if (importedData.import == "single") {
        joinedData.currentClass = importedData.currentClass;
        joinedData.classes[importedData.currentClass] =
          importedData.classes[importedData.currentClass];
        window.localStorage.removeItem("talentsDataImported");
        window.localStorage.setItem(
          "talentsDataImported",
          JSON.stringify(joinedData)
        );
      } else {
        window.localStorage.removeItem("talentsDataImported");
        window.localStorage.setItem(
          "talentsDataImported",
          JSON.stringify(importedData)
        );
      }
      this.data = JSON.parse(
        window.localStorage.getItem("talentsDataImported")
      );
    },
    popupExportMessageSingle: function() {
      let base64 = require("js-base64").Base64;
      let exportedData = this.data;
      let classData = exportedData.classes[this.data.currentClass];
      this.popupExportChoiceActive = false;
      this.popupExportActive = true;
      exportedData.import = "single";
      for (let i = 0; i < exportedData.classes.length; i++) {
        if (i !== classData.id) {
          delete exportedData.classes[i].talentPath;
          delete exportedData.classes[i].talentTrees;
        }
      }
      this.exportMessage = base64.encode(impexpjson.pack(exportedData));
    },
    popupExportMessageAll: function() {
      let base64 = require("js-base64").Base64;
      let exportedData = this.data;
      exportedData.import = "all";
      this.popupExportChoiceActive = false;
      this.popupExportActive = true;
      this.exportMessage = base64.encode(impexpjson.pack(exportedData));
    },
    copyTextArea: function() {
      let el = this.$refs.textToexport.$el;
      let elCount = this.$refs.textToexport.$el.value.length;
      let oldContentEditable = el.contentEditable,
        oldReadOnly = el.readOnly,
        range = document.createRange();
      let s = window.getSelection();

      el.contentEditable = true;
      el.readOnly = false;
      range.selectNodeContents(el);

      s.removeAllRanges();
      s.addRange(range);

      el.setSelectionRange(0, elCount);

      el.contentEditable = oldContentEditable;
      el.readOnly = oldReadOnly;

      document.execCommand("Copy");
      this.popupExportActive = false;
    },
    saveTalentTrees: function() {
      let localData = JSON.stringify(this.data);
      window.localStorage.setItem("talentsDataSaved", localData);
    }
  }
};
</script>
