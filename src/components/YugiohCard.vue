<template>
  <div class="yugioh-card-container">
    <!-- 顶部标题栏 -->
      <div class="form-header">
        <div class="form-title">
        <span>游戏王卡片生成器</span>
          <Icon
            class="github-icon"
            icon="ri:github-fill"
            width="24"
            height="24"
            @click="toGithub"
          />
        </div>
        <div class="form-description">
        <span>使用Canvas渲染经典游戏王卡片的工具</span>
      </div>
        </div>
    
    <!-- 卡片预览区域 -->
    <div class="yugioh-card">
      <div class="card-container" ref="cardContainer">
        <div 
          ref="card" 
          class="card" 
          :style="{transform: `scale(${currentScale}) translate(${position.x}px, ${position.y}px)`}"
          @mousedown="startDrag"
          @touchstart="startDrag"
        />
      </div>

      <div class="card-controls">
        <el-tooltip content="重置位置和缩放" placement="top">
          <el-button circle @click="resetPosition" class="control-btn">
            <el-icon><Refresh /></el-icon>
          </el-button>
        </el-tooltip>
        
        <el-tooltip content="缩小" placement="top">
          <el-button circle @click="zoomOut" class="control-btn">
            <el-icon><ZoomOut /></el-icon>
          </el-button>
        </el-tooltip>
        
        <el-tooltip content="放大" placement="top">
          <el-button circle @click="zoomIn" class="control-btn">
            <el-icon><ZoomIn /></el-icon>
          </el-button>
        </el-tooltip>
        
        <el-tooltip content="导出图片" placement="top">
          <el-button type="primary" circle @click="exportImage" class="control-btn">
            <el-icon><Download /></el-icon>
          </el-button>
        </el-tooltip>
      </div>
    </div>
    
    <div class="divider"></div>
    
    <!-- 表单编辑区域 -->
    <div class="form-container">
      <div class="form-main">
        <el-form :model="form.data" label-position="top" size="small">
          <!-- 使用标签页替换折叠面板 -->
          <el-tabs v-model="activeTab" class="card-tabs">
            <!-- AI生成 -->
            <el-tab-pane label="AI生成" name="ai">
              <div class="ai-generator-wrapper">
                <AICardGenerator @apply-to-card="applyGeneratedCard" />
              </div>
            </el-tab-pane>
            
            <!-- 基本信息 -->
            <el-tab-pane label="基本信息" name="1">
              <div class="form-row">
                <el-form-item label="卡名" class="form-item-wide">
                  <el-input v-model="form.data.name" placeholder="例如：青眼白龙" />
                </el-form-item>
              </div>
              
              <div class="form-row">
                <el-form-item label="语言" class="form-item">
                  <el-select v-model="form.data.language" placeholder="选择语言">
                    <el-option label="简体中文" value="sc" />
                    <el-option label="繁体中文" value="tc" />
                    <el-option label="日文" value="jp" />
                    <el-option label="韩文" value="kr" />
                    <el-option label="英文" value="en" />
                    <el-option label="星光界文" value="astral" />
                  </el-select>
                </el-form-item>
                
                <el-form-item label="卡片类型" class="form-item">
                  <el-select v-model="form.data.type" placeholder="选择卡片类型">
                    <el-option label="怪兽卡" value="monster" />
                    <el-option label="魔法卡" value="spell" />
                    <el-option label="陷阱卡" value="trap" />
                    <el-option label="灵摆卡" value="pendulum" />
                  </el-select>
                </el-form-item>
              </div>

              <!-- 魔法/陷阱属性 -->
              <div v-if="form.data.type === 'spell' || form.data.type === 'trap'">
                <el-form-item label="图标">
                  <el-select v-model="form.data.icon" placeholder="选择图标">
                    <el-option label="无" value="" />
                    <el-option label="装备" value="equip" v-if="form.data.type === 'spell'" />
                    <el-option label="场地" value="field" v-if="form.data.type === 'spell'" />
                    <el-option label="速攻" value="quick-play" v-if="form.data.type === 'spell'" />
                    <el-option label="仪式" value="ritual" v-if="form.data.type === 'spell'" />
                    <el-option label="永续" value="continuous" />
                    <el-option label="反击" value="counter" v-if="form.data.type === 'trap'" />
                  </el-select>
                </el-form-item>
              </div>

              <!-- 灵摆属性 -->
              <div v-if="form.data.type === 'pendulum'">
                <el-form-item label="灵摆刻度">
                  <el-input-number v-model="form.data.pendulumScale" :min="0" :max="13" />
                </el-form-item>
                
                <el-form-item label="灵摆效果">
                  <el-input
                    type="textarea"
                    v-model="form.data.pendulumDescription"
                    :rows="4"
                    placeholder="输入灵摆效果描述..."
                  />
                </el-form-item>
              </div>
            </el-tab-pane>
            
            <!-- 怪兽属性 -->
            <el-tab-pane label="怪兽属性" name="2" v-if="form.data.type === 'monster' || form.data.type === 'pendulum'">
              <div class="form-row">
                <el-form-item label="怪兽种类" class="form-item">
                  <el-select v-model="form.data.cardType" placeholder="选择怪兽种类">
                    <el-option label="通常怪兽" value="normal" />
                    <el-option label="效果怪兽" value="effect" />
                    <el-option label="仪式怪兽" value="ritual" />
                    <el-option label="融合怪兽" value="fusion" />
                    <el-option label="同调怪兽" value="synchro" />
                    <el-option label="超量怪兽" value="xyz" />
                    <el-option label="连接怪兽" value="link" />
                    <el-option label="衍生物" value="token" />
                  </el-select>
                </el-form-item>
                
                <el-form-item label="属性" class="form-item">
                  <el-select v-model="form.data.attribute" placeholder="选择属性">
                    <el-option label="暗" value="dark" />
                    <el-option label="光" value="light" />
                    <el-option label="地" value="earth" />
                    <el-option label="水" value="water" />
                    <el-option label="炎" value="fire" />
                    <el-option label="风" value="wind" />
                    <el-option label="神" value="divine" />
                  </el-select>
                </el-form-item>
              </div>
              
              <div class="form-row">
                <el-form-item label="星级/阶级" class="form-item" v-if="form.data.cardType !== 'link'">
                  <el-input-number v-model="form.data.level" :min="0" :max="12" v-if="form.data.cardType !== 'xyz'" />
                  <el-input-number v-model="form.data.rank" :min="0" :max="13" v-else />
                </el-form-item>
                
                <el-form-item label="怪兽种族/类型" class="form-item">
                  <el-input v-model="form.data.monsterType" placeholder="例如：龙族/通常" />
                </el-form-item>
              </div>
              
              <div class="form-row">
                <el-form-item label="攻击力/防御力" class="form-item-wide">
                  <div class="atk-def-container">
                    <el-input-number v-model="form.data.atk" :min="-2" :max="9999" placeholder="ATK" />
                    <span class="separator">/</span>
                    <el-input-number v-model="form.data.def" :min="-2" :max="9999" placeholder="DEF" v-if="form.data.cardType !== 'link'" />
                  </div>
                  <div class="atk-def-note">注：-1代表?，-2代表∞</div>
                </el-form-item>
              </div>
              
              <el-form-item label="连接箭头" v-if="form.data.cardType === 'link'">
                <div class="link-arrows">
                  <div class="arrow-grid">
                    <el-checkbox v-model="form.data.arrowList" :label="7"></el-checkbox>
                    <el-checkbox v-model="form.data.arrowList" :label="0"></el-checkbox>
                    <el-checkbox v-model="form.data.arrowList" :label="1"></el-checkbox>
                    <el-checkbox v-model="form.data.arrowList" :label="6"></el-checkbox>
                    <div class="center-box">连接</div>
                    <el-checkbox v-model="form.data.arrowList" :label="2"></el-checkbox>
                    <el-checkbox v-model="form.data.arrowList" :label="5"></el-checkbox>
                    <el-checkbox v-model="form.data.arrowList" :label="4"></el-checkbox>
                    <el-checkbox v-model="form.data.arrowList" :label="3"></el-checkbox>
                  </div>
                </div>
              </el-form-item>
            </el-tab-pane>
            
            <!-- 卡片效果与图片 -->
            <el-tab-pane label="卡片效果与图片" name="3">
              <el-form-item label="效果描述">
                <el-input
                  type="textarea"
                  v-model="form.data.description"
                  :rows="6"
                  placeholder="输入卡片效果或描述..."
                />
              </el-form-item>
              
              <div class="form-row">
                <el-form-item label="文本缩放" class="form-item">
                  <el-slider v-model="form.data.descriptionZoom" :min="0.5" :max="1.5" :step="0.1" />
                </el-form-item>
                
                <div class="form-item checkbox-group">
                  <el-checkbox v-model="form.data.descriptionAlign">文本居中</el-checkbox>
                  <el-checkbox v-model="form.data.firstLineCompress">首行压缩</el-checkbox>
                </div>
              </div>

              <!-- 卡片图片 -->
              <el-form-item label="卡片图片">
                <el-upload
                  class="card-image-uploader"
                  action="#"
                  :auto-upload="false"
                  :show-file-list="false"
                  :on-change="handleImageChange"
                >
                  <img v-if="imageUrl" :src="imageUrl" class="card-image" />
                  <el-icon v-else class="uploader-icon"><Plus /></el-icon>
                </el-upload>
                <div class="upload-tips">点击上方区域上传卡片图片</div>
              </el-form-item>
            </el-tab-pane>
            
            <!-- 附加信息 -->
            <el-tab-pane label="附加信息" name="4">
              <div class="form-row">
                <el-form-item label="卡包编号" class="form-item">
                  <el-input v-model="form.data.package" placeholder="例如：SD25-SC001" />
                </el-form-item>
                
                <el-form-item label="卡片密码" class="form-item">
                  <el-input v-model="form.data.password" placeholder="例如：89631139" />
                </el-form-item>
              </div>
              
              <div class="form-row">
                <el-form-item label="罕贵度" class="form-item">
                  <el-select v-model="form.data.rare" placeholder="选择罕贵度">
                    <el-option label="无" value="" />
                    <el-option label="DT" value="dt" />
                    <el-option label="UR" value="ur" />
                    <el-option label="GR" value="gr" />
                    <el-option label="HR" value="hr" />
                    <el-option label="SER" value="ser" />
                    <el-option label="GSER" value="gser" />
                    <el-option label="PSER" value="pser" />
                  </el-select>
                </el-form-item>
                
                <el-form-item label="角标" class="form-item">
                  <el-select v-model="form.data.laser" placeholder="选择角标">
                    <el-option label="无" value="" />
                    <el-option label="样式一" value="laser1" />
                    <el-option label="样式二" value="laser2" />
                    <el-option label="样式三" value="laser3" />
                    <el-option label="样式四" value="laser4" />
                  </el-select>
                </el-form-item>
              </div>
              
              <div class="form-row">
                <el-form-item label="版权" class="form-item">
                  <el-select v-model="form.data.copyright" placeholder="选择版权">
                    <el-option label="无" value="" />
                    <el-option label="简体中文" value="sc" />
                    <el-option label="日文" value="jp" />
                    <el-option label="英文" value="en" />
                  </el-select>
                </el-form-item>
                
                <div class="form-item checkbox-group">
                  <el-checkbox v-model="form.data.twentieth">20周年样式</el-checkbox>
                  <el-checkbox v-model="form.data.radius">圆角</el-checkbox>
                </div>
              </div>

              <!-- 卡片缩放设置 -->
              <el-form-item label="卡片缩放">
                <el-slider v-model="form.data.scale" :min="0.5" :max="2" :step="0.1" />
              </el-form-item>
            </el-tab-pane>
          </el-tabs>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { Icon } from '@iconify/vue';
  import { Plus, Download, ZoomIn, ZoomOut, Refresh } from '@element-plus/icons-vue';
  import { onBeforeUnmount, onMounted, reactive, ref, shallowRef, watch } from 'vue';
  import { YugiohCard } from 'yugioh-card';
  import yugiohDemo from '@/assets/demo/yugioh-demo';
  import AICardGenerator from './AICardGenerator.vue';

  const card = ref(null);
  const cardContainer = ref(null);
  const cardLeaf = shallowRef(null);
  const activeTab = ref('ai'); // 默认选中AI生成标签页
  const imageUrl = ref('');
  
  // 卡片拖拽和缩放相关
  const currentScale = ref(0.9); // 初始缩放略小以确保完整显示
  const position = reactive({ x: 0, y: 0 });
  const dragStart = reactive({ x: 0, y: 0 });
  const isDragging = ref(false);
  
  const form = reactive({
    data: JSON.parse(JSON.stringify(yugiohDemo)), // 深拷贝避免直接修改原始数据
  });

  onMounted(() => {
    initCard();
    if (form.data.image) {
      imageUrl.value = form.data.image;
    }
    
    // 添加鼠标移动和抬起事件监听
    window.addEventListener('mousemove', onMouseMove);
    window.addEventListener('mouseup', stopDrag);
    window.addEventListener('touchmove', onTouchMove);
    window.addEventListener('touchend', stopDrag);
    
    // 添加滚轮缩放事件
    cardContainer.value.addEventListener('wheel', handleWheel);
    
    // 自动调整卡片到最佳大小
    setTimeout(adjustCardToFit, 500);
  });
  
  // 调整卡片到最佳大小，确保完整显示
  function adjustCardToFit() {
    if (!cardContainer.value || !card.value) return;
    
    const containerWidth = cardContainer.value.clientWidth;
    const containerHeight = cardContainer.value.clientHeight;
    const cardWidth = card.value.clientWidth;
    const cardHeight = card.value.clientHeight;
    
    if (cardWidth === 0 || cardHeight === 0) return;
    
    // 计算宽度和高度缩放比例，取最小值确保完整显示
    const widthRatio = (containerWidth * 0.8) / cardWidth;
    const heightRatio = (containerHeight * 0.9) / cardHeight;
    currentScale.value = Math.min(widthRatio, heightRatio, 1); // 不超过原始大小
  }

  onBeforeUnmount(() => {
    cardLeaf.value?.leafer.destroy();
    
    // 移除事件监听
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mouseup', stopDrag);
    window.removeEventListener('touchmove', onTouchMove);
    window.removeEventListener('touchend', stopDrag);
    
    cardContainer.value?.removeEventListener('wheel', handleWheel);
  });

  function initCard() {
    cardLeaf.value?.leafer.destroy();
    cardLeaf.value = new YugiohCard({
      view: card.value,
      data: form.data,
      resourcePath: process.env.NODE_ENV === 'production' ? './assets/yugioh-card' : 'src/assets/yugioh-card',
    });
  }

  function exportImage() {
    cardLeaf.value.leafer.export('卡片.png');
  }

  function resetCard() {
    form.data = JSON.parse(JSON.stringify(yugiohDemo));
    if (form.data.image) {
      imageUrl.value = form.data.image;
    }
    cardLeaf.value.setData(form.data);
    resetPosition();
  }
  
  function handleImageChange(file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      imageUrl.value = e.target.result;
      form.data.image = e.target.result;
      cardLeaf.value.setData(form.data);
    };
    reader.readAsDataURL(file.raw);
  }
  
  // 拖拽相关函数
  function startDrag(event) {
    isDragging.value = true;
    
    if (event.type === 'mousedown') {
      dragStart.x = event.clientX - position.x;
      dragStart.y = event.clientY - position.y;
    } else if (event.type === 'touchstart') {
      const touch = event.touches[0];
      dragStart.x = touch.clientX - position.x;
      dragStart.y = touch.clientY - position.y;
    }
    
    event.preventDefault();
  }
  
  function onMouseMove(event) {
    if (!isDragging.value) return;
    
    position.x = event.clientX - dragStart.x;
    position.y = event.clientY - dragStart.y;
  }
  
  function onTouchMove(event) {
    if (!isDragging.value) return;
    
    const touch = event.touches[0];
    position.x = touch.clientX - dragStart.x;
    position.y = touch.clientY - dragStart.y;
    
    event.preventDefault();
  }
  
  function stopDrag() {
    isDragging.value = false;
  }
  
  function resetPosition() {
    position.x = 0;
    position.y = 0;
    adjustCardToFit();
  }
  
  function zoomIn() {
    // 使用相对缩放比例而不是固定值，更精确
    const zoomFactor = 1.1; // 放大10%
    const newScale = Math.min(currentScale.value * zoomFactor, 3);
    
    // 计算缩放前后的位置差，保持缩放的中心点
    const scaleDiff = newScale - currentScale.value;
    if (scaleDiff > 0) {
      // 调整位置以保持中心点
      position.x = position.x - (position.x * scaleDiff / currentScale.value);
      position.y = position.y - (position.y * scaleDiff / currentScale.value);
    }
    
    currentScale.value = newScale;
  }
  
  function zoomOut() {
    // 使用相对缩放比例而不是固定值，更精确
    const zoomFactor = 0.9; // 缩小10%
    const newScale = Math.max(currentScale.value * zoomFactor, 0.3);
    
    // 检查卡片是否完整显示在容器中
    if (card.value && cardContainer.value) {
      const containerWidth = cardContainer.value.clientWidth;
      const containerHeight = cardContainer.value.clientHeight;
      const cardWidth = card.value.clientWidth / currentScale.value; // 获取原始卡片宽度
      const cardHeight = card.value.clientHeight / currentScale.value; // 获取原始卡片高度
      
      // 如果卡片当前超出容器或者接近容器边缘
      const isCardTooLarge = 
        cardWidth * newScale > containerWidth * 0.95 || 
        cardHeight * newScale > containerHeight * 0.95;
        
      // 如果卡片太大，自动计算合适的缩放比例
      if (isCardTooLarge) {
        const widthRatio = (containerWidth * 0.85) / cardWidth;
        const heightRatio = (containerHeight * 0.85) / cardHeight;
        const fitScale = Math.min(widthRatio, heightRatio);
        
        // 如果计算出的合适比例小于当前新比例，使用合适比例
        if (fitScale < newScale) {
          // 限制一次缩放幅度，避免突变
          const cappedScale = Math.max(newScale * 0.8, fitScale);
          currentScale.value = cappedScale;
          
          // 重置位置到中心
          position.x = 0;
          position.y = 0;
          return;
        }
      }
    }
    
    // 计算缩放前后的位置差，保持缩放的中心点
    const scaleDiff = currentScale.value - newScale;
    if (scaleDiff > 0) {
      // 调整位置以保持中心点
      position.x = position.x - (position.x * scaleDiff / currentScale.value);
      position.y = position.y - (position.y * scaleDiff / currentScale.value);
    }
    
    currentScale.value = newScale;
  }
  
  function handleWheel(event) {
    // 获取鼠标相对于卡片容器的位置
    const rect = cardContainer.value.getBoundingClientRect();
    const mouseX = event.clientX - rect.left;
    const mouseY = event.clientY - rect.top;
    
    // 获取鼠标相对于卡片中心的位置
    const centerX = rect.width / 2;
    const centerY = rect.height / 2;
    
    // 计算鼠标到中心的距离
    const distX = mouseX - centerX;
    const distY = mouseY - centerY;
    
    if (event.deltaY < 0) {
      // 放大时，向鼠标位置方向微调位置
      zoomIn();
      position.x += distX * 0.02;
      position.y += distY * 0.02;
    } else {
      // 缩小时，远离鼠标位置方向微调位置
      zoomOut();
      position.x -= distX * 0.01;
      position.y -= distY * 0.01;
    }
    
    event.preventDefault();
  }

  // 监听表单数据变化，实时更新卡片
  watch(() => form.data, () => {
    cardLeaf.value.setData(form.data);
  }, { deep: true });
  
  // 监听窗口大小变化，重新调整卡片大小
  window.addEventListener('resize', adjustCardToFit);

  // 监听卡片类型变化，自动切换到合适的标签页
  watch(() => form.data.type, (newType) => {
    if (newType === 'monster' || newType === 'pendulum') {
      // 切换到怪兽属性
      activeTab.value = '2';
    } else if (newType === 'spell' || newType === 'trap') {
      // 非怪兽卡，回到基本信息
      activeTab.value = '1';
    }
  });

  function toGithub() {
    open('https://github.com/kooriookami/yugioh-card');
  }
  
  // 处理AI生成的卡片数据
  function applyGeneratedCard(generatedData) {
    // 设置卡片类型
    if (generatedData.cardType === 'monster') {
      form.data.type = 'monster';
      
      // 检查怪兽类型中是否包含"效果"关键字
      if (generatedData.monsterType && generatedData.monsterType.includes('效果')) {
        form.data.cardType = 'effect'; // 设置为效果怪兽
      } else {
        form.data.cardType = 'normal'; // 默认设置为通常怪兽
      }
    } else if (generatedData.cardType === 'spell') {
      form.data.type = 'spell';
    } else if (generatedData.cardType === 'trap') {
      form.data.type = 'trap';
    }
    
    if (generatedData.name) {
      form.data.name = generatedData.name;
    }
    
    if (generatedData.description) {
      form.data.description = generatedData.description;
    }
    
    if (generatedData.imageUrl) {
      imageUrl.value = generatedData.imageUrl;
      form.data.image = generatedData.imageUrl;
    }
    
    // 应用怪兽属性（如果是怪兽卡）
    if (form.data.type === 'monster') {
      if (generatedData.monsterType) {
        form.data.monsterType = generatedData.monsterType;
      }
      
      if (generatedData.level) {
        form.data.level = generatedData.level;
      }
      
      if (generatedData.atk) {
        form.data.atk = generatedData.atk;
      }
      
      if (generatedData.def) {
        form.data.def = generatedData.def;
      }
      
      // 应用属性
      if (generatedData.attribute) {
        form.data.attribute = generatedData.attribute;
      }
    }
    
    // 更新卡片
    cardLeaf.value.setData(form.data);
  }
</script>

<style lang="scss" scoped>
  .yugioh-card-container {
    display: flex;
    flex-direction: column;
    min-height: 100%;
    width: 100%;
    background-color: #f5f7fa;
    overflow-y: auto;
    overflow-x: hidden;
    position: relative;
  }

  .form-header {
    padding: 12px 15px;
    background-color: var(--primary-color);
    color: #fff;
    position: sticky;
    top: 0;
    z-index: 10;
    
    .form-title {
      display: flex;
      align-items: center;
      font-size: 16px;
      font-weight: bold;
      
      .github-icon {
        margin-left: 10px;
        cursor: pointer;
        color: #fff;
      }
    }
    
    .form-description {
      margin-top: 4px;
      font-size: 12px;
      opacity: 0.8;
    }
  }

  .yugioh-card {
    height: 45vh;
    min-height: 280px;
    max-height: 400px;
    display: flex;
    flex-direction: column;
    position: relative;
    
    .card-container {
      flex: 1;
      display: flex;
      justify-content: center;
      align-items: center;
      position: relative;
      overflow: hidden;
      background-color: #fff;

      .card {
        display: inline-block;
        cursor: move;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        transition: box-shadow 0.3s ease;
        will-change: transform;
        position: relative;
        
        &:hover {
          box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
        }
      }
    }
    
    .card-controls {
      display: flex;
      justify-content: center;
      gap: 10px;
      padding: 8px;
      background-color: #fff;
      border-top: 1px solid var(--border-color);
      
      .control-btn {
        box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
      }
    }
  }
  
  .divider {
    width: 100%;
    height: 1px;
    background-color: var(--border-color);
  }

  .form-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    background-color: #fff;
    overflow: visible;
    
    .form-main {
      padding: 0;
      flex: 1;
      overflow: visible;
      
      .card-tabs {
        width: 100%;
        
        :deep(.el-tabs__header) {
          margin-bottom: 15px;
          padding: 0 15px;
          background-color: #f9f9f9;
          border-bottom: 1px solid var(--border-color);
          position: sticky;
          top: 0;
          z-index: 5;
        }
        
        :deep(.el-tabs__nav) {
          border: none;
          width: 100%;
          display: flex;
          justify-content: space-between;
        }
        
        :deep(.el-tabs__nav-next.is-disabled),
        :deep(.el-tabs__nav-prev.is-disabled) {
          display: none;
        }
        
        :deep(.el-tabs__item) {
          height: 40px;
          line-height: 40px;
          font-size: 14px;
          color: #606266;
          flex: 1;
          text-align: center;
          padding: 0 5px;
          
          &.is-active {
            color: var(--primary-color);
            font-weight: 500;
          }
        }
        
        :deep(.el-tabs__content) {
          padding: 0 15px 15px;
          overflow: visible;
        }
      }
      
      .form-row {
        display: flex;
        flex-wrap: wrap;
        gap: 12px;
        margin-bottom: 15px;
        
        .form-item {
          flex: 1 0 100%;
          min-width: 0;
          
          @media (min-width: 768px) {
            flex: 1;
            min-width: 200px;
          }
        }
        
        .form-item-wide {
          flex-basis: 100%;
        }
        
        .checkbox-group {
          display: flex;
          align-items: center;
          gap: 12px;
          flex-wrap: wrap;
          justify-content: flex-start;
        }
      }
      
      .atk-def-container {
        display: flex;
        align-items: center;

        .separator {
          margin: 0 10px;
        }
      }
      
      .atk-def-note {
        font-size: 12px;
        color: #999;
        margin-top: 5px;
      }
      
      .card-image-uploader {
        width: 100%;
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
        transition: border-color 0.3s;
        
        &:hover {
          border-color: var(--primary-color);
        }
        
        .card-image {
          width: 100%;
          height: 180px;
          display: block;
          object-fit: cover;
        }
        
        .uploader-icon {
          font-size: 28px;
          color: #8c939d;
          width: 100%;
          height: 150px;
          display: flex;
          justify-content: center;
          align-items: center;
        }
      }
      
      .upload-tips {
        font-size: 12px;
        color: #999;
        margin-top: 5px;
        text-align: center;
      }
      
      .link-arrows {
        .arrow-grid {
          display: grid;
          grid-template-columns: repeat(3, 1fr);
          grid-template-rows: repeat(3, 1fr);
          width: 150px;
          height: 150px;
          gap: 5px;
          
          .center-box {
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: #f0f0f0;
            border-radius: 4px;
            font-size: 14px;
          }
        }
      }
    }
  }
  
  // 移动设备优化
  @media (max-width: 768px) {
    .yugioh-card {
      height: 40vh;
      min-height: 260px;
    }
    
    .form-container .form-main .card-tabs :deep(.el-tabs__header) {
      padding: 0 8px;
    }
    
    .form-container .form-main .card-tabs :deep(.el-tabs__content) {
      padding: 0 8px 15px;
    }
    
    .form-container .form-main .card-tabs :deep(.el-tabs__item) {
      padding: 0 5px;
      font-size: 13px;
    }
    
    :deep(.el-form-item__label) {
      padding-bottom: 4px;
    }
    
    :deep(.el-form-item) {
      margin-bottom: 12px;
    }
    
    :deep(.el-input-number) {
      width: 100% !important;
    }
  }
  
  // AI生成器相关样式
  .ai-generator-wrapper {
    padding: 0 5px;
  }
  
  :deep(.ai-generator-container) {
    width: 100%;
    max-width: 100%;
  }
  
  :deep(.generated-image) {
    display: block;
    margin: 0 auto;
  }
  
  :deep(.modern-form) {
    padding: 12px 10px;
  }
  
  :deep(.form-row) {
    margin-bottom: 8px;
  }
  
  :deep(.horizontal-checkbox-group) {
    flex-wrap: wrap;
  }
</style>

