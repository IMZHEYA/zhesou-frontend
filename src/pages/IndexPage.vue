<template>
  <div class="index-page">
  <a-input-search
      v-model:value="searchParams.text"
      placeholder="input search text"
      enter-button
      @search="onSearch"
    />
        <MyDivider/>
      <a-tabs v-model:activeKey="activeKey" @change="onTabChange">
    <a-tab-pane key="post" tab="文章" > <PostList :post-list="postList"/></a-tab-pane>
    <a-tab-pane key="picture" tab="图片" force-render><PictureList :picture-list="pictureList"/></a-tab-pane>
    
    <a-tab-pane key="user" tab="用户">
      <UserList :user-list="userList"/></a-tab-pane>
    
  </a-tabs>
  </div>
  
</template>

<script setup lang="ts">
import {ref, watchEffect} from 'vue';
import PostList from '@/components/PostList.vue';
import PictureList from '@/components/PictureList.vue';
import UserList from '@/components/UserList.vue';
import MyDivider from '@/components/MyDivider.vue';
import { useLink, useRoute, useRouter } from 'vue-router';
import myAxios from '@/plugins/myAxios';
//创建路由实例
const router = useRouter();
//获取路由所有信息
const route = useRoute();
const activeKey = route.params.category;
const initSearchParams = {
  text:"",
  pageSize: 10,
  pageNum: 1,
};
//把initSearchParams变为响应式对象
const searchParams = ref(initSearchParams);
const postList = ref([]);
const userList = ref([]);
const pictureList = ref([]);
//旧
const loadDataOld = (params: any) => {
  const pictureQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/picture/search/page/vo",pictureQuery).then((res: any) => {
  pictureList.value = res.records;
});
  const postQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/post/list/page/vo", postQuery).then((res: any) => {
  postList.value = res.records;
});
const userQuery = {
  ...params,
  userName: params.text,
}
myAxios.post("/user/list/page/vo", userQuery).then((res: any) => {
  userList.value = res.records;
});
};
//新
const loadData = (params: any) => {
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/search/all",query).then((res: any) => {
  userList.value = res.userList;
  pictureList.value = res.pictureList;
  postList.value = res.postList;
});
};
//监视路由变化
watchEffect(() =>{
  searchParams.value = {
  ...initSearchParams,
  text: route.query.text,
  } as any;
});
//用户每次搜索
loadData(initSearchParams);
const onSearch = (value: string) =>{
  //把搜索框里面的内容压到路由中
  router.push({
    query: searchParams.value,
  });
  loadData(searchParams.value);
};
//每次导航栏切换
const onTabChange = (key: string) => {
  router.push({
    path: `/${key}`,
    query: searchParams.value,
  });
};
</script>
