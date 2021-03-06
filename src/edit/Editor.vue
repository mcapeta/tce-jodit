<template>
  <div class="jodit_wrapper">
    <jodit-vue ref="jodit" @input="input" :config="config" :value="value" />
  </div>
</template>

<script>
import { Jodit, JoditVue } from 'jodit-vue';
import AutofocusPlugin from './plugins/autofocus';
import ExternalToolbarPlugin from './plugins/external-toolbar';
import FontControlsPlugin from './plugins/font-controls';
import MdiIconsPlugin from './plugins/mdi-icons';
import pluginsAdapter from './plugins-adapter';
import SourceEditorPlugin from './plugins/source-editor';
import TablePopupsPlugin from './plugins/table-popups';
import Toolbar from '@/edit/Toolbar';
import ToolbarBuilderPlugin from './plugins/toolbar-builder';
import ToolbarPopupsPlugin from './plugins/toolbar-popups';
import TooltipPlugin from './plugins/tooltip';

// NOTE: Fixes production build issues: https://github.com/xdan/jodit/issues/225
//       caused by: https://github.com/xdan/jodit/blob/3.2.55/src/modules/helpers/checker/isJoditObject.ts#L18
Object.defineProperty(Jodit, 'name', { value: 'Jodit' });

const JODIT_READY_EVENT = 'joditReady';

/** @type {import('jodit/src/Config').Config & import('jodit/src/plugins')} */
const joditConfig = {
  autofocus: true,
  addNewLineOnDBLClick: false,
  showTooltipDelay: 350,
  colorPickerDefaultTab: 'color',
  disablePlugins: ['fullsize'],
  language: 'en'
};

pluginsAdapter(Jodit);

const plugins = [{
  use: TooltipPlugin
}, {
  use: ToolbarBuilderPlugin,
  options: {
    buttons: Toolbar.$buttons
  }
}, {
  use: ExternalToolbarPlugin,
  options: {
    readyEvent: JODIT_READY_EVENT,
    toolbarContainer: Toolbar.$containerId
  }
}, {
  use: FontControlsPlugin
}, {
  use: MdiIconsPlugin
}, {
  use: ToolbarPopupsPlugin
}, {
  use: SourceEditorPlugin
}, {
  use: TablePopupsPlugin
}, {
  use: AutofocusPlugin,
  options: {
    readyEvent: JODIT_READY_EVENT
  }
}];

export default {
  props: {
    value: { type: String, required: true },
    minHeight: { type: Number, required: true },
    placeholder: { type: String, default: 'Insert text here...' },
    readonly: { type: Boolean, default: false }
  },
  computed: {
    config: vm => ({
      ...joditConfig,
      minHeight: vm.minHeight,
      placeholder: vm.placeholder,
      plugins
    })
  },
  methods: {
    input(value) {
      const innerText = this.$refs.jodit.$el.innerText;
      return this.$emit('input', innerText ? value : '');
    }
  },
  watch: {
    readonly(state) {
      const { editor } = this.$refs.jodit;
      if (!editor) return;
      editor.setReadOnly(state);
      if (!state) editor.selection.focus();
    }
  },
  components: {
    JoditVue
  }
};
</script>

<style lang="scss" scoped>
$icon-color: #333;
$icon-size: 18px;
$statusbar-height: 26px;
$statusbar-border-size: 1px;
$min-height: 140px;
$font-family-monospace: "Menlo", "Ubuntu Mono", "Consolas", "source-code-pro", monospace;

.jodit_wrapper ::v-deep {
  .jodit_container:not(.jodit_inline) {
    min-height: $min-height;

    .jodit_workplace {
      border: none;
    }
  }

  .jodit_placeholder {
    font-style: italic;
  }

  .jodit_source .ace_editor {
    font-size: 13px;
    font-family: $font-family-monospace;
  }

  .jodit_statusbar {
    height: $statusbar-height;
    line-height: $statusbar-height - $statusbar-border-size;
    background-color: transparent;
    border: none;

    .jodit_statusbar_item {
      line-height: inherit;
    }

    .jodit_toolbar_btn {
      line-height: inherit;
      vertical-align: top;

      & > a {
        vertical-align: middle;
      }

      .jodit_icon {
        display: inline-block;
        width: $icon-size;
        height: $icon-size;
        color: $icon-color;
        font-size: $icon-size;
        line-height: $icon-size;
      }
    }
  }
}
</style>
