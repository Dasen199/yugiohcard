<template>
  <YugiohCard />
</template>

<script setup>
  import YugiohCard from './components/YugiohCard.vue';
</script>

<style lang="scss">
  * {
    box-sizing: border-box;
    overflow-wrap: break-word;
    -webkit-tap-highlight-color: transparent;
    margin: 0;
    padding: 0;
  }

  :root {
    --primary-color: #409EFF;
    --success-color: #67C23A;
    --warning-color: #E6A23C;
    --danger-color: #F56C6C;
    --info-color: #909399;
    --main-color: #303133;
    --text-color: #606266;
    --border-color: #DCDFE6;
    --background-color: #F5F7FA;
    --default-font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  }

  html, body {
    height: 100%;
    width: 100%;
    overflow-x: hidden;
    overflow-y: auto;
    position: relative;
  }

  body {
    font-family: var(--default-font-family);
    color: var(--main-color);
    font-size: 14px;
    background-color: var(--background-color);
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    position: relative;
  }

  #app {
    min-height: 100%;
    width: 100%;
    display: flex;
    flex-direction: column;
  }

  .el-form-item__label {
    font-weight: 500;
  }
  
  /* 移动设备优化 */
  @media (max-width: 768px) {
    .el-form-item {
      margin-bottom: 12px;
    }
    
    .el-form-item__label {
      padding-bottom: 4px;
      line-height: 1.2;
      font-size: 13px;
    }
    
    .el-input__inner {
      font-size: 14px;
    }
    
    .el-button {
      font-size: 14px;
    }
    
    .el-select {
      width: 100%;
    }
    
    .el-input-number {
      width: 100% !important;
    }
    
    .el-checkbox__label {
      font-size: 13px;
    }
    
    .el-radio__label {
      font-size: 13px;
    }
  }
</style>
