<script setup lang="ts">
import { MdPreview } from "md-editor-v3";
import "md-editor-v3/lib/preview.css";
import type { Release } from "~/app.vue";

import { useToast } from "primevue/usetoast";
const toast = useToast();

const selectedArch = ref("amd64");
const archList = [
  { name: "Intel/AMD 架构（amd64）", value: "amd64" },
  { name: "ARM 架构（arm64）", value: "arm64" },
  { name: "龙架构·新世界（loong64）", value: "loong64" },
];

const sDark = inject("sDark", false);

const dialog = ref(false);

const downloadStarted = ref(false);

const startDownload = () => {
  downloadStarted.value = true;
  if (selectedArch.value === "loong64") {
    toast.add({
      severity: "error",
      summary: "请前往 Gitee 手动下载",
      detail:
        "loong64 架构的安装包不同发行版有所不同，请前往 Gitee 手动下载。\n5s 后自动跳转。",
      life: 5000,
    });
    setTimeout(() => {
      window.open(
        "https://gitee.com/spark-store-project/spark-store/releases/latest",
        "_blank"
      );
    }, 5000);
  } else {
    // 创建下载任务
    const downloadLink = downloadUrl.value;
    if (downloadLink) {
      const a = document.createElement("a");
      a.href = downloadLink;
      a.download = `spark-store-${selectedArch.value}.deb`;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      toast.add({
        severity: "success",
        summary: "下载任务已创建",
        detail: `正在下载适用于 ${selectedArch.value} 架构的星火应用商店安装包，请在浏览器下载管理器中查看进度。`,
        life: 5000,
      });
    } else {
      toast.add({
        severity: "error",
        summary: "下载链接错误",
        detail: "无法找到适合您架构的下载链接，请稍后再试。",
        life: 5000,
      });
    }
  }
};

watchEffect(() => {
  if (dialog.value) {
    // 重置下载状态
    downloadStarted.value = false;
  }
});

const latestRelease = inject<Ref<Release>>("latestRelease");

const downloadUrl = computed(() => {
  if (selectedArch.value === "loong64") {
    return "https://gitee.com/spark-store-project/spark-store/releases/latest";
  }
  for (const asset of latestRelease?.value.assets || []) {
    if (asset.name.includes(selectedArch.value)) {
      return asset.browser_download_url;
    }
  }
  return "";
});

const consoleDownloadUrl = computed(() => {
  for (const asset of latestRelease?.value.assets || []) {
    if (asset.name.includes("all")) {
      return asset.browser_download_url;
    }
  }
  return "";
});

const { data: faqContent } = await useAsyncData("/download/faq", () => {
  return queryCollection("download").path("/download/faq").first();
});
</script>

