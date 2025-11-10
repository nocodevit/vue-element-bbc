<template>
  <div class="settings-page">
    <el-row :gutter="24" class="page-layout">
      <el-col :lg="8" :md="10" :sm="24" class="layout-left">
        <div class="phone-preview">
          <div class="phone-header">
            <span class="status-dot" />
            <span class="status-bar" />
            <span class="status-dot" />
          </div>
          <div class="phone-screen" v-if="!isPolicyScreen">
            <div v-if="isCustomPolicyEnabled && selectedLanguages.length > 0" class="preview-lang-switcher">
              <el-select v-model="previewLanguage" size="small" :popper-class="'preview-lang-popper'"
                placeholder="选择语言">
                <template #value="{ value }">
                  <span class="flag-only" v-if="value">
                    {{ languageMap[value]?.flag || '' }}
                  </span>
                  <span v-else>--</span>
                </template>
                <el-option v-for="code in selectedLanguages" :key="code" :value="code" :label="languageMap[code].flag">
                  <span class="lang-option">
                    <span class="flag">{{ languageMap[code].flag }}</span>
                    <span>{{ languageMap[code].shortLabel }}</span>
                  </span>
                </el-option>
              </el-select>
            </div>
            <div class="login-logo">Brand</div>
            <div class="login-title">欢迎回来</div>
            <div class="login-subtitle">请使用账号登录当前系统</div>
            <div class="login-input" />
            <div class="login-input" />
            <div class="login-button">登录</div>
            <div class="privacy-link" role="button" tabindex="0" @click="showPolicyScreen"
              @keyup.enter="showPolicyScreen" @keyup.space.prevent="showPolicyScreen">
              Privacy Policy
            </div>
          </div>
          <div class="phone-screen policy-screen" v-else :dir="isPolicyRtl ? 'rtl' : 'ltr'">
            <div class="policy-header">
              <el-icon class="back-icon" @click="hidePolicyScreen">
                <ArrowLeft />
              </el-icon>
              <span>{{ policyScreenTitle }}</span>
              <div v-if="isCustomPolicyEnabled && selectedLanguages.length > 0" class="preview-lang-switcher">
                <el-select v-model="previewLanguage" size="small" :popper-class="'preview-lang-popper'"
                  placeholder="选择语言">
                  <template #value="{ value }">
                    <span class="flag-only" v-if="value">
                      {{ languageMap[value]?.flag || '' }}
                    </span>
                    <span v-else>--</span>
                  </template>
                  <el-option v-for="code in selectedLanguages" :key="code" :value="code"
                    :label="languageMap[code].flag">
                    <span class="lang-option">
                      <span class="flag">{{ languageMap[code].flag }}</span>
                      <span>{{ languageMap[code].shortLabel }}</span>
                    </span>
                  </el-option>
                </el-select>
              </div>
            </div>
            <div class="policy-body" :class="{ 'rtl-content': isPolicyRtl }" v-html="activePolicyHtml" />
          </div>
        </div>
      </el-col>
      <el-col :lg="16" :md="14" :sm="24" class="layout-right">
        <el-tabs v-model="activeTab" tab-position="top" class="settings-tabs">
          <el-tab-pane label="品牌设置" name="first">
            <el-card class="tab-card" shadow="hover">
              <div class="tab-placeholder">品牌设置</div>
            </el-card>
          </el-tab-pane>
          <el-tab-pane label="登录页" name="second">
            <el-card class="tab-card" shadow="hover">
              <div class="tab-placeholder">
                登录页
                <p class="tab-subtext">信息采集配置</p>
              </div>
            </el-card>
          </el-tab-pane>
          <el-tab-pane label="隐私政策" name="third">
            <el-card class="tab-card" shadow="hover">
              <el-form label-width="120px">
                <el-form-item label="自定义策略">
                  <el-switch v-model="isCustomPolicyEnabled" active-text="开启" inactive-text="关闭" />
                </el-form-item>

                <el-form-item v-if="!isCustomPolicyEnabled" label="当前策略">
                  <el-alert type="info" show-icon title="当前使用系统默认隐私政策。开启自定义后即可编辑并覆盖默认内容。" />
                </el-form-item>

                <template v-else>
                  <el-form-item label="语言选择">
                    <el-select v-model="languageToAdd" placeholder="请选择语言" clearable @change="handleAddLanguage"
                      :disabled="selectedLanguages.length >= 3 || availableLanguageOptions.length === 0">
                      <el-option v-for="option in availableLanguageOptions" :key="option.value" :value="option.value"
                        :label="option.label">
                        <span class="lang-option">
                          <span class="flag">{{ option.flag }}</span>
                          <span>{{ option.label }}</span>
                        </span>
                      </el-option>
                    </el-select>
                  </el-form-item>

                  <el-form-item v-if="selectedLanguages.length > 0" label="语言内容">
                    <el-tabs v-model="activeLanguage" class="language-tabs">
                      <el-tab-pane v-for="code in selectedLanguages" :key="code" :name="code">
                        <template #label>
                          <span class="lang-tab-label">
                            <span class="flag">{{ languageMap[code].flag }}</span>
                            <span>{{ languageMap[code].shortLabel }}</span>
                            <el-tooltip content="设置默认回退语言：未配置的语言将使用该语言的隐私政策。" placement="top">
                              <el-icon class="default-lang" :class="{ active: defaultLanguage === code }"
                                @click.stop="setDefaultLanguage(code)">
                                <StarFilled v-if="defaultLanguage === code" />
                                <Star v-else />
                              </el-icon>
                            </el-tooltip>
                            <el-icon class="remove-lang" @click.stop="removeLanguage(code)">
                              <Delete />
                            </el-icon>
                          </span>
                        </template>
                      </el-tab-pane>
                    </el-tabs>
                  </el-form-item>

                  <el-form-item label="标题">
                    <el-input v-model="currentTitle" :placeholder="currentTitlePlaceholder" :disabled="!activeLanguage"
                      maxlength="60" show-word-limit />
                  </el-form-item>

                  <el-form-item label="编辑器工具栏">
                    <div class="toolbar">
                      <el-tooltip content="一级标题" placement="top">
                        <el-button size="small" class="toolbar-btn heading heading-1" @click="insertHeading(1)"
                          :disabled="!activeLanguage">
                          H1
                        </el-button>
                      </el-tooltip>
                      <el-tooltip content="二级标题" placement="top">
                        <el-button size="small" class="toolbar-btn heading heading-2" @click="insertHeading(2)"
                          :disabled="!activeLanguage">
                          H2
                        </el-button>
                      </el-tooltip>
                      <el-tooltip content="三级标题" placement="top">
                        <el-button size="small" class="toolbar-btn heading heading-3" @click="insertHeading(3)"
                          :disabled="!activeLanguage">
                          H3
                        </el-button>
                      </el-tooltip>
                      <el-divider direction="vertical" />
                      <el-tooltip content="加粗" placement="top">
                        <el-button size="small" class="toolbar-btn emphasis bold" @click="wrapSelection('**', '**')"
                          :disabled="!activeLanguage">
                          B
                        </el-button>
                      </el-tooltip>
                      <el-tooltip content="斜体" placement="top">
                        <el-button size="small" class="toolbar-btn emphasis italic" @click="wrapSelection('*', '*')"
                          :disabled="!activeLanguage">
                          I
                        </el-button>
                      </el-tooltip>
                      <el-tooltip content="正文段落" placement="top">
                        <el-button size="small" class="toolbar-btn paragraph" @click="insertParagraph"
                          :disabled="!activeLanguage">
                          P
                        </el-button>
                      </el-tooltip>
                    </div>
                  </el-form-item>

                  <el-form-item label="自定义内容">
                    <el-input ref="editorRef" v-model="currentPolicy" type="textarea"
                      :autosize="{ minRows: 16, maxRows: 28 }" :placeholder="currentPlaceholder"
                      :disabled="!activeLanguage" :class="{ 'rtl-textarea': isRtlLanguage }" />
                  </el-form-item>

                  <el-form-item>
                    <el-button type="primary" @click="handleSave" :loading="isSaving"
                      :disabled="!activeLanguage || !isDirty || isSaving">
                      保存
                    </el-button>
                  </el-form-item>
                </template>
              </el-form>
            </el-card>
          </el-tab-pane>
          <el-tab-pane label="套餐列表" name="fifth">
            <el-card class="tab-card" shadow="hover">
              <div class="tab-placeholder">套餐列表</div>
            </el-card>
          </el-tab-pane>
          <el-tab-pane label="支付&订阅页" name="sixth">
            <el-card class="tab-card" shadow="hover">
              <div class="tab-placeholder">支付页</div>
            </el-card>
          </el-tab-pane>
        </el-tabs>
      </el-col>
    </el-row>
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, ref, watch } from 'vue';
import MarkdownIt from 'markdown-it';
import { ArrowLeft, Delete, Star, StarFilled } from '@element-plus/icons-vue';
import { ElMessage } from 'element-plus';
import type { InputInstance } from 'element-plus';

