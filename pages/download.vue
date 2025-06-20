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
  <div class="page flex flex-col items-center relative justify-between">
    <div
      class="pt-27 flex flex-col justify-center flex-grow relative w-full overflow-hidden gap-10"
    >
      <div class="flex flex-col">
        <h1 class="text-center text-5xl font-bold text-primary-color">
          下载星火应用商店
        </h1>
        <p class="text-center text-lg leading-[2]">
          基于 DTK 框架构建星火应用商店客户端，极小安装包体积，让 Linux
          生态触手可及。
        </p>
      </div>
      <div class="flex justify-center gap-6">
        <div class="flex flex-col gap-2">
          <p class="text-1.8xl font-bold">
            在这里选择计算机架构 <i class="pi pi-question-circle"></i>
          </p>
          <Select
            v-model="selectedArch"
            :options="archList"
            option-label="name"
            option-value="value"
            class="min-w-72 border-2! s-deco-primary-color s-bg-primary-200 s-bg-2-primary-400 s-bg-3-secondary-600 dark:s-bg-primary-900 dark:s-bg-2-secondary-700 dark:s-bg-3-secondary-500"
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
                  '--p-select-option-border-radius': 'calc(2 * var(--spacing))',
                  '--p-select-option-selected-background': 'transparent',
                },
              },
            }"
          />
        </div>
        <button
          class="relative items-start flex flex-col py-2 pl-4 pr-26 text-2xl font-bold text-white rounded-2xl overflow-hidden from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600"
          @click="dialog = true"
        >
          <p class="translate-y-0.5">下载最新版本</p>
          <p class="text-lg opacity-75">
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
          :href="consoleDownloadUrl"
          class="relative items-center gap-2 flex py-2 px-4 font-bold rounded-2xl overflow-hidden from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600 before:content-[''] before:absolute before:top-0 before:left-0 before:w-full before:h-full before:border-5 before:border-[transparent] before:rounded-2xl before:bg-white before:bg-clip-content before:opacity-50 dark:before:bg-secondary-950 dark:before:opacity-80"
        >
          <Icon name="proicons:terminal" class="text-primary-500 text-6xl" />
          <div class="flex flex-col items-start">
            <p class="text-xl text-primary-color z-[0]">下载终端版</p>
            <p class="text-sm text-primary-color opacity-50">
              全平台支持 仅包含基本功能
            </p>
          </div>
        </a>
      </div>
      <div class="flex flex-col items-center justify-center gap-2">
        <div class="flex gap-8 font-bold text-lg">
          <a
            class="border-b-2 border-primary-500 hover:border-primary-color"
            href="https://gitee.com/spark-store-project/spark-store/releases"
            >查看更新日志</a
          >
          <a
            class="border-b-2 border-primary-500 hover:border-primary-color"
            href="https://gitee.com/spark-store-project/spark-store"
            >访问 Gitee 源码仓库</a
          >
          <a
            class="border-b-2 border-primary-500 hover:border-primary-color"
            href="https://github.com/spark-store-project/spark-store"
            >访问 GitHub 源码仓库</a
          >
        </div>
        <p class="opacity-60">
          星火应用商店遵循 GNU GPLv3 协议开放源代码，<a
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
    <div class="flex gap-36 py-8">
      <div class="flex flex-col gap-4">
        <h1 class="text-3xl font-bold">使⽤时遇到问题？<br /></h1>
        <p class="text-lg leading-[2] max-w-[40.8em]">
          敬请参阅<NuxtLink to="/faq" class="text-primary-color underline"
            >星火应用商店 FAQ 与支持指南</NuxtLink
          >。如果您有其他问题或建议，可以通过论坛、交流群或电子邮件等方式与我们取得联系；此外，也可以在源码仓库
          Issue 页面上提交反馈。
        </p>
      </div>
      <Icon
        name="s:download-deco"
        class="s-deco-secondary-900 s-bg-primary-200 scale-120 dark:s-deco-secondary-100 dark:s-bg-primary-500"
        mode="svg"
        :style="{
          height: '100%',
          width: 'auto',
        }"
      />
    </div>
    <Dialog
      v-model:visible="dialog"
      modal
      :show-header="false"
      class="w-210 h-128 rounded-3xl!"
      :style="{
        '--p-dialog-border-color': 'transparent',
      }"
      content-class="flex p-5! w-full h-full gap-4 overflow-hidden!"
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
        class="p-4 rounded-xl flex flex-col justify-between from-primary-400 to-primary-500 bg-linear-to-b dark:from-primary-500 dark:to-primary-600"
      >
        <Icon
          name="sp:spark"
          class="s-color-primary-50 w-20"
          mode="svg"
          :style="{
            height: 'auto',
          }"
        />
        <p class="font-(family-name:--s-title-font) text-5xl text-primary-100">
          SPARK
        </p>
      </div>
      <div v-if="!downloadStarted" class="flex flex-col flex-grow gap-4">
        <div class="flex flex-col gap-1">
          <h2 class="text-3xl font-bold">下载开始前，让我们做好准备</h2>
          <p class="text-lg">
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
            class="relative py-2 px-9 text-lg font-bold text-primary-color rounded-xl from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600 before:content-[''] before:absolute before:top-0 before:left-0 before:z-[0] before:w-full before:h-full before:border-2 before:border-[transparent] before:rounded-xl before:bg-white before:bg-clip-content before:opacity-80 dark:before:bg-secondary-950 dark:before:opacity-90"
            @click="dialog = false"
          >
            <span class="relative z-[0] text-primary-500 dark:text-primary-400"
              >取消</span
            >
          </button>

          <button
            class="py-2 px-9 text-lg font-bold text-white rounded-xl from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600"
            @click="startDownload"
          >
            下一步
          </button>
        </div>
      </div>
      <div v-else class="flex flex-col flex-grow gap-4 pl-4">
        <div class="flex flex-col flex-grow justify-center gap-4">
          <div class="flex gap-4">
            <Icon
              name="line-md:download-loop"
              class="text-white p-4 rounded-full from-primary-500 to-primary-600 bg-linear-to-r dark:from-primary-400 dark:to-primary-500"
              mode="svg"
              :style="{
                width: 'auto',
                height: '100%',
              }"
            />
            <h2 class="text-3xl font-bold leading-[1.5]">
              感谢您下载<br />星火应用商店！
            </h2>
          </div>
          <p class="text-lg leading-[2]">
            应用商店安装包下载已开始，<br />如果创建下载任务失败，请<a
              :href="downloadUrl"
              class="text-primary-color underline"
              >点击此处重试</a
            >。
          </p>
        </div>
        <div class="flex justify-end gap-4">
          <button
            class="py-2 px-9 text-lg font-bold text-white rounded-xl from-primary-400 to-primary-500 bg-linear-to-r dark:from-primary-500 dark:to-primary-600"
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
.page {
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
