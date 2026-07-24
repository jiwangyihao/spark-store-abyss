<script setup lang="ts">
import {
  onMounted,
  watchEffect,
  ref,
  inject,
  type Ref,
  type ComponentPublicInstance,
} from "vue";
import { range } from "~/utils/index";
import Dialog from "primevue/dialog";

const scrollPanel = inject<Ref<ComponentPublicInstance>>("scrollPanel");
const sProgress = ref(0);
const contactDialog = ref(false);

onMounted(() => {
  watchEffect(() => {
    const scrollTop =
      (scrollPanel?.value as unknown as { lastScrollTop: number })
        .lastScrollTop || 0;
    const clientHeight = scrollPanel?.value?.$el.clientHeight || 1;
    sProgress.value = scrollTop / clientHeight;
  });
});

const scrollToNext = () => {
  const container = scrollPanel?.value?.$el.firstChild.firstChild;
  if (container) {
    container.scrollBy({ top: window.innerHeight, behavior: "smooth" });
  }
};

const sendEmail = () => {
  const to = "shenmo@spark-app.store";
  const cc = "momen@spark-app.store";
  const subject = encodeURIComponent("商业版咨询");
  const body = encodeURIComponent("您好，我想咨询商业版的相关信息：\n\n");
  window.location.href = `mailto:${to}?cc=${cc}&subject=${subject}&body=${body}`;
  contactDialog.value = true;
};
</script>