const activeTab = ref('third');
const isPolicyScreen = ref(false);

const systemDefaultPolicy = ref<string>(
  [
    '# Privacy Policy',
    '',
    'We are committed to protecting your privacy. This policy explains how we collect, use, and store your information.',
    '',
    '## Information Collection',
    '',
    'We only collect the data necessary to deliver and improve our services.',
    '',
    '## Information Usage',
    '',
    'All collected information is used internally to enhance your experience and will never be sold to third parties.',
  ].join('\n'),
);

const languageOptions = [
  {
    value: 'zh',
    label: '中 | Chinese',
    shortLabel: '中文',
    flag: '🇨🇳',
    placeholder: '# 自定义隐私政策\n',
    headingPlaceholder: '标题',
    titlePlaceholder: '隐私政策',
    paragraphPlaceholder: '请输入正文内容',
  },
  {
    value: 'ja',
    label: '日 | Japanese',
    shortLabel: '日本語',
    flag: '🇯🇵',
    placeholder: '# カスタムプライバシーポリシー\n',
    headingPlaceholder: 'タイトル',
    titlePlaceholder: 'プライバシーポリシー',
    paragraphPlaceholder: '本文を入力してください',
  },
  {
    value: 'en',
    label: '英 | English',
    shortLabel: 'English',
    flag: '🇺🇸',
    placeholder: '# Custom Privacy Policy\n',
    headingPlaceholder: 'Heading',
    titlePlaceholder: 'Privacy Policy',
    paragraphPlaceholder: 'Enter body content',
  },
  {
    value: 'ar',
    label: '阿拉伯 | Arabic',
    shortLabel: 'العربية',
    flag: '🇸🇦',
    placeholder: '# سياسة الخصوصية المخصصة\n',
    headingPlaceholder: 'عنوان',
    titlePlaceholder: 'سياسة الخصوصية',
    paragraphPlaceholder: 'يرجى إدخال محتوى النص',
  },
];

