<template>
  <div class="ai-generator-container">
    <el-form :model="generatorForm" label-width="80px" size="small" class="modern-form">
      <!-- 第一行 -->
      <div class="form-row">
        <el-form-item label="卡片主题" class="form-item theme-item">
          <el-input 
            v-model="generatorForm.theme" 
            placeholder="输入卡片主题，例如：龙、魔法师、战士等"
          ></el-input>
        </el-form-item>
      </div>
      
      <!-- 第二行 -->
      <div class="form-row">
        <el-form-item label="卡片类型" class="form-item">
          <el-select v-model="generatorForm.cardType" placeholder="选择生成的卡片类型">
            <el-option label="怪兽卡" value="monster" />
            <el-option label="魔法卡" value="spell" />
            <el-option label="陷阱卡" value="trap" />
          </el-select>
        </el-form-item>
        
        <el-form-item label="卡片属性" v-if="generatorForm.cardType === 'monster'" class="form-item">
          <el-select v-model="generatorForm.attribute" placeholder="选择怪兽属性">
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
      
      <!-- 第三行 -->
      <div class="form-row">
        <el-form-item label="卡片强度" class="form-item">
          <el-radio-group v-model="generatorForm.power" class="radio-group">
            <el-radio value="weak">弱</el-radio>
            <el-radio value="medium">中</el-radio>
            <el-radio value="strong">强</el-radio>
          </el-radio-group>
        </el-form-item>
      </div>
      
      <!-- 第四行 - 选项 -->
      <div class="form-row options-row">
        <div class="checkbox-container">
          <el-checkbox-group v-model="generatorForm.options" class="horizontal-checkbox-group">
            <el-checkbox value="name">生成卡名</el-checkbox>
            <el-checkbox value="description">生成效果</el-checkbox>
            <el-checkbox value="image">生成图片</el-checkbox>
          </el-checkbox-group>
        </div>
      </div>
      
      <!-- 按钮 -->
      <div class="form-row button-row">
        <el-button type="primary" @click="generateCard" :loading="loading" class="generate-button">生成卡片</el-button>
      </div>
    </el-form>
    
    <div v-if="loading" class="loading-container">
      <el-progress type="circle" :percentage="generationProgress"></el-progress>
      <div class="loading-text">{{ loadingMessage }}</div>
    </div>
    
    <div v-if="hasResults" class="results-container">
      <div class="result-section">
        <h4>生成的卡片名称</h4>
        <div class="result-content">{{ generatedData.name }}</div>
      </div>
      
      <div class="result-section" v-if="generatedData.description">
        <h4>生成的卡片效果</h4>
        <div class="result-content">{{ generatedData.description }}</div>
      </div>
      
      <div class="result-section" v-if="generatedData.imageUrl">
        <h4>生成的卡片图片</h4>
        <img class="generated-image" :src="generatedData.imageUrl" alt="生成的卡片图片">
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue';

const emit = defineEmits(['apply-to-card']);

// 表单数据
const generatorForm = reactive({
  theme: '',
  cardType: 'monster',
  attribute: 'dark',
  power: 'medium',
  options: ['name', 'description', 'image'] // 默认全选
});

// 状态变量
const loading = ref(false);
const generationProgress = ref(0);
const loadingMessage = ref('');
const generatedData = reactive({
  name: '',
  description: '',
  monsterType: '',
  level: 0,
  atk: 0,
  def: 0,
  imageUrl: '',
  cardType: '',
  attribute: ''
});

// 计算属性
const hasResults = computed(() => {
  return generatedData.name || generatedData.description || generatedData.imageUrl;
});

// 生成卡片
async function generateCard() {
  if (!generatorForm.theme) {
    alert('请输入卡片主题');
    return;
  }
  
  loading.value = true;
  loadingMessage.value = '正在思考卡片设计...';
  generationProgress.value = 10;
  
  try {
    // 设置卡片类型
    generatedData.cardType = generatorForm.cardType;
    
    // 设置属性（如果是怪兽卡）
    if (generatorForm.cardType === 'monster') {
      generatedData.attribute = generatorForm.attribute;
    }
    
    // 生成卡片名称和基本属性
    if (generatorForm.options.includes('name')) {
      await generateCardName();
    }
    
    // 生成卡片效果描述
    if (generatorForm.options.includes('description')) {
      await generateCardDescription();
    }
    
    // 生成卡片图片
    if (generatorForm.options.includes('image')) {
      await generateCardImage();
    }
    
    loadingMessage.value = '生成完成!';
    generationProgress.value = 100;
    
    // 自动应用生成的卡片数据
    applyToCard();
  } catch (error) {
    console.error('卡片生成失败:', error);
    loadingMessage.value = `生成失败: ${error.message || '未知错误'}`;
  } finally {
    loading.value = false;
  }
}

