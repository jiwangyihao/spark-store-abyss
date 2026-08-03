<script setup lang="ts">
import ScrollPanel from "primevue/scrollpanel";
import Button from "primevue/button";

import { useToast } from "primevue/usetoast";

const appConfig = useAppConfig();

const path = computed(() => {
  return useRoute().path;
});
const sProgress = ref(path.value === "/commercial" ? 0 : 1);
const scrollPanel = useTemplateRef<ComponentPublicInstance>("scrollPanel");
const header = useTemplateRef<HTMLElement>("header");
const sX = ref(0);
const sY = ref(0);
const sWidth = ref(0);
const sHeight = ref(0);
const mounted = ref(false);

provide("scrollPanel", scrollPanel);

const handleScrollOrResize = () => {
  const navEl = document.querySelector("header nav");
  const el = document.querySelector("header .active");
  if (el && navEl) {
    sX.value = el.getBoundingClientRect().x - navEl.getBoundingClientRect().x;
    sY.value = el.getBoundingClientRect().y - navEl.getBoundingClientRect().y;
    sWidth.value = el.getBoundingClientRect().width;
    sHeight.value = el.getBoundingClientRect().height;
  }
  const scrollTop = (scrollPanel.value as unknown as { lastScrollTop: number })
    .lastScrollTop;
  const clientHeight = scrollPanel.value?.$el.clientHeight;
  if (path.value === "/commercial") {
    sProgress.value = 0;
    return;
  }
  if (path.value === "/") {
    sProgress.value =
      1 -
      range(0, 1, scrollTop / clientHeight) +
      1 -
      Math.abs(range(-1, 1, scrollTop / clientHeight - 6));
    return;
  }
  sProgress.value = 1 - range(0, 1, scrollTop / clientHeight);
};

const handleHeaderFocus = () => {
  mounted.value = false;
  setTimeout(() => {
    handleScrollOrResize();
    nextTick(() => {
      mounted.value = true;
    });
  }, 300);
};

onMounted(() => {
  addEventListener("resize", handleScrollOrResize);
  watchEffect(handleScrollOrResize);
  nextTick(() => {
    mounted.value = true;
  });

  watchEffect(() => {
    if (path.value) {
      scrollPanel.value?.$el.firstChild.firstChild.scrollTo({
        top: 0,
        behavior: "instant",
      });
    }
  });
});

const qrWidth = ref(0);

onMounted(() => {
  const qrEl = document.querySelector("footer h2+svg");
  if (qrEl) {
    qrEl.addEventListener("resize", () => {
      qrWidth.value = qrEl.getBoundingClientRect().height;
    });
    qrWidth.value = qrEl.getBoundingClientRect().height;
  }
});

const sDarkConfig = ref("auto");

const isDarkMode = ref(false);

const sDark = computed(() => {
  return sDarkConfig.value === "auto"
    ? isDarkMode.value
      ? true
      : false
    : sDarkConfig.value === "dark"
      ? true
      : false;
});

provide("sDark", sDark);

const enableTransitions = () =>
  "startViewTransition" in document &&
  window.matchMedia("(prefers-reduced-motion: no-preference)").matches;

const mouseX = ref(0);
const mouseY = ref(0);

const toggleDarkMode = async ({ clientX: x, clientY: y }: MouseEvent) => {
  if (sDarkConfig.value === "auto") {
    sDarkConfig.value = isDarkMode.value ? "light" : "dark";
  } else if (sDarkConfig.value === "dark" && isDarkMode.value) {
    sDarkConfig.value = "auto";
  } else if (sDarkConfig.value === "light" && !isDarkMode.value) {
    sDarkConfig.value = "auto";
  } else {
    sDarkConfig.value = isDarkMode.value ? "dark" : "light";
  }
  localStorage.setItem("darkMode", sDarkConfig.value);

  mouseX.value = x;
  mouseY.value = y;
};

