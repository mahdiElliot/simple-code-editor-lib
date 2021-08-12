<template>
  <!-- <div class="editor">
    <textarea
      v-model="lines"
      id="lines"
      name="lines"
      rows="30"
      disabled
      @scroll="changeScrollLeft"
      ref="line"
    />
    <textarea
      id="editor"
      name="editor"
      rows="30"
      dir="auto"
      v-model="text"
      @input="typed"
      @scroll="changeScrollRight"
      ref="editor"
    />
  </div> -->
  <div class="editor">
    <!-- <div id="lines" @scroll="changeScrollLeft" ref="line">
      <span v-for="i in lineNumber" :key="i">{{ i }}</span>
    </div> -->
    <!-- <div
      id="editor"
      contenteditable="true"
      @keydown="typed"
      @keyup="up"
      ref="editor"
      dir="auto"
      @scroll="changeScrollRight"
      spellcheck="false"
    ></div> -->
    <div id="editor" ref="editor" dir="auto" @click="clickedEditor">
      <div v-for="i in lineNumber" :key="i" :id="i" @click="clickedLine">
        <span @click="lineNumberClicked"></span>
        <div
          style="width: 100%"
          contenteditable="true"
          @keyup="up"
          @keydown="typed"
        ></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
export default Vue.extend({
  data() {
    return {
      lineNumber: 1,
      text: "",
      isSyncingLeftScroll: false,
      isSyncingRightScroll: false,
    };
  },
  methods: {
    t(el: any) {
      for (const node of el.children) {
        const s = node.innerText
          .replace(/(\/\/.*)/g, "<em>$1</em>")
          .replace(
            /\b(new|if|else|do|while|switch|for|in|of|continue|break|return|typeof|function|var|const|let|\.length|\.\w+)(?=[^\w])/g,
            "<strong>$1</strong>"
          )
          .replace(/(".*?"|'.*?'|`.*?`)/g, "<strong><em>$1</em></strong>")
          .replace(/\b(\d+)/g, "<em><strong>$1</strong></em>");
        node.innerHTML = s.split("\n").join("<br/>");
      }
    },
    caret() {
      const temp = window.getSelection();
      if (temp !== null) {
        const range = temp.getRangeAt(0);
        const prefix = range.cloneRange();
        prefix.selectNodeContents(this.$refs.editor as any);
        prefix.setEnd(range.endContainer, range.endOffset);
        return prefix.toString().length;
      }
      return null;
    },
    setCaret(pos: any) {
      for (const node of (this.$refs.editor as any).childNodes) {
        if (node.nodeType == Node.TEXT_NODE) {
          if (node.length >= pos) {
            const range = document.createRange();
            const sel = window.getSelection();
            range.setStart(node, pos);
            range.collapse(true);
            sel?.removeAllRanges();
            sel?.addRange(range);
            return -1;
          } else {
            pos = pos - node.length;
          }
        } else {
          pos = this.setCaret(pos);
          if (pos < 0) {
            return pos;
          }
        }
      }
      return pos;
    },
    async typed(e: any) {
      if (
        (this.$refs.editor as any).firstChild.id !==
          document.activeElement?.parentElement?.id &&
        e.keyCode === 8
      ) {
        if (document.activeElement?.textContent === "") {
          e.preventDefault();
          const active = document.activeElement;
          (
            document.activeElement.parentElement?.previousSibling
              ?.lastChild as any
          ).focus();
          await active.parentElement?.remove();
          // this.lineNumber = (this.$refs.editor as any).children.length;
        }
      }
      if (e.keyCode === 13) {
        await this.lineNumber++;
        (
          document.activeElement?.parentNode?.nextSibling?.lastChild as any
        ).focus();
        e.preventDefault();
      }
    },
    up(e: any) {
      if (e.keyCode === 13) {
        e.preventDefault();
      }
    },
    clickedEditor(e: any) {
      if (e.target.id === "editor")
        (this.$refs.editor as any).lastChild.lastChild.focus();
    },
    clickedLine(e: any) {
      if (e.target.lastChild !== null) e.target.lastChild.focus();
    },
    lineNumberClicked(e: any) {
      e.target.nextSibling.focus();
    },
    changeScrollRight() {
      if (!this.isSyncingRightScroll) {
        this.isSyncingLeftScroll = true;
        (this.$refs.line as any).scrollTop = (
          this.$refs.editor as any
        ).scrollTop;
      }
      this.isSyncingRightScroll = false;
    },
    changeScrollLeft() {
      if (!this.isSyncingLeftScroll) {
        this.isSyncingRightScroll = true;
        (this.$refs.editor as any).scrollTop = (
          this.$refs.line as any
        ).scrollTop;
      }
      this.isSyncingLeftScroll = false;
    },
  },
  mounted() {
    (this.$refs.editor as any).firstChild?.nextSibling?.focus();
  },
});
</script>

<style lang="scss">
.editor {
  width: 100%;
  height: 640px;
  display: flex;
  border-style: outset;
  border-width: 8px;
  border-color: rgb(143, 143, 143);
  background: black;
}
#editor {
  cursor: text;
  width: 100%;
  height: 95%;
  outline: none;
  overflow-y: auto;
  resize: none;
  line-height: 1.5;
  font-size: 2em;
  color: white;
  counter-reset: line;
  position: relative;
}

#editor div {
  outline: none;
  // border: 2px solid white;
  display: flex;
  padding-left: 1rem;
  padding-right: 1rem;
}

#editor div div {
  padding-left: 0;
}

#editor div span {
  width: 48px;
  // border-right: 1px solid white;
}

#editor div span::before {
  content: counter(line);
  counter-increment: line;
  color: white;
}

// #editor div::before {
//   content: counter(line);
//   counter-increment: line;
//   color: white;
//   margin-right: 16px;
// }

#lines {
  width: 48px;
  height: 95%;
  font-size: 2em;
  overflow-y: scroll;
  -ms-overflow-style: none; /* Internet Explorer 10+ */
  scrollbar-width: none; /* Firefox */
  background: rgb(141, 141, 141);
  color: white;
  line-height: 1.5;
  padding: 1rem;
  outline: none;
  border-right: 6px inset rgb(105, 105, 105);

  text-align: center;
  resize: none;
  opacity: 0.7;
  display: flex;
  flex-direction: column;
}

#lines span {
  content: counter(line);
  counter-increment: line;
  color: white;
  margin-right: 16px;
}

#lines::-webkit-scrollbar {
  display: none; /* Safari and Chrome */
}
</style>