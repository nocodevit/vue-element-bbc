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
          <div class="phone-screen policy-screen" v-else>
            <div class="policy-header">
              <el-icon class="back-icon" @click="hidePolicyScreen">
                <ArrowLeft />
              </el-icon>
              <span>隐私政策</span>
            </div>
            <div class="policy-body" v-html="activePolicyHtml" />
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
                  <el-form-item label="编辑器工具栏">
                    <div class="toolbar">
                      <el-tooltip content="一级标题" placement="top">
                        <el-button size="small" @click="insertHeading(1)">H1</el-button>
                      </el-tooltip>
                      <el-tooltip content="二级标题" placement="top">
                        <el-button size="small" @click="insertHeading(2)">H2</el-button>
                      </el-tooltip>
                      <el-tooltip content="三级标题" placement="top">
                        <el-button size="small" @click="insertHeading(3)">H3</el-button>
                      </el-tooltip>
                      <el-divider direction="vertical" />
                      <el-tooltip content="加粗" placement="top">
                        <el-button size="small" @click="wrapSelection('**', '**')">B</el-button>
                      </el-tooltip>
                      <el-tooltip content="斜体" placement="top">
                        <el-button size="small" @click="wrapSelection('*', '*')">I</el-button>
                      </el-tooltip>
                      <el-tooltip content="正文段落" placement="top">
                        <el-button size="small" @click="insertParagraph">P</el-button>
                      </el-tooltip>
                    </div>
                  </el-form-item>

                  <el-form-item label="自定义内容">
                    <el-input ref="editorRef" v-model="customPolicy" type="textarea"
                      :autosize="{ minRows: 16, maxRows: 28 }" placeholder="在此输入自定义隐私政策内容，支持 Markdown 基础语法。" />
                    <el-alert class="custom-hint" type="info" show-icon :closable="false" title="自定义仅支持设置一种语言。" />
                  </el-form-item>

                  <el-form-item>
                    <el-button type="primary" @click="handleSave" :loading="isSaving" :disabled="!isDirty || isSaving">
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
import { computed, nextTick, ref } from 'vue';
import MarkdownIt from 'markdown-it';
import { ArrowLeft } from '@element-plus/icons-vue';
import { ElMessage } from 'element-plus';
import type { InputInstance } from 'element-plus';

const activeTab = ref('third');
const isPolicyScreen = ref(false);

const systemDefaultPolicy = ref<string>(
  [
    '# 隐私政策',
    '',
    '我们致力于保护您的隐私。本隐私政策阐述了我们如何收集、使用与存储您的信息。',
    '',
    '## 信息收集',
    '',
    '我们仅在提供服务所需的情况下收集必要的信息。',
    '',
    '## 信息使用',
    '',
    '所收集的数据仅用于改进产品体验，不会对外出售。',
  ].join('\n'),
);

const isCustomPolicyEnabled = ref<boolean>(false);
const customPolicy = ref<string>(
  [
    '# 自定义隐私政策',
    '',
    '请在此处填写符合您业务需求的隐私政策内容。',
  ].join('\n'),
);

const savedCustomPolicy = ref(customPolicy.value);

const isSaving = ref(false);
const editorRef = ref<InputInstance>();

const activePolicy = computed<string>(() =>
  isCustomPolicyEnabled.value ? customPolicy.value : systemDefaultPolicy.value,
);

const markdownRenderer = new MarkdownIt({
  html: false,
  linkify: false,
  typographer: false,
});

const activePolicyHtml = computed<string>(() => markdownRenderer.render(activePolicy.value));
const isDirty = computed<boolean>(() => customPolicy.value !== savedCustomPolicy.value);

function showPolicyScreen() {
  isPolicyScreen.value = true;
}

function hidePolicyScreen() {
  isPolicyScreen.value = false;
}

function handleSave() {
  isSaving.value = true;

  window.setTimeout(() => {
    savedCustomPolicy.value = customPolicy.value;
    isSaving.value = false;
    ElMessage.success('隐私政策已保存');
  }, 800);
}

function insertHeading(level: number) {
  const hashes = '#'.repeat(Math.min(level, 6));
  insertContent(`${hashes} `, '', '标题');
}

function insertParagraph() {
  insertContent('', '', '请输入正文内容');
}

function wrapSelection(prefix: string, suffix: string) {
  insertContent(prefix, suffix, '文本');
}

function insertContent(prefix: string, suffix: string, placeholder: string) {
  const textarea = editorRef.value?.textarea;
  if (!textarea) {
    customPolicy.value = `${prefix}${placeholder}${suffix}`;
    return;
  }

  const { selectionStart, selectionEnd, value } = textarea;
  const selectedText = value.slice(selectionStart, selectionEnd) || placeholder;
  const nextValue =
    value.slice(0, selectionStart) + prefix + selectedText + suffix + value.slice(selectionEnd);

  customPolicy.value = nextValue;

  nextTick(() => {
    textarea.focus();
    const start = selectionStart + prefix.length;
    const end = start + selectedText.length;
    textarea.setSelectionRange(start, end);
  });
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
}

.policy-screen {
  gap: 0;
  padding: 24px 20px;
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

.custom-hint {
  margin-top: 12px;
  width: 100%;
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
