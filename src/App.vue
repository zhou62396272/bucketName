<template>
  <div class="width">
    <a-form :model="bucketInfo">
      <a-form-item
        label="Bucket名称"
        name="bucketName"
        :rules="rule.bucketNameRules()"
      >
        <a-input style="width: 500px;" v-model:value="bucketInfo.bucketName" :maxlength="63" placeholder="Bucket名称" />
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup>
// 一个一个 慢慢输入 => 正常提示
// 快速输入后停止时，如何能正常触发校验提示？？？

import "ant-design-vue/dist/antd.css";
import { reactive } from "vue";
import axios from "axios";

const bucketInfo = reactive({
  bucketName: ''
})

const abortPromise = (promise1) => {
  let abort;
  const promise2 = new Promise((resolve, reject) => {
    abort = reject;
  });
  const p = Promise.race([promise1, promise2]);
  p.abort = abort;
  return p;
};

const debouncePromise = (success, fail, time) => {
  let promise;
  return function (...rest) {
    if (promise && typeof promise.abort === "function") {
      promise.abort();
    }
    const timeoutPromise = new Promise((resolve) => {
      setTimeout(() => {
        resolve(undefined); //个人觉得就是每次输入都会立刻resolve，所以页面会来不及更新
      }, time);
    });
    promise = abortPromise(timeoutPromise);
    return promise.then(
      () => {
        return success(...rest);
      },
      () => {
        return fail(...rest);
      }
    );
  };
};

const fail = () => {
  console.log("由于防抖中断了第一次的请求");
  return Promise.resolve(); // 忽视它暂时是正确的
};

const success = (rule, value) => {
  console.log("请求后台接口");
  const reg = /^(?!(-|[-a-z0-9]*--|[-a-z0-9]*-$))[-a-z0-9]*[-a-z0-9]*$/;
  if (value.length < 1) {
    return Promise.reject("名称不能为空");
  }
  if (value.length < 3 || value.length > 63) {
    return Promise.reject("请输入 3~63 个字符");
  } else if (!reg.test(value)) {
    return Promise.reject("只允许小写字母、数字、短横线（-），且不能以短横线开头或结尾");
  }
  // 调用接口查询是否重复
  axios
    .get(
      "https://www.fastmock.site/mock/a535d266440f87553f2748030e9505cb/starship-ss3/api/check",
      value
    )
    .then((res) => {
      console.log(res.data);
    });
};

const check = debouncePromise(success, fail, 200);

const rule = {
  bucketNameRules() {
    return { validator: check, trigger: "change", required: true };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.width {
  width: 100%;
  display: flex;
  justify-content: center;
}
.w500 {
  width: 500px;
}
</style>