// 生成卡片名称
async function generateCardName() {
  loadingMessage.value = '正在生成卡片名称...';
  generationProgress.value = 30;
  
  const cardTypeText = {
    'monster': '怪兽',
    'spell': '魔法',
    'trap': '陷阱'
  }[generatorForm.cardType];
  
  const prompt = `为一张游戏王卡片创建一个名称，主题是"${generatorForm.theme}"，卡片类型是${cardTypeText}卡。名称应该简洁有力，符合游戏王卡片的命名风格，不要超过10个汉字。只返回卡片名称，不要有其他说明。`;
  
  try {
    // 使用Pollinations文本API生成卡名
    const response = await fetch(`https://text.pollinations.ai/${encodeURIComponent(prompt)}?model=openai&private=true`);
    
    if (!response.ok) {
      throw new Error(`API错误 (${response.status})`);
    }
    
    const name = await response.text();
    generatedData.name = name.trim();
    
    // 如果是怪兽卡，还需要生成额外属性
    if (generatorForm.cardType === 'monster') {
      await generateMonsterAttributes();
    }
    
  } catch (error) {
    console.error('卡名生成失败:', error);
    throw new Error('卡名生成失败');
  }
}

// 生成怪兽属性
async function generateMonsterAttributes() {
  const powerMapping = {
    'weak': { levelMin: 1, levelMax: 4, atkMin: 500, atkMax: 1500, defMin: 500, defMax: 1500 },
    'medium': { levelMin: 5, levelMax: 7, atkMin: 1600, atkMax: 2500, defMin: 1200, defMax: 2500 },
    'strong': { levelMin: 8, levelMax: 12, atkMin: 2600, atkMax: 4000, defMin: 2000, defMax: 3500 }
  };
  
  const power = powerMapping[generatorForm.power];
  
  // 生成随机等级、攻击力和防御力
  generatedData.level = Math.floor(Math.random() * (power.levelMax - power.levelMin + 1)) + power.levelMin;
  generatedData.atk = Math.floor(Math.random() * (power.atkMax - power.atkMin + 1)) + power.atkMin;
  generatedData.def = Math.floor(Math.random() * (power.defMax - power.defMin + 1)) + power.defMin;
  
  // 生成怪兽类型
  const prompt = `给一个名为"${generatedData.name}"的游戏王怪兽卡片生成一个合适的怪兽类型，考虑到它的主题是"${generatorForm.theme}"。格式为"XXX族"或"XXX族/效果"，直接返回类型，不要有其他说明。`;
  
  try {
    const response = await fetch(`https://text.pollinations.ai/${encodeURIComponent(prompt)}?model=openai&private=true`);
    
    if (!response.ok) {
      throw new Error(`API错误 (${response.status})`);
    }
    
    const monsterType = await response.text();
    generatedData.monsterType = monsterType.trim();
  } catch (error) {
    console.error('怪兽类型生成失败:', error);
    generatedData.monsterType = '???族';
  }
}

// 生成卡片效果描述
async function generateCardDescription() {
  loadingMessage.value = '正在生成卡片效果...';
  generationProgress.value = 50;
  
  let prompt = '';
  
  if (generatorForm.cardType === 'monster') {
    prompt = `为一张游戏王怪兽卡创建效果文本，卡片名称是"${generatedData.name || '未命名怪兽'}"，主题是"${generatorForm.theme}"，怪兽类型是"${generatedData.monsterType || '未知族'}"，等级为${generatedData.level}，攻击力${generatedData.atk}，防御力${generatedData.def}。`;
    
    // 根据强度调整效果的强度
    prompt += `卡片强度为${{'weak':'弱', 'medium':'中等', 'strong':'强大'}[generatorForm.power]}。效果应该平衡且符合游戏王卡片的风格。请仅返回效果文本内容，不要包含怪兽的类型、等级、攻防等信息，不要有其他说明。`;
  } else if (generatorForm.cardType === 'spell') {
    prompt = `为一张游戏王魔法卡创建效果文本，卡片名称是"${generatedData.name || '未命名魔法卡'}"，主题是"${generatorForm.theme}"。`;
    prompt += `卡片强度为${{'weak':'弱', 'medium':'中等', 'strong':'强大'}[generatorForm.power]}。效果应该平衡且符合游戏王卡片的风格。请仅返回效果文本，不要有任何其他说明或附加信息。`;
  } else {
    prompt = `为一张游戏王陷阱卡创建效果文本，卡片名称是"${generatedData.name || '未命名陷阱卡'}"，主题是"${generatorForm.theme}"。`;
    prompt += `卡片强度为${{'weak':'弱', 'medium':'中等', 'strong':'强大'}[generatorForm.power]}。效果应该平衡且符合游戏王卡片的风格。请仅返回效果文本，不要有任何其他说明或附加信息。`;
  }
  
  try {
    // 使用Pollinations文本API生成卡片效果
    const response = await fetch(`https://text.pollinations.ai/${encodeURIComponent(prompt)}?model=openai&private=true`);
    
    if (!response.ok) {
      throw new Error(`API错误 (${response.status})`);
    }
    
    const description = await response.text();
    
    // 进一步处理描述文本，确保只包含效果
    let cleanedDescription = description.trim();
    
    // 移除可能包含的"【效果】"等标记
    cleanedDescription = cleanedDescription.replace(/^【.*?】\s*/g, '');
    
    // 移除可能包含的类型信息（如"[天使族/效果]"）
    cleanedDescription = cleanedDescription.replace(/^\[.*?\]\s*/g, '');
    
    // 移除可能包含的星级/攻防信息
    cleanedDescription = cleanedDescription.replace(/级\s*\d+\s*星/g, '');
    cleanedDescription = cleanedDescription.replace(/攻\s*\d+\s*防\s*\d+/g, '');
    
    // 保存处理后的描述
    generatedData.description = cleanedDescription;
  } catch (error) {
    console.error('效果生成失败:', error);
    throw new Error('效果生成失败');
  }
}

// 生成卡片图片
async function generateCardImage() {
  loadingMessage.value = '正在生成卡片图片...';
  generationProgress.value = 70;
  
  // 固定样式后缀
  const fixedStyleSuffix = ", Bright colors, illustrated in dynamic comic book style, Yu-Gi-Oh style";
  
  // 先生成基于卡片名称和主题的详细描述提示
  let detailedPrompt = '';
  try {
    const nameToUse = generatedData.name || generatorForm.theme;
    const promptForDescription = `创建一个详细的英文描述，用于AI图像生成，描述一张名为"${nameToUse}"的游戏王卡片形象。主题是"${generatorForm.theme}"。描述应包含角色/生物的详细外观、姿势、动作、背景环境、氛围和色调。描述应该生动、具体且富有想象力，长度约40-60个英文单词。只返回英文描述，不要有其他说明。`;
    
    // 使用Pollinations文本API生成详细描述
    const response = await fetch(`https://text.pollinations.ai/${encodeURIComponent(promptForDescription)}?model=openai&private=true`);
    
    if (response.ok) {
      detailedPrompt = await response.text();
      detailedPrompt = detailedPrompt.trim();
    }
  } catch (error) {
    console.error('生成详细描述失败:', error);
    // 如果详细描述生成失败，使用备用提示
  }
  
  // 如果无法生成详细描述，使用备用提示词
  if (!detailedPrompt) {
    if (generatorForm.cardType === 'monster') {
      detailedPrompt = `Detailed Yugioh card art of a ${generatedData.name || generatorForm.theme}, ${generatorForm.attribute} attribute monster, detailed creature design, dynamic pose, dramatic lighting, mystical aura, intricate details`;
    } else if (generatorForm.cardType === 'spell') {
      detailedPrompt = `Detailed Yugioh card art of a ${generatedData.name || generatorForm.theme} magic spell, arcane symbols, magical effects, energy swirls, mystical artifacts, glowing runes, powerful magical atmosphere`;
    } else {
      detailedPrompt = `Detailed Yugioh card art of a ${generatedData.name || generatorForm.theme} trap, tactical defensive scene, clever mechanism, dramatic moment, hidden dangers, strategic elements, suspenseful atmosphere`;
    }
  }
  
  // 添加额外的卡片类型特定内容
  if (generatorForm.cardType === 'monster') {
    detailedPrompt += `, ${generatorForm.attribute} attribute`;
  }
  
  // 添加固定风格后缀
  const imagePrompt = detailedPrompt + fixedStyleSuffix;
  
  // 随机种子
  const seed = Math.floor(Math.random() * 9999);
  
  try {
    // 尝试使用默认模型生成图片
    let imageUrl = `https://image.pollinations.ai/prompt/${encodeURIComponent(imagePrompt)}?width=1024&height=1024&nologo=true&seed=${seed}`;
    
    // 通过创建Image对象确认图像已加载
    await new Promise((resolve, reject) => {
      const img = new Image();
      img.onload = () => resolve();
      img.onerror = () => reject(new Error('图片生成失败'));
      img.src = imageUrl;
      
      // 超时处理 - 10秒后继续，避免无限等待
      setTimeout(() => resolve(), 10000);
    });
    
    // 如果成功获取到图片
    generatedData.imageUrl = imageUrl;
    
  } catch (error) {
    console.error('图片生成失败，尝试备用选项:', error);
    
    try {
      // 备用选项: 使用不同的API端点
      const backupImageUrl = `https://image.pollinations.ai/prompt/${encodeURIComponent(imagePrompt)}?width=512&height=512&nologo=true&seed=${seed}`;
      
      await new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => resolve();
        img.onerror = () => reject(new Error('备用图片加载失败'));
        img.src = backupImageUrl;
        setTimeout(() => resolve(), 10000);
      });
      
      generatedData.imageUrl = backupImageUrl;
    } catch (backupError) {
      console.error('备用图片也失败了:', backupError);
      generatedData.imageUrl = ''; // 直接设为空，不再使用预设图片
    }
  }
}