const languageMap = languageOptions.reduce(
  (acc, option) => Object.assign(acc, { [option.value]: option }),
  {} as Record<string, (typeof languageOptions)[number]>,
);

type CustomPolicyEntry = {
  title: string;
  content: string;
};

const isCustomPolicyEnabled = ref<boolean>(false);
const languageToAdd = ref<string>('');
const customPolicies = ref<Record<string, CustomPolicyEntry>>({});
const savedCustomPolicies = ref<Record<string, CustomPolicyEntry>>({});
const activeLanguage = ref<string>('');
const defaultLanguage = ref<string>('');
const savedDefaultLanguage = ref<string>('');
const previewLanguage = ref<string>('');

const isSaving = ref(false);
const editorRef = ref<InputInstance>();

const selectedLanguages = computed<string[]>(() => Object.keys(customPolicies.value));

const availableLanguageOptions = computed(() =>
  languageOptions.filter((option) => !selectedLanguages.value.includes(option.value)),
);

const markdownRenderer = new MarkdownIt({
  html: false,
  linkify: false,
  typographer: false,
});

const currentPolicy = computed({
  get() {
    if (!activeLanguage.value) {
      return '';
    }
    const entry = customPolicies.value[activeLanguage.value];
    if (!entry) {
      return '';
    }
    return entry.content ?? '';
  },
  set(value: string) {
    if (!activeLanguage.value) {
      return;
    }
    const entry = customPolicies.value[activeLanguage.value] ?? {
      title: languageMap[activeLanguage.value]?.titlePlaceholder ?? '',
      content: '',
    };
    customPolicies.value = {
      ...customPolicies.value,
      [activeLanguage.value]: {
        ...entry,
        content: value,
      },
    };
  },
});