onMounted(() => {
  const schemeQuery = window.matchMedia("(prefers-color-scheme: dark)");
  isDarkMode.value = schemeQuery.matches;
  schemeQuery.addEventListener("change", (e) => {
    isDarkMode.value = e.matches;
  });
  const savedDarkMode = localStorage.getItem("darkMode");
  if (savedDarkMode) {
    sDarkConfig.value = savedDarkMode;
  }
  watch(
    sDark,
    async (newValue, oldValue) => {
      console.log("Dark mode changed:", newValue, oldValue);
      const sDarkValue = newValue;

      if (oldValue === undefined) {
        return;
      }

      if (!enableTransitions()) {
        document.documentElement.classList.toggle("s-dark", sDarkValue);
        return;
      }

      const clipPath = [
        `circle(0px at ${mouseX.value}px ${mouseY.value}px)`,
        `circle(${Math.hypot(
          Math.max(mouseX.value, innerWidth - mouseX.value),
          Math.max(mouseY.value, innerHeight - mouseY.value),
        )}px at ${mouseX.value}px ${mouseY.value}px)`,
      ];

      await document.startViewTransition(async () => {
        document.documentElement.classList.toggle("s-dark", sDarkValue);
        await nextTick();
      }).ready;

      document.documentElement.animate(
        { clipPath: sDarkValue ? clipPath.reverse() : clipPath },
        {
          duration: 300,
          easing: "ease-in",
          pseudoElement: `::view-transition-${
            sDarkValue ? "old" : "new"
          }(root)`,
        },
      );
    },
    { immediate: true },
  );
});

// 数据获取
export interface Release {
  assets: {
    name: string;
    browser_download_url: string;
  }[];
  tag_name: string;
  created_at: string;
  body: string;
}

const { data: latestRelease } = await useAsyncData(
  "latestRelease",
  async () => {
    if (appConfig.latestRelease) {
      return appConfig.latestRelease;
    } else {
      return await $fetch(
        "https://gitee.com/api/v5/repos/spark-store-project/spark-store/releases/latest",
      );
    }
  },
);

provide("latestRelease", latestRelease);

const dialog = ref(false);

const joinGroup = () => {
  dialog.value = true;
  window.open(
    "https://qm.qq.com/cgi-bin/qm/qr?authKey=1Oz2h5JBFSxBsHM63bwY1x0OswtQ08SBKj%2Fw9CRTxZ9%2B8%2FRKNIEv35s%2FhjOdRqkj&k=e7gkRbeGAjTfGupzsWbSbI8mOJM3ZqHQ&noverify=0",
    "_blank",
  );
};

const toast = useToast();

const copyGroupNumber = async () => {
  try {
    await navigator.clipboard.writeText("424677882");
    toast.add({
      severity: "success",
      summary: "复制成功",
      detail: "QQ群号已复制到剪贴板",
      life: 5000,
    });
  } catch {
    toast.add({
      severity: "error",
      summary: "复制失败",
      detail: "请手动复制QQ群号：424677882",
      life: 5000,
    });
  }
};
</script>