// 应用到卡片编辑器
function applyToCard() {
  emit('apply-to-card', {...generatedData});
}
</script>

<style scoped>
.ai-generator-container {
  padding: 8px 0;
  width: 100%;
  max-width: 100%;
  margin: 0 auto;
}

.modern-form {
  background-color: #f9fafc;
  border-radius: 8px;
  padding: 16px 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
}

.form-row {
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 10px;
  justify-content: space-between;
  gap: 10px;
}

.form-item {
  flex: 1 0 100%;
  margin-right: 0;
}

.theme-item {
  flex: 1 0 100%;
}

.radio-group {
  display: flex;
  justify-content: space-between;
  width: 100%;
  max-width: 100%;
}

.options-row {
  display: flex;
  justify-content: center;
  margin: 12px 0;
}

.button-row {
  display: flex;
  justify-content: center;
  margin-top: 16px;
}

.checkbox-container {
  display: flex;
  justify-content: center;
  width: 100%;
}

.horizontal-checkbox-group {
  display: flex;
  justify-content: center;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  width: 100%;
  gap: 8px;
}

.horizontal-checkbox-group :deep(.el-checkbox) {
  margin: 4px;
  white-space: nowrap;
}

.generate-button {
  width: 100%;
  max-width: 180px;
  height: 40px;
  font-size: 16px;
  font-weight: 500;
  border-radius: 20px;
}

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 20px 0;
}

.loading-text {
  margin-top: 16px;
  color: #606266;
  text-align: center;
}

.results-container {
  margin-top: 20px;
  border-top: 1px solid #EBEEF5;
  padding-top: 16px;
}

.result-section {
  margin-bottom: 16px;
}

.result-section h4 {
  color: #303133;
  margin-bottom: 8px;
  font-size: 15px;
}

.result-content {
  padding: 10px;
  background-color: #fff;
  border: 1px solid #EBEEF5;
  border-radius: 4px;
  white-space: pre-wrap;
  word-break: break-word;
  font-size: 14px;
  line-height: 1.5;
}

.generated-image {
  max-width: 100%;
  max-height: 280px;
  object-fit: contain;
  border: 1px solid #EBEEF5;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  display: block;
  margin: 0 auto;
}

@media (min-width: 768px) {
  .form-item {
    flex: 1;
    min-width: 200px;
    margin-right: 10px;
  }
  
  .radio-group {
    max-width: 200px;
  }
  
  .horizontal-checkbox-group {
    flex-wrap: nowrap;
  }
  
  .generate-button {
    width: 180px;
  }
}
</style> 