<template>
  <div>
    <section class="flex flex-col items-center relative justify-between">
      <div
        class="pt-20 xs:pt-27 px-8 flex flex-col justify-center flex-grow relative w-full overflow-hidden gap-4 md:gap-10"
      >
        <div class="flex flex-col gap-4 md:gap-0">
          <h1
            class="md:text-center text-4xl md:text-5xl font-bold text-primary-color"
          >
            下载<br class="hidden xs:block sm:hidden" />星火应用商店
          </h1>
          <p class="md:text-center md:text-lg md:leading-loose">
            星火应用商店兼容所有主流 Linux 发行版，让 Linux
            生态触手可及。
            <span class="md:hidden">
              <br />我们推荐您在 Linux 计算机上访问此页面，下载安装更便捷。
            </span>
          </p>
        </div>
        <div class="flex justify-center gap-4 lg:gap-6 flex-col md:flex-row">
          <div class="flex flex-col gap-2">
            <p class="font-bold">
              在这里选择计算机架构 <i class="pi pi-question-circle"></i>
            </p>
            <Select
              v-model="selectedArch"
              :options="archList"
              option-label="name"
              option-value="value"
              class="lg:min-w-72 border-2! s-deco-primary-color s-bg-primary-200 s-bg-2-primary-400 s-bg-3-secondary-600 dark:s-bg-primary-900 dark:s-bg-2-secondary-700 dark:s-bg-3-secondary-500"
              :style="{
                '--p-select-border-radius': 'calc(3 * var(--spacing))',
                '--p-select-border-color': 'var(--s-bg-2)',
                '--p-select-hover-border-color': 'var(--s-bg-3)',
                '--p-select-background': 'var(--s-bg)',
                '--p-select-color': 'var(--s-deco)',
                '--p-select-dropdown-color': 'var(--s-deco)',
              }"
              :pt="{
                overlay: {
                  style: {
                    transform: 'translateY(calc(2 * var(--spacing)))',
                    '--p-select-overlay-border-radius':
                      'calc(3 * var(--spacing))',
                    '--p-select-overlay-border-color': 'transparent',
                    '--p-select-option-border-radius':
                      'calc(2 * var(--spacing))',
                    '--p-select-option-selected-background': 'transparent',
                  },
                },
              }"
            />
          </div>
          <div
            class="flex flex-col xs:flex-row gap-4 lg:gap-6 grow-1 lg:grow-0"
          >
            <button
              class="relative items-start flex flex-col justify-center py-2 pl-4 pr-4 lg:pr-26 grow-1 lg:grow-0 text-xl lg:text-2xl font-bold text-white rounded-2xl overflow-hidden from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600"
              @click="dialog = true"
            >
              <p class="translate-y-0.5">下载最新版本</p>
              <p class="text-base lg:text-lg opacity-75">
                V{{ latestRelease?.tag_name }} |
                {{
                  new Date(latestRelease!.created_at)
                    .toLocaleDateString()
                    .replace(/\//g, "-")
                }}
              </p>
              <Icon
                name="sp:spark"
                class="absolute top-0 right-0 s-color-primary-200 w-20 translate-x-1/6 -translate-y-2/9 opacity-60"
                mode="svg"
                :style="{
                  height: 'auto',
                }"
              />
            </button>
            <a
              href="https://amber-pm.spark-app.store/"
              class="relative items-center gap-2 flex py-2 px-4 lg:pr-26 grow-1 lg:grow-0 font-bold rounded-2xl overflow-hidden from-primary-200 to-primary-300 bg-linear-to-r dark:from-primary-800 dark:to-primary-900"
            >
              <div class="flex flex-col items-start">
                <p
                  class="text-md sm:text-lg lg:text-xl text-primary-color z-[0]"
                >
                  <span class="hidden sm:inline"> </span>了解 APM 包管理器
                </p>
                <p class="text-xs lg:text-sm text-primary-color opacity-50">
                  星火跨平台兼容容器
                </p>
              </div>
              <Icon
                name="s:amber-pm"
                class="absolute top-0 right-0 s-deco-primary-500 s-bg-primary-300 w-20 translate-x-1/6 -translate-y-2/9 opacity-60 dark:s-bg-primary-900 dark:opacity-40"
                mode="svg"
                :style="{
                  height: 'auto',
                }"
              />
            </a>
          </div>
        </div>
        <div class="flex flex-col items-center justify-center gap-2">
          <div
            class="flex flex-col xs:flex-row items-center gap-4 sm:gap-8 font-bold text-sm sm:text-base md:text-lg"
          >
            <div class="flex gap-4 sm:gap-8">
              <a
                class="border-b-2 border-primary-500 hover:border-primary-color"
                href="https://gitee.com/spark-store-project/spark-store/releases"
                ><span class="hidden md:inline">查看</span>更新日志</a
              >
              <a
                class="border-b-2 border-primary-500 hover:border-primary-color"
                href="https://gitee.com/spark-store-project/spark-store"
                ><span class="hidden md:inline">访问 </span>Gitee 源码仓库</a
              >
            </div>
            <div class="flex gap-4 sm:gap-8">
              <a
                class="border-b-2 border-primary-500 hover:border-primary-color"
                href="https://github.com/spark-store-project/spark-store"
                ><span class="hidden md:inline">访问 </span>GitHub 源码仓库</a
              >
              <a
                class="border-b-2 border-primary-500 hover:border-primary-color"
                :href="consoleDownloadUrl"
                ><span class="hidden md:inline">下载 </span>TUI 终端版本</a
              >
            </div>
          </div>
          <p class="opacity-60 text-sm md:text-base text-center">
            星火应用商店遵循 GNU GPLv3 协议开放<br
              class="sm:hidden"
            />源代码，<a
              href="https://gitee.com/spark-store-project/spark-store/blob/dev/LICENSE"
              class="text-primary-color underline"
              >点此阅读版权声明</a
            >
          </p>
        </div>

        <div
          class="absolute w-[500vw] h-[500vw] rounded-full bottom-0 left-1/2 transform -translate-x-1/2 z-[-2] bg-primary-100 dark:opacity-10 dark:bg-primary-500"
        ></div>

        <Icon
          name="sp:spark"
          class="absolute top-0 left-0 z-[-1] s-color-primary-200 w-120 -translate-x-2/7 -translate-y-1/4 dark:opacity-10 dark:s-color-primary-500"
          mode="svg"
          :style="{
            height: 'auto',
          }"
        />
      </div>
      <div class="flex gap-16 md:gap-36 p-8 flex-row-reverse md:flex-row">
        <div class="flex flex-col gap-4">
          <h1 class="text-3xl font-bold">
            使用时<br class="sm:hidden" />遇到问题？<br />
          </h1>
          <p class="hidden md:block text-lg leading-[2] max-w-[40.8em]">
            敬请参阅<NuxtLink to="/faq" class="text-primary-color underline"
              >星火应用商店 FAQ 与支持指南</NuxtLink
            >。如果您有其他问题或建议，可以通过论坛、交流群或电子邮件等方式与我们取得联系；此外，也可以在源码仓库
            Issue 页面上提交反馈。
          </p>
        </div>
        <Icon
          name="s:download-deco"
          class="scale-120 sm:scale-200 md:scale-120 s-deco-secondary-900 s-bg-primary-200 dark:s-deco-secondary-100 dark:s-bg-primary-500"
          mode="svg"
          :style="{
            height: '100%',
            width: 'auto',
          }"
        />
      </div>
    </section>
    <div class="md:hidden px-8 pb-8">
      <p class="text-base sm:text-lg leading-[2] max-w-[40.8em]">
        敬请参阅<NuxtLink to="/faq" class="text-primary-color underline"
          >星火应用商店 FAQ 与支持指南</NuxtLink
        >。<br />如果您有其他问题或建议，可以通过论坛、交流群或电子邮件等方式与我们取得联系；此外，也可以在源码仓库
        Issue 页面上提交反馈。
      </p>
    </div>
    <Dialog
      v-model:visible="dialog"
      modal
      :show-header="false"
      class="max-w-7/8 w-210 h-7/8 md:h-128 rounded-3xl!"
      :style="{
        '--p-dialog-border-color': 'transparent',
      }"
      content-class="flex flex-col md:flex-row p-5! w-full h-full gap-4 overflow-hidden!"
      :pt="{
        mask: {
          class:
            'backdrop-blur-2xl s-bg-[rgba(255,255,255,0.4)] s-deco-surface-950 dark:s-bg-[rgba(0,0,0,0.4)] dark:s-deco-surface-100',
          style: {
            '--p-mask-background': 'var(--s-bg)',
            '--p-dialog-color': 'var(--s-deco)',
          },
        },
      }"
    >
      <div
        class="p-2 md:p-4 rounded-xl flex md:flex-col items-end md:items-start justify-between from-primary-400 to-primary-500 bg-linear-to-b dark:from-primary-500 dark:to-primary-600"
      >
        <Icon
          name="sp:spark"
          class="s-color-primary-50 w-16 sm:w-20"
          mode="svg"
          :style="{
            height: 'auto',
          }"
        />
        <p
          class="font-(family-name:--s-title-font) text-3xl sm:text-4xl md:text-5xl text-primary-100"
        >
          SPARK
        </p>
      </div>
      <div v-if="!downloadStarted" class="flex flex-col flex-grow gap-4">
        <div class="flex flex-col gap-1">
          <h2 class="text-xl sm:text-2xl md:text-3xl font-bold">
            下载开始前，让我们做好准备
          </h2>
          <p class="text-sm sm:text-base md:text-lg">
            阅读以下安装须知，了解星火应用商店在不同发行版的安装方式。
          </p>
        </div>
        <Message v-if="selectedArch === 'loong64'" severity="error">
          loong64 架构的安装包不同发行版有所不同，请前往 Gitee 手动下载。
        </Message>
        <div class="flex-grow bg-surface-100 rounded-xl h-[0] dark:bg-black">
          <ScrollPanel
            ref="scrollPanel"
            class="overflow-hidden w-full h-full s-deco-surface-800 dark:s-deco-surface-200"
            :style="{
              '--p-scrollpanel-bar-background': 'var(--s-deco)',
              '--p-scrollpanel-bar-size': 'calc(4 * var(--spacing))',
            }"
            :pt="{
              barY: {
                class:
                  'border-[calc(1.5*var(--spacing))]! border-[transparent]! border-solid! rounded-full! bg-clip-content! -translate-x-2.5',
              },
            }"
          >
            <MdPreview
              :model-value="faqContent?.rawbody.replaceAll('\\n', '\n') || ''"
              :theme="sDark ? 'dark' : 'light'"
              preview-theme="github"
            />
            <a
              href="https://gitee.com/spark-store-project/spark-store/releases/latest"
              class="font-bold text-xl text-primary-color hover:underline pl-4"
              target="_blank"
            >
              V{{ latestRelease?.tag_name }}
            </a>
            <MdPreview
              :model-value="latestRelease?.body || ''"
              :theme="sDark ? 'dark' : 'light'"
              preview-theme="github"
            />
          </ScrollPanel>
        </div>
        <div class="flex justify-between gap-4">
          <button
            class="grow-1 md:grow-0 relative py-2 px-9 text-lg font-bold text-primary-color rounded-xl from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600 before:content-[''] before:absolute before:top-0 before:left-0 before:z-[0] before:w-full before:h-full before:border-2 before:border-[transparent] before:rounded-xl before:bg-white before:bg-clip-content before:opacity-80 dark:before:bg-secondary-950 dark:before:opacity-90"
            @click="dialog = false"
          >
            <span class="relative z-[0] text-primary-500 dark:text-primary-400"
              >取消</span
            >
          </button>

          <button
            class="grow-1 md:grow-0 py-2 px-9 text-lg font-bold text-white rounded-xl from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600"
            @click="startDownload"
          >
            下一步
          </button>
        </div>
      </div>
      <div v-else class="flex flex-col flex-grow gap-4 md:pl-4">
        <div class="flex flex-col flex-grow justify-center gap-4">
          <div
            class="flex gap-4 flex-col md:flex-row items-center md:items-start"
          >
            <Icon
              name="line-md:download-loop"
              class="w-24 h-24 md:w-auto md:h-full text-white p-4 rounded-full from-primary-500 to-primary-600 bg-linear-to-r dark:from-primary-400 dark:to-primary-500"
              mode="svg"
            />
            <h2
              class="text-2xl md:text-3xl font-bold leading-[1.5] text-center md:text-start"
            >
              感谢您下载<br />&nbsp;&nbsp;星火应用商店！
            </h2>
          </div>
          <p
            class="text-base md:text-lg leading-loose text-center md:text-start"
          >
            应用商店安装包下载已开始，<br />如果创建下载任务失败，<br
              class="md:hidden"
            />请<a :href="downloadUrl" class="text-primary-color underline"
              >点击此处重试</a
            >。
          </p>
        </div>
        <div class="flex justify-end gap-4">
          <button
            class="w-full md:w-auto py-2 px-9 text-lg font-bold text-white rounded-xl from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600"
            @click="dialog = false"
          >
            关闭
          </button>
        </div>
      </div>
    </Dialog>
    <Toast />
  </div>
</template>

<style scoped>
section {
  width: 100%;
  height: 100vh;
  scroll-snap-align: start;
}
</style>

<style lang="scss">
.p-select-option.p-select-option-selected {
  font-weight: bold;
}
.md-editor {
  --md-bk-color: transparent;
  padding: calc(4 * var(--spacing));

  .md-editor-preview {
    font-size: calc(3 * var(--spacing));

    .md-editor-code {
      margin: 1em 0;
    }

    ol,
    ul {
      padding-left: 2em;
    }
  }

  div.github-theme {
    --md-theme-link-color: var(--p-primary-color);
    --md-theme-link-hover-color: var(--p-primary-color);

    h3 {
      font-size: 1em;
    }

    p {
      margin-bottom: 1em;
    }

    ol,
    ul,
    menu {
      list-style: revert;
    }
  }
}
</style>
