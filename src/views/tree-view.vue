<template>
  <div id="starcounter-debug-aid-tree-view">
    <div id="starcounter-debug-aid-tree-toolbar">
      <label>
        <input name="view-switch" value="tree" v-model="treeOrPlain" type="radio" checked>
        Tree
      </label>
      <label>
        <input name="view-switch" value="plain" v-model="treeOrPlain" type="radio">
        Text
      </label>
      <button v-on:click="dumbToConsole">Dump to console</button>
    </div>
    <div id="starcounter-debug-aid-tree-view-view">
      <div
        id="starcounter-debug-aid-jsoneditor-tree-view"
        v-bind:class="[treeOrPlain === 'tree' ? '' : 'hidden']"
      ></div>
      <pre v-bind:class="[treeOrPlain === 'tree' ? 'hidden' : '']">{{json}}</pre>
    </div>
  </div>
</template>

<script>
import JSONEditor from "jsoneditor";
import jsonpatch from "fast-json-patch";

export default {
  name: "tree-view",
  data() {
    return {
      json: {},
      treeOrPlain: "tree"
    };
  },
  methods: {
    dumbToConsole() {
      const cloneToRemoveProxification = JSON.parse(
        JSON.stringify(this.palindromClient.obj)
      );
      window.opener.console.info(cloneToRemoveProxification);
    },
    updateListener() {
      if (
        this.palindromClient &&
        this.palindromClient.palindrom &&
        this.palindromClient.palindrom.obj
      ) {
        const currentTree = JSON.parse(JSON.stringify(this.json));
        const nextTree = JSON.parse(JSON.stringify(this.palindromClient.obj));
        const diff = jsonpatch.compare(currentTree, nextTree);

        if (diff.length > 0) {
          this.editor.update((this.json = nextTree));
        }
      }
    }
  },
  destroyed() {
    const index = this.listener.updateListeners.indexOf(this.updateListener);
    if (index > -1) {
      this.listener.updateListeners.splice(index, 1);
    }
  },
  mounted() {
    const options = {
      history: false,
      onChange: () => {
        const currentState = this.json;
        const newState = this.editor.get();
        const patches = jsonpatch.compare(currentState, newState);
        jsonpatch.applyPatch(this.palindromClient.obj, patches);
      },
      enableTransform: false,
      enableSort: false
    };

    this.editor = new JSONEditor(
      this.$el.querySelector("#starcounter-debug-aid-jsoneditor-tree-view"),
      options
    );

    this.listener = window.opener.starcounterDebugAidListener;
    this.listener.updateListeners.push(this.updateListener);
    this.palindromClient = this.listener.getPalindromClient();
    this.json = JSON.parse(JSON.stringify(this.palindromClient.obj));

    this.editor.set(this.json);
  }
};
</script>

<style scoped>
@import "../assets/jsoneditor.css";

#starcounter-debug-aid-tree-view {
  width: 100%;
}
#starcounter-debug-aid-jsoneditor-tree-view {
  width: 100%;
}

#starcounter-debug-aid-tree-toolbar {
  margin-bottom: 4px;
  flex: 0 0 auto;
}

#starcounter-debug-aid-tree-view-view {
  flex: 1 1 auto;
  width: 100%;
  background-color: white;
  max-height: 75vh;
}

.hidden {
  display: none;
}

#plain {
  margin: 0;
  padding: 16px;
  background: #f7f7f7;
  border-radius: 3px;
  white-space: pre-wrap;
  font: 12px Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>