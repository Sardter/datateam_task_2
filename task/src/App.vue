<template>
  <div id="app">
    <h1>Text Selection and Clipboard Demo</h1>
    <div class="text-fields">
      <div>
        <h2>Text Field 1</h2>
        <textarea
          ref="textArea1"
          v-model="text1"
          @mouseup="handleSelection('textArea1')"
          @keydown="handleKeydown($event, 'textArea1')"
          placeholder="Type your formulas here"
        ></textarea>
        <button @click="toggleAppendMode('textArea1')">
          {{ appendMode1 ? "Disable Append Mode" : "Enable Append Mode" }}
        </button>
      </div>
      <div>
        <h2>Text Field 2</h2>
        <textarea
          ref="textArea2"
          v-model="text2"
          @mouseup="handleSelection('textArea2')"
          @keydown="handleKeydown($event, 'textArea2')"
          placeholder="Type your formulas here"
        ></textarea>
        <button @click="toggleAppendMode('textArea2')">
          {{ appendMode2 ? "Disable Append Mode" : "Enable Append Mode" }}
        </button>
      </div>
    </div>
    <button @click="copyToClipboard">Copy Selected to Clipboard</button>
    <button @click="toggleCopyOnSelection()">
      {{ copyOnSelection ? "Disable Copy on Selection" : "Enable Copy on Selection" }}
    </button>
    <button @click="toggleCopyOnKey()">
      {{ copyOnKey ? "Disable Copy on Key" : "Enable Copy on Key" }}
    </button>
    <p>Selected Text: {{ selectedText }}</p>
    <p>Clipboard: {{ clipboard }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      text1: "AVG(balance_real)\nvar = 123", // Initial text for field 1
      text2: "SUM(revenue)\nvalue = 456", // Initial text for field 2
      selectedText: "", // Holds the selected text
      clipboard: "", // Simulates the clipboard for demo purposes
      activeTextArea: null, // Keeps track of which textarea is active
      variables: {}, // Store parsed variables
      appendMode1: false, // Toggle state for text field 1
      appendMode2: false, // Toggle state for text field 2
      copyOnSelection: true, // Controls if copy happens on selection
      copyOnKey: true, // Controls if copy happens on key press
    };
  },
  methods: {
    toggleCopyOnSelection() {
      this.copyOnSelection = !this.copyOnSelection;
    },
    toggleCopyOnKey() {
      this.copyOnKey = !this.copyOnKey;
    },
    getVariables() {
      const areas = ['textArea1', 'textArea2'];
      this.variables = {};

      areas.forEach((element) => {
        const sentences = this.$refs[element].value.split('\n');
        sentences.forEach((sentence) => {
          const assignmentMatch = sentence.match(/(\w+)\s*=\s*(\w+)/);
          if (assignmentMatch) {
            const [, key, value] = assignmentMatch;
            this.variables[key] = value;
          }
        });
      });
    },
    handleSelection(refName) {
      const textarea = this.$refs[refName];
      const start = textarea.selectionStart;
      const end = textarea.selectionEnd;
      this.getVariables();
      let selected = textarea.value.substring(start, end);

      // Replace only the standalone variables within functions like AVG(), SUM()
      Object.entries(this.variables).forEach(([key, value]) => {
        const regex = new RegExp(`\\b(${key})\\b(?!\\s*=)`, 'g');
        selected = selected.replace(regex, value);
      });

      this.selectedText = selected;
      this.activeTextArea = refName; // Update active textarea reference

      if (!this.copyOnSelection && !this.copyOnKey) return;

      if (this.appendMode1) {
        this.appendToTextArea("textArea1");
      }
      if (this.appendMode2) {
        this.appendToTextArea("textArea2");
      }
      this.copyToClipboard();
    },
    handleKeydown(event, refName) {
      // Check if Ctrl+C or Command+C is pressed
      if (!this.copyOnKey) return;
      if ((event.ctrlKey || event.metaKey) && event.key === 'c') {
        event.preventDefault(); // Prevent the default Ctrl+C behavior
        this.handleSelection(refName); // Handle the selection manually
      }
    },
    toggleAppendMode(refName) {
      if (refName === 'textArea1') {
        this.appendMode1 = !this.appendMode1;
      } else if (refName === 'textArea2') {
        this.appendMode2 = !this.appendMode2;
      }
    },
    appendToTextArea(refName) {
      if (this.selectedText) {
        if (refName === 'textArea1') {
          this.text1 += '\n' + this.selectedText;
        } else if (refName === 'textArea2') {
          this.text2 += '\n' + this.selectedText;
        }
      }
    },
    copyToClipboard() {
      if (this.selectedText) {
        this.clipboard = this.selectedText; // Simulate clipboard copy

        // Use the Clipboard API if available
        if (navigator.clipboard) {
          navigator.clipboard.writeText(this.selectedText).then(
            () => {
              console.log("Text copied to clipboard");
            },
            (err) => {
              console.error("Failed to copy text to clipboard", err);
            }
          );
        }
      }
    },
  },
};
</script>

<style scoped>
#app {
  max-width: 600px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.text-fields {
  width: 100%;
  margin-bottom: 20px;
}

textarea {
  width: 100%;
  height: 150px;
  padding: 10px;
  font-family: inherit;
  font-size: 14px;
  border-radius: 4px;
  border: 1px solid #ccc;
  box-sizing: border-box;
  resize: none;
  margin-bottom: 10px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 4px;
  border: none;
  background-color: #007bff;
  color: white;
  cursor: pointer;
  margin-bottom: 10px;
}

button:hover {
  background-color: #0056b3;
}

p {
  margin: 5px 0;
}
</style>