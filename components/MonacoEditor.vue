<template>
  <div class="container">
    <div class="editor"></div>
    <button v-if="!loader" @click="saveValue">Run Code</button>
  </div>
</template>

<script>
import loader from "@monaco-editor/loader";

export default {
  data() {
    return {
      jsonValue: "",
      editorInstance: {},
      loader: true,
    };
  },
  created() {
    loader.init().then((monaco) => {
      this.loader = false;
      let editor = monaco.editor.create(document.querySelector(".editor"), {
        theme: "vs-dark",
        language: "json",
        minimap: false,
        value: "console.log('hellow')",
      });

      function createDependencyProposals(range) {
        // returning a static list of proposals, not even looking at the prefix (filtering is done by the Monaco editor),
        // here you could do a server side lookup
        return [
          {
            label: '"lodash"',
            kind: monaco.languages.CompletionItemKind.Property,
            documentation: "The Lodash library exported as Node.js modules.",
            insertText: '"lodash": "*"',
            range: range,
          },
          {
            label: '"express"',
            kind: monaco.languages.CompletionItemKind.Property,
            documentation: "Fast, unopinionated, minimalist web framework",
            insertText: '"express": "*"',
            range: range,
          },
          {
            label: '"mkdirp"',
            kind: monaco.languages.CompletionItemKind.Function,
            documentation: "Recursively mkdir, like <code>mkdir -p</code>",
            insertText: '"mkdirp": "*"',
            range: range,
          },
          {
            label: '"my-third-party-library"',
            kind: monaco.languages.CompletionItemKind.Function,
            documentation: "Describe your library here",
            insertText: '"${1:my-third-party-library}": "${2:1.2.3}"',
            insertTextRules:
              monaco.languages.CompletionItemInsertTextRule.InsertAsSnippet,
            range: range,
          },
        ];
      }

      monaco.languages.registerCompletionItemProvider("json", {
        provideCompletionItems: function (model, position) {
          // find out if we are completing a property in the 'dependencies' object.
          let textUntilPosition = model.getValueInRange({
            startLineNumber: 1,
            startColumn: 1,
            endLineNumber: position.lineNumber,
            endColumn: position.column,
          });
          let match = textUntilPosition.match(
            /"scheme"\s*:\s*\{\s*("[^"]*"\s*:\s*"[^"]*"\s*,\s*)*([^"]*)?$/
          );
          if (!match) {
            return { suggestions: [] };
          }
          let word = model.getWordUntilPosition(position);
          let range = {
            startLineNumber: position.lineNumber,
            endLineNumber: position.lineNumber,
            startColumn: word.startColumn,
            endColumn: word.endColumn,
          };
          return {
            suggestions: createDependencyProposals(range),
          };
        },
      });

      editor.onDidChangeModelContent(() => {
        console.log(editor.getValue());
      });
    });
  },
  methods: {
    saveVal() {},
  },
};
</script>

<style>
.container {
  position: relative;
  width: 500px;
  height: 500px;
}
.editor {
  width: 100%;
  height: 100%;
}

button {
  position: absolute;
  bottom: 10px;
  right: 20px;
  padding: 0.5rem 1rem;
  border: none;
  outline: none;
  background-color: blueviolet;
  color: white;
  text-transform: uppercase;
  cursor: pointer;
}
</style>
