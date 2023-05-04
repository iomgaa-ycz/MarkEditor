<template>
  <div ref="editor" class="prosemirror-editor"></div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { schema } from 'prosemirror-schema-basic';
import { EditorState } from 'prosemirror-state';
import { EditorView } from 'prosemirror-view';
import { keymap } from 'prosemirror-keymap';
import { baseKeymap, toggleMark } from 'prosemirror-commands';
import { undo, redo } from 'prosemirror-history';
import { history } from 'prosemirror-history';
import { InputRule } from 'prosemirror-inputrules';
import { inputRules } from 'prosemirror-inputrules';
import { Schema } from 'prosemirror-model';

const strong = schema.marks.strong;

const strongRule = new InputRule(/\*\*(.+?)\*\*$/, (state, match, start, end) => {
  const attrs = {};
  const tr = state.tr;
  const startPos = tr.mapping.map(start);
  const endPos = tr.mapping.map(end);
  tr.addMark(startPos, endPos, strong.create(attrs));
  tr.insertText(match[0], startPos, endPos);
  return tr;
});

const unStrongRule = new InputRule(/(?:\*\*(.+?)\*|\*(.+?)\*\*)$/, (state, match, start, end) => {
  const tr = state.tr;
  const startPos = tr.mapping.map(start);
  const endPos = tr.mapping.map(end);

  // 获取匹配的文本，根据匹配的组来判断
  const matchedText = match[0];

  // 获取当前文本位置的 ResolvedPos 对象
  const resolvedPos = state.doc.resolve(startPos);

  // 获取当前文本位置的 marks
  const marksAtPos = resolvedPos.marks();

  // 检查当前选中的文本是否加粗
  const isStrong = marksAtPos.some(mark => mark.type === strong);

  // 如果文本是加粗的，那么在减少星号时取消加粗
  if (isStrong) {
    tr.removeMark(startPos, endPos, strong);
    // 插入匹配的文本
    tr.insertText(matchedText, startPos, endPos);
    return tr;
  }

  // 如果文本不是加粗的，返回 null，这样输入规则就不会删除 *
  return null;
});




const customNodes = schema.spec.nodes;
const customMarks = schema.spec.marks.addBefore('link', 'strong', { attrs: { level: { default: 1 } } });

const customSchema = new Schema({
  nodes: customNodes,
  marks: customMarks
});

export default {
  setup() {
    const editor = ref(null);
    let view = null;

    onMounted(() => {
      const state = EditorState.create({
        schema: customSchema,
        plugins: [
          keymap({
            'Mod-z': undo,
            'Mod-y': redo,
            'Mod-Shift-z': redo,
            'Mod-b': toggleStrong,
            ...baseKeymap,
          }),
          history(),
          inputRules({ rules: [strongRule, unStrongRule] }),
        ],
      });

      view = new EditorView(editor.value, {
        state,
      });
    });

    onBeforeUnmount(() => {
      if (view) {
        view.destroy();
      }
    });

    function toggleStrong() {
      return toggleMark(strong)(view.state, view.dispatch);
    }

    return { editor };
  },
};
</script>

<style scoped>
.prosemirror-editor {
  border: 1px solid gray;
  padding: 8px;
  min-height: 300px;
}

.prosemirror-editor strong {
  font-weight: bold;
}
</style>