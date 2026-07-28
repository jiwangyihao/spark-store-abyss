<script setup lang="ts">
import {
  onMounted,
  onBeforeUnmount,
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
const governancePreviewVisible = ref(false);
const governanceSection = ref<HTMLElement | null>(null);
const governanceTabs = ref<HTMLElement | null>(null);
const customizationSection = ref<HTMLElement | null>(null);
const governanceVisible = ref(false);
const activeGovernanceScene = ref(0);
const governanceScenes = [
  {
    title: "总览",
    desc: "企业应用商店运行状态集中总览",
    hint: "授权状态  |  客户端统计  |  仓库概览  |  暂存操作",
    image: "/images/02-dashboard.png",
    features: [
      ["核心状态总览", "集中展示授权有效性、客户端配额与服务器运行信息"],
      ["多架构仓库统计", "快速掌握 amd64、arm64 与 loong64 仓库的应用数量和总体积"],
      ["变更动态追踪", "查看待推送改动与最近暂存操作，及时掌握内容变更"],
    ],
  },
  {
    title: "应用管理",
    desc: "多架构应用仓库统一维护",
    hint: "应用检索  |  分类管理  |  移动下架  |  暂存变更",
    image: "/images/03-apps.png",
    features: [
      ["多架构仓库", "按 amd64、arm64 与 loong64 分别维护应用"],
      ["应用全程管理", "集中查看详情、调整分类、移动或下架应用"],
      ["变更统一推送", "暂存应用调整，确认后一次性保存并推送"],
    ],
  },
  {
    title: "应用上架",
    desc: "安装包解析与信息录入",
    hint: "上传安装包  |  自动解析  |  架构选择  |  信息完善",
    image: "/images/04-publish.png",
    features: [
      ["安装包上传", "支持拖入或选择对应架构的 deb 安装包"],
      ["信息自动解析", "上传后自动解析软件包基础元数据"],
      ["分步上架", "按上传、编辑和确认流程规范发布应用"],
    ],
  },
  {
    title: "首页推荐",
    desc: "客户端首页内容统一运营",
    hint: "推荐配置  |  内容排序  |  多架构适配  |  即时发布",
    image: "/images/05-home.png",
    features: [
      ["推荐内容配置", "集中选择并维护客户端首页推荐应用"],
      ["展示顺序调整", "根据企业需要灵活调整推荐内容顺序"],
      ["多架构运营", "分别维护不同架构客户端的首页内容"],
    ],
  },
  {
    title: "客户端管理",
    desc: "终端接入与状态集中管理",
    hint: "终端总览  |  状态查询  |  接入管理  |  权限控制",
    image: "/images/06-clients.png",
    features: [
      ["终端统一纳管", "集中查看已接入企业内网的客户端"],
      ["运行状态总览", "快速掌握终端版本、架构和在线状态"],
      ["接入权限控制", "通过后端统一维护终端访问权限"],
    ],
  },
  {
    title: "运维管理",
    desc: "仓库发布与历史版本维护",
    hint: "暂存推送  |  更新发布  |  旧版清理  |  仓库维护",
    image: "/images/07-maintenance.png",
    features: [
      ["变更暂存推送", "审核仓库变更后统一保存并推送生效"],
      ["旧版本清理", "清理历史软件包，控制仓库存储占用"],
      ["发布过程可控", "形成应用变更、审核和发布管理闭环"],
    ],
  },
  {
    title: "客户端定制",
    desc: "企业品牌与客户端能力配置",
    hint: "品牌定制  |  客户端配置  |  内网部署  |  统一交付",
    image: "/images/08-customize.png",
    features: [
      ["企业形象定制", "配置客户端名称、标识与品牌视觉"],
      ["功能统一配置", "集中维护客户端功能和默认参数"],
      ["专属客户端交付", "生成满足企业内网部署需求的客户端"],
    ],
  },
  {
    title: "授权管理",
    desc: "客户端授权许可集中管控",
    hint: "许可状态  |  授权配额  |  有效期限  |  公钥校验",
    image: "/images/09-license.png",
    features: [
      ["授权状态总览", "集中查看许可状态、有效期和终端配额"],
      ["许可安全校验", "使用信任公钥校验授权许可真实性"],
      ["终端额度管控", "统一控制企业客户端可用授权数量"],
    ],
  },
  {
    title: "系统设置",
    desc: "服务端与仓库运行参数配置",
    hint: "仓库路径  |  服务端口  |  激活模式  |  密码策略",
    image: "/images/10-settings.png",
    features: [
      ["服务端配置", "维护对外地址、仓库路径和服务端口"],
      ["客户端激活模式", "根据部署要求选择自动或手动激活"],
      ["管理安全策略", "配置许可公钥、密码过期和管理员密码"],
    ],
  },
];
let governanceObserver: IntersectionObserver | undefined;
let governanceWheelCleanup: (() => void) | undefined;
let governanceWheelLock = false;

const isGovernanceScrollMode = () =>
  window.matchMedia("(min-width: 1200px)").matches &&
  !window.matchMedia("(prefers-reduced-motion: reduce)").matches;

const getScrollContainer = () =>
  scrollPanel?.value?.$el.firstChild.firstChild as HTMLElement | undefined;

const getGovernanceSectionTop = () => {
  const container = getScrollContainer();
  if (!container || !governanceSection.value) return 0;
  return governanceSection.value.offsetTop;
};

const scrollToGovernanceScene = (index: number) => {
  const container = getScrollContainer();
  if (!container) return;

  container.scrollTo({
    top: getGovernanceSectionTop() + index * container.clientHeight,
    behavior: "smooth",
  });
};

const syncGovernanceTabsScroll = () => {
  const tabs = governanceTabs.value;
  if (!tabs) return;

  window.requestAnimationFrame(() => {
    const activeTab = tabs.querySelector<HTMLElement>(".governance-tab.active");
    if (!activeTab) return;

    const tabsRect = tabs.getBoundingClientRect();
    const activeRect = activeTab.getBoundingClientRect();
    const edgeGap = 12;

    if (activeRect.top < tabsRect.top + edgeGap) {
      tabs.scrollBy({
        top: activeRect.top - tabsRect.top - edgeGap,
        behavior: "smooth",
      });
      return;
    }

    if (activeRect.bottom > tabsRect.bottom - edgeGap) {
      tabs.scrollBy({
        top: activeRect.bottom - tabsRect.bottom + edgeGap,
        behavior: "smooth",
      });
    }
  });
};

const handleGovernanceWheel = (event: WheelEvent) => {
  const container = getScrollContainer();
  if (
    !container ||
    !governanceSection.value ||
    governancePreviewVisible.value ||
    !isGovernanceScrollMode()
  ) {
    return;
  }

  const sectionTop = getGovernanceSectionTop();
  const stepHeight = container.clientHeight;
  const sectionEnd = sectionTop + governanceScenes.length * stepHeight;
  const currentTop = container.scrollTop;
  const direction = Math.sign(event.deltaY);

  const isEnteringFromBefore =
    direction > 0 &&
    currentTop < sectionTop &&
    sectionTop - currentTop <= stepHeight;

  if (isEnteringFromBefore) {
    event.preventDefault();
    if (governanceWheelLock) return;
    governanceWheelLock = true;
    activeGovernanceScene.value = 0;
    syncGovernanceTabsScroll();
    scrollToGovernanceScene(0);
    window.setTimeout(() => {
      governanceWheelLock = false;
    }, 760);
    return;
  }

  const isInsideGovernance =
    currentTop >= sectionTop - 2 && currentTop < sectionEnd - 2;

  if (!isInsideGovernance) return;

  if (
    (direction > 0 && activeGovernanceScene.value >= governanceScenes.length - 1) ||
    (direction < 0 && activeGovernanceScene.value <= 0)
  ) {
    return;
  }

  event.preventDefault();

  if (governanceWheelLock || direction === 0) return;
  governanceWheelLock = true;

  const nextScene = Math.min(
    governanceScenes.length - 1,
    Math.max(0, activeGovernanceScene.value + direction),
  );
  activeGovernanceScene.value = nextScene;
  syncGovernanceTabsScroll();
  scrollToGovernanceScene(nextScene);

  window.setTimeout(() => {
    governanceWheelLock = false;
  }, 760);
};

onMounted(() => {
  watchEffect(() => {
    const scrollTop =
      (scrollPanel?.value as unknown as { lastScrollTop: number })
        .lastScrollTop || 0;
    const clientHeight = scrollPanel?.value?.$el.clientHeight || 1;
    sProgress.value = scrollTop / clientHeight;

    if (
      governanceSection.value &&
      isGovernanceScrollMode()
    ) {
      const container = getScrollContainer();
      if (!container) return;

      const sectionTop = getGovernanceSectionTop();
      const progress = Math.min(
        governanceScenes.length - 1,
        Math.max(0, (container.scrollTop - sectionTop) / container.clientHeight),
      );
      activeGovernanceScene.value = Math.round(progress);
      syncGovernanceTabsScroll();
    }
  });

  governanceObserver = new IntersectionObserver(
    ([entry]) => {
      if (entry?.isIntersecting) {
        governanceVisible.value = true;
      }
    },
    { threshold: 0 },
  );

  if (governanceSection.value) {
    governanceObserver.observe(governanceSection.value);
  }

  const container = getScrollContainer();
  if (container) {
    container.addEventListener("wheel", handleGovernanceWheel, { passive: false });
    governanceWheelCleanup = () =>
      container.removeEventListener("wheel", handleGovernanceWheel);
  }
});

onBeforeUnmount(() => {
  governanceObserver?.disconnect();
  governanceWheelCleanup?.();
});

const scrollToNext = () => {
  const container = getScrollContainer();
  if (container && customizationSection.value) {
    container.scrollTo({
      top: customizationSection.value.offsetTop,
      behavior: "smooth",
    });
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
      ref="governanceSection"
      class="governance-section relative flex items-center justify-center overflow-hidden px-5 sm:px-8 lg:px-12 py-6 lg:py-8"
      :class="{ 'is-visible': governanceVisible }"
    >
      <div class="governance-sticky relative z-10 w-full max-w-[92rem]">
        <div class="governance-heading flex items-center justify-center gap-4 sm:gap-5 mb-3 lg:mb-4">
          <img src="/assets/images/commercial/spark-store-enterprise.png" alt="星火应用商店商业版" class="w-[1em] h-[1em] !text-6xl sm:!text-7xl md:!text-[6rem] lg:!text-[7rem] object-contain" />
          <h2 class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold leading-[1.3]">
            <span class="text-[#292929] dark:text-surface-50">星火应用商店</span><br />
            <span class="text-primary-color dark:text-primary-400">商业版</span>
          </h2>
        </div>
        <div class="governance-intro">
          <p>为政府、企事业单位客户提供专业的应用商店解决方案：</p>
          <div>
            <span><i class="pi pi-check" />支持安装在主流信创设备上</span>
            <span><i class="pi pi-check" />支持定制客户端和服务器端本地部署</span>
            <span><i class="pi pi-check" />支持指定软件适配服务和信创设备迁移</span>
          </div>
          <p>助力客户构建完善的软件分发体系。</p>
          <div class="governance-actions">
            <button type="button" @click="sendEmail">联系客服</button>
            <button type="button" @click="scrollToNext">了解更多</button>
          </div>
        </div>
        <div class="governance-panel">
          <nav ref="governanceTabs" class="governance-tabs" aria-label="内网管控场景">
            <button v-for="(scene, index) in governanceScenes" :key="scene.title" type="button" class="governance-tab" :class="{ active: activeGovernanceScene === index }" @click="activeGovernanceScene = index; syncGovernanceTabsScroll(); if (isGovernanceScrollMode()) scrollToGovernanceScene(index);">
              <span class="governance-tab-title">{{ scene.title }}</span>
              <span class="governance-tab-desc">{{ scene.desc }}</span>
            </button>
          </nav>
          <div class="governance-detail">
            <Transition name="governance-scene" mode="out-in">
              <div :key="activeGovernanceScene" class="governance-detail-copy">
                <div>
                  <p class="text-lg text-surface-600 dark:text-surface-300">{{ governanceScenes[activeGovernanceScene]?.desc }}</p>
                  <h3 class="text-2xl sm:text-3xl lg:text-4xl font-bold mt-1">{{ governanceScenes[activeGovernanceScene]?.title }}</h3>
                </div>
                <p class="governance-hint">{{ governanceScenes[activeGovernanceScene]?.hint }}</p>
                <ul class="governance-features">
                  <li v-for="feature in governanceScenes[activeGovernanceScene]?.features" :key="feature[0]"><strong>{{ feature[0] }}</strong><span>{{ feature[1] }}</span></li>
                </ul>
              </div>
            </Transition>
            <div class="governance-visual relative">
              <Transition name="governance-image" mode="out-in">
                <button :key="governanceScenes[activeGovernanceScene]?.image" type="button" class="governance-image-frame" :aria-label="`放大查看${governanceScenes[activeGovernanceScene]?.title}后台界面`" @click="governancePreviewVisible = true">
                  <img :src="governanceScenes[activeGovernanceScene]?.image" :alt="`${governanceScenes[activeGovernanceScene]?.title}后台界面`" class="governance-dashboard" />
                  <span class="governance-zoom-hint"><i class="pi pi-search-plus" />点击放大</span>
                </button>
              </Transition>
            </div>
          </div>
        </div>
      </div>
    </section>
    <section
      v-if="false"
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
      ref="customizationSection"
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
      v-model:visible="governancePreviewVisible"
      modal
      dismissable-mask
      :show-header="false"
      class="governance-preview-dialog"
      content-class="governance-preview-content"
      :pt="{
        root: {
          style: 'padding:0; overflow:hidden;',
        },
        content: {
          style: 'padding:0; margin:0; overflow:auto;',
        },
        mask: {
          class: 'backdrop-blur-xl bg-black/70!',
        },
      }"
    >
      <button
        type="button"
        class="governance-preview-close"
        aria-label="关闭图片预览"
        @click="governancePreviewVisible = false"
      >
        <i class="pi pi-times" />
      </button>
      <img
        :src="governanceScenes[activeGovernanceScene]?.image"
        :alt="`${governanceScenes[activeGovernanceScene]?.title}后台界面大图`"
        class="governance-preview-image"
      />
    </Dialog>

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
  min-height: 100dvh;
  height: 100dvh;
  scroll-snap-align: start;
}

.governance-section {
  isolation: isolate;
  perspective: 1400px;
}

@media (min-width: 1200px) and (prefers-reduced-motion: no-preference) {
  .governance-section {
    height: 900vh;
    min-height: 900vh;
    align-items: flex-start;
    overflow: visible;
    padding-block: 0;
    scroll-snap-align: start;
    scroll-snap-stop: always;
  }

  .governance-sticky {
    position: sticky;
    top: 6.75rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    height: calc(100dvh - 6.75rem);
    padding-block: 0.5rem;
  }
}

.governance-intro {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 0.5rem 1rem;
  max-width: 68rem;
  margin: 0 auto 1rem;
  color: var(--p-surface-600);
  font-size: 0.9rem;
}

.governance-intro > p:first-child {
  font-weight: 700;
}

.governance-intro > div:not(.governance-actions) {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.4rem 0.9rem;
}

.governance-intro span {
  display: inline-flex;
  align-items: center;
  gap: 0.35rem;
}

.governance-intro .pi-check {
  color: var(--p-primary-500);
}

.governance-intro > p:last-of-type {
  font-weight: 700;
}

.governance-actions {
  display: flex;
  gap: 0.75rem;
}

.governance-actions button {
  padding: 0.55rem 1rem;
  border-radius: 9999px;
  font-weight: 700;
  transition: transform 200ms ease, background-color 200ms ease;
}

.governance-actions button:hover {
  transform: translateY(-2px);
}

.governance-actions button:first-child {
  border: 0;
  color: white;
  background: var(--p-primary-500);
}

.governance-actions button:last-child {
  border: 1px solid var(--p-primary-500);
  color: var(--p-primary-600);
  background: transparent;
}

.s-dark .governance-intro {
  color: var(--p-surface-300);
}

@media (max-width: 767px) {
  .governance-intro {
    align-items: flex-start;
    flex-direction: column;
    margin-bottom: 0.75rem;
    font-size: 0.82rem;
  }

  .governance-intro > div:not(.governance-actions) {
    justify-content: flex-start;
  }
}

.governance-heading,
.governance-visual {
  opacity: 0;
  transition:
    opacity 700ms ease,
    transform 900ms cubic-bezier(0.22, 1, 0.36, 1);
}

.governance-heading {
  transform: translateY(-30px);
}

.governance-visual {
  transform: translateX(90px) translateY(28px) rotateY(-10deg) scale(0.78);
  transform-origin: center bottom;
  transition-delay: 220ms;
}

.governance-section.is-visible .governance-heading,
.governance-section.is-visible .governance-visual {
  opacity: 1;
  transform: translate3d(0, 0, 0) rotateY(0) scale(1);
}

.governance-panel {
  display: grid;
  grid-template-columns: 13rem minmax(0, 1fr);
  height: min(34rem, calc(100dvh - 15rem));
  min-height: 22rem;
  overflow: hidden;
  border: 1px solid color-mix(in srgb, var(--p-primary-400) 18%, var(--p-surface-200));
  border-radius: 1.75rem;
  background: color-mix(in srgb, var(--p-surface-0) 88%, transparent);
  box-shadow: 0 24px 70px color-mix(in srgb, var(--p-primary-900) 12%, transparent);
  backdrop-filter: blur(18px);
}

.governance-tabs {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  padding: 1rem 0.75rem;
  overflow-y: auto;
  border-right: 1px solid color-mix(in srgb, var(--p-primary-400) 15%, var(--p-surface-200));
  background: color-mix(in srgb, var(--p-surface-50) 72%, transparent);
}

.governance-tab {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  width: 100%;
  padding: 0.65rem 0.85rem 0.65rem 1.15rem;
  border: 0;
  border-radius: 1rem;
  color: var(--p-surface-600);
  background: transparent;
  text-align: left;
  cursor: pointer;
  transition: color 250ms ease, background-color 250ms ease, transform 250ms ease;
}

.governance-tab::before {
  content: "";
  position: absolute;
  left: 0;
  top: 20%;
  width: 0.25rem;
  height: 60%;
  border-radius: 9999px;
  background: var(--p-primary-500);
  opacity: 0;
  transform: scaleY(0.3);
  transition: opacity 250ms ease, transform 250ms ease;
}

.governance-tab:hover {
  color: var(--p-primary-600);
  background: color-mix(in srgb, var(--p-primary-400) 7%, transparent);
  transform: translateX(3px);
}

.governance-tab.active {
  color: var(--p-primary-700);
  background: color-mix(in srgb, var(--p-primary-400) 13%, transparent);
}

.governance-tab.active::before {
  opacity: 1;
  transform: scaleY(1);
}

.governance-tab-title {
  font-size: 1.05rem;
  font-weight: 800;
}

.governance-tab-desc {
  color: var(--p-surface-500);
  font-size: 0.8rem;
  line-height: 1.5;
}

.governance-detail {
  display: grid;
  grid-template-columns: minmax(18rem, 0.72fr) minmax(28rem, 1.28fr);
  align-items: center;
  gap: 2rem;
  padding: 2.25rem;
  overflow: hidden;
}

.governance-detail-copy {
  display: flex;
  flex-direction: column;
  gap: 1.35rem;
}

.governance-hint {
  color: var(--p-primary-600);
  font-size: 0.92rem;
  font-weight: 700;
  line-height: 1.7;
}

.governance-features {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.governance-features li {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
  padding-left: 1rem;
  border-left: 2px solid color-mix(in srgb, var(--p-primary-400) 45%, transparent);
}

.governance-features strong {
  font-size: 1rem;
}

.governance-features span {
  color: var(--p-surface-500);
  font-size: 0.875rem;
  line-height: 1.6;
}

.governance-visual {
  align-self: end;
}

.governance-image-frame {
  position: relative;
  display: block;
  width: 100%;
  aspect-ratio: 1.72;
  padding: 0;
  overflow: hidden;
  border: 1px solid color-mix(in srgb, var(--p-primary-400) 18%, transparent);
  border-radius: 0.85rem;
  background: var(--p-surface-100);
  box-shadow: 0 20px 50px color-mix(in srgb, var(--p-primary-900) 15%, transparent);
  cursor: zoom-in;
  text-align: initial;
}

.governance-image-frame:focus-visible {
  outline: 3px solid var(--p-primary-400);
  outline-offset: 3px;
}

.governance-zoom-hint {
  position: absolute;
  right: 0.8rem;
  bottom: 0.8rem;
  z-index: 2;
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.5rem 0.7rem;
  border-radius: 9999px;
  color: white;
  background: rgb(15 23 42 / 72%);
  box-shadow: 0 8px 24px rgb(0 0 0 / 20%);
  backdrop-filter: blur(10px);
  font-size: 0.75rem;
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 220ms ease, transform 220ms ease;
}

.governance-image-frame:hover .governance-zoom-hint,
.governance-image-frame:focus-visible .governance-zoom-hint {
  opacity: 1;
  transform: translateY(0);
}

.governance-image-frame:hover .governance-dashboard {
  transform: translateX(0) scale(1.02);
}

.governance-dashboard {
  display: block;
  width: 100%;
  height: auto;
  opacity: 0;
  transform: translateX(60px) scale(0.98);
  transition:
    opacity 850ms ease 260ms,
    transform 950ms cubic-bezier(0.22, 1, 0.36, 1) 260ms;
}

.governance-section.is-visible .governance-dashboard {
  opacity: 1;
  transform: translateX(0) scale(1);
}

.governance-preview-dialog {
  width: min(94vw, 90rem) !important;
  max-height: 94dvh;
  overflow: hidden;
  border: 0 !important;
  border-radius: 1.25rem !important;
  background: var(--p-surface-0) !important;
}

:deep(.governance-preview-content) {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 0;
  max-height: 94dvh;
  padding: 0 !important;
  overflow: auto !important;
  background: var(--p-surface-0);
}

.governance-preview-close {
  position: absolute;
  top: 0.75rem;
  right: 0.75rem;
  z-index: 3;
  display: grid;
  place-items: center;
  width: 2.5rem;
  height: 2.5rem;
  margin: 0;
  border: 1px solid rgb(255 255 255 / 30%);
  border-radius: 9999px;
  color: white;
  background: rgb(15 23 42 / 76%);
  cursor: pointer;
  backdrop-filter: blur(12px);
}

.governance-preview-image {
  display: block;
  width: 100%;
  height: auto;
  margin: 0;
  object-fit: contain;
  vertical-align: top;
}

@media (prefers-color-scheme: dark) {
  .governance-preview-dialog,
  :deep(.governance-preview-content) {
    background: var(--p-surface-900) !important;
  }

  .governance-preview-image {
    border-color: var(--p-surface-700);
  }
}

@media (max-width: 767px) {
  .governance-preview-dialog {
    width: calc(100vw - 1rem) !important;
    max-height: 92dvh;
    border-radius: 1rem !important;
  }

  :deep(.governance-preview-content) {
    max-height: 92dvh;
    padding: 0 !important;
  }

  .governance-zoom-hint {
    opacity: 1;
    transform: none;
  }
}

.governance-image-enter-active,
.governance-image-leave-active {
  transition: opacity 260ms ease, transform 320ms ease;
}

.governance-image-enter-from {
  opacity: 0;
  transform: translateX(24px);
}

.governance-image-leave-to {
  opacity: 0;
  transform: translateX(-18px);
}

.governance-scene-enter-active,
.governance-scene-leave-active {
  transition: opacity 220ms ease, transform 280ms ease;
}

.governance-scene-enter-from {
  opacity: 0;
  transform: translateY(16px);
}

.governance-scene-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

@media (prefers-color-scheme: dark) {
  .governance-panel {
    background: color-mix(in srgb, var(--p-surface-900) 88%, transparent);
  }

  .governance-tabs {
    background: color-mix(in srgb, var(--p-surface-950) 55%, transparent);
  }

  .governance-tab.active {
    color: var(--p-primary-300);
  }
}

@media (max-width: 1199px) {
  .governance-section {
    height: auto;
    min-height: 100dvh;
    overflow: visible;
    padding-block: max(5rem, env(safe-area-inset-top)) 2rem;
  }

  .governance-panel {
    grid-template-columns: 1fr;
    width: 100%;
    height: auto;
    min-height: 0;
  }

  .governance-tabs {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    overflow: visible;
    border-right: 0;
    border-bottom: 1px solid color-mix(in srgb, var(--p-primary-400) 15%, var(--p-surface-200));
  }

  .governance-tab {
    min-height: 4.5rem;
  }

  .governance-detail {
    grid-template-columns: minmax(15rem, 0.78fr) minmax(24rem, 1.22fr);
    padding: 1.75rem;
  }

  .governance-visual {
    transform: translateY(24px) scale(0.92);
  }
}

@media (max-width: 767px) {
  .governance-section {
    min-height: 100dvh;
    padding: max(5rem, calc(env(safe-area-inset-top) + 1.25rem)) 1rem 2rem;
  }

  .governance-heading {
    justify-content: flex-start;
    gap: 0.75rem;
    margin-bottom: 0.75rem;
  }

  .governance-heading :deep(svg) {
    flex: 0 0 auto;
    font-size: 3.25rem !important;
  }

  .governance-panel {
    border-radius: 1.25rem;
  }

  .governance-tabs {
    display: flex;
    flex-flow: row nowrap;
    gap: 0.35rem;
    padding: 0.65rem;
    overflow-x: auto;
    overscroll-behavior-x: contain;
    scrollbar-width: none;
    scroll-snap-type: x proximity;
  }

  .governance-tabs::-webkit-scrollbar {
    display: none;
  }

  .governance-tab {
    flex: 0 0 auto;
    width: auto;
    min-width: 7.75rem;
    min-height: 2.75rem;
    padding: 0.7rem 0.9rem 0.7rem 1.1rem;
    scroll-snap-align: start;
  }

  .governance-tab-desc {
    display: none;
  }

  .governance-tab-title {
    font-size: 0.95rem;
    white-space: nowrap;
  }

  .governance-detail {
    grid-template-columns: minmax(0, 1fr);
    gap: 1.25rem;
    padding: 1.1rem;
    overflow: visible;
  }

  .governance-detail-copy {
    gap: 0.9rem;
  }

  .governance-detail-copy h3 {
    font-size: 1.5rem;
  }

  .governance-detail-copy > div > p {
    font-size: 0.95rem;
  }

  .governance-hint {
    font-size: 0.78rem;
  }

  .governance-features {
    gap: 0.7rem;
  }

  .governance-features li {
    padding-left: 0.75rem;
  }

  .governance-features strong {
    font-size: 0.9rem;
  }

  .governance-features span {
    font-size: 0.78rem;
  }

  .governance-visual {
    align-self: auto;
    transform: translateY(20px) scale(0.94);
  }

  .governance-image-frame {
    aspect-ratio: 1.55;
    border-radius: 0.65rem;
  }
}

@media (max-width: 390px) {
  .governance-section {
    padding-inline: 0.75rem;
  }

  .governance-heading :deep(svg) {
    font-size: 2.85rem !important;
  }

  .governance-heading h2 {
    font-size: 1.25rem;
  }

  .governance-heading h2 span {
    font-size: 0.75rem;
  }
}

@media (min-width: 1200px) and (max-height: 760px) {
  .governance-panel {
    height: calc(100dvh - 17rem);
    min-height: 18rem;
  }

  .governance-sticky {
    justify-content: flex-start;
  }

  .governance-heading {
    margin-bottom: 0.5rem;
  }

  .governance-detail {
    min-height: 0;
    padding-block: 1.25rem;
    overflow-y: auto;
  }
}

@media (prefers-reduced-motion: reduce) {
  .governance-heading,
  .governance-copy,
  .governance-visual {
    opacity: 1;
    transform: none;
    transition: none;
  }

  .governance-dashboard,
  .governance-callout {
    opacity: 1;
    transform: none;
    transition: none;
  }
}
</style>