const currentPlaceholder = computed(() => {
  if (!activeLanguage.value) {
    return '请选择语言后开始编辑内容';
  }
  return languageMap[activeLanguage.value]?.placeholder ?? '请输入隐私政策内容';
});

const currentTitle = computed({
  get() {
    if (!activeLanguage.value) {
      return '';
    }
    const entry = customPolicies.value[activeLanguage.value];
    if (!entry) {
      return languageMap[activeLanguage.value]?.titlePlaceholder ?? '';
    }
    return entry.title ?? '';
  },
  set(value: string) {
    if (!activeLanguage.value) {
      return;
    }
    const entry = customPolicies.value[activeLanguage.value] ?? {
      title: languageMap[activeLanguage.value]?.titlePlaceholder ?? '',
      content: '',
    };
    customPolicies.value = {
      ...customPolicies.value,
      [activeLanguage.value]: {
        ...entry,
        title: value,
      },
    };
  },
});

const currentTitlePlaceholder = computed(() => {
  if (!activeLanguage.value) {
    return '请选择语言后设置标题';
  }
  return languageMap[activeLanguage.value]?.titlePlaceholder ?? 'Privacy Policy';
});

const isRtlLanguage = computed(() => activeLanguage.value === 'ar');

const previewPolicyLanguage = computed(() => {
  if (!isCustomPolicyEnabled.value) {
    return '';
  }
  return (
    previewLanguage.value || defaultLanguage.value || selectedLanguages.value[0] || ''
  );
});

const isPolicyRtl = computed(() => previewPolicyLanguage.value === 'ar');

const activePolicy = computed<string>(() => {
  if (!isCustomPolicyEnabled.value) {
    return systemDefaultPolicy.value;
  }
  const language = previewPolicyLanguage.value;
  if (language && customPolicies.value[language]) {
    return customPolicies.value[language].content;
  }
  return '';
});

const activePolicyHtml = computed<string>(() => markdownRenderer.render(activePolicy.value));

const isDirty = computed<boolean>(() => {
  if (selectedLanguages.value.length === 0) {
    return false;
  }
  return (
    JSON.stringify(customPolicies.value) !== JSON.stringify(savedCustomPolicies.value) ||
    defaultLanguage.value !== savedDefaultLanguage.value
  );
});

const activePolicyTitle = computed(() => {
  const language = previewPolicyLanguage.value;
  if (!language) {
    return 'Privacy Policy';
  }
  const entry = customPolicies.value[language];
  if (entry) {
    return entry.title || languageMap[language]?.titlePlaceholder || 'Privacy Policy';
  }
  return languageMap[language]?.titlePlaceholder || 'Privacy Policy';
});

const policyScreenTitle = computed(() => {
  if (!isCustomPolicyEnabled.value) {
    return 'Privacy Policy';
  }
  return activePolicyTitle.value;
});

watch(selectedLanguages, (langs) => {
  if (langs.length === 0) {
    activeLanguage.value = '';
    defaultLanguage.value = '';
    savedDefaultLanguage.value = '';
    previewLanguage.value = '';
    isPolicyScreen.value = false;
    return;
  }
  if (!langs.includes(activeLanguage.value)) {
    activeLanguage.value = langs[0];
  }
  if (!langs.includes(defaultLanguage.value)) {
    defaultLanguage.value = langs[0];
  }
  if (!langs.includes(previewLanguage.value)) {
    previewLanguage.value = defaultLanguage.value
      ? defaultLanguage.value
      : langs[0];
  }
});

