<script setup>
import { ref, watch } from "vue";
import * as ElementPlusIconsVue from "@element-plus/icons-vue";
import { useRouter, useRoute } from "vue-router";
import { useNavsStore } from "@/stores/navs";
const router = useRouter();
const route = useRoute();
const navsStore = useNavsStore();
const activeIndex = ref("1");
const list = ref([]);

const getIcon = (iconName) => {
  if (!ElementPlusIconsVue[iconName]) {
    console.error(`ElementPlusIconsVue 中不存在名为 ${iconName} 的图标`);
    return null; // 或者返回一个默认的图标组件
  }
  return ElementPlusIconsVue[iconName];
};
const select = (val) => {
  activeIndex.value = val;
  const url = findUrlByIndex(val);
  if (url) {
    router.push(url);
  }
};
//找到index对应的url
const findUrlByIndex = (index) => {
  let url = "";
  const findUrl = (list) => {
    for (let i = 0; i < list.length; i++) {
      const item = list[i];
      if (item.id.toString() === index) {
        url = item.url;
        break;
      } else if (item.children) {
        findUrl(item.children);
      }
    }
  };
  findUrl(list.value);
  return url;
};
// 找到当前url对应的index
const findIndexByUrl = (url) => {
  let index = "";
  const findIndex = (list) => {
    for (let i = 0; i < list.length; i++) {
      const item = list[i];
      if (item.url === url) {
        index = item.id.toString();
        break;
      } else if (item.children) {
        findIndex(item.children);
      }
    }
  };
  findIndex(list.value);
  return index;
};

router.afterEach((to, from) => {
  if (to.fullPath) {
    activeIndex.value = findIndexByUrl(to.fullPath);
  }
});

watch(
  () => navsStore.navsTree,
  (newStore, oldStore) => {
    if (!newStore) return;
    list.value = newStore;
    if (route.fullPath) {
      activeIndex.value = findIndexByUrl(route.fullPath);
    }
  },
  { deep: true, immediate: true },
);
</script>
<template>
  <div class="navBox">
    <el-menu class="menu" :default-active="activeIndex" @select="select">
      <div class="item" v-for="(item, index) in list" :key="index">
        <el-menu-item :index="item.id.toString()" v-if="!item.children">
          <el-icon v-if="item.icon">
            <component :is="getIcon(item.icon)" />
          </el-icon>
          <span>{{ item.name }}</span>
        </el-menu-item>
        <el-sub-menu :index="item.id.toString()" v-if="item.children">
          <template #title>
            <el-icon v-if="item.icon">
              <component :is="getIcon(item.icon)" />
            </el-icon>
            <span>{{ item.name }}</span>
          </template>
          <div
            class="items"
            v-for="(items, indexs) in item.children"
            :key="indexs + 's'"
          >
            <el-menu-item :index="items.id.toString()" v-if="!items.children"
              >{{ items.name }}
            </el-menu-item>
            <el-sub-menu :index="items.id.toString()" v-if="items.children">
              <template #title>
                <span>{{ items.name }}</span>
              </template>
              <div
                class="itemss"
                v-for="(itemss, indexss) in items.children"
                :key="indexss + 'ss'"
              >
                <el-menu-item :index="itemss.id.toString()"
                  >{{ itemss.name }}
                </el-menu-item>
              </div>
            </el-sub-menu>
          </div>
        </el-sub-menu>
      </div>
    </el-menu>
  </div>
</template>
<style scoped>
.navBox {
  width: 100%;
  height: 100%;
  background-color: var(--c-bg);
  overflow: hidden;
}
.navBox .menu {
  height: 100%;
  padding: 5px 0;
  overflow-y: auto;
  border-right: 0;
}

.navBox :deep(.el-menu-item),
.navBox :deep(.el-sub-menu__title),
.navBox :deep(.el-menu-item-group__title) {
  font-size: 14px;
  height: 34px;
  line-height: 34px;
}
.navBox :deep(.el-menu-item:hover),
.navBox :deep(.el-sub-menu__title:hover),
.navBox .items :deep(.el-menu-item:hover),
.navBox .itemss :deep(.el-menu-item:hover),
.navBox .items :deep(.el-sub-menu__title:hover),
.navBox .itemss :deep(.el-sub-menu__title:hover) {
  background-color: transparent;
}
</style>