<template>
  <ScrollPanel
    ref="scrollPanel"
    class="overflow-hidden"
    style="width: 100vw; height: 100vh"
    :pt="{
      content: {
        class: 'snap-y snap-mandatory scroll-smooth overflow-x-hidden!',
      },
    }"
  >
    <header
      ref="header"
      tabindex="0"
      class="fixed w-full h-15 z-10 px-4 sm:px-8 lg:px-12 translate-y-[calc(var(--s-progress)*4*var(--spacing))] sm:translate-y-[calc(var(--s-progress)*8*var(--spacing))] lg:translate-y-[calc(var(--s-progress)*12*var(--spacing))] before:translate-x-[calc(var(--s-progress)*4*var(--spacing))] sm:before:translate-x-[calc(var(--s-progress)*8*var(--spacing))] lg:before:translate-x-[calc(var(--s-progress)*12*var(--spacing))] before:w-[calc(100%-var(--s-progress)*8*var(--spacing))] sm:before:w-[calc(100%-var(--s-progress)*16*var(--spacing))] lg:before:w-[calc(100%-var(--s-progress)*24*var(--spacing))] sm:h-auto focus-within:h-auto before:h-15 focus-within:before:h-93.5 sm:before:h-full focus-within:sm:before:h-full overflow-hidden focus-within:overflow-visible transition-discrete group"
      :class="{ 'commercial-header': path === '/commercial' }"
      :style="{ '--s-progress': sProgress }"
      @click="if (!header?.matches(':focus-within')) header?.focus();"
      @focus="handleHeaderFocus"
    >
      <nav
        class="relative flex px-2.5 md:px-4 lg:px-8 items-center flex-col lg:flex-row before:opacity-0 group-focus-within:before:opacity-100 sm:before:opacity-100 before:origin-top before:scale-0 group-focus-within:before:scale-100 sm:before:scale-100 before:translate-x-[calc(50vw-14.5*var(--spacing))] before:translate-y-15 group-focus-within:before:translate-x-[calc(var(--s-x)*1px)] group-focus-within:before:translate-y-[calc(var(--s-y)*1px)] sm:before:translate-x-[calc(var(--s-x)*1px)] sm:before:translate-y-[calc(var(--s-y)*1px)]"
        :class="{ mounted }"
        :style="{
          '--s-x': sX,
          '--s-y': sY,
          '--s-width': sWidth,
          '--s-height': sHeight,
        }"
      >
        <div
          class="flex h-15 w-full sm:w-auto px-2 justify-between items-center"
        >
          <NuxtLink class="flex items-center">
            <Icon
              name="sp:spark"
              class="text-4xl sm:mr-2 s-color-primary-color"
              mode="svg"
            />
          </NuxtLink>
          <NuxtLink class="flex items-center">
            <h1 class="font-(family-name:--s-title-font) text-surface-500">
              SPARK
            </h1>
          </NuxtLink>
          <div class="sm:hidden relative">
            <span
              class="pi pi-ellipsis-v text-surface-400 font-bold! text-xl! group-focus-within:opacity-0 group-focus-within:rotate-90 transition-all duration-300"
            ></span>
            <span
              class="absolute top-0 left-0 pi pi-times text-surface-400 font-bold! text-xl! opacity-0 -rotate-90 group-focus-within:opacity-100 group-focus-within:rotate-0 transition-all duration-300"
            ></span>
          </div>
        </div>
        <div
          class="flex grow pb-2.5 lg:pt-2.5 items-end sm:items-center flex-row-reverse sm:flex-row w-full sm:w-auto justify-between lg:justify-end opacity-0 group-focus-within:opacity-100 sm:opacity-100 origin-top scale-0 group-focus-within:scale-100 sm:scale-100 transition-transform duration-300"
        >
          <div
            class="flex flex-col gap-1 mr-1.5 lg:mr-1 lg:gap-2 lg:mr-2 sm:flex-row items-end sm:items-center"
          >
            <NuxtLink to="/" class="nav-link" active-class="active">
              首页
            </NuxtLink>
            <NuxtLink to="/download" class="nav-link" active-class="active">
              下载
            </NuxtLink>
            <NuxtLink to="/commercial" class="nav-link" active-class="active">
              商业版
            </NuxtLink>
            <a
              href="https://bbs.spark-app.store/"
              class="nav-link"
              active-class="active"
            >
              社区
            </a>
            <a
              href="https://wiki.spark-app.store/"
              class="nav-link"
              active-class="active"
            >
              帮助
            </a>
            <NuxtLink to="/about" class="nav-link" active-class="active">
              关于
            </NuxtLink>
            <a href="https://gxde.top/" class="nav-link" active-class="active">
              GXDE OS
            </a>
          </div>
          <Button
            :icon="`pi ${
              sDarkConfig === 'auto'
                ? 'pi-bullseye'
                : sDarkConfig === 'dark'
                  ? 'pi-moon'
                  : 'pi-sun'
            }`"
            aria-label="Toggle Dark Mode"
            size="small"
            class="shrink-0 m-1.5 sm:m-0"
            rounded
            severity="secondary"
            @click="toggleDarkMode"
          />
        </div>
      </nav>
    </header>
    <div>
      <NuxtPage />
    </div>
    <footer
      class="w-full flex justify-around bg-white snap-start overflow-hidden p-8 md:p-14 gap-4 dark:bg-surface-900 flex-col-reverse md:flex-row"
    >
      <div
        class="flex md:flex-col justify-end items-center md:items-start gap-2 md:gap-6"
      >
        <Icon
          name="sp:spark"
          class="md:grow w-8 h-8 md:w-auto md:h-auto max-w-40 max-h-40"
          mode="svg"
        />
        <h2
          class="font-(family-name:--s-title-font) text-base md:text-5xl text-surface-500 dark:text-surface-100"
        >
          SPARK
        </h2>
        <p class="grow text-end text-base md:text-2xl md:hidden">
          © 2020-{{ new Date().getFullYear() }} 星火社区
        </p>
      </div>
      <div class="flex flex-col gap-2 md:gap-4">
        <h2 class="text-lg md:text-2xl font-bold dark:text-surface-100">
          关于星火社区
        </h2>
        <p
          class="text-sm md:text-base leading-loose text-surface-600 md:max-w-[25.8em] dark:text-surface-200"
        >
          星火社区是以丰富 Linux
          应用生态为核心目标的开源协作平台，汇聚年轻开发者团队。核心项目星火应用商店通过整合分散资源、提供海量软件下载及安装功能，解决
          Linux 用户获取应用难题。社区以「星火燎原」为理念，通过论坛、Wiki
          和形象 IP 等传播方式构建开放协作生态。
        </p>
      </div>
      <div class="flex-col items-end gap-4 lg:gap-8 hidden md:flex">
        <div class="flex flex-col lg:flex-row grow items-end gap-2 lg:gap-6">
          <h2 class="whitespace-nowrap text-right text-2xl font-bold">
            加入星火应用商店<br />
            用户交流群
          </h2>
          <Icon
            name="s:qrcode-3"
            class="s-color-[black] dark:s-color-[white]"
            mode="svg"
            :style="{
              width: qrWidth + 'px',
              height: '100%',
            }"
          />
        </div>
        <p class="text-lg lg:text-2xl">
          © 2020-{{ new Date().getFullYear() }} 星火社区
        </p>
      </div>
      <button
        class="md:hidden self-center flex items-center gap-2 px-6 py-2 text-lg font-bold text-primary-600 bg-primary-200 rounded-full hover:bg-primary-300 dark:text-primary-400 dark:bg-[rgba(from_var(--p-primary-500)r_g_b/0.3)] dark:hover:bg-[rgba(from_var(--p-primary-500)r_g_b/0.5)] cursor-pointer"
        @click="joinGroup"
      >
        <Icon name="simple-icons:qq" class="text-2xl! font-bold!" />
        点击这里加入用户交流群
      </button>
      <Dialog
        v-model:visible="dialog"
        modal
        :show-header="false"
        class="w-7/8 rounded-3xl!"
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
        <p class="flex items-center justify-center gap-2 font-bold">
          <Icon name="simple-icons:qq" class="text-2xl!" />
          正在拉起手机 QQ
          <i class="pi pi-spin pi-spinner font-bold!"></i>
        </p>
        <p class="text-sm text-surface-600">
          如未响应，可通过以下方式加入交流群
        </p>
        <Icon
          name="s:qrcode-3"
          class="s-color-primary-600 bg-primary-100 dark:s-color-primary-400 dark:bg-[rgba(from_var(--p-primary-500)r_g_b/0.3)] w-48 h-48 p-3 rounded-2xl"
          mode="svg"
        />
        <ol
          class="text-sm flex flex-col gap-2 *:pl-6 *:relative *:before:content-[counter(qq)] *:before:counter-increment-[qq] *:before:absolute *:before:left-0 *:before:bg-primary-600 *:before:w-5 *:before:h-5 *:before:text-center *:before:leading-5 *:before:text-xs *:before:font-bold *:before:rounded-full *:before:text-white counter-reset-[qq]"
        >
          <li>
            长按保存或截图上方群二维码，启动
            QQ，点击右上角“+”-“扫一扫”-“相册”，选中此二维码以识别，申请加入群聊；
          </li>
          <li>
            搜索 QQ 群号<span
              class="font-bold py-0.5 px-2 mx-1 bg-primary-100 text-primary-600 rounded-full dark:text-primary-400 dark:bg-[rgba(from_var(--p-primary-500)r_g_b/0.3)]"
              >424677882</span
            >
            <i
              class="pi pi-clone text-xs! bg-surface-50 hover:bg-surface-100 px-2 py-0.5 rounded-full font-bold! dark:bg-surface-800 dark:hover:bg-surface-700 dark:text-surface-300 cursor-pointer"
              @click="copyGroupNumber"
            ></i>
          </li>
        </ol>
      </Dialog>
      <Toast />
    </footer>
  </ScrollPanel>
</template>

<style scoped lang="scss">
header {
  transform: translateY();
  transition: {
    property: transform, overflow;
    duration: 0.1s, 0s;
    delay: 0s, 0.3s;
  }

  &.commercial-header {
    transition: none;

    &::before {
      transition: none;
    }
  }

  &::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    border-radius: calc(var(--s-progress) * var(--radius-2xl));
    border-bottom-right-radius: calc(var(--s-progress) * var(--radius-2xl));
    border-bottom-left-radius: calc(var(--s-progress) * var(--radius-2xl));
    background-color: var(--p-surface-0);
    backdrop-filter: blur(calc(4 * var(--spacing)));
    z-index: -1;
    transition: {
      property: transform, width, border-radius, background-color, height;
      duration: 0.1s, 0.1s, 0.1s, 0.3s, 0.3s;
    }
  }

  &:focus-within::before,
  &:hover::before {
    background-color: rgba(from var(--p-surface-0) r g b / 0.8);
    border-bottom-right-radius: var(--radius-2xl);
    border-bottom-left-radius: var(--radius-2xl);
  }

  h1 {
    font-size: 1.75em;
  }

  nav::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: calc(var(--s-width) * 1px);
    height: calc(var(--s-height) * 1px);
    background-color: var(--p-primary-200);
    border-radius: calc(var(--spacing) * 4.75);
    z-index: -1;
    visibility: hidden;
  }

  .nav-link {
    padding: calc(var(--spacing) * 2) calc(var(--spacing) * 4);
    border-radius: calc(var(--spacing) * 4.75);
    font-weight: bold;
    color: var(--p-surface-600);
    white-space: nowrap;

    &.active {
      color: var(--p-primary-600);
      background-color: var(--p-primary-200);
    }
  }

  nav.mounted {
    &::before {
      visibility: visible;
      transition: {
        property: width, transform, opacity, scale, translate;
        duration: 0.3s;
      }
    }

    .nav-link {
      background-color: unset;

      &:hover {
        background-color: rgba(from var(--p-surface-400) r g b / 0.1);
      }
    }
  }
}

.s-dark header {
  &::before {
    background-color: var(--p-surface-900);
  }

  h1 {
    color: var(--p-surface-100);
  }

  nav::before {
    background-color: rgb(from var(--p-primary-500) r g b / 0.3);
  }

  .nav-link {
    color: var(--p-surface-300);

    &.active {
      color: var(--p-primary-400);
      background-color: rgb(from var(--p-primary-500) r g b / 0.3);
    }
  }

  nav.mounted {
    .nav-link {
      background-color: unset;

      &:hover {
        background-color: rgba(from var(--p-surface-400) r g b / 0.1);
      }
    }
  }
}
</style>