watch(isCustomPolicyEnabled, (enabled) => {
  if (!enabled) {
    isPolicyScreen.value = false;
    previewLanguage.value = '';
  } else if (selectedLanguages.value.length > 0 && !activeLanguage.value) {
    activeLanguage.value = selectedLanguages.value[0];
  }
  if (enabled && !previewLanguage.value && selectedLanguages.value.length > 0) {
    previewLanguage.value = defaultLanguage.value || selectedLanguages.value[0];
  }
});

function showPolicyScreen() {
  isPolicyScreen.value = true;
}

function hidePolicyScreen() {
  isPolicyScreen.value = false;
}

function handleSave() {
  isSaving.value = true;

  window.setTimeout(() => {
    savedCustomPolicies.value = JSON.parse(JSON.stringify(customPolicies.value));
    savedDefaultLanguage.value = defaultLanguage.value;
    previewLanguage.value = previewPolicyLanguage.value;
    isSaving.value = false;
    ElMessage.success('隐私政策已保存');
  }, 800);
}

function handleAddLanguage(value: string) {
  if (!value || selectedLanguages.value.includes(value) || selectedLanguages.value.length >= 3) {
    languageToAdd.value = '';
    return;
  }
  customPolicies.value = {
    ...customPolicies.value,
    [value]: {
      title: languageMap[value]?.titlePlaceholder ?? '',
      content: '',
    },
  };
  savedCustomPolicies.value = {
    ...savedCustomPolicies.value,
    [value]: {
      title: languageMap[value]?.titlePlaceholder ?? '',
      content: '',
    },
  };
  activeLanguage.value = value;
  if (!defaultLanguage.value) {
    defaultLanguage.value = value;
    savedDefaultLanguage.value = value;
  }
  if (!previewLanguage.value) {
    previewLanguage.value = value;
  }
  languageToAdd.value = '';
}

function removeLanguage(code: string) {
  const { [code]: removed, ...rest } = customPolicies.value;
  const remaining = Object.keys(rest);
  customPolicies.value = rest;
  const { [code]: savedRemoved, ...savedRest } = savedCustomPolicies.value;
  savedCustomPolicies.value = savedRest;
  activeLanguage.value = remaining[0] || '';
  if (defaultLanguage.value === code) {
    defaultLanguage.value = remaining[0] || '';
  }
  if (previewLanguage.value === code) {
    previewLanguage.value = remaining[0] || defaultLanguage.value || '';
  }
  if (!defaultLanguage.value) {
    savedDefaultLanguage.value = '';
  }
}

function insertHeading(level: number) {
  const hashes = '#'.repeat(Math.min(level, 6));
  const fallback = level === 1 ? 'Heading' : 'Heading';
  const label = activeLanguage.value
    ? languageMap[activeLanguage.value]?.headingPlaceholder ?? fallback
    : fallback;
  insertContent(`${hashes} `, '', label);
}

function insertParagraph() {
  const fallback = 'Enter body content';
  const label = activeLanguage.value
    ? languageMap[activeLanguage.value]?.paragraphPlaceholder ?? fallback
    : fallback;
  insertContent('', '', label);
}

function wrapSelection(prefix: string, suffix: string) {
  insertContent(prefix, suffix, '文本');
}

function insertContent(prefix: string, suffix: string, placeholder: string) {
  const textarea = editorRef.value?.textarea;
  if (!textarea) {
    currentPolicy.value = `${prefix}${placeholder}${suffix}`;
    return;
  }

  const { selectionStart, selectionEnd, value } = textarea;
  const selectedText = value.slice(selectionStart, selectionEnd) || placeholder;
  const nextValue =
    value.slice(0, selectionStart) + prefix + selectedText + suffix + value.slice(selectionEnd);

  currentPolicy.value = nextValue;

  nextTick(() => {
    textarea.focus();
    const start = selectionStart + prefix.length;
    const end = start + selectedText.length;
    textarea.setSelectionRange(start, end);
  });
}

