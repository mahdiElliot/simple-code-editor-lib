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
    <!-- <div id="editor" ref="editor" dir="auto" @click="clickedEditor">
      <div v-for="i in lineNumber" :key="i" :id="i" @click="clickedLine">
        <span @click="lineNumberClicked"></span>
        <div
          style="width: 100%"
          contenteditable="true"
          @keyup="up"
          @keydown="typed"
        ></div>
      </div>
    </div> -->
    <div id="editor" ref="editor" dir="auto" @click="clickedEditor">
      <div id="1" @click="clickedLine">
        <span @click="lineNumberClicked"></span>
        <div contenteditable="true" @keyup="up" @keydown="typed"></div>
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
    getCharacterPreceedingCaret(e: any): string {
      let sel = window.getSelection();
      if (sel) {
        if (sel.rangeCount > 0) {
          const range = sel.getRangeAt(0).cloneRange();
          range.collapse(true);
          range.setStart(e.target, 0);
          const precedingChar = range.toString().slice(-1);
          return precedingChar;
        }
      }
      return "";
    },
    deleteKey(e: any) {
      if (this.getCharacterPreceedingCaret(e) === "") {
        e.preventDefault();
        const active = document.activeElement;
        const content = active?.textContent;
        (
          document.activeElement?.parentElement?.previousSibling
            ?.lastChild as any
        ).focus();
        active?.parentElement?.remove();
        if (
          document.activeElement &&
          document.activeElement.childNodes.length
        ) {
          const len = document.activeElement.textContent?.length;
          document.activeElement.textContent =
            (document.activeElement.textContent || "") + (content || "");
          const range = document.createRange();
          const sel = window.getSelection();
          range.setStart(document.activeElement.childNodes[0], len || 1);
          range.collapse(true);
          sel?.removeAllRanges();
          sel?.addRange(range);
        }
      }
    },
    enterKey(e: any) {
      let rest = "";
      if (e.target.childNodes.length) {
        let sel = window.getSelection();
        if (sel) {
          if (sel.rangeCount > 0) {
            const range = sel.getRangeAt(0).cloneRange();
            const range2 = sel.getRangeAt(0).cloneRange();
            range.collapse(true);
            range.setEnd(
              e.target.childNodes[0],
              document.activeElement?.textContent?.length || 1
            );
            rest = range.toString();
            range2.setStart(e.target.childNodes[0], 0);
            e.target.textContent = range2.toString();
          }
        }
      }
      this.lineNumber++;
      const el = document.createElement("div");
      el.setAttribute("id", this.lineNumber + "");
      el.addEventListener("click", this.clickedLine);
      const span = document.createElement("span");
      span.addEventListener("click", this.lineNumberClicked);
      el.appendChild(span);
      const el2 = document.createElement("div");
      el2.setAttribute("contenteditable", "true");
      el2.textContent = rest;
      el2.addEventListener("keydown", this.typed);
      el2.addEventListener("keyup", this.up);
      el.appendChild(el2);
      (this.$refs.editor as any).insertBefore(
        el,
        e.target.parentNode.nextSibling
      );
      (e.target.parentNode?.nextSibling?.lastChild as any).focus();
      e.preventDefault();
    },
    upKey(e: any) {
      if (e.target.parentNode?.id === (this.$refs.editor as any).firstChild?.id)
        return;

      const sel = window.getSelection();
      const start = sel?.getRangeAt(0).startOffset || 1;
      e.target.parentNode?.previousSibling?.lastChild.focus();
      if (document.activeElement && document.activeElement.childNodes.length) {
        const len = document.activeElement.textContent?.length || 1;
        const range = document.createRange();
        const sel = window.getSelection();
        const pos = start > len ? len : start;
        range.setStart(document.activeElement.childNodes[0], pos);
        range.collapse(true);
        sel?.removeAllRanges();
        sel?.addRange(range);
      }

      e.preventDefault();
    },
    donwKey(e: any) {
      if (e.target.parentNode?.id === (this.$refs.editor as any).lastChild?.id)
        return;

      const sel = window.getSelection();
      const start = sel?.getRangeAt(0).startOffset || 1;
      e.target.parentNode?.nextSibling?.lastChild.focus();
      if (document.activeElement && document.activeElement.childNodes.length) {
        const len = document.activeElement.textContent?.length || 1;
        const range = document.createRange();
        const sel = window.getSelection();
        const pos = start > len ? len : start;
        range.setStart(document.activeElement.childNodes[0], pos);
        range.collapse(true);
        sel?.removeAllRanges();
        sel?.addRange(range);
      }
      e.preventDefault();
    },
    typed(e: any) {
      if (
        (this.$refs.editor as any).firstChild.id !==
          document.activeElement?.parentElement?.id &&
        e.keyCode === 8
      )
        this.deleteKey(e);
      else if (e.keyCode === 13) this.enterKey(e);
      else if (e.keyCode === 38) this.upKey(e);
      else if (e.keyCode === 40) this.donwKey(e);
    },
    up(e: any) {
      this.text = (this.$refs.editor as any).innerText;
    },
    clickedEditor(e: any) {
      if (e.target.id === "editor")
        (this.$refs.editor as any).lastChild.lastChild.focus();
    },
    clickedLine(e: any) {
      if (e.target.id !== "") e.target.lastChild.focus();
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
  position: relative;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  counter-reset: line;
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
  width: 100%;
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