<template>
    <div class="main">
        <h1><heart-filled style="color: red;"/>希希<heart-filled style="color: red"/>的考勤计算</h1>
        <div style="height: 20px"></div>
        <a-form @submit.prevent="handleSubmit">
            <a-form-item label="选择月份" :rules="[{ required: true, message: '请选择月份' }]" has-feedback :validate-status="monthsValidate">
                <a-select
                    style="width: 90%"
                    v-model="selectedMonth"
                    @change="selectedMonthChange"
                >
                    <a-select-option v-for="month in months" :key="month.value" :value="month.value">{{ month.label }}</a-select-option>
                </a-select>
            </a-form-item>
            <a-form-item label="选择指纹打卡文件" :rules="[{ required: true, message: '请选择指纹打卡文件' }]" has-feedback :validate-status="file1Validate">
                <a-upload-dragger :autoUpload="false" :beforeUpload="beforeUploadFile1" style="width: 90%">
                    <p class="ant-upload-drag-icon">
                        <inbox-outlined></inbox-outlined>
                    </p>
                    <p class="ant-upload-text">
                        点击或者将指纹打卡记录拖拽到此处上传
                    </p>
                    <p class="ant-upload-hint">
                        文件中必须包含一个[被计算年+被计算月]的sheet页哦,例如:202306
                    </p>
                </a-upload-dragger>
            </a-form-item>
            <a-form-item label="选择销客打卡文件" :rules="[{ required: true, message: '请选择销客打卡文件' }]" has-feedback :validate-status="file2Validate">
                <a-upload-dragger :autoUpload="false" :beforeUpload="beforeUploadFile2" style="width: 90%">
                    <p class="ant-upload-drag-icon">
                        <inbox-outlined></inbox-outlined>
                    </p>
                    <p class="ant-upload-text">
                        点击或者将销客打卡记录拖拽到此处上传
                    </p>
                    <p class="ant-upload-hint">
                        文件中必须包含一个[签到详单统计报表]的sheet页哦
                    </p>
                </a-upload-dragger>
            </a-form-item>
            <a-form-item>
                <a-button type="primary" html-type="submit">提交</a-button>
            </a-form-item>
        </a-form>
    </div>
</template>

<script>
import axios from 'axios';
import { InboxOutlined, HeartFilled } from '@ant-design/icons-vue';


export default {
    components: {
        InboxOutlined,
        HeartFilled
    },
  name: 'MainPage',
    computed: {
        file1Validate() {
            return this.file1 ? 'success' : 'error';
        },
        file2Validate() {
            return this.file2 ? 'success' : 'error';
        },
    },
  data() {
    return {
      selectedMonth: '',
      months: [
        { value: 1, label: '一月' },
        { value: 2, label: '二月' },
        { value: 3, label: '三月' },
        { value: 4, label: '四月' },
        { value: 5, label: '五月' },
        { value: 6, label: '六月' },
        { value: 7, label: '七月' },
        { value: 8, label: '八月' },
        { value: 9, label: '九月' },
        { value: 10, label: '十月' },
        { value: 11, label: '十一月' },
        { value: 12, label: '十二月' },
      ],
      monthsValidate: "error", // 月份验证状态
      file1: null,
      file2: null,
    };
  },
  methods: {
      beforeUploadFile1(file) {
          // 处理文件1的操作
          this.file1 = file;
          return false; // 取消自动上传
      },
      beforeUploadFile2(file) {
          // 处理文件2的操作
          this.file2 = file;
          return false; // 取消自动上传
      },
    handleSubmit() {
          if (!this.selectedMonth) {
              this.monthsValidate = "error";
              this.$message.warn("请选择月份");
              return;
          }
          if (!this.file1) {
              this.file1Validate = "error";
              this.$message.warn("请选择指纹打卡文件");
              return;
          }
          if (!this.file2) {
              this.file2Validate = "error";
              this.$message.warn("请选择销客打卡文件");
              return;
          }
      const formData = new FormData();
      formData.append('month', this.selectedMonth);
      formData.append('file1', this.file1);
      formData.append('file2', this.file2);
      axios.post('https://xixi.home.qyzhg.com:10443/getClockExcel', formData)
          .then(response => {
              if (response.data.code === 0) {
                  this.downloadFile(response.data.filename)
                  this.$message.success('打卡数据处理成功');
              } else {
                  this.$message.warn(response.data.errMsg? "打卡数据处理失败::" + response.data.errMsg:'打卡数据处理失败')
              }
          })
          .catch(error => {
              this.$message.error("系统内部错误:" + error);
          });
    },
  downloadFile(fileName) {
      const fileUrl = 'https://xixi.home.qyzhg.com:10443/download/' + fileName; // 替换成你要下载的文件链接
      // 创建XHR对象
      const xhr = new XMLHttpRequest();
      xhr.open('GET', fileUrl, true);
      xhr.responseType = 'blob';
      // 请求完成后的回调函数
      xhr.onload = function () {
          if (xhr.status === 200) {
              // 创建一个<a>元素
              const link = document.createElement('a');
              link.href = window.URL.createObjectURL(xhr.response);
              link.download = fileName; // 可以根据需要设置文件名
              // 将<a>元素添加到DOM树中
              document.body.appendChild(link);
              // 模拟点击<a>元素，触发文件下载
              link.click();
              // 删除<a>元素
              document.body.removeChild(link);
          }
      };
      // 发送请求
      xhr.send();
  },
      selectedMonthChange(value) {
          this.selectedMonth = value;
          this.monthsValidate = "success";
      }
  },
};
</script>

<style scoped>
</style>
