<template>
  <div class="form-container">
    <input type="text" v-model="strValue" placeholder="请输入内容" class="input">
    <input type="text" v-model="inputValue" placeholder="请输入内容，只能输入数字" @input="validateFloat" class="input">
    <select v-model="selectedOption" class="select">
      <option disabled value="">请选择一个选项</option>
      <option v-for="option in options" :key="option.value" :value="option.value">
        {{ option.text }}
      </option>
    </select>
    <div class="file-upload-wrapper">
      <button @click="triggerFileInput" class="file-upload-button">选择文件</button>
      <span class="file-name">{{ fileName }}</span>
      <input type="file" @change="loadJsonFile" accept=".json" ref="fileInput" class="file-input" />
    </div>

    <button @click="submit" class="button">导出为JSON</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const inputValue = ref('');
const strValue = ref('');
const selectedOption = ref('');
const options = ref([
  { value: '选项1', text: '选项1' },
  { value: '选项2', text: '选项2' },
  { value: '选项3', text: '选项3' },
]);

const fileInput = ref(null);
const fileName = ref("未选择文件");

const triggerFileInput = () => {
  fileInput.value.click(); // 触发input的点击事件
};

// 输入有效性验证
const validateFloat = (event) => {
  // 使用正则表达式验证输入是否为浮点数，允许负数和小数点
  if (!event.target.value.match(/^[-]?[0-9]*\.?[0-9]*$/)) {
    alert('只能输入数字')
    event.target.value = inputValue.value;
  } else {
    // 更新为最新的有效值
    inputValue.value = event.target.value;
  }
};


// 从JSON文件加载数据
const loadJsonFile = (event) => {
  const file = event.target.files[0];
  fileName.value = file ? file.name : "未选择文件";
  
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const content = e.target.result;
        const jsonData = JSON.parse(content);
        alert("open success")
        // 用读取的数据来更新前端的数据
        inputValue.value = jsonData["key1"]
        selectedOption.value = jsonData["key2"]
        strValue.value = jsonData["key3"]
      } catch (error) {
        console.error("Error parsing JSON:", error);
        alert("文件内容不是有效的JSON格式！");
      }
    };
    reader.onerror = (error) => {
      console.error("File reading error:", error);
      alert("读取文件时发生错误，请重试。");
    };
    console.log("here")
    reader.readAsText(file); // 以文本形式读取文件
  }
};

// 提交选择并导出json文件
const submit = () => {
  const data = {
    key1: inputValue.value,
    key2: selectedOption.value,
    key3: strValue.value
  };
  const jsonString = JSON.stringify(data);
  const blob = new Blob([jsonString], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  
  // 创建一个链接元素用于下载
  const link = document.createElement('a');
  link.href = url;
  link.download = 'data.json'; // 文件名
  document.body.appendChild(link); // 必须将链接元素添加到文档中才能正常工作
  link.click(); // 模拟点击实现下载
  
  document.body.removeChild(link); // 下载后移除元素
  URL.revokeObjectURL(url); // 释放掉blob对象的URL
};
</script>

<style scoped>
.form-container {
  max-width: 400px;
  margin: 40px auto;
  padding: 20px 100px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.input, .select, .button {
  width: 100%;
  padding: 12px 15px; /* 略微增加内边距以提供更多的空间和舒适度 */
  margin: 12px 0; /* 增加外边距以改进元素之间的空间 */
  border-radius: 8px; /* 增加边框圆角，使元素更加现代和圆润 */
  border: 1px solid #d1d5db; /* 使用更柔和的边框颜色 */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* 添加轻微的阴影，增加层次感 */
  transition: all 0.3s ease; /* 添加过渡效果，使状态变化更平滑 */
}

.input:focus, .select:focus {
  border-color: #93c5fd; /* 聚焦时改变边框颜色，提高用户交互体验 */
  box-shadow: 0 0 0 3px rgba(147,197,253,0.5); /* 聚焦时增加更明显的阴影，突出元素 */
}

.input {
  width: 90%;
}

.button {
  color: #ffffff; /* 设置按钮文字颜色为白色 */
  background-color: #3b82f6; /* 使用更鲜艳的背景色 */
  cursor: pointer; /* 确保鼠标悬停时显示为指针 */
}

.button:hover {
  background-color: #2563eb; /* 鼠标悬停时改变按钮背景色，增加交互性 */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* 鼠标悬停时增加阴影，增加立体感 */
}

.file-upload-wrapper {
  display: flex;
  justify-content: space-between; /* 使按钮和文件名分布在两端 */
  align-items: center;
  width: 100%; /* 容器宽度为100% */
}

.file-upload-button {
  padding: 10px 20px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.file-upload-button:hover {
  background-color: #0056b3;
}

.file-input {
  position: absolute;
  width: 1px;
  height: 1px;
  opacity: 0;
  overflow: hidden;
  clip: rect(1px, 1px, 1px, 1px);
}

.file-name {
  flex-grow: 1; /* 文件名占据剩余空间 */
  padding: 0 10px; /* 保持文字不贴边 */
  text-align: left;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis; /* 长文件名用省略号表示 */
}

</style>