function setDefaultLanguage(code: string) {
  defaultLanguage.value = code;
  previewLanguage.value = code;
}
</script>

<style scoped>
.settings-page {
  padding: 32px 48px;
  background: #eef1f5;
  min-height: 100vh;
}

.page-layout {
  align-items: stretch;
  gap: 12px;
  flex-wrap: nowrap;
}

.layout-left,
.layout-right {
  display: flex;
  flex-direction: column;
  padding: 0;
}

.layout-left {
  align-items: flex-end;
}

.layout-right {
  align-items: stretch;
  width: 100%;
}

.settings-tabs {
  width: 100%;
}

.settings-tabs :deep(.el-tabs__header) {
  margin-bottom: 20px;
}

.settings-tabs :deep(.el-tabs__content) {
  overflow: visible;
}

.settings-tabs :deep(.el-tab-pane) {
  padding: 0 4px 20px;
}

.tab-card {
  border-radius: 16px;
  background: #ffffff;
  box-shadow: 0 16px 32px rgba(15, 23, 42, 0.12);
  border: 1px solid rgba(15, 23, 42, 0.05);
  overflow: hidden;
}

.tab-card :deep(.el-card__body) {
  padding: 24px 28px;
  background: inherit;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.privacy-policy-setting {
  padding-top: 8px;
}

.tab-placeholder {
  padding: 32px;
  background-color: rgba(15, 23, 42, 0.03);
  border-radius: 12px;
  color: var(--el-text-color-primary);
  font-size: 16px;
  border: 1px dashed rgba(15, 23, 42, 0.12);
}

.tab-subtext {
  margin-top: 8px;
  color: var(--el-text-color-regular);
  font-size: 14px;
}

.language-tabs {
  width: 100%;
}

.language-tabs :deep(.el-tabs__header) {
  justify-content: flex-start;
  margin-bottom: 8px;
}

.language-tabs :deep(.el-tabs__item) {
  display: flex;
  align-items: center;
}

.lang-option,
.lang-tab-label {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  line-height: 1;
}

.lang-tab-label :deep(.el-icon) {
  display: flex;
  align-items: center;
}

.flag {
  display: inline-flex;
  align-items: center;
  font-size: 16px;
}

.flag-only {
  display: inline-flex;
  justify-content: center;
  width: 100%;
  font-size: 16px;
}

.default-lang {
  cursor: pointer;
  color: var(--el-text-color-secondary);
  font-size: 16px;
  margin-left: 8px;
}

.default-lang.active {
  color: var(--el-color-primary);
}

.rtl-textarea :deep(textarea) {
  direction: rtl;
  text-align: right;
}

.phone-preview {
  position: relative;
  width: 360px;
  min-width: 360px;
  margin: 0;
  padding: 18px 14px;
  border-radius: 28px;
  background: #121826;
  box-shadow: 0 16px 32px rgba(18, 24, 38, 0.25);
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.phone-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.6);
}

.status-bar {
  flex: 1;
  height: 6px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.6);
}

.phone-screen {
  background: #f5f7fa;
  border-radius: 20px;
  padding: 32px 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  min-height: 540px;
  position: relative;
}

.policy-screen {
  gap: 0;
  padding: 24px 20px;
}

.policy-screen[dir='rtl'] .policy-header {
  justify-content: flex-end;
}

.policy-screen[dir='rtl'] .back-icon {
  transform: scaleX(-1);
}

.policy-header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding-bottom: 16px;
  border-bottom: 1px solid rgba(31, 45, 61, 0.12);
  color: var(--el-text-color-primary);
  font-weight: 600;
}

.policy-header .preview-lang-switcher {
  position: static;
  margin-left: auto;
  width: 72px;
}

.policy-header .preview-lang-switcher :deep(.el-select__wrapper) {
  height: 32px;
  align-items: center;
}

.policy-header .preview-lang-switcher :deep(.el-select__selection) {
  justify-content: center;
}

.policy-header .preview-lang-switcher :deep(.el-select__selection-text) {
  display: none;
}