<template>
  <div class="page">
    <section
      class="relative flex items-center justify-center gap-10 pt-24 pb-24 sm:pb-0 flex-col sm:flex-row"
    >
      <div
        class="fixed w-full h-full s-size-4 lg:s-size-5.5 top-0 left-0 z-[-1] flex items-end sm:justify-center"
        :style="{
          opacity: 1 - range(0, 0.5, sProgress) * 2,
          '--s-n': 16,
          '--s-progress': range(0, 0.5, sProgress) * 2,
        }"
      >
        <div class="absolute top-0 left-0 flex flex-col">
          <div v-for="r in 16" :key="r" class="flex">
            <i
              v-for="c in Math.min(16, 16 - r + 3)"
              :key="`${r}-${c}`"
              class="s-dots"
              :style="{
                '--s-row': r,
                '--s-col': c,
              }"
            />
          </div>
        </div>
        <div class="absolute bottom-0 right-0 flex flex-col rotate-180">
          <div v-for="r in 16" :key="r" class="flex">
            <i
              v-for="c in Math.min(16, 16 - r + 3)"
              :key="`${r}-${c}`"
              class="s-dots"
              :style="{
                '--s-row': r,
                '--s-col': c,
                '--s-n': 16,
              }"
            />
          </div>
        </div>
      </div>
      <div
        class="card flex flex-col lg:flex-row items-center lg:items-center px-4 md:px-6 lg:px-9 rounded-3xl gap-10 sm:gap-14 py-8 pb-15 sm:py-15 w-full max-w-7xl"
      >
        <div class="flex flex-col gap-6 lg:gap-8 flex-1">
          <div class="flex flex-col text-center lg:text-left gap-4">
            <h1
              class="font-bold text-4xl sm:text-5xl lg:text-6xl text-surface-900 dark:text-surface-50"
            >
              星火应用商店<br />
              <span class="text-primary-500 mt-2 block">商业版</span>
            </h1>
          </div>
          <div
            class="flex flex-col text-surface-500 gap-2 text-base sm:text-lg lg:text-xl text-center lg:text-left dark:text-surface-300"
          >
            <p>为政府、企事业单位客户提供专业的应用商店解决方案：</p>
            <div class="flex flex-col gap-1 mt-2 lg:items-start items-center">
              <span class="flex items-center gap-2"
                ><i class="pi pi-check text-primary-500" />
                支持安装在主流信创设备上</span
              >
              <span class="flex items-center gap-2"
                ><i class="pi pi-check text-primary-500" />
                支持定制客户端和服务器端本地部署</span
              >
              <span class="flex items-center gap-2"
                ><i class="pi pi-check text-primary-500" />
                支持指定软件适配服务和信创设备迁移</span
              >
            </div>
            <p class="mt-4 font-semibold">助力客户构建完善的软件分发体系。</p>
          </div>
          <div class="flex gap-4 mt-2 justify-center lg:justify-start">
            <button
              @click="sendEmail"
              class="px-6 py-3 bg-primary-500 hover:bg-primary-600 text-white rounded-full font-bold shadow-lg shadow-primary-500/30 transition-all hover:-translate-y-1"
            >
              联系客服
            </button>
            <button
              @click="scrollToNext"
              class="px-6 py-3 border-2 border-primary-500 text-primary-500 hover:bg-primary-50 dark:hover:bg-primary-900/20 rounded-full font-bold transition-all hover:-translate-y-1 cursor-pointer"
            >
              了解更多
            </button>
          </div>
        </div>

        <!-- 模拟企业控制台 UI 窗口 -->
        <div
          class="hidden sm:flex flex-col flex-1 max-w-lg w-full rounded-2xl bg-surface-0/60 dark:bg-surface-800/80 shadow-[0_20px_50px_rgba(0,0,0,0.1)] dark:shadow-[0_20px_50px_rgba(0,0,0,0.5)] overflow-hidden border border-surface-200/50 dark:border-surface-700/50 h-[380px] relative font-mono backdrop-blur-xl"
        >
          <!-- 动态背景光晕 -->
          <div
            class="absolute -top-20 -right-20 w-64 h-64 bg-primary-500/10 dark:bg-primary-500/5 blur-3xl rounded-full"
          ></div>
          <div
            class="absolute -bottom-20 -left-20 w-64 h-64 bg-secondary-500/10 dark:bg-secondary-500/5 blur-3xl rounded-full"
          ></div>

          <div
            class="h-12 border-b border-surface-200/50 dark:border-surface-700/50 flex items-center px-4 bg-surface-50/80 dark:bg-surface-900/80 backdrop-blur-md"
          >
            <div class="flex gap-2">
              <div class="w-3 h-3 rounded-full bg-red-400"></div>
              <div class="w-3 h-3 rounded-full bg-yellow-400"></div>
              <div class="w-3 h-3 rounded-full bg-green-400"></div>
            </div>
            <div
              class="ml-4 h-4 w-40 bg-surface-100 dark:bg-surface-800 rounded-lg border border-surface-200 dark:border-surface-700"
            ></div>
            <div class="ml-auto flex gap-3">
              <div
                class="h-4 w-12 bg-primary-100 dark:bg-primary-900/50 rounded-lg border border-primary-200 dark:border-primary-800"
              ></div>
            </div>
          </div>
          <div class="flex flex-1 overflow-hidden relative z-10">
            <!-- 左侧边栏 -->
            <div
              class="w-1/3 border-r border-surface-100 dark:border-surface-700/50 p-4 flex flex-col gap-4 bg-surface-50/50 dark:bg-surface-800/50 backdrop-blur-sm"
            >
              <div
                class="h-8 w-full bg-primary-500 text-white rounded-lg flex items-center justify-center text-xs font-bold tracking-wider shadow-sm"
              >
                SPARK PRO
              </div>
              <div class="flex flex-col gap-2 mt-2">
                <div
                  class="h-5 w-full bg-surface-100 dark:bg-surface-700 rounded-md"
                ></div>
                <div
                  class="h-5 w-5/6 bg-surface-100 dark:bg-surface-700 rounded-md"
                ></div>
                <div
                  class="h-5 w-4/5 bg-surface-100 dark:bg-surface-700 rounded-md"
                ></div>
              </div>
              <div
                class="mt-auto p-3 rounded-xl bg-primary-50 dark:bg-primary-900/20 border border-primary-100 dark:border-primary-800"
              >
                <div
                  class="w-8 h-8 rounded-full bg-primary-200 dark:bg-primary-800 mb-2 flex items-center justify-center"
                >
                  <div class="w-4 h-4 rounded-full bg-primary-500"></div>
                </div>
                <div
                  class="h-2 w-1/2 bg-primary-300 dark:bg-primary-700 rounded-full mb-1"
                ></div>
                <div
                  class="h-2 w-3/4 bg-primary-200 dark:bg-primary-800 rounded-full"
                ></div>
              </div>
            </div>
            <!-- 右侧主体内容 -->
            <div
              class="w-2/3 p-6 flex flex-col gap-5 bg-surface-0/50 dark:bg-surface-800/20 backdrop-blur-sm"
            >
              <div class="flex gap-4">
                <div
                  class="flex-[2] h-16 rounded-xl from-primary-400 to-primary-600 bg-linear-to-br shadow-md p-3 flex flex-col justify-end text-white"
                >
                  <div class="h-2 w-1/3 bg-white/30 rounded-full mb-1"></div>
                  <div class="h-3 w-1/2 bg-white/50 rounded-full"></div>
                </div>
                <div
                  class="flex-1 h-16 rounded-xl bg-surface-100 dark:bg-surface-700 flex items-center justify-center border border-surface-200 dark:border-surface-600"
                >
                  <div
                    class="w-8 h-8 rounded-lg bg-surface-300 dark:bg-surface-500"
                  ></div>
                </div>
              </div>
              <div
                class="flex-1 rounded-xl border border-surface-100 dark:border-surface-700 bg-surface-50/50 dark:bg-surface-800/80 p-4 flex flex-col gap-3"
              >
                <div class="flex justify-between items-center mb-2">
                  <div
                    class="h-3 w-20 bg-surface-300 dark:bg-surface-600 rounded-full"
                  ></div>
                  <div class="h-3 w-10 bg-green-400 rounded-full"></div>
                </div>
                <div class="flex items-end gap-2 flex-1 pt-4">
                  <div
                    class="w-1/5 rounded-t-md bg-primary-200 dark:bg-primary-900 h-[40%]"
                  ></div>
                  <div
                    class="w-1/5 rounded-t-md bg-primary-300 dark:bg-primary-800 h-[60%]"
                  ></div>
                  <div
                    class="w-1/5 rounded-t-md bg-primary-400 dark:bg-primary-700 h-[100%]"
                  ></div>
                  <div
                    class="w-1/5 rounded-t-md bg-primary-300 dark:bg-primary-800 h-[80%]"
                  ></div>
                  <div
                    class="w-1/5 rounded-t-md bg-primary-200 dark:bg-primary-900 h-[50%]"
                  ></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section
      class="flex flex-col items-center justify-center gap-6 lg:gap-8 px-8 pb-24 pt-24"
    >
      <div
        class="flex items-center justify-center self-start sm:self-center gap-4 sm:gap-6"
      >
        <Icon
          name="s:commercial-customization-title"
          mode="svg"
          class="!text-6xl sm:!text-7xl md:!text-[6rem] lg:!text-[7rem] text-primary-color s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
        />
        <h2
          class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl text-primary-color font-bold leading-[1.3] dark:text-primary-400"
        >
          <span
            class="font-(family-name:--s-title-font) text-primary-400 font-normal dark:opacity-50 tracking-widest uppercase"
            >CUSTOMIZATION</span
          ><br />
          专业定制&nbsp;&nbsp;专属服务
        </h2>
      </div>
      <div
        class="flex gap-8 lg:gap-16 max-w-7xl w-full flex-col-reverse sm:flex-row items-center justify-center"
      >
        <div
          class="flex flex-col py-0 sm:py-8 lg:py-12 gap-6 items-center sm:items-start flex-1"
        >
          <div
            class="flex flex-col text-center sm:text-start items-center sm:items-start"
          >
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl leading-[1.5]"
            >
              专属企业形象
            </p>
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold leading-[1.5] whitespace-nowrap"
            >
              满足企业个性化需求
            </p>
          </div>
          <p
            class="text-base sm:text-lg text-center sm:text-start text-surface-700 leading-[1.8] dark:text-surface-300"
          >
            商业版提供专属定制服务，可根据企业需求修改应用商店名称、logo、主题风格等，打造独一无二的企业级应用商店，提升企业品牌形象。从界面呈现到功能模块设定，全方位实现开箱即用的企业级定制。
          </p>
          <p
            class="font-bold text-surface-700 dark:text-surface-300 text-base md:text-lg leading-[2]"
          >
            专属定制 / Logo 替换 / 风格定制
          </p>
        </div>
        <Icon
          name="s:commercial-customization"
          class="w-full max-w-[20rem] sm:max-w-[24rem] lg:max-w-[30rem] h-auto object-contain hover:-translate-y-2 transition-transform duration-500 s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
          mode="svg"
        />
      </div>
    </section>

    <section
      class="flex flex-col items-center justify-center gap-6 lg:gap-8 px-8 pb-24 pt-24"
    >
      <div
        class="flex items-center justify-center self-start sm:self-center gap-4 sm:gap-6"
      >
        <Icon
          name="s:commercial-architecture-title"
          mode="svg"
          class="!text-6xl sm:!text-7xl md:!text-[6rem] lg:!text-[7rem] text-primary-color s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
        />
        <h2
          class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl text-primary-color font-bold leading-[1.3] dark:text-primary-400"
        >
          <span
            class="font-(family-name:--s-title-font) text-primary-400 font-normal dark:opacity-50 tracking-widest uppercase"
            >ARCHITECTURE</span
          ><br />
          多重架构&nbsp;&nbsp;全系覆盖
        </h2>
      </div>
      <div
        class="flex gap-8 lg:gap-16 max-w-7xl w-full flex-col sm:flex-row items-center justify-center"
      >
        <Icon
          name="s:commercial-architecture"
          class="w-full max-w-[20rem] sm:max-w-[24rem] lg:max-w-[30rem] h-auto object-contain hover:-translate-y-2 transition-transform duration-500 s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
          mode="svg"
        />
        <div
          class="flex flex-col py-0 sm:py-8 lg:py-12 gap-6 items-center sm:items-start flex-1"
        >
          <div
            class="flex flex-col text-center sm:text-start items-center sm:items-start"
          >
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl leading-[1.5]"
            >
              不畏多重阻碍
            </p>
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold leading-[1.5] whitespace-nowrap"
            >
              支持国内全平台硬件
            </p>
          </div>
          <p
            class="text-base sm:text-lg text-center sm:text-start text-surface-700 leading-[1.8] dark:text-surface-300"
          >
            商业版支持 amd64、arm64、loongarch64 等多种 CPU
            架构部署，可在不同硬件平台上稳定运行，满足企业多样化的复杂部署需求，无缝适配各类国产化终端设备。
          </p>
          <p
            class="font-bold text-surface-700 dark:text-surface-300 text-base md:text-lg leading-[2]"
          >
            龙芯 / 兆芯 / 飞腾 / 海光 / 华为麒麟 / 华为鲲鹏
          </p>
        </div>
      </div>
    </section>

    <section
      class="flex flex-col items-center justify-center gap-6 lg:gap-8 px-8 pb-24 pt-24"
    >
      <div
        class="flex items-center justify-center self-start sm:self-center gap-4 sm:gap-6"
      >
        <Icon
          name="s:commercial-distribution-title"
          mode="svg"
          class="!text-6xl sm:!text-7xl md:!text-[6rem] lg:!text-[7rem] text-primary-color s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
        />
        <h2
          class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl text-primary-color font-bold leading-[1.3] dark:text-primary-400"
        >
          <span
            class="font-(family-name:--s-title-font) text-primary-400 font-normal dark:opacity-50 tracking-widest uppercase"
            >DISTRIBUTION</span
          ><br />
          跨发行版&nbsp;&nbsp;全面兼容
        </h2>
      </div>
      <div
        class="flex gap-8 lg:gap-16 max-w-7xl w-full flex-col-reverse sm:flex-row items-center justify-center"
      >
        <!-- 左侧内容区域 -->
        <div
          class="flex flex-col py-0 sm:py-8 lg:py-12 gap-6 items-center sm:items-start flex-1"
        >
          <div
            class="flex flex-col text-center sm:text-start items-center sm:items-start"
          >
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl leading-[1.5]"
            >
              融合多元生态
            </p>
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold leading-[1.5] whitespace-nowrap"
            >
              适配主流 Linux 发行版
            </p>
          </div>
          <p
            class="text-base sm:text-lg text-center sm:text-start text-surface-700 leading-[1.8] dark:text-surface-300"
          >
            商业版支持多种主流国产操作系统，包括但不限于统信
            UOS、银河麒麟、中科方德等，确保在不同发行版本及不同的软件基础包环境下都能高度稳定运行，为企业提供全生态覆盖、没有短板的应用商店解决方案。
          </p>
          <p
            class="font-bold text-surface-700 dark:text-surface-300 text-base md:text-lg leading-[2]"
          >
            统信 UOS / 银河麒麟 / 中科方德
          </p>
        </div>
        <!-- 右侧图片区域 -->
        <Icon
          name="s:commercial-distribution"
          class="w-full max-w-[20rem] sm:max-w-[24rem] lg:max-w-[30rem] h-auto object-contain hover:-translate-y-2 transition-transform duration-500 s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
          mode="svg"
        />
      </div>
    </section>

    <section
      class="flex flex-col items-center justify-center gap-6 lg:gap-8 px-8 pb-24 pt-24"
    >
      <div
        class="flex items-center justify-center self-start sm:self-center gap-4 sm:gap-6"
      >
        <Icon
          name="s:commercial-governance-title"
          mode="svg"
          class="!text-6xl sm:!text-7xl md:!text-[6rem] lg:!text-[7rem] text-primary-color s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
        />
        <h2
          class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl text-primary-color font-bold leading-[1.3] dark:text-primary-400"
        >
          <span
            class="font-(family-name:--s-title-font) text-primary-400 font-normal dark:opacity-50 tracking-widest uppercase"
            >GOVERNANCE</span
          ><br />
          内网管控&nbsp;&nbsp;应用中枢
        </h2>
      </div>
      <div
        class="flex gap-8 lg:gap-16 max-w-7xl w-full flex-col sm:flex-row items-center justify-center"
      >
        <Icon
          name="s:commercial-governance"
          class="w-full max-w-[20rem] sm:max-w-[24rem] lg:max-w-[30rem] h-auto object-contain hover:-translate-y-2 transition-transform duration-500 s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
          mode="svg"
        />
        <div
          class="flex flex-col py-0 sm:py-8 lg:py-12 gap-6 items-center sm:items-start flex-1"
        >
          <div
            class="flex flex-col text-center sm:text-start items-center sm:items-start"
          >
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl leading-[1.5]"
            >
              内网应用管控
            </p>
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold leading-[1.5] whitespace-nowrap"
            >
              统一上架授权运维
            </p>
          </div>
          <p
            class="text-base sm:text-lg text-center sm:text-start text-surface-700 leading-[1.8] dark:text-surface-300"
          >
            后台管理端面向企业内网运维场景，提供应用上架、下架、分类编辑、首页推荐、客户端授权许可、暂存推送、旧版本清理和客户端定制等能力。管理员可通过后端统一维护软件仓库与终端接入权限，实现应用分发、更新发布和安全管控闭环。
          </p>
          <p
            class="font-bold text-surface-700 dark:text-surface-300 text-base md:text-lg leading-[2]"
          >
            应用管理 / 终端授权 / 暂存推送 / 安全管控
          </p>
        </div>
      </div>
    </section>

    <section
      class="flex flex-col items-center justify-center gap-6 lg:gap-8 px-8 pb-24 pt-24"
    >
      <div
        class="flex items-center justify-center self-start sm:self-center gap-4 sm:gap-6"
      >
        <Icon
          name="s:commercial-case-title"
          mode="svg"
          class="!text-6xl sm:!text-7xl md:!text-[6rem] lg:!text-[7rem] text-primary-color s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
        />
        <h2
          class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl text-primary-color font-bold leading-[1.3] dark:text-primary-400"
        >
          <span
            class="font-(family-name:--s-title-font) text-primary-400 font-normal dark:opacity-50 tracking-widest uppercase"
            >CASE STUDY</span
          ><br />
          成功案例&nbsp;&nbsp;稳如泰山
        </h2>
      </div>
      <div
        class="flex gap-8 lg:gap-16 max-w-7xl w-full flex-col sm:flex-row items-center justify-center"
      >
        <Icon
          name="s:commercial-case"
          class="w-full max-w-[20rem] sm:max-w-[24rem] lg:max-w-[30rem] h-auto object-contain hover:-translate-y-2 transition-transform duration-500 s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
          mode="svg"
        />
        <div
          class="flex flex-col py-0 sm:py-8 lg:py-12 gap-6 items-center sm:items-start flex-1"
        >
          <div
            class="flex flex-col text-center sm:text-start items-center sm:items-start"
          >
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl leading-[1.5]"
            >
              无惧极端异构
            </p>
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold leading-[1.5] whitespace-nowrap"
            >
              政务终端统一纳管
            </p>
          </div>
          <p
            class="text-base sm:text-lg text-center sm:text-start text-surface-700 leading-[1.8] dark:text-surface-300"
          >
            某机关单位信创改造项目中，面临高度复杂的异构终端环境。该单位同时部署了搭载兆芯、飞腾、麒麟芯片的设备，并交叉安装统信
            UOS、银河麒麟与中科方德。通过部署星火应用商店专业版进行整合，该单位最终成功实现了跨异构、跨操作系统的软件高效率统一分发与纳管。
          </p>
          <div class="flex gap-4 mt-2">
            <!--<NuxtLink
              to="/cases"
              class="text-primary-500 font-bold hover:text-primary-600 dark:hover:text-primary-400 transition-colors flex items-center gap-2"
              >查看详细案例 <i class="pi pi-arrow-right text-sm"
            /></NuxtLink>-->
          </div>
        </div>
      </div>
    </section>

    <section
      class="flex flex-col items-center justify-center gap-6 lg:gap-8 px-8 pb-24 pt-24"
    >
      <div
        class="flex items-center justify-center self-start sm:self-center gap-4 sm:gap-6"
      >
        <Icon
          name="s:commercial-contact-title"
          mode="svg"
          class="!text-6xl sm:!text-7xl md:!text-[6rem] lg:!text-[7rem] text-primary-color s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
        />
        <h2
          class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl text-primary-color font-bold leading-[1.3] dark:text-primary-400"
        >
          <span
            class="font-(family-name:--s-title-font) text-primary-400 font-normal dark:opacity-50 tracking-widest uppercase"
            >CONTACT US</span
          ><br />
          联系客服&nbsp;&nbsp;定制报价
        </h2>
      </div>
      <div
        class="flex gap-8 lg:gap-16 max-w-7xl w-full flex-col-reverse sm:flex-row items-center justify-center"
      >
        <div
          class="flex flex-col py-0 sm:py-8 lg:py-12 gap-6 items-center sm:items-start flex-1"
        >
          <div
            class="flex flex-col text-center sm:text-start items-center sm:items-start"
          >
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl leading-[1.5]"
            >
              全天候客服护航
            </p>
            <p
              class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold leading-[1.5] whitespace-nowrap"
            >
              按需提供专业解决方案
            </p>
          </div>
          <p
            class="text-base sm:text-lg text-center sm:text-start text-surface-700 leading-[1.8] dark:text-surface-300"
          >
            商业版采用定制化报价模式，根据企业客户的具体需求（如定制程度、部署规模、私有化部署要求及驻点技术支持等）提供更具性价比的解决方案。期待与您的团队达成信创合作共识，我们将为您提供专业透明的咨询服务和报价方案。
          </p>
          <div class="flex gap-4 mt-2">
            <button
              @click="contactDialog = true"
              class="px-8 sm:px-14 py-3 text-xl font-bold text-white from-primary-400 to-primary-500 bg-linear-to-r rounded-full shadow-lg shadow-primary-500/30 dark:from-primary-500 dark:to-primary-600 hover:-translate-y-1 transition-transform"
            >
              <i class="pi pi-envelope text-xl! font-bold! pr-2 sm:pr-2" />
              马上联系
            </button>
          </div>
        </div>
        <Icon
          name="s:commercial-contact"
          class="w-full max-w-[20rem] sm:max-w-[24rem] lg:max-w-[30rem] h-auto object-contain hover:-translate-y-2 transition-transform duration-500 s-deco-primary-700 s-bg-primary-100 s-bg-2-primary-200 s-bg-3-primary-400 dark:s-deco-primary-400 dark:s-bg-primary-800 dark:s-bg-2-primary-600 dark:s-bg-3-primary-900"
          mode="svg"
        />
      </div>
    </section>

    <Dialog
      v-model:visible="contactDialog"
      modal
      :show-header="false"
      class="!w-auto !min-w-[320px] !max-w-[calc(100vw-2rem)] rounded-3xl!"
      :style="{
        '--p-dialog-border-color': 'transparent',
      }"
      content-class="flex flex-col items-center p-5! w-full h-full gap-4 overflow-hidden!"
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
      dismissable-mask
    >
      <p class="flex items-center justify-center gap-2 font-bold mb-2">
        联系商业版专线客服
      </p>

      <div class="flex flex-col gap-4 text-sm w-full font-medium">
        <p>通过邮箱联系：</p>
        <div
          class="flex flex-col gap-2 p-4 bg-surface-100 dark:bg-surface-800 rounded-xl"
        >
          <div class="flex items-center justify-between">
            <span class="text-surface-500">主收件：</span>
            <span class="select-all font-mono">shenmo@spark-app.store</span>
          </div>
          <div class="flex items-center justify-between">
            <span class="text-surface-500">抄送：</span>
            <span class="select-all font-mono">momen@spark-app.store</span>
          </div>
        </div>
      </div>

      <div class="flex flex-col gap-4 text-sm w-full font-medium mt-2">
        <p>也可以通过 QQ 联系：</p>
        <div
          class="flex flex-col gap-2 p-4 bg-surface-100 dark:bg-surface-800 rounded-xl"
        >
          <div class="flex items-center justify-between">
            <span class="text-surface-500">shenmo：</span>
            <span class="select-all font-mono">1422953823</span>
          </div>
          <div class="flex items-center justify-between">
            <span class="text-surface-500">momen：</span>
            <span class="select-all font-mono">954551908</span>
          </div>
        </div>
      </div>

      <button
        @click="contactDialog = false"
        class="mt-4 px-8 py-2 font-bold text-white bg-primary-500 rounded-full hover:bg-primary-600 transition-colors"
      >
        关闭
      </button>
    </Dialog>
  </div>
</template>

<style scoped>
section {
  width: 100%;
  height: 100vh;
  scroll-snap-align: start;
}
</style>