.preview-lang-switcher :deep(.el-select-dropdown__item) {
  font-size: 12px;
}

.back-icon {
  cursor: pointer;
  font-size: 20px;
  color: var(--el-text-color-regular);
}

.back-icon:hover {
  color: var(--el-color-primary);
}

.policy-body {
  flex: 1;
  margin-top: 16px;
  padding-right: 4px;
  overflow-y: auto;
  color: var(--el-text-color-primary);
  line-height: 1.6;
}

.policy-body.rtl-content {
  direction: rtl;
  text-align: right;
}

.policy-body h1,
.policy-body h2,
.policy-body h3,
.policy-body h4,
.policy-body h5,
.policy-body h6 {
  font-weight: 600;
  margin: 12px 0 8px;
}

.policy-body p {
  margin: 8px 0;
}

.login-logo {
  align-self: center;
  font-size: 18px;
  font-weight: 600;
  letter-spacing: 0.4px;
  color: var(--el-text-color-primary);
}

.login-title {
  font-size: 20px;
  font-weight: 600;
  color: var(--el-text-color-primary);
}

.login-subtitle {
  font-size: 14px;
  color: var(--el-text-color-regular);
}

.login-input {
  height: 42px;
  border-radius: 12px;
  background: #ffffff;
  border: 1px solid rgba(31, 45, 61, 0.08);
}

.login-button {
  margin-top: 8px;
  height: 44px;
  border-radius: 12px;
  background: linear-gradient(90deg, #ea1845 0%, #ff7e27 100%);
  color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  letter-spacing: 0.2px;
}

.privacy-link {
  margin-top: auto;
  font-size: 12px;
  text-align: center;
  color: var(--el-color-primary);
  text-decoration: underline;
  cursor: pointer;
  font-weight: 500;
}

.privacy-link:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.3);
  border-radius: 4px;
}

.privacy-link:hover {
  color: var(--el-color-primary-dark-2);
}

.toolbar {
  display: flex;
  align-items: center;
  gap: 8px;
}

.toolbar :deep(.el-button.is-disabled) {
  opacity: 0.5;
  cursor: not-allowed;
}

.toolbar-btn {
  width: 42px;
  justify-content: center;
  font-weight: 600;
}

.toolbar-btn.heading {
  color: var(--el-text-color-primary);
}

.toolbar-btn.heading-1 {
  font-size: 18px;
}

.toolbar-btn.heading-2 {
  font-size: 16px;
}

.toolbar-btn.heading-3 {
  font-size: 14px;
}

.toolbar-btn.emphasis {
  font-size: 16px;
}

.toolbar-btn.bold {
  font-weight: 700;
}

.toolbar-btn.italic {
  font-style: italic;
}

.toolbar-btn.paragraph {
  font-size: 14px;
  font-weight: 500;
}

:deep(.el-form-item__label) {
  color: var(--el-text-color-regular);
  font-weight: 500;
}

:deep(.el-form-item__content) {
  color: var(--el-text-color-primary);
}

.dialog-content {
  color: var(--el-text-color-primary);
  line-height: 1.6;
}

.policy-dialog {
  color: var(--el-text-color-primary);
}

:deep(.policy-dialog .el-dialog__body) {
  max-height: 60vh;
  overflow-y: auto;
}

.preview-lang-switcher {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 10;
}

.preview-lang-popper {
  max-height: 200px;
  overflow-y: auto;
}

@media screen and (max-width: 1024px) {
  .settings-page {
    padding: 24px;
  }

  .page-layout {
    flex-wrap: wrap;
    gap: 16px;
  }

  .layout-left,
  .layout-right {
    width: 100%;
    align-items: center;
  }

  .layout-right {
    align-items: stretch;
  }

  .settings-tabs :deep(.el-tab-pane) {
    padding-bottom: 24px;
  }
}

@media screen and (max-width: 768px) {
  .phone-preview {
    width: 320px;
    min-width: 320px;
    margin: 0 auto;
  }

  .phone-screen {
    padding: 28px 20px;
    min-height: 480px;
  }
}
</style>
