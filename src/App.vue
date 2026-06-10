<template>
  <section class="lens-page">
    <div class="bg-grid"></div>
    <div class="scan-line"></div>

    <header class="topbar">
      <div class="brand">
        <div class="logo"></div>
        <div>
          <div class="eyebrow">PHYSICS OPTICS LAB</div>
          <h1>凸透镜成像 3D 互动实验台</h1>
        </div>
      </div>

      <div class="top-actions">
        <button class="ghost guide-top-btn" @click="showGuideDialog = true">使用引导</button>
        <button class="ghost" :class="{ active: cameraView === 'standard' }" @click="setCameraView('standard')">标准视角</button>
        <button class="ghost" :class="{ active: cameraView === 'top' }" @click="setCameraView('top')">俯视光路</button>
        <button class="ghost top-snapshot-btn" @click="snapshotExperiment">快照当前实验</button>
        <button class="ghost" :disabled="experimentLogs.length === 0" @click="exportExperimentLogsAsImage">导出日志图片</button>
        <button class="ghost" :disabled="experimentLogs.length === 0" @click="clearExperimentLogs">清空日志</button>
        <button class="ghost" @click="resetExperiment">重置</button>
      </div>
    </header>

    <Teleport to="body">
      <div v-if="showGuideDialog" class="guide-mask" @click.self="showGuideDialog = false">
        <section class="guide-modal">
          <header class="guide-header">
            <div>
              <div class="guide-kicker">USER GUIDE</div>
              <h2>凸透镜成像 3D 互动实验台 · 使用引导</h2>
              <p>适合课堂导入、实验演示、规律归纳、学生闯关和实验记录导出。</p>
            </div>
            <button class="guide-close" @click="showGuideDialog = false">×</button>
          </header>

          <div class="guide-body">
            <div class="guide-card guide-main-card">
              <h3>推荐上课流程</h3>
              <ol class="guide-steps">
                <li><b>先看结构：</b>用“标准视角”认识物体、凸透镜、光屏、主光轴、F 与 2F。</li>
                <li><b>再拖参数：</b>调整物距 u 和焦距 f，让学生观察像距 v、像的大小和倒正变化。</li>
                <li><b>切典型位置：</b>依次点击 u&gt;2f、u=2f、f&lt;u&lt;2f、u=f、u&lt;f，对照右侧规律表。</li>
                <li><b>做光屏寻像：</b>开启光屏寻像后拖动光屏，越接近真实像距，投影越清晰。</li>
                <li><b>最后留痕：</b>点击“快照当前实验”，把参数、现象和结论加入实验日志，再导出图片。</li>
              </ol>
            </div>

            <div class="guide-grid">
              <div class="guide-card">
                <h3>左侧怎么用</h3>
                <ul>
                  <li><b>实验参数：</b>控制物距 u、焦距 f、物高 H。</li>
                  <li><b>典型位置：</b>快速进入教材中常见的五种成像情况。</li>
                  <li><b>场景显示：</b>控制光线、光屏、标签等图层，适合分层讲解。</li>
                  <li><b>生活应用：</b>切换光具座、照相机、投影仪、放大镜，建立生活联系。</li>
                  <li><b>课堂闯关：</b>随机生成题目，让学生判断成像规律。</li>
                </ul>
              </div>

              <div class="guide-card">
                <h3>右侧怎么看</h3>
                <ul>
                  <li><b>实时数据：</b>看 u、f、v、放大率、成像性质和光屏结果。</li>
                  <li><b>成像规律表：</b>当前参数对应的规律会自动高亮。</li>
                  <li><b>公式推导：</b>展示 v = f·u / (u - f) 的计算过程。</li>
                  <li><b>实验日志：</b>保存每次观察记录，便于复盘和导出。</li>
                </ul>
              </div>

              <div class="guide-card">
                <h3>重点物理规律</h3>
                <ul>
                  <li><b>u &gt; 2f：</b>倒立、缩小、实像，像在 f 与 2f 之间。</li>
                  <li><b>u = 2f：</b>倒立、等大、实像，像在 2f 处。</li>
                  <li><b>f &lt; u &lt; 2f：</b>倒立、放大、实像，像在 2f 以外。</li>
                  <li><b>u = f：</b>出射光线近似平行于主光轴，不在有限位置成清晰像。</li>
                  <li><b>u &lt; f：</b>正立、放大、虚像，光屏不能承接。</li>
                </ul>
              </div>

              <div class="guide-card">
                <h3>课堂提示</h3>
                <ul>
                  <li><b>不要一开始全开：</b>可以先关掉部分标签，只讲主光轴和像，再逐步打开光线。</li>
                  <li><b>实像要看光屏：</b>只有实像能被光屏承接；虚像只能通过反向延长线理解。</li>
                  <li><b>靠近焦点要提醒：</b>u 接近 f 时，像距会快速变大，这是学生最容易疑惑的地方。</li>
                  <li><b>导出日志可作实验单：</b>适合课后整理、公开课展示或学生观察记录。</li>
                </ul>
              </div>
            </div>
          </div>

          <footer class="guide-footer">
            <span>建议：先用“f &lt; u &lt; 2f”观察倒立放大实像，再切换到“u &lt; f”理解放大镜虚像。</span>
            <button class="guide-start" @click="showGuideDialog = false">开始使用</button>
          </footer>
        </section>
      </div>
    </Teleport>

    <main class="layout">
      <aside class="panel left-panel">
        <div class="panel-title"><span></span>实验控制</div>

        <div class="block">
          <h3>① 实验参数</h3>
          <RangeRow label="物距 u" :value="state.u" suffix="cm" :min="6" :max="60" :step="0.1" @update:value="setManualValue('u', $event)" />
          <RangeRow label="焦距 f" :value="state.f" suffix="cm" :min="6" :max="20" :step="0.1" @update:value="setManualValue('f', $event)" />
          <RangeRow label="物高 H" :value="state.h" suffix="cm" :min="2" :max="8" :step="0.1" @update:value="setManualValue('h', $event)" />
          <p class="tip">先调物距、焦距和物高，3D 光具座会实时改变光路、像距和成像性质。</p>
        </div>

        <div class="block">
          <h3>② 典型位置</h3>
          <div class="preset-grid">
            <button :class="{ active: presetKey === 'far' }" @click="setPreset('far')">u &gt; 2f</button>
            <button :class="{ active: presetKey === 'twof' }" @click="setPreset('twof')">u = 2f</button>
            <button :class="{ active: presetKey === 'between' }" @click="setPreset('between')">f &lt; u &lt; 2f</button>
            <button :class="{ active: presetKey === 'focus' }" @click="setPreset('focus')">u = f</button>
            <button :class="{ active: presetKey === 'inside' }" @click="setPreset('inside')">u &lt; f</button>
            <button :class="{ active: presetKey === 'near' }" @click="setPreset('near')">靠近焦点</button>
          </div>
          <p class="tip">建议课堂上按顺序点击：缩小实像 → 等大实像 → 放大实像 → 不成有限像 → 放大虚像。</p>
        </div>

        <div class="block control-block">
          <h3>③ 场景显示</h3>
          <div class="layer-grid">
            <button :class="{ active: showRays }" @click="toggleLayer('rays')">三条特殊光线</button>
            <button :class="{ active: showScreen }" @click="toggleLayer('screen')">光屏</button>
            <button :class="{ active: showLabels }" @click="toggleLayer('labels')">标签</button>
          </div>
          <p class="tip">这是图层开关：可先隐藏标签讲现象，再打开光线和标注讲原理。</p>
        </div>

        <div class="block feature-block">
          <h3>④ 生活应用</h3>
          <div class="mode-grid">
            <button :class="{ active: appMode === 'bench' }" @click="setAppMode('bench')">光具座</button>
            <button :class="{ active: appMode === 'camera' }" @click="setAppMode('camera')">照相机</button>
            <button :class="{ active: appMode === 'projector' }" @click="setAppMode('projector')">投影仪</button>
            <button :class="{ active: appMode === 'magnifier' }" @click="setAppMode('magnifier')">放大镜</button>
          </div>
          <p class="tip">应用模式会自动切换到对应成像条件：照相机 u&gt;2f，投影仪 f&lt;u&lt;2f，放大镜 u&lt;f。</p>
        </div>

        <div class="block feature-block">
          <h3>⑤ 光屏寻像</h3>
          <div class="challenge-row">
            <button class="wide" :class="{ active: focusChallenge }" @click="toggleFocusChallenge">
              {{ focusChallenge ? '退出光屏寻像' : '开启光屏寻像' }}
            </button>
            <button v-if="focusChallenge" class="wide ghost-btn" @click="snapScreenToImage">一键对焦到像距</button>
          </div>

          <RangeRow
            v-if="focusChallenge"
            label="光屏位置"
            :value="state.screenCm"
            suffix="cm"
            :min="0"
            :max="120"
            :step="0.1"
            @update:value="setManualValue('screenCm', $event)"
          />
          <p class="tip">光屏寻像只用于实像。若当前是虚像或 u=f，开启时会自动切回 f&lt;u&lt;2f，避免物理意义混乱。</p>
        </div>

        <div class="block challenge-card-left">
          <h3>⑥ 课堂闯关</h3>
          <div class="challenge-status">
            <b>{{ challengeActive ? '正在闯关' : '未开始' }}</b>
            <span>{{ challengeActive ? '观察 3D 场景后选择对应规律' : '随机生成一个成像位置，让学生判断规律' }}</span>
          </div>

          <div class="challenge-actions">
            <button class="wide" @click="startChallenge">{{ challengeActive ? '换一题' : '开始闯关' }}</button>
            <button class="wide ghost-btn" :disabled="!challengeActive" @click="exitChallenge">退出闯关</button>
          </div>

          <div v-if="challengeActive" class="choice-list">
            <button
              v-for="choice in challengeChoices"
              :key="choice.key"
              :class="{
                active: selectedChallengeKey === choice.key,
                correct: challengeResultVisible && choice.key === challengeExpectedKey,
                wrong: challengeResultVisible && selectedChallengeKey === choice.key && selectedChallengeKey !== challengeExpectedKey,
              }"
              @click="selectChallengeChoice(choice.key)"
            >
              <b>{{ choice.title }}</b>
              <span>{{ choice.result }}</span>
            </button>
          </div>

          <div v-if="challengeActive" class="challenge-actions">
            <button class="wide" :disabled="!selectedChallengeKey" @click="verifyChallengeAnswer">验证答案</button>
          </div>

          <div v-if="challengeResultVisible" class="challenge-result" :class="{ pass: challengePassed }">
            <b>{{ challengePassed ? '判断正确' : '再观察一下' }}</b>
            <span>{{ challengeFeedback }}</span>
          </div>
        </div>
      </aside>

      <section class="stage-card">
        <div ref="canvasWrapRef" class="canvas-wrap"></div>

        <div class="scene-title">
          <b>{{ imageType.title }}</b>
          <span>{{ appModeLabel }} · u={{ formatNumber(metrics.u) }}cm · f={{ formatNumber(metrics.f) }}cm · {{ imageType.nature }}</span>
        </div>

        <div class="legend-panel">
          <div class="legend-title">图例</div>
          <div><i class="ray yellow"></i>平行主轴光线</div>
          <div><i class="ray cyan"></i>过光心光线</div>
          <div><i class="ray green"></i>过焦点光线</div>
          <div><i class="ray red"></i>虚像反向延长线</div>
        </div>

        <div class="toast">{{ imageType.toast }}</div>

        <div class="metric-strip">
          <div class="metric-card">
            <span>像距 v（带符号）</span>
            <b>{{ isFiniteNumber(metrics.v) ? `${formatNumber(metrics.v)} cm` : '∞' }}</b>
          </div>
          <div class="metric-card">
            <span>放大率 |m|</span>
            <b>{{ isFiniteNumber(metrics.m) ? formatNumber(Math.abs(metrics.m), 2) : '∞' }}</b>
          </div>
          <div class="metric-card active">
            <span>成像性质</span>
            <b>{{ imageType.short }}</b>
          </div>
          <div class="metric-card">
            <span>{{ focusChallenge ? '调焦清晰度' : '光屏判断' }}</span>
            <b>{{ focusChallenge ? focusStatusText : imageType.screen }}</b>
          </div>
        </div>
      </section>

      <aside class="panel right-panel">
        <div class="panel-title"><span></span>知识速览</div>

        <div class="data-card">
          <div class="card-head">实时数据</div>
          <div class="big-row">
            <span>物距 u</span><b>{{ formatNumber(metrics.u) }} cm</b>
          </div>
          <div class="big-row">
            <span>焦距 f</span><b>{{ formatNumber(metrics.f) }} cm</b>
          </div>
          <div class="big-row">
            <span>像距 v（带符号）</span><b>{{ isFiniteNumber(metrics.v) ? `${formatNumber(metrics.v)} cm` : '∞' }}</b>
          </div>
          <div class="small-grid">
            <div>
              <span>物高 H</span><b>{{ formatNumber(metrics.h) }} cm</b>
            </div>
            <div>
              <span>像高 h'</span><b>{{ isFiniteNumber(metrics.imageHeight) ? `${formatNumber(Math.abs(metrics.imageHeight))} cm` : '∞' }}</b>
            </div>
            <div>
              <span>放大率</span><b>{{ isFiniteNumber(metrics.m) ? formatNumber(Math.abs(metrics.m), 2) : '∞' }}</b>
            </div>
            <div>
              <span>光屏</span><b>{{ focusChallenge ? focusStatusText : imageType.screen }}</b>
            </div>
          </div>
        </div>

        <div class="knowledge-card log-card">
          <h3>实验日志 · 时间快照</h3>
          <div v-if="experimentLogs.length === 0" class="log-empty">暂无快照。点击顶部“快照当前实验”，可以记录每次实验的参数、现象和结论。</div>
          <div v-else class="log-list">
            <div v-for="(item, index) in experimentLogs" :key="item.id" class="log-item">
              <div class="log-head">
                <b>#{{ index + 1 }} {{ item.mode }}</b>
                <span>{{ item.time }}</span>
              </div>
              <p><b>参数：</b>u={{ formatNumber(item.u) }}cm，f={{ formatNumber(item.f) }}cm，H={{ formatNumber(item.h) }}cm，v={{ item.vText }}</p>
              <p><b>现象：</b>{{ item.nature }}；{{ item.position }}</p>
              <p><b>光屏：</b>{{ item.screen }}</p>
              <p><b>结论：</b>{{ item.conclusion }}</p>
            </div>
          </div>
        </div>

        <div class="knowledge-card rule-table-card">
          <h3>成像规律表 · 自动高亮</h3>
          <div class="rule-table">
            <div v-for="row in ruleRows" :key="row.key" class="rule-row" :class="{ active: currentRuleKey === row.key }">
              <div class="rule-left">
                <b>{{ row.title }}</b>
                <span>{{ row.range }}</span>
              </div>
              <div class="rule-right">{{ row.result }}</div>
            </div>
          </div>
        </div>

        <div class="knowledge-card step-guide-card">
          <h3>实验步骤引导</h3>
          <div class="step-progress">
            <span>步骤 {{ lessonStepIndex + 1 }} / {{ lessonSteps.length }}</span>
            <b>{{ currentLessonStep!.title }}</b>
          </div>
          <p class="step-text">{{ currentLessonStep!.text }}</p>
          <div class="step-actions">
            <button class="ghost-btn" :disabled="lessonStepIndex === 0" @click="prevLessonStep">上一步</button>
            <button @click="applyCurrentLessonStep">应用本步</button>
            <button class="ghost-btn" :disabled="lessonStepIndex === lessonSteps.length - 1" @click="nextLessonStep">下一步</button>
          </div>
        </div>

        <div class="knowledge-card">
          <h3>课堂任务</h3>
          <ol class="task-list">
            <li>把物体放在 2f 以外，观察像为什么缩小。</li>
            <li>把物体移到 f 与 2f 之间，观察像距和像的大小变化。</li>
            <li>把物体移到焦点以内，判断为什么光屏接不到像。</li>
            <li>改变焦距 f，比较同一物距下成像位置的变化。</li>
          </ol>
        </div>

        <div class="knowledge-card feature-summary-card">
          <h3>特色模式 · 当前场景</h3>
          <ul>
            <li>
              <b>{{ appModeLabel }}：</b>{{ appModeDescription }}
            </li>
            <li v-if="focusChallenge">
              <b>光屏寻像：</b>光屏位置 {{ formatNumber(state.screenCm) }}cm，距离真实像距 {{ focusDeltaText }}，清晰度 {{ focusScore }}%。
            </li>
            <li v-else><b>光屏寻像：</b>可在左侧开启，让学生自己拖动光屏找清晰像。</li>
          </ul>
        </div>

        <div class="knowledge-card report-card">
          <h3>实验记录 · 可直接用于课堂</h3>
          <pre>{{ experimentReport }}</pre>
        </div>

        <div class="knowledge-card formula-card">
          <h3>公式推导 · 当前参数代入</h3>

          <div class="formula-item strong">
            <b>① 凸透镜成像公式</b>
            <code>1/f = 1/u + 1/v</code>
            <span>{{ formulaText }}</span>
          </div>

          <div class="formula-item">
            <b>② 像距计算过程</b>
            <code>1/v = 1/f - 1/u = (u - f) / (f · u)</code>
            <span>{{ imageDistanceDerivation }}</span>
          </div>

          <div class="formula-item">
            <b>③ 放大率与像高</b>
            <code>m = -v/u，h' = m · H</code>
            <span>{{ magnificationDerivation }}</span>
          </div>

          <div class="formula-item">
            <b>④ 当前成像判定</b>
            <code>根据 u 与 f、2f 的位置关系判断</code>
            <span>{{ currentRuleText }}</span>
          </div>
        </div>

        <div class="knowledge-card convention-card">
          <h3>符号约定 · 防止误解</h3>
          <ul>
            <li><b>v &gt; 0：</b>像在透镜另一侧，是实像位置，光屏可以承接。</li>
            <li><b>v &lt; 0：</b>像在物体同侧，是虚像位置；显示为负号不是距离为负，而是表示方向。</li>
            <li><b>|m|：</b>表示放大倍数；m 的正负用于表示像的正立或倒立。</li>
          </ul>
        </div>

        <div class="knowledge-card observation-card">
          <h3>观察方法 · 这节课怎么讲</h3>
          <ul>
            <li><b>先看物距：</b>判断蜡烛在 2f 外、2f 处、f 与 2f 之间、f 处还是 f 内。</li>
            <li><b>再看光线：</b>平行主轴光线、过光心光线、过焦点光线最终是否会聚。</li>
            <li><b>最后看光屏：</b>会聚在另一侧是实像，可以用光屏承接；反向延长线相交是虚像，光屏接不到。</li>
          </ul>
        </div>

        <div class="knowledge-card observation-card">
          <h3>三条特殊光线</h3>
          <ul>
            <li><b>黄色：</b>平行主光轴入射，经过凸透镜后通过另一侧焦点。</li>
            <li><b>蓝色：</b>通过光心 O，方向近似不变。</li>
            <li><b>绿色：</b>入射光线的延长线通过物侧焦点，经过凸透镜后近似平行主光轴。</li>
          </ul>
        </div>

        <div class="knowledge-card application-card">
          <h3>生活应用对应</h3>
          <div class="app-grid">
            <div><b>照相机</b><span>u &gt; 2f，倒立缩小实像</span></div>
            <div><b>投影仪</b><span>f &lt; u &lt; 2f，倒立放大实像</span></div>
            <div><b>放大镜</b><span>u &lt; f，正立放大虚像</span></div>
          </div>
        </div>

        <div class="knowledge-card">
          <h3>五种典型规律</h3>
          <ul>
            <li><b>u &gt; 2f：</b>倒立、缩小、实像，像在 f 与 2f 之间。</li>
            <li><b>u = 2f：</b>倒立、等大、实像，像在 2f 处。</li>
            <li><b>f &lt; u &lt; 2f：</b>倒立、放大、实像，像在 2f 以外。</li>
            <li><b>u = f：</b>出射光线近似平行于主光轴，不在有限位置成像。</li>
            <li><b>u &lt; f：</b>正立、放大、虚像，光屏不能承接。</li>
          </ul>
        </div>

        <div class="knowledge-card">
          <h3>易错点提醒</h3>
          <ul>
            <li>实像能用光屏承接，虚像不能用光屏承接。</li>
            <li>不要只背表格，关键看物体相对 f 和 2f 的位置。</li>
            <li>物体越靠近焦点，实像像距越大，像也越大。</li>
            <li>焦点以内成正立放大虚像，是放大镜的基本原理。</li>
          </ul>
        </div>
      </aside>
    </main>
  </section>
</template>

<script setup lang="ts">
import { computed, defineComponent, h, nextTick, onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue'
import { ElSlider } from 'element-plus'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

type PresetKey = 'far' | 'twof' | 'between' | 'focus' | 'inside' | 'near' | ''
type CameraView = 'standard' | 'top'
type AppMode = 'bench' | 'camera' | 'projector' | 'magnifier'
type RuleKey = 'far' | 'twof' | 'between' | 'focus' | 'inside'

type ExperimentLog = {
  id: number
  time: string
  mode: string
  u: number
  f: number
  h: number
  vText: string
  nature: string
  position: string
  screen: string
  conclusion: string
}

type RayStyle = 'yellow' | 'cyan' | 'green' | 'red'

type OpticalMetrics = {
  u: number
  f: number
  h: number
  v: number
  m: number
  real: boolean
  virtual: boolean
  imageHeight: number
}

const CM_TO_UNIT = 0.085
const HEIGHT_TO_UNIT = 0.13
const AXIS_Y = 0
const LENS_X = 0
const BENCH_Z = 0
// 光具座进一步缩到约 ±120cm：进场和标准视角都能直接看清蜡烛、透镜、成像和光屏。
// 有限像距仍然按真实比例绘制，但通过参数保护避免极端像距把场景拉得过长。
const MIN_X = -10.5
const MAX_X = 10.5
const BENCH_LIMIT_CM = 120
// 参数保护：除 u=f 这个特殊演示点外，限制有限像距 |v| 不超过光具座范围。
// 这样不会再出现 v=几千厘米 这种对课堂演示意义不大的极端参数。
const MAX_ABS_IMAGE_DISTANCE_CM = BENCH_LIMIT_CM
// 场景里用于判断像高是否超出可视范围的参考高度。
// 配合 ±120cm 光具座，限制可视高度，避免像高过大导致进场看不清。
const MAX_DISPLAY_IMAGE_Y = 9.2
const EPS = 0.05

const canvasWrapRef = ref<HTMLDivElement | null>(null)
const playing = ref(false)
const showRays = ref(true)
const showScreen = ref(true)
const showLabels = ref(true)
const focusChallenge = ref(false)
const appMode = ref<AppMode>('bench')
const presetKey = ref<PresetKey>('between')
const cameraView = ref<CameraView>('standard')
const showGuideDialog = ref(false)
const experimentLogs = ref<ExperimentLog[]>([])
let experimentLogId = 0

const challengeActive = ref(false)
const selectedChallengeKey = ref<RuleKey | ''>('')
const challengeResultVisible = ref(false)
const challengeExpectedKey = ref<RuleKey>('between')
const lessonStepIndex = ref(0)

const state = reactive({
  // 初始状态改为 f < u < 2f：方便一打开就看到倒立放大的实像。
  u: 14.5,
  f: 10,
  h: 4,
  screenCm: 32.2,
})

let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let controls: OrbitControls
let resizeObserver: ResizeObserver | null = null
let frameId = 0
let lastTime = 0
let demoT = 0
let dynamicUpdateFrame = 0
let isUserOrbiting = false

let rootGroup: THREE.Group
let staticGroup: THREE.Group
let dynamicGroup: THREE.Group
let rayGroup: THREE.Group
let labelGroup: THREE.Group
let screenGroup: THREE.Group
let lensMesh: THREE.Mesh
let glowLight: THREE.PointLight

const RangeRow = defineComponent({
  name: 'RangeRow',
  props: {
    label: { type: String, required: true },
    value: { type: Number, required: true },
    suffix: { type: String, default: '' },
    min: { type: Number, required: true },
    max: { type: Number, required: true },
    step: { type: Number, default: 1 },
  },
  emits: ['update:value'],
  setup(props, { emit }) {
    const valueText = () => `${formatNumber(props.value, props.step < 1 ? 1 : 0)} ${props.suffix}`

    return () =>
      h('div', { class: 'range-row' }, [
        h('div', { class: 'range-head' }, [h('span', { class: 'range-label' }, props.label), h('span', { class: 'range-value' }, valueText())]),
        h(ElSlider, {
          modelValue: props.value,
          min: props.min,
          max: props.max,
          step: props.step,
          showTooltip: false,
          // @ts-ignore
          onInput: (value: number) => emit('update:value', Number(value)),
          // @ts-ignore
          'onUpdate:modelValue': (value: number) => emit('update:value', Number(value)),
        }),
      ])
  },
})

const metrics = computed<OpticalMetrics>(() => {
  const u = state.u
  const f = state.f
  const h = state.h
  const v = Math.abs(u - f) < EPS ? Infinity : (f * u) / (u - f)
  const m = Number.isFinite(v) ? v / u : Infinity
  const real = Number.isFinite(v) && v > 0
  const virtual = Number.isFinite(v) && v < 0
  const imageHeight = Number.isFinite(m) ? -m * h : Infinity

  return { u, f, h, v, m, real, virtual, imageHeight }
})

const imageType = computed(() => {
  const { u, f, v, virtual } = metrics.value

  if (!Number.isFinite(v)) {
    return {
      title: 'u = f：不在有限位置成像',
      short: '不成有限像',
      nature: '出射光线近似平行于主光轴，光屏上不能得到清晰像',
      pos: '像距趋于无穷远',
      screen: '不可承接',
      toast: '物体在焦点处时，出射光线近似平行于主光轴，光屏无法在有限位置接到清晰像。',
    }
  }

  if (virtual) {
    return {
      title: 'u < f：正立放大虚像',
      short: '正立放大虚像',
      nature: '正立、放大、虚像',
      pos: '在物体同侧，由折射光线反向延长线相交得到',
      screen: '不可承接',
      toast: '物体进入焦点以内时，形成正立放大的虚像，光屏不能承接。',
    }
  }

  if (Math.abs(u - 2 * f) < 0.25) {
    return {
      title: 'u = 2f：倒立等大实像',
      short: '倒立等大实像',
      nature: '倒立、等大、实像',
      pos: '在透镜另一侧 2f 处',
      screen: '可承接',
      toast: '物体在 2f 处时，像也在另一侧 2f 处，大小相等但上下倒置。',
    }
  }

  if (u > 2 * f) {
    return {
      title: 'u > 2f：倒立缩小实像',
      short: '倒立缩小实像',
      nature: '倒立、缩小、实像',
      pos: '在透镜另一侧，f 与 2f 之间',
      screen: '可承接',
      toast: '照相机成像常对应这一类：物体较远，形成倒立缩小实像。',
    }
  }

  return {
    title: 'f < u < 2f：倒立放大实像',
    short: '倒立放大实像',
    nature: '倒立、放大、实像',
    pos: '在透镜另一侧，2f 以外',
    screen: '可承接',
    toast: '投影仪成像常对应这一类：物体在 f 与 2f 之间，形成倒立放大实像。',
  }
})

const formulaText = computed(() => {
  const { u, f, v } = metrics.value
  const vText = Number.isFinite(v) ? `1 / (${formatNumber(v)})` : '1 / ∞'
  return `1/${formatNumber(f)} = 1/${formatNumber(u)} + ${vText}`
})

const imageDistanceDerivation = computed(() => {
  const { u, f, v } = metrics.value
  if (Math.abs(u - f) < EPS) {
    return `u=f=${formatNumber(f)}cm，分母 u-f=0，所以 v 趋向无穷远，不在有限位置成清晰像。`
  }

  const numerator = f * u
  const denominator = u - f
  const signText = v > 0 ? 'v 为正，像在透镜另一侧，为实像位置。' : `v 为负，表示虚像在物体同侧；虚像距为 ${formatNumber(Math.abs(v))}cm。`
  return `v = f·u/(u-f) = ${formatNumber(f)}×${formatNumber(u)}/(${formatNumber(u)}-${formatNumber(f)}) = ${formatNumber(numerator)}/${formatNumber(denominator)} = ${formatNumber(v)}cm。${signText}`
})

const magnificationDerivation = computed(() => {
  const { u, v, h, imageHeight } = metrics.value
  if (!Number.isFinite(v)) return `像距趋于无穷远，放大率在理想薄透镜模型中也趋于无穷大，有限光屏上不能接到清晰像。`

  const signedM = -v / u
  const directionText = signedM < 0 ? 'm<0，表示像相对物体倒立。' : 'm>0，表示像相对物体正立。'
  return `m = -v/u = -(${formatNumber(v)})/${formatNumber(u)} = ${formatNumber(signedM, 2)}；h' = m·H = ${formatNumber(signedM, 2)}×${formatNumber(h)} = ${formatNumber(imageHeight)}cm。${directionText}`
})

const currentRuleText = computed(() => {
  const { u, f, v } = metrics.value
  if (!Number.isFinite(v)) return `当前 u≈f，出射光线近似平行于主光轴，不能在有限位置成像。`
  if (u > 2 * f) return `当前 u>${formatNumber(2 * f)}cm，即 u>2f：倒立、缩小、实像，像在 f 与 2f 之间。`
  if (Math.abs(u - 2 * f) < 0.25) return `当前 u≈2f：倒立、等大、实像，像在另一侧 2f 附近。`
  if (u > f && u < 2 * f) return `当前 f<u<2f：倒立、放大、实像，像在 2f 以外。`
  return `当前 u<f：正立、放大、虚像，光屏不能承接。`
})

const appModeLabel = computed(() => {
  const map: Record<AppMode, string> = {
    bench: '光具座实验',
    camera: '照相机模式',
    projector: '投影仪模式',
    magnifier: '放大镜模式',
  }
  return map[appMode.value]
})

const appModeDescription = computed(() => {
  const map: Record<AppMode, string> = {
    bench: '保留标准实验台，适合讲清物距、像距、焦距和三条特殊光线。',
    camera: '对应 u>2f，得到倒立缩小实像，强调照相机把远处景物成在感光元件上。',
    projector: '对应 f<u<2f，得到倒立放大实像，强调投影仪把小物体放大到屏幕上。',
    magnifier: '对应 u<f，得到正立放大虚像，强调放大镜看到的是不能被光屏承接的虚像。',
  }
  return map[appMode.value]
})

const focusDelta = computed(() => {
  const v = metrics.value.v
  if (!focusChallenge.value || !metrics.value.real || !Number.isFinite(v)) return Infinity
  return Math.abs(state.screenCm - v)
})

const focusScore = computed(() => {
  if (!Number.isFinite(focusDelta.value)) return 0
  return Math.round(clamp(100 - focusDelta.value * 8, 0, 100))
})

const focusStatusText = computed(() => {
  if (!focusChallenge.value) return imageType.value.screen
  if (!metrics.value.real || !Number.isFinite(metrics.value.v)) return '无法承接'
  if (focusScore.value >= 92) return '清晰成像'
  if (focusScore.value >= 55) return '略微模糊'
  return '明显模糊'
})

const focusDeltaText = computed(() => {
  if (!Number.isFinite(focusDelta.value)) return '不可计算'
  return `${formatNumber(focusDelta.value)}cm`
})

const ruleRows: Array<{ key: RuleKey; title: string; range: string; result: string }> = [
  { key: 'far', title: 'u > 2f', range: '物体在二倍焦距以外', result: '倒立、缩小、实像，像在 f 与 2f 之间' },
  { key: 'twof', title: 'u = 2f', range: '物体在二倍焦距处', result: '倒立、等大、实像，像在 2f 处' },
  { key: 'between', title: 'f < u < 2f', range: '物体在一倍到二倍焦距之间', result: '倒立、放大、实像，像在 2f 以外' },
  { key: 'focus', title: 'u = f', range: '物体在焦点处', result: '折射光线近似平行，不成有限清晰像' },
  { key: 'inside', title: 'u < f', range: '物体在焦点以内', result: '正立、放大、虚像，光屏不能承接' },
]

const challengeChoices = ruleRows

const currentRuleKey = computed<RuleKey>(() => {
  const { u, f, v } = metrics.value
  if (!Number.isFinite(v) || Math.abs(u - f) < EPS) return 'focus'
  if (Math.abs(u - 2 * f) < 0.25) return 'twof'
  if (u > 2 * f) return 'far'
  if (u > f && u < 2 * f) return 'between'
  return 'inside'
})

const currentRuleRow = computed(() => ruleRows.find(row => row.key === currentRuleKey.value) ?? ruleRows[0])

const challengePassed = computed(() => challengeResultVisible.value && selectedChallengeKey.value === challengeExpectedKey.value)

const challengeFeedback = computed(() => {
  const expected = ruleRows.find(row => row.key === challengeExpectedKey.value)
  if (!expected) return ''
  if (challengePassed.value) return `当前属于 ${expected.title}，判断为：${expected.result}。`
  const selected = ruleRows.find(row => row.key === selectedChallengeKey.value)
  return `你选的是${selected ? `「${selected.title}」` : '未选择'}，正确应为「${expected.title}」：${expected.result}。`
})

const lessonSteps = [
  {
    title: '认识光具座和焦距',
    text: '先固定 f=10cm，观察 O、F、2F 的位置，理解焦距是光心到焦点的距离。',
    preset: 'between' as PresetKey,
    challenge: false,
  },
  {
    title: '观察倒立缩小实像',
    text: '把物体放到 2f 以外，对应照相机成像，重点观察像在 f 与 2f 之间。',
    preset: 'far' as PresetKey,
    challenge: false,
  },
  {
    title: '观察倒立放大实像',
    text: '把物体放到 f 与 2f 之间，对应投影仪成像，重点观察像在 2f 以外。',
    preset: 'between' as PresetKey,
    challenge: false,
  },
  {
    title: '开启光屏寻像',
    text: '开启光屏寻像挑战，拖动光屏寻找清晰像，体验真实实验中的调焦过程。',
    preset: 'between' as PresetKey,
    challenge: true,
  },
  {
    title: '观察正立放大虚像',
    text: '把物体放到焦点以内，对应放大镜成像，重点理解虚像不能被光屏承接。',
    preset: 'inside' as PresetKey,
    challenge: false,
  },
  {
    title: '记录并导出实验日志',
    text: '点击顶部“快照当前实验”记录参数和结论，积累多条后可以导出实验日志图片。',
    preset: 'between' as PresetKey,
    challenge: false,
  },
]

const currentLessonStep = computed(() => lessonSteps[lessonStepIndex.value])

const experimentReport = computed(() => {
  const m = metrics.value
  const vText = Number.isFinite(m.v) ? `${formatNumber(m.v)}cm` : '趋向无穷远'
  const screenText = focusChallenge.value ? `光屏位置 ${formatNumber(state.screenCm)}cm，${focusStatusText.value}` : imageType.value.screen
  return `【实验参数】u=${formatNumber(m.u)}cm，f=${formatNumber(m.f)}cm，H=${formatNumber(m.h)}cm，v=${vText}
【观察现象】${imageType.value.nature}，${imageType.value.pos}。
【光屏结果】${screenText}。
【课堂结论】${currentRuleText.value}`
})

function snapshotExperiment() {
  const m = metrics.value
  const now = new Date()
  const time = now.toLocaleTimeString('zh-CN', { hour12: false })
  const vText = Number.isFinite(m.v) ? `${formatNumber(m.v)}cm` : '趋向无穷远'
  const screen = focusChallenge.value ? `光屏位置 ${formatNumber(state.screenCm)}cm，${focusStatusText.value}` : imageType.value.screen

  experimentLogs.value.unshift({
    id: ++experimentLogId,
    time,
    mode: appModeLabel.value,
    u: m.u,
    f: m.f,
    h: m.h,
    vText,
    nature: imageType.value.nature,
    position: imageType.value.pos,
    screen,
    conclusion: currentRuleText.value,
  })

  if (experimentLogs.value.length > 12) experimentLogs.value.length = 12
}

function clearExperimentLogs() {
  experimentLogs.value = []
}

function exportExperimentLogsAsImage() {
  if (!experimentLogs.value.length) return

  const width = 1500
  const padding = 70
  const contentWidth = width - padding * 2
  const cardRadius = 24
  const cardGap = 30
  const titleHeight = 150
  const cardInnerPadX = 34
  const cardInnerX = padding + cardInnerPadX
  const cardInnerWidth = contentWidth - cardInnerPadX * 2
  const columnGap = 28
  const columnWidth = (cardInnerWidth - columnGap) / 2
  const sectionTitleH = 34
  const sectionLineH = 31
  const conclusionLineH = 34

  type ExportCard = {
    item: ExperimentLog
    index: number
    paramLines: string[]
    observeLines: string[]
    screenLines: string[]
    conclusionLines: string[]
    topTwoColumnH: number
    screenSectionH: number
    conclusionBoxH: number
    height: number
  }

  const measureCanvas = document.createElement('canvas')
  const measureCtx = measureCanvas.getContext('2d')!

  const cards: ExportCard[] = experimentLogs.value.map((item, index) => {
    measureCtx.font = '700 24px Microsoft YaHei, PingFang SC, Arial'
    const paramText = `u=${formatNumber(item.u)}cm    f=${formatNumber(item.f)}cm    H=${formatNumber(item.h)}cm    v=${item.vText}`
    const paramLines = wrapCanvasText(measureCtx, paramText, columnWidth)

    measureCtx.font = '700 23px Microsoft YaHei, PingFang SC, Arial'
    const observeLines = wrapCanvasText(measureCtx, `${item.nature}；${item.position}`, columnWidth)

    // 光屏结果单独占一整行，避免左列高度和右列高度不一致时，课堂结论看起来“飘”在卡片底部。
    const screenLines = wrapCanvasText(measureCtx, item.screen, cardInnerWidth)

    measureCtx.font = '800 24px Microsoft YaHei, PingFang SC, Arial'
    const conclusionLines = wrapCanvasText(measureCtx, item.conclusion, cardInnerWidth - 52)

    const headerH = 76
    const contentTop = 92
    const sectionBottomPad = 10
    const topTwoColumnH = Math.max(
      sectionTitleH + paramLines.length * sectionLineH + sectionBottomPad,
      sectionTitleH + observeLines.length * sectionLineH + sectionBottomPad,
    )
    const screenSectionH = sectionTitleH + screenLines.length * sectionLineH + sectionBottomPad
    const conclusionBoxH = 94 + conclusionLines.length * conclusionLineH
    const bottomPad = 36

    const height = contentTop + topTwoColumnH + 22 + screenSectionH + 22 + conclusionBoxH + bottomPad

    return {
      item,
      index,
      paramLines,
      observeLines,
      screenLines,
      conclusionLines,
      topTwoColumnH,
      screenSectionH,
      conclusionBoxH,
      height: Math.max(height, headerH + 210),
    }
  })

  const totalCardsHeight = cards.reduce((sum, card) => sum + card.height, 0) + Math.max(0, cards.length - 1) * cardGap
  const height = Math.max(900, padding + titleHeight + totalCardsHeight + padding)
  const canvas = document.createElement('canvas')
  canvas.width = width
  canvas.height = height
  const ctx = canvas.getContext('2d')!

  const bg = ctx.createLinearGradient(0, 0, width, height)
  bg.addColorStop(0, '#050d19')
  bg.addColorStop(0.48, '#081d33')
  bg.addColorStop(1, '#050914')
  ctx.fillStyle = bg
  ctx.fillRect(0, 0, width, height)

  drawExportBackground(ctx, width, height)

  const headerX = padding
  const headerY = padding
  const headerW = contentWidth
  const headerH = 118
  const headerGrad = ctx.createLinearGradient(headerX, headerY, headerX + headerW, headerY + headerH)
  headerGrad.addColorStop(0, 'rgba(52,217,255,0.18)')
  headerGrad.addColorStop(0.55, 'rgba(255,209,102,0.10)')
  headerGrad.addColorStop(1, 'rgba(126,232,255,0.08)')
  ctx.fillStyle = headerGrad
  roundRect(ctx, headerX, headerY, headerW, headerH, 28, true, false)
  ctx.strokeStyle = 'rgba(126,232,255,0.34)'
  ctx.lineWidth = 2
  roundRect(ctx, headerX, headerY, headerW, headerH, 28, false, true)

  ctx.fillStyle = '#eaf7ff'
  ctx.font = '900 46px Microsoft YaHei, PingFang SC, Arial'
  ctx.fillText('凸透镜成像 3D互动实验日志', headerX + 34, headerY + 52)

  ctx.fillStyle = 'rgba(234,247,255,0.72)'
  ctx.font = '700 22px Microsoft YaHei, PingFang SC, Arial'
  ctx.fillText(`导出时间：${new Date().toLocaleString('zh-CN', { hour12: false })}`, headerX + 36, headerY + 92)

  drawExportChip(ctx, headerX + headerW - 260, headerY + 32, 200, 48, `共 ${experimentLogs.value.length} 条记录`, '#ffd166')

  let y = padding + titleHeight
  cards.forEach(card => {
    const { item, index } = card
    const cardX = padding
    const cardY = y
    const cardW = contentWidth
    const cardH = card.height
    const innerX = cardX + cardInnerPadX
    const innerW = cardW - cardInnerPadX * 2
    const leftX = innerX
    const rightX = innerX + columnWidth + columnGap

    ctx.fillStyle = 'rgba(5,15,27,0.78)'
    roundRect(ctx, cardX, cardY, cardW, cardH, cardRadius, true, false)
    ctx.strokeStyle = index === 0 ? 'rgba(255,209,102,0.52)' : 'rgba(126,232,255,0.22)'
    ctx.lineWidth = index === 0 ? 2.5 : 1.6
    roundRect(ctx, cardX, cardY, cardW, cardH, cardRadius, false, true)

    const badgeX = cardX + 36
    const badgeY = cardY + 38
    ctx.fillStyle = index === 0 ? '#ffd166' : '#34d9ff'
    ctx.beginPath()
    ctx.arc(badgeX, badgeY, 20, 0, Math.PI * 2)
    ctx.fill()
    ctx.fillStyle = '#06111f'
    ctx.font = '900 22px Microsoft YaHei, PingFang SC, Arial'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.fillText(String(index + 1), badgeX, badgeY + 1)
    ctx.textAlign = 'left'
    ctx.textBaseline = 'alphabetic'

    ctx.fillStyle = '#ffd166'
    ctx.font = '900 27px Microsoft YaHei, PingFang SC, Arial'
    ctx.fillText(item.mode, cardX + 70, cardY + 46)

    ctx.fillStyle = 'rgba(234,247,255,0.68)'
    ctx.font = '700 20px Microsoft YaHei, PingFang SC, Arial'
    ctx.textAlign = 'right'
    ctx.fillText(item.time, cardX + cardW - 34, cardY + 45)
    ctx.textAlign = 'left'

    const contentY = cardY + 92
    drawExportSection(ctx, leftX, contentY, columnWidth, '实验参数', card.paramLines, '#7ee8ff')
    drawExportSection(ctx, rightX, contentY, columnWidth, '观察现象', card.observeLines, '#64f4ac')

    const screenY = contentY + card.topTwoColumnH + 22
    drawExportSection(ctx, innerX, screenY, innerW, '光屏结果', card.screenLines, '#ffdf87')

    const conclusionY = screenY + card.screenSectionH + 22
    ctx.fillStyle = 'rgba(255,209,102,0.08)'
    roundRect(ctx, innerX, conclusionY, innerW, card.conclusionBoxH, 18, true, false)
    ctx.strokeStyle = 'rgba(255,209,102,0.28)'
    ctx.lineWidth = 1.5
    roundRect(ctx, innerX, conclusionY, innerW, card.conclusionBoxH, 18, false, true)

    ctx.fillStyle = '#ffd166'
    ctx.font = '900 21px Microsoft YaHei, PingFang SC, Arial'
    ctx.fillText('课堂结论', innerX + 24, conclusionY + 34)

    ctx.fillStyle = '#fff3c7'
    ctx.font = '800 24px Microsoft YaHei, PingFang SC, Arial'
    drawCanvasLines(ctx, card.conclusionLines, innerX + 24, conclusionY + 78, conclusionLineH)

    y += cardH + cardGap
  })

  const url = canvas.toDataURL('image/png')
  const a = document.createElement('a')
  a.href = url
  a.download = `凸透镜成像实验日志_${Date.now()}.png`
  a.click()
}

function drawExportBackground(ctx: CanvasRenderingContext2D, width: number, height: number) {
  ctx.save()
  ctx.strokeStyle = 'rgba(126,232,255,0.07)'
  ctx.lineWidth = 1
  for (let x = 0; x < width; x += 52) {
    ctx.beginPath()
    ctx.moveTo(x, 0)
    ctx.lineTo(x, height)
    ctx.stroke()
  }
  for (let y = 0; y < height; y += 52) {
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(width, y)
    ctx.stroke()
  }

  const glow1 = ctx.createRadialGradient(240, 130, 0, 240, 130, 360)
  glow1.addColorStop(0, 'rgba(52,217,255,0.20)')
  glow1.addColorStop(1, 'rgba(52,217,255,0)')
  ctx.fillStyle = glow1
  ctx.fillRect(0, 0, width, height)

  const glow2 = ctx.createRadialGradient(width - 180, 260, 0, width - 180, 260, 420)
  glow2.addColorStop(0, 'rgba(255,209,102,0.14)')
  glow2.addColorStop(1, 'rgba(255,209,102,0)')
  ctx.fillStyle = glow2
  ctx.fillRect(0, 0, width, height)
  ctx.restore()
}

function drawExportChip(ctx: CanvasRenderingContext2D, x: number, y: number, width: number, height: number, text: string, color: string) {
  ctx.fillStyle = color === '#ffd166' ? 'rgba(255,209,102,0.14)' : 'rgba(52,217,255,0.12)'
  roundRect(ctx, x, y, width, height, 999, true, false)
  ctx.strokeStyle = color === '#ffd166' ? 'rgba(255,209,102,0.42)' : 'rgba(52,217,255,0.38)'
  ctx.lineWidth = 1.5
  roundRect(ctx, x, y, width, height, 999, false, true)
  ctx.fillStyle = color
  ctx.font = '900 22px Microsoft YaHei, PingFang SC, Arial'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText(text, x + width / 2, y + height / 2 + 1)
  ctx.textAlign = 'left'
  ctx.textBaseline = 'alphabetic'
}

function drawExportSection(ctx: CanvasRenderingContext2D, x: number, y: number, width: number, title: string, lines: string[], color: string) {
  ctx.fillStyle = color
  ctx.font = '900 21px Microsoft YaHei, PingFang SC, Arial'
  ctx.fillText(title, x, y)
  ctx.fillStyle = '#dff8ff'
  ctx.font = '700 24px Microsoft YaHei, PingFang SC, Arial'
  drawCanvasLines(ctx, lines, x, y + 36, 31)
}

function wrapCanvasText(ctx: CanvasRenderingContext2D, text: string, maxWidth: number) {
  const lines: string[] = []
  let line = ''
  for (const char of text) {
    const testLine = line + char
    if (ctx.measureText(testLine).width > maxWidth && line) {
      lines.push(line)
      line = char
    } else {
      line = testLine
    }
  }
  if (line) lines.push(line)
  return lines.length ? lines : ['']
}

function drawCanvasLines(ctx: CanvasRenderingContext2D, lines: string[], x: number, y: number, lineHeight: number) {
  lines.forEach((line, index) => {
    ctx.fillText(line, x, y + index * lineHeight)
  })
}

function drawWrappedText(ctx: CanvasRenderingContext2D, text: string, x: number, y: number, maxWidth: number, lineHeight: number) {
  let line = ''
  let offsetY = 0
  for (const char of text) {
    const testLine = line + char
    if (ctx.measureText(testLine).width > maxWidth && line) {
      ctx.fillText(line, x, y + offsetY)
      line = char
      offsetY += lineHeight
    } else {
      line = testLine
    }
  }
  ctx.fillText(line, x, y + offsetY)
}

function roundRect(
  ctx: CanvasRenderingContext2D,
  x: number,
  y: number,
  width: number,
  height: number,
  radius: number,
  fill: boolean,
  stroke: boolean,
) {
  ctx.beginPath()
  ctx.moveTo(x + radius, y)
  ctx.lineTo(x + width - radius, y)
  ctx.quadraticCurveTo(x + width, y, x + width, y + radius)
  ctx.lineTo(x + width, y + height - radius)
  ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height)
  ctx.lineTo(x + radius, y + height)
  ctx.quadraticCurveTo(x, y + height, x, y + height - radius)
  ctx.lineTo(x, y + radius)
  ctx.quadraticCurveTo(x, y, x + radius, y)
  ctx.closePath()
  if (fill) ctx.fill()
  if (stroke) ctx.stroke()
}

function formatNumber(value: number, digits = 1) {
  if (!Number.isFinite(value)) return '∞'
  return Number(value).toFixed(digits)
}

function isFiniteNumber(value: number) {
  return Number.isFinite(value)
}

function clamp(value: number, min: number, max: number) {
  return Math.max(min, Math.min(max, value))
}

function cmToX(cm: number) {
  return cm * CM_TO_UNIT
}

// 不再把真实像距压缩到光具座边缘：
// v 算出来是多少，就用 cmToX(v) 放到对应位置。
function getImageDisplayX(v: number) {
  return Number.isFinite(v) ? cmToX(v) : v
}

function getImageDisplayY(rawY: number) {
  return rawY
}

function isPointOutsideScene(x: number, y: number) {
  if (!Number.isFinite(x) || !Number.isFinite(y)) return true
  return x < MIN_X || x > MAX_X || y < -MAX_DISPLAY_IMAGE_Y || y > MAX_DISPLAY_IMAGE_Y
}

function isVirtualImageOutOfRange(m: OpticalMetrics, rawX: number, rawY: number) {
  if (!m.virtual) return false
  // 只有无穷远/非有限值才不绘制。
  // 只要算得出有限的 v，就按真实位置绘制虚像蜡烛，不再压缩、不再按 ±60cm/±165cm 隐藏。
  if (!Number.isFinite(m.v) || !Number.isFinite(rawX) || !Number.isFinite(rawY)) return true
  return false
}

function getVirtualOutOfRangeTipPosition(rawX: number) {
  const x = rawX < MIN_X ? MIN_X + 1.25 : MAX_X - 1.25
  return new THREE.Vector3(x, 2.18, 1.2)
}

function getSafeUByImageDistance(rawU: number, f: number) {
  const uMin = 6
  const uMax = 60
  const u = clamp(rawU, uMin, uMax)

  // 保留 u=f 作为一个明确的课堂演示点：出射光线近似平行于主光轴，不在有限位置成像。
  if (Math.abs(u - f) < EPS) return f

  // 限制有限像距：|v| = |f·u/(u-f)| <= MAX_ABS_IMAGE_DISTANCE_CM。
  // 对 u<f：u <= Lf/(L+f)；对 u>f：u >= Lf/(L-f)。
  const limit = MAX_ABS_IMAGE_DISTANCE_CM
  const leftSafeMax = (limit * f) / (limit + f)
  const rightSafeMin = (limit * f) / (limit - f)

  // 物体在焦点以内，但太靠近焦点，会产生特别远的虚像；推回到安全边界。
  if (u < f && u > leftSafeMax) return clamp(leftSafeMax, uMin, uMax)

  // 物体在焦点以外，但太靠近焦点，会产生特别远的实像；推回到安全边界。
  if (u > f && u < rightSafeMin) return clamp(rightSafeMin, uMin, uMax)

  return u
}

function setManualValue(key: keyof typeof state, value: number) {
  if (key === 'screenCm') {
    state.screenCm = clamp(value, 0, BENCH_LIMIT_CM)
    return
  }

  if (key === 'u') {
    state.u = getSafeUByImageDistance(value, state.f)
  } else if (key === 'f') {
    state.f = value
    state.u = getSafeUByImageDistance(state.u, state.f)
  } else {
    state[key] = value
  }

  presetKey.value = ''
  playing.value = false
}

function startChallenge() {
  const keys: RuleKey[] = ['far', 'twof', 'between', 'focus', 'inside']
  const nextKey = keys[Math.floor(Math.random() * keys.length)]
  challengeActive.value = true
  selectedChallengeKey.value = ''
  challengeResultVisible.value = false
  challengeExpectedKey.value = nextKey!
  focusChallenge.value = false
  appMode.value = 'bench'
  setPreset(nextKey!)
}

function selectChallengeChoice(key: RuleKey) {
  selectedChallengeKey.value = key
  challengeResultVisible.value = false
}

function verifyChallengeAnswer() {
  if (!selectedChallengeKey.value) return
  challengeExpectedKey.value = currentRuleKey.value
  challengeResultVisible.value = true
}

function exitChallenge() {
  challengeActive.value = false
  selectedChallengeKey.value = ''
  challengeResultVisible.value = false
}

function applyCurrentLessonStep() {
  const step = currentLessonStep.value!
  state.f = 10
  state.h = 4
  appMode.value = 'bench'
  setPreset(step!.preset)

  if (step.preset === 'far') appMode.value = 'camera'
  if (step.preset === 'between' && lessonStepIndex.value === 2) appMode.value = 'projector'
  if (step.preset === 'inside') appMode.value = 'magnifier'

  focusChallenge.value = false
  if (step.challenge) {
    focusChallenge.value = true
    if (metrics.value.real && Number.isFinite(metrics.value.v)) {
      state.screenCm = clamp(metrics.value.v + 16, 0, BENCH_LIMIT_CM)
    }
  }

  updateDynamicScene()
}

function nextLessonStep() {
  if (lessonStepIndex.value >= lessonSteps.length - 1) return
  lessonStepIndex.value += 1
  applyCurrentLessonStep()
}

function prevLessonStep() {
  if (lessonStepIndex.value <= 0) return
  lessonStepIndex.value -= 1
  applyCurrentLessonStep()
}

function setPreset(key: PresetKey) {
  if (!key) return
  const map: Record<Exclude<PresetKey, ''>, number> = {
    far: state.f * 3,
    twof: state.f * 2,
    between: state.f * 1.45,
    focus: state.f,
    inside: state.f * 0.62,
    near: state.f * 1.08,
  }

  // u=f 保留为“焦点处不成有限清晰像”的特殊演示；其他预设都走像距保护。
  state.u = key === 'focus' ? state.f : getSafeUByImageDistance(map[key], state.f)

  presetKey.value = key
  playing.value = false
}

function setAppMode(mode: AppMode) {
  appMode.value = mode
  focusChallenge.value = false
  showScreen.value = true

  if (mode === 'camera') setPreset('far')
  if (mode === 'projector') setPreset('between')
  if (mode === 'magnifier') setPreset('inside')
  if (mode === 'bench') presetKey.value = ''

  updateDynamicScene()
}

function toggleFocusChallenge() {
  focusChallenge.value = !focusChallenge.value
  showScreen.value = true

  if (focusChallenge.value) {
    // 光屏寻像只对实像有物理意义；如果当前是虚像或 u=f，先切到 f<u<2f 的实像状态。
    if (!metrics.value.real || !Number.isFinite(metrics.value.v)) {
      setPreset('between')
      appMode.value = 'bench'
    }

    // 开启挑战时故意把光屏放偏一点，让学生拖动寻找清晰像。
    if (metrics.value.real && Number.isFinite(metrics.value.v)) {
      state.screenCm = clamp(metrics.value.v + 16, 0, BENCH_LIMIT_CM)
    } else {
      state.screenCm = 20
    }
  } else if (metrics.value.real && Number.isFinite(metrics.value.v)) {
    state.screenCm = clamp(metrics.value.v, 0, BENCH_LIMIT_CM)
  }

  updateDynamicScene()
}

function snapScreenToImage() {
  if (!metrics.value.real || !Number.isFinite(metrics.value.v)) return
  state.screenCm = clamp(metrics.value.v, 0, BENCH_LIMIT_CM)
  showScreen.value = true
  updateDynamicScene()
}

function togglePlay() {
  playing.value = !playing.value
}

function toggleLayer(layer: 'rays' | 'screen' | 'labels') {
  if (layer === 'rays') showRays.value = !showRays.value
  if (layer === 'screen') showScreen.value = !showScreen.value
  if (layer === 'labels') showLabels.value = !showLabels.value
  updateDynamicScene()
}

function resetExperiment() {
  // 重置后也回到 f < u < 2f，而不是 u > 2f。
  state.u = 14.5
  state.f = 10
  state.h = 4
  presetKey.value = 'between'
  appMode.value = 'bench'
  focusChallenge.value = false
  challengeActive.value = false
  selectedChallengeKey.value = ''
  challengeResultVisible.value = false
  lessonStepIndex.value = 0
  state.screenCm = metrics.value.real && Number.isFinite(metrics.value.v) ? clamp(metrics.value.v, 0, BENCH_LIMIT_CM) : 32.2
  playing.value = false
  demoT = 0
  setCameraView('standard')
}

function initThree() {
  if (!canvasWrapRef.value) return

  scene = new THREE.Scene()
  scene.fog = new THREE.Fog(0x07111f, 9, 30)

  const rect = canvasWrapRef.value.getBoundingClientRect()
  const width = Math.max(1, rect.width)
  const height = Math.max(1, rect.height)

  camera = new THREE.PerspectiveCamera(40, width / height, 0.1, 90)
  // 初始镜头再拉近：默认进来直接看清蜡烛、透镜、光屏和成像，不再像远景总览。
  camera.position.set(4.8, 3.7, 9.2)

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 1.55))
  renderer.setSize(width, height, false)
  renderer.setClearColor(0x07111f, 0)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  canvasWrapRef.value.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.target.set(0, 0.55, 0)
  controls.minDistance = 2.2
  controls.maxDistance = 24
  controls.enablePan = false
  controls.addEventListener('start', () => {
    isUserOrbiting = true
  })
  controls.addEventListener('end', () => {
    isUserOrbiting = false
  })

  const ambient = new THREE.AmbientLight(0xffffff, 1.35)
  scene.add(ambient)

  const key = new THREE.DirectionalLight(0xffffff, 1.8)
  key.position.set(4, 6, 5)
  key.castShadow = true
  key.shadow.mapSize.set(1024, 1024)
  scene.add(key)

  glowLight = new THREE.PointLight(0x34d9ff, 1.2, 7)
  glowLight.position.set(0, 1.4, 0)
  scene.add(glowLight)

  rootGroup = new THREE.Group()
  staticGroup = new THREE.Group()
  dynamicGroup = new THREE.Group()
  rayGroup = new THREE.Group()
  labelGroup = new THREE.Group()
  screenGroup = new THREE.Group()
  scene.add(rootGroup)
  rootGroup.add(staticGroup, dynamicGroup, rayGroup, labelGroup, screenGroup)

  createStaticScene()
  updateDynamicScene()
  setCameraView('standard')

  resizeObserver = new ResizeObserver(() => requestAnimationFrame(resizeRenderer))
  resizeObserver.observe(canvasWrapRef.value)
  window.addEventListener('resize', resizeRenderer, { passive: true })

  lastTime = performance.now()
  animate(lastTime)
}

function createStaticScene() {
  clearGroup(staticGroup)

  const floor = new THREE.Mesh(
    new THREE.PlaneGeometry(MAX_X - MIN_X + 1.4, 5.2),
    new THREE.MeshStandardMaterial({
      color: 0x071827,
      roughness: 0.72,
      metalness: 0.18,
      transparent: true,
      opacity: 0.72,
      emissive: 0x041927,
      emissiveIntensity: 0.32,
    }),
  )
  floor.rotation.x = -Math.PI / 2
  floor.position.y = -0.025
  floor.receiveShadow = true
  staticGroup.add(floor)

  createTechBenchDecorations()

  addTube(staticGroup, new THREE.Vector3(MIN_X, AXIS_Y, BENCH_Z), new THREE.Vector3(MAX_X, AXIS_Y, BENCH_Z), 0.012, 0xeaf7ff, 0.92)
  addTube(staticGroup, new THREE.Vector3(MIN_X, -0.1, -0.45), new THREE.Vector3(MAX_X, -0.1, -0.45), 0.025, 0x5fb8d8, 0.55)
  addTube(staticGroup, new THREE.Vector3(MIN_X, -0.1, 0.45), new THREE.Vector3(MAX_X, -0.1, 0.45), 0.025, 0x5fb8d8, 0.55)

  createLens()
  createTickMarks()
}

function createTechBenchDecorations() {
  // 不再使用 GridHelper 网格平面，改成几条发光边框和节点，保留科技感但不干扰光路教学。
  const y = -0.012
  const zFront = 1.96
  const zBack = -1.96
  addTube(staticGroup, new THREE.Vector3(MIN_X, y, zFront), new THREE.Vector3(MAX_X, y, zFront), 0.012, 0x34d9ff, 0.38)
  addTube(staticGroup, new THREE.Vector3(MIN_X, y, zBack), new THREE.Vector3(MAX_X, y, zBack), 0.012, 0x34d9ff, 0.28)
  addTube(staticGroup, new THREE.Vector3(MIN_X, y, zBack), new THREE.Vector3(MIN_X, y, zFront), 0.012, 0x7ee8ff, 0.26)
  addTube(staticGroup, new THREE.Vector3(MAX_X, y, zBack), new THREE.Vector3(MAX_X, y, zFront), 0.012, 0x7ee8ff, 0.26)

  for (let cm = -120; cm <= 120; cm += 30) {
    const x = cmToX(cm)
    const node = new THREE.Mesh(
      new THREE.CylinderGeometry(0.05, 0.05, 0.01, 24),
      new THREE.MeshBasicMaterial({ color: 0x34d9ff, transparent: true, opacity: 0.72, blending: THREE.AdditiveBlending }),
    )
    node.rotation.x = Math.PI / 2
    node.position.set(x, y + 0.006, zFront)
    staticGroup.add(node)
  }
}

function createLens() {
  const lensGeo = new THREE.SphereGeometry(1, 48, 32)
  const lensMat = new THREE.MeshPhysicalMaterial({
    color: 0x8feeff,
    transmission: 0.5,
    transparent: true,
    opacity: 0.44,
    roughness: 0.06,
    metalness: 0,
    ior: 1.45,
    thickness: 0.8,
    side: THREE.DoubleSide,
  })

  lensMesh = new THREE.Mesh(lensGeo, lensMat)
  lensMesh.scale.set(0.13, 1.55, 0.86)
  lensMesh.position.set(LENS_X, 0.65, BENCH_Z)
  lensMesh.castShadow = true
  lensMesh.receiveShadow = true
  staticGroup.add(lensMesh)

  const ring = new THREE.Mesh(
    new THREE.TorusGeometry(0.88, 0.012, 12, 96),
    new THREE.MeshBasicMaterial({ color: 0x7ee8ff, transparent: true, opacity: 0.82 }),
  )
  ring.rotation.y = Math.PI / 2
  ring.position.copy(lensMesh.position)
  staticGroup.add(ring)

  labelGroup.add(makeTextSprite('凸透镜', '#7ee8ff', new THREE.Vector3(0, 2.62, 0.58), 0.82, 40))
}

function createTickMarks() {
  for (let cm = -120; cm <= 120; cm += 10) {
    const x = cmToX(cm)
    const isMajor = cm % 60 === 0
    addTube(
      staticGroup,
      new THREE.Vector3(x, -0.06, isMajor ? -0.16 : -0.1),
      new THREE.Vector3(x, -0.06, isMajor ? 0.16 : 0.1),
      isMajor ? 0.007 : 0.005,
      0xdff8ff,
      isMajor ? 0.58 : 0.32,
    )
  }
}

function updateDynamicScene() {
  if (!dynamicGroup || !rayGroup || !labelGroup || !screenGroup) return

  clearGroup(dynamicGroup)
  clearGroup(rayGroup)
  clearGroup(screenGroup)

  // 标签组里静态刻度和凸透镜标签在 createStaticScene 生成；这里重建动态标签时保留简单做法：全部重建。
  clearGroup(labelGroup)
  labelGroup.visible = showLabels.value
  createTickLabelsAndLensLabel()

  const m = metrics.value
  const objectX = -cmToX(m.u)
  const objectTopY = m.h * HEIGHT_TO_UNIT
  const rawImageX = cmToX(m.v)
  const rawImageTopY = m.imageHeight * HEIGHT_TO_UNIT

  // 实像 / 虚像都按真实像距位置绘制，不再把 v=135cm 画到 60cm 边缘。
  const imageX = getImageDisplayX(m.v)
  const imageTopY = getImageDisplayY(rawImageTopY)
  const imageOutOfRange = isPointOutsideScene(imageX, imageTopY)
  const virtualOutOfRange = isVirtualImageOutOfRange(m, rawImageX, rawImageTopY)
  const compressedImage = false

  createFocusMarkers(m.f)
  createObjectArrow(objectX, objectTopY)
  createApplicationModeModel(m, objectX, imageX)
  createSceneAnnotations(m, objectX, objectTopY, imageX, imageTopY, compressedImage, virtualOutOfRange, rawImageX)

  if (Number.isFinite(m.v)) {
    if (!imageOutOfRange && !virtualOutOfRange) {
      createImageArrow(imageX, imageTopY, m.real, compressedImage)
    } else if (m.real && imageOutOfRange) {
      labelGroup.add(
        makeTextSprite(
          `实像真实位置已超过当前光具座\nv = ${formatNumber(m.v)}cm，未压缩显示\n请调整参数`,
          '#34d9ff',
          new THREE.Vector3(MAX_X - 2.0, 2.7, 1.18),
          0.66,
          38,
        ),
      )
    }
    if (showScreen.value && m.real) {
      const screenCm = focusChallenge.value ? state.screenCm : m.v
      const screenX = cmToX(screenCm)
      if (screenX >= MIN_X && screenX <= MAX_X) {
        createScreen(screenX, compressedImage)
        if (focusChallenge.value) createScreenProjection(screenX, imageTopY, focusScore.value)
      }
    }
  } else {
    labelGroup.add(makeTextSprite('折射光线近似平行\n像距趋于无穷远', '#ffdf87', new THREE.Vector3(2.35, 1.95, 0.78), 0.92, 40))
  }

  rayGroup.visible = showRays.value
  if (showRays.value) createRays(m, objectX, objectTopY, imageX, imageTopY, virtualOutOfRange)
}

function createTickLabelsAndLensLabel() {
  // 静态标签只保留少量刻度，避免和 F / 2F / 焦距线段文字堆在一起。
  labelGroup.add(makeTextSprite('凸透镜', '#7ee8ff', new THREE.Vector3(0, 2.7, 1.08), 0.86, 42))
  for (let cm = -120; cm <= 120; cm += 30) {
    if (cm === 0) continue
    const x = cmToX(cm)
    labelGroup.add(makeTextSprite(`${Math.abs(cm)}cm`, 'rgba(222,245,255,0.86)', new THREE.Vector3(x, -0.22, -0.92), 0.42, 28))
  }
}

function createFocusMarkers(f: number) {
  const items = [
    { x: -cmToX(f), label: 'F', color: '#ffd166' },
    { x: cmToX(f), label: 'F', color: '#ffd166' },
    { x: -cmToX(2 * f), label: '2F', color: '#ff9f43' },
    { x: cmToX(2 * f), label: '2F', color: '#ff9f43' },
  ]

  items.forEach(item => {
    if (item.x < MIN_X || item.x > MAX_X) return
    const sphere = new THREE.Mesh(new THREE.SphereGeometry(0.055, 16, 12), new THREE.MeshBasicMaterial({ color: item.color }))
    sphere.position.set(item.x, 0, 0)
    dynamicGroup.add(sphere)
    addTube(dynamicGroup, new THREE.Vector3(item.x, -0.18, 0), new THREE.Vector3(item.x, 0.18, 0), 0.006, new THREE.Color(item.color).getHex(), 0.7)
    // 焦点文字直接压在对应焦点球体的正上方：x / z 与球体完全一致，只改 y。
    // 不再为了避开 mesh 改 z，文字 Sprite 已 depthTest=false，不会被遮挡。
    const labelY = item.label === 'F' ? 0.58 : 0.82
    labelGroup.add(makeTextSprite(item.label, item.color, new THREE.Vector3(item.x, labelY, 0), 0.82, 46))
  })
}

function createObjectArrow(x: number, topY: number) {
  // 教材里凸透镜成像常用“蜡烛”做物体，比单纯箭头更接近真实实验。
  createCandleModel(dynamicGroup, {
    x,
    topY,
    bodyColor: 0xfff1c2,
    bodyAccentColor: 0xffd166,
    flameColor: 0xff9f43,
    glowColor: 0xffd166,
    opacity: 1,
    radiusScale: 1,
    withLight: true,
  })
  // 物距已经由下方尺寸线标出，这里只保留“蜡烛”身份标签。
  // 标签的 x 坐标严格使用蜡烛 x，y 坐标按蜡烛真实视觉顶部计算，保证文字在蜡烛正上方。
  labelGroup.add(makeTextSprite('发光蜡烛', '#ffd166', getCandleLabelPosition(x, topY, 0.82), 0.78, 44))
}

function getCandleLabelY(topY: number, offset = 0.78) {
  // 文字要在“蜡烛模型视觉外轮廓”的正上方。
  // 由于蜡烛模型可能正立，也可能倒立，不能简单用 topY。
  // 正立物体：火焰在上方，标签放在火焰上方。
  // 倒立实像：最高点反而接近主光轴的底座，标签放在主光轴上方一点，避免跑到倒像下方。
  const absH = Math.max(Math.abs(topY), 0.22)
  const bodyHeight = Math.max(0.16, absH * 0.68)
  const flameHeight = Math.max(0.12, absH * 0.22)
  const wickHeight = Math.max(0.035, absH * 0.06)
  const uprightTop = bodyHeight + wickHeight + flameHeight
  const visualTop = topY >= 0 ? uprightTop : 0.08
  return visualTop + offset
}

function getCandleLabelPosition(x: number, topY: number, offset = 0.78) {
  // 正上方：x 与蜡烛完全一致，z 与蜡烛完全一致，只改变 y。
  return new THREE.Vector3(x, getCandleLabelY(topY, offset), 0)
}

function createImageArrow(x: number, topY: number, real: boolean, compressed = false) {
  if (x < MIN_X || x > MAX_X) return

  // 实像 / 虚像都继续用“蜡烛像”，只通过透明度、标签和辅助线区分。
  // 这样学生看到的是同一种物体的成像变化，而不是突然变成另一个图形。
  createCandleModel(dynamicGroup, {
    x,
    topY,
    bodyColor: 0xfff1c2,
    bodyAccentColor: real ? 0x34d9ff : 0xff6b7a,
    flameColor: 0xff9f43,
    glowColor: real ? 0x34d9ff : 0xff6b7a,
    opacity: real ? 0.78 : 0.48,
    radiusScale: 1,
    withLight: false,
    ghost: !real,
  })

  if (real) {
    createProjectedImagePatch(x, topY)
  } else {
    createVirtualImageHalo(x, topY)
  }

  // 像的文字也按蜡烛像的外轮廓放置，不再用固定偏移，避免看起来不在像的正上方。
  labelGroup.add(makeTextSprite(real ? '倒立实像' : '正立虚像', real ? '#34d9ff' : '#ff6b7a', getCandleLabelPosition(x, topY, 0.78), 0.72, 40))
}

type CandleOptions = {
  x: number
  topY: number
  bodyColor: number
  bodyAccentColor: number
  flameColor: number
  glowColor: number
  opacity?: number
  radiusScale?: number
  withLight?: boolean
  ghost?: boolean
}

function createCandleModel(group: THREE.Group, options: CandleOptions) {
  const sign = options.topY >= 0 ? 1 : -1
  const absH = Math.max(Math.abs(options.topY), 0.22)
  const opacity = options.opacity ?? 1
  const radiusScale = options.radiusScale ?? 1
  const bodyHeight = Math.max(0.16, absH * 0.68)
  const flameHeight = Math.max(0.12, absH * 0.22)
  const wickHeight = Math.max(0.035, absH * 0.06)
  const radius = Math.max(0.045, Math.min(0.085, absH * 0.11)) * radiusScale
  const transparent = opacity < 1

  const waxMat = new THREE.MeshStandardMaterial({
    color: options.bodyColor,
    roughness: 0.55,
    metalness: 0.02,
    transparent,
    opacity,
    emissive: options.ghost ? options.bodyAccentColor : 0x000000,
    emissiveIntensity: options.ghost ? 0.18 : 0,
  })

  const body = new THREE.Mesh(new THREE.CylinderGeometry(radius * 0.92, radius, bodyHeight, 28), waxMat)
  body.position.set(options.x, sign * bodyHeight * 0.5, 0)
  body.castShadow = true
  body.receiveShadow = true
  group.add(body)

  const rimMat = new THREE.MeshBasicMaterial({ color: options.bodyAccentColor, transparent: true, opacity: Math.min(0.9, opacity + 0.12) })
  const topRim = new THREE.Mesh(new THREE.TorusGeometry(radius * 0.96, radius * 0.08, 8, 32), rimMat)
  topRim.rotation.x = Math.PI / 2
  topRim.position.set(options.x, sign * bodyHeight, 0)
  group.add(topRim)

  const base = new THREE.Mesh(
    new THREE.CylinderGeometry(radius * 1.55, radius * 1.7, 0.045, 28),
    new THREE.MeshStandardMaterial({
      color: options.bodyAccentColor,
      roughness: 0.45,
      metalness: 0.18,
      transparent,
      opacity: Math.min(0.82, opacity),
    }),
  )
  base.position.set(options.x, sign * 0.022, 0)
  group.add(base)

  const wickStart = new THREE.Vector3(options.x, sign * bodyHeight, 0)
  const wickEnd = new THREE.Vector3(options.x, sign * (bodyHeight + wickHeight), 0)
  addTube(group, wickStart, wickEnd, radius * 0.11, 0x1b1b1b, opacity)

  const flameMat = new THREE.MeshBasicMaterial({
    color: options.flameColor,
    transparent: true,
    opacity: Math.min(0.95, opacity + 0.08),
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  })
  const flame = new THREE.Mesh(new THREE.ConeGeometry(radius * 1.1, flameHeight, 24), flameMat)
  flame.position.set(options.x, sign * (bodyHeight + wickHeight + flameHeight * 0.5), 0)
  if (sign < 0) flame.rotation.z = Math.PI
  group.add(flame)

  const innerFlame = new THREE.Mesh(
    new THREE.ConeGeometry(radius * 0.56, flameHeight * 0.62, 20),
    new THREE.MeshBasicMaterial({
      color: 0xfff3a0,
      transparent: true,
      opacity: Math.min(0.9, opacity),
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  innerFlame.position.set(options.x, sign * (bodyHeight + wickHeight + flameHeight * 0.46), 0.006)
  if (sign < 0) innerFlame.rotation.z = Math.PI
  group.add(innerFlame)

  const glow = new THREE.Mesh(
    new THREE.SphereGeometry(flameHeight * 0.75, 20, 14),
    new THREE.MeshBasicMaterial({
      color: options.glowColor,
      transparent: true,
      opacity: options.ghost ? 0.12 : 0.2,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
    }),
  )
  glow.scale.set(0.75, 1.15, 0.75)
  glow.position.set(options.x, sign * (bodyHeight + wickHeight + flameHeight * 0.5), 0)
  group.add(glow)

  if (options.withLight) {
    const flameLight = new THREE.PointLight(options.glowColor, 0.8, 2.1)
    flameLight.position.set(options.x, sign * (bodyHeight + wickHeight + flameHeight * 0.5), 0.08)
    group.add(flameLight)
  }
}

function createProjectedImagePatch(x: number, topY: number) {
  const height = Math.max(0.28, Math.abs(topY) + 0.28)
  const patch = new THREE.Mesh(
    new THREE.PlaneGeometry(0.52, height),
    new THREE.MeshBasicMaterial({
      color: 0x34d9ff,
      transparent: true,
      opacity: 0.18,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
      side: THREE.DoubleSide,
    }),
  )
  patch.rotation.y = Math.PI / 2
  patch.position.set(x + 0.012, topY * 0.5, -0.018)
  dynamicGroup.add(patch)
}

function createVirtualImageHalo(x: number, topY: number) {
  const height = Math.max(0.32, Math.abs(topY) + 0.28)
  const halo = new THREE.Mesh(
    new THREE.PlaneGeometry(0.62, height),
    new THREE.MeshBasicMaterial({
      color: 0xff6b7a,
      transparent: true,
      opacity: 0.13,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
      side: THREE.DoubleSide,
    }),
  )
  halo.rotation.y = Math.PI / 2
  halo.position.set(x - 0.012, topY * 0.5, 0.028)
  dynamicGroup.add(halo)
}

function createSceneAnnotations(
  m: OpticalMetrics,
  objectX: number,
  objectY: number,
  imageX: number,
  imageY: number,
  compressedImage = false,
  virtualOutOfRange = false,
  rawImageX = imageX,
) {
  addDimensionSegment(new THREE.Vector3(objectX, -1.02, 1.08), new THREE.Vector3(0, -1.02, 1.08), '#ffd166', `物距 u = ${formatNumber(m.u)}cm`, 0.06)

  if (Number.isFinite(m.v)) {
    const label = m.real ? `像距 v = ${formatNumber(m.v)}cm` : `虚像距 |v| = ${formatNumber(Math.abs(m.v))}cm\n(v = ${formatNumber(m.v)}cm)`

    if (virtualOutOfRange) {
      // 只有 v 趋向无穷远 / 非有限值时才走这里。
      // 有限的大虚像（例如 v=-323cm）会在真实位置直接绘制出来。
      labelGroup.add(
        makeTextSprite(
          `虚像趋向无穷远
v = ${formatNumber(m.v)}cm
当前无法在有限光具座内绘制`,
          '#ff6b7a',
          getVirtualOutOfRangeTipPosition(rawImageX),
          0.62,
          36,
        ),
      )
    } else {
      addDimensionSegment(new THREE.Vector3(0, -1.24, 1.08), new THREE.Vector3(imageX, -1.24, 1.08), m.real ? '#34d9ff' : '#ff6b7a', label, 0.06)
      if (m.virtual && Math.abs(m.v) > BENCH_LIMIT_CM * 0.65) {
        labelGroup.add(
          makeTextSprite(
            `虚像按真实位置绘制
距透镜 ${formatNumber(Math.abs(m.v))}cm，可切换俯视或稍微拉远观察`,
            '#ff6b7a',
            new THREE.Vector3(imageX, Math.min(Math.max(imageY + 1.6, 2.0), MAX_DISPLAY_IMAGE_Y - 1.0), 1.18),
            0.62,
            36,
          ),
        )
      }
    }

    if (compressedImage) {
      labelGroup.add(makeTextSprite('真实像距或像高较大\n光屏边缘提示显示', '#34d9ff', new THREE.Vector3(imageX, 1.98, 1.18), 0.54, 32))
      addTube(dynamicGroup, new THREE.Vector3(imageX - 0.18, -0.2, -0.34), new THREE.Vector3(imageX + 0.18, -0.2, -0.34), 0.01, 0x34d9ff, 0.92)
      addTube(dynamicGroup, new THREE.Vector3(imageX - 0.08, -0.28, -0.34), new THREE.Vector3(imageX + 0.28, -0.28, -0.34), 0.01, 0x34d9ff, 0.72)
    }
  }

  // 多余的焦距 f 文字和短竖线已去掉，只保留 F / 2F 焦点标识。

  // 光心 O 放在透镜中心右侧：靠近光心，但不压进透明透镜里。
  // z 不偏移，避免斜视角下投影跑偏。
  labelGroup.add(makeTextSprite('光心 O', '#ffffff', new THREE.Vector3(0.58, 0.72, 0), 0.72, 42))
  labelGroup.add(makeTextSprite('主光轴', '#eaf7ff', new THREE.Vector3(MAX_X - 0.55, 0.48, 1.16), 0.68, 40))

  if (showRays.value && Number.isFinite(m.v)) {
    createAngleMarker(objectX, objectY, imageX, imageY, m.real, virtualOutOfRange)
  }
}

function createFocalDistanceSegments(_f: number) {
  // 已取消独立“焦距 f”短竖线，避免和 F / 2F、物距、像距标签混在一起。
  // 焦距位置直接通过 F、2F 焦点球体和上方文字表达。
}

function addDimensionSegment(start: THREE.Vector3, end: THREE.Vector3, color: string, label: string, labelYOffset = 0.14) {
  const hex = new THREE.Color(color).getHex()
  addTube(dynamicGroup, start, end, 0.009, hex, 0.88)
  addTube(dynamicGroup, new THREE.Vector3(start.x, start.y - 0.07, start.z), new THREE.Vector3(start.x, start.y + 0.07, start.z), 0.006, hex, 0.8)
  addTube(dynamicGroup, new THREE.Vector3(end.x, end.y - 0.07, end.z), new THREE.Vector3(end.x, end.y + 0.07, end.z), 0.006, hex, 0.8)
  addConeArrow(dynamicGroup, start, end, hex, 0.035, 0.09)
  addConeArrow(dynamicGroup, end, start, hex, 0.035, 0.09)
  const mid = new THREE.Vector3().addVectors(start, end).multiplyScalar(0.5)
  labelGroup.add(makeTextSprite(label, color, new THREE.Vector3(mid.x, mid.y - labelYOffset, mid.z), 0.58, 34))
}

function createAngleMarker(objectX: number, objectY: number, imageX: number, imageY: number, real: boolean, virtualOutOfRange = false) {
  // 所有角度标注都放在真实光路所在的 x-y 平面上（z=0），不再用 z 前移。
  // 文字 Sprite 不参与深度测试，所以不需要为了“看得见”去改 z。
  if (!Number.isFinite(imageX)) return

  const incidentLabelX = (objectX + 0) * 0.5
  labelGroup.add(makeTextSprite('入射光线', '#ffd166', new THREE.Vector3(incidentLabelX, objectY + 0.34, 0), 0.62, 36))

  const center = new THREE.Vector3(0, objectY, 0)
  const angleRad = Math.atan2(Math.abs(imageY - objectY), Math.max(0.001, Math.abs(imageX)))
  const signedAngle = imageY >= objectY ? angleRad : -angleRad
  const angleDeg = (angleRad * 180) / Math.PI

  addArcMarker(center, 0.42, 0, signedAngle, real ? 0x34d9ff : 0xff6b7a)
  labelGroup.add(
    makeTextSprite(
      `∠出射光线与主光轴≈${formatNumber(angleDeg, 0)}°`,
      real ? '#34d9ff' : '#ff6b7a',
      getAngleLabelPosition(center, 0.82, signedAngle),
      0.62,
      34,
    ),
  )

  if (!real && !virtualOutOfRange) {
    labelGroup.add(makeTextSprite('反向延长线交点\n形成虚像', '#ff6b7a', new THREE.Vector3(imageX, getCandleLabelY(imageY, 1.18), 0), 0.68, 38))
  }
}

function getAngleLabelPosition(center: THREE.Vector3, radius: number, endAngle: number) {
  const midAngle = endAngle * 0.5
  return new THREE.Vector3(center.x + Math.cos(midAngle) * radius, center.y + Math.sin(midAngle) * radius, center.z)
}

function addArcMarker(center: THREE.Vector3, radius: number, startAngle: number, endAngle: number, color: number) {
  const points: THREE.Vector3[] = []
  const steps = 30
  for (let i = 0; i <= steps; i++) {
    const t = i / steps
    const a = startAngle + (endAngle - startAngle) * t
    points.push(new THREE.Vector3(center.x + Math.cos(a) * radius, center.y + Math.sin(a) * radius, center.z))
  }
  const geo = new THREE.BufferGeometry().setFromPoints(points)
  const mat = new THREE.LineBasicMaterial({ color, transparent: true, opacity: 0.96, linewidth: 2 })
  const arc = new THREE.Line(geo, mat)
  arc.renderOrder = 30
  dynamicGroup.add(arc)

  // 弧线两端加两条短边，明确这是“∠”的夹角范围。
  const a0 = new THREE.Vector3(center.x + Math.cos(startAngle) * radius, center.y + Math.sin(startAngle) * radius, center.z)
  const a1 = new THREE.Vector3(center.x + Math.cos(endAngle) * radius, center.y + Math.sin(endAngle) * radius, center.z)
  addTube(dynamicGroup, center, a0, 0.006, color, 0.78)
  addTube(dynamicGroup, center, a1, 0.006, color, 0.78)
}

function createApplicationModeModel(m: OpticalMetrics, objectX: number, imageX: number) {
  if (appMode.value === 'bench') return

  if (appMode.value === 'camera') {
    createCameraModel(new THREE.Vector3(Math.min(MAX_X - 1.25, imageX + 0.75), 0.33, -1.34))
    labelGroup.add(
      makeTextSprite('照相机：接收倒立缩小实像', '#7ee8ff', new THREE.Vector3(Math.min(MAX_X - 1.25, imageX + 0.75), 1.35, -1.34), 0.58, 34),
    )
  }

  if (appMode.value === 'projector') {
    createProjectorModel(new THREE.Vector3(objectX - 0.55, 0.28, -1.34))
    labelGroup.add(makeTextSprite('投影仪：小物体放大到屏幕', '#ffd166', new THREE.Vector3(objectX - 0.55, 1.25, -1.34), 0.58, 34))
  }

  if (appMode.value === 'magnifier') {
    createMagnifierModel(new THREE.Vector3(0.62, 0.72, -1.3))
    labelGroup.add(makeTextSprite('放大镜：看到正立放大虚像', '#ff6b7a', new THREE.Vector3(0.62, 1.8, -1.3), 0.58, 34))
  }
}

function createCameraModel(pos: THREE.Vector3) {
  const bodyMat = new THREE.MeshStandardMaterial({
    color: 0x132b46,
    roughness: 0.42,
    metalness: 0.34,
    emissive: 0x071827,
    emissiveIntensity: 0.38,
  })

  const body = new THREE.Mesh(new THREE.BoxGeometry(0.92, 0.52, 0.42), bodyMat)
  body.position.copy(pos)
  body.castShadow = true
  dynamicGroup.add(body)

  const prism = new THREE.Mesh(new THREE.BoxGeometry(0.38, 0.2, 0.34), bodyMat)
  prism.position.set(pos.x - 0.08, pos.y + 0.34, pos.z)
  prism.rotation.z = 0.06
  dynamicGroup.add(prism)

  const grip = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.66, 0.4), bodyMat)
  grip.position.set(pos.x + 0.43, pos.y - 0.04, pos.z)
  dynamicGroup.add(grip)

  const lensOuter = new THREE.Mesh(
    new THREE.CylinderGeometry(0.22, 0.25, 0.24, 36),
    new THREE.MeshStandardMaterial({ color: 0x0b1422, roughness: 0.28, metalness: 0.55, emissive: 0x06111f, emissiveIntensity: 0.25 }),
  )
  lensOuter.rotation.x = Math.PI / 2
  lensOuter.position.set(pos.x - 0.1, pos.y, pos.z + 0.33)
  dynamicGroup.add(lensOuter)

  const lensGlass = new THREE.Mesh(
    new THREE.CylinderGeometry(0.15, 0.15, 0.03, 36),
    new THREE.MeshBasicMaterial({ color: 0x34d9ff, transparent: true, opacity: 0.78, blending: THREE.AdditiveBlending }),
  )
  lensGlass.rotation.x = Math.PI / 2
  lensGlass.position.set(pos.x - 0.1, pos.y, pos.z + 0.47)
  dynamicGroup.add(lensGlass)

  const flash = new THREE.Mesh(new THREE.BoxGeometry(0.2, 0.08, 0.045), new THREE.MeshBasicMaterial({ color: 0xffd166 }))
  flash.position.set(pos.x - 0.31, pos.y + 0.19, pos.z + 0.24)
  dynamicGroup.add(flash)

  const shutter = new THREE.Mesh(new THREE.CylinderGeometry(0.055, 0.055, 0.035, 20), new THREE.MeshBasicMaterial({ color: 0xffd166 }))
  shutter.rotation.x = Math.PI / 2
  shutter.position.set(pos.x + 0.22, pos.y + 0.32, pos.z + 0.02)
  dynamicGroup.add(shutter)
}

function createProjectorModel(pos: THREE.Vector3) {
  const mat = new THREE.MeshStandardMaterial({
    color: 0x16283d,
    roughness: 0.46,
    metalness: 0.26,
    emissive: 0x101e30,
    emissiveIntensity: 0.38,
  })
  const body = new THREE.Mesh(new THREE.BoxGeometry(0.98, 0.34, 0.58), mat)
  body.position.copy(pos)
  body.castShadow = true
  dynamicGroup.add(body)

  const top = new THREE.Mesh(
    new THREE.BoxGeometry(0.78, 0.08, 0.42),
    new THREE.MeshStandardMaterial({ color: 0x203a58, roughness: 0.35, metalness: 0.28 }),
  )
  top.position.set(pos.x - 0.04, pos.y + 0.22, pos.z)
  dynamicGroup.add(top)

  const lensShell = new THREE.Mesh(
    new THREE.CylinderGeometry(0.16, 0.18, 0.2, 32),
    new THREE.MeshStandardMaterial({ color: 0x0c1522, roughness: 0.28, metalness: 0.52, emissive: 0x221708, emissiveIntensity: 0.25 }),
  )
  lensShell.rotation.x = Math.PI / 2
  lensShell.position.set(pos.x + 0.46, pos.y + 0.04, pos.z + 0.32)
  dynamicGroup.add(lensShell)

  const lens = new THREE.Mesh(
    new THREE.CylinderGeometry(0.12, 0.12, 0.035, 28),
    new THREE.MeshBasicMaterial({ color: 0xffd166, transparent: true, opacity: 0.86, blending: THREE.AdditiveBlending }),
  )
  lens.rotation.x = Math.PI / 2
  lens.position.set(pos.x + 0.46, pos.y + 0.04, pos.z + 0.43)
  dynamicGroup.add(lens)

  // 投影仪镜头是朝 +Z 方向的：光锥的锥顶必须贴在镜头口，底面向投影幕方向展开。
  // ConeGeometry 默认锥尖在 +Y 端、底面在 -Y 端；rotation.x = -PI/2 后，锥尖转到 -Z，底面转到 +Z。
  // 因此把 cone center 放在镜头前方半个高度处，就能让锥顶刚好对准镜头。
  const coneLength = 1.85
  const cone = new THREE.Mesh(
    new THREE.ConeGeometry(0.48, coneLength, 32, 1, true),
    new THREE.MeshBasicMaterial({
      color: 0xffd166,
      transparent: true,
      opacity: 0.12,
      blending: THREE.AdditiveBlending,
      side: THREE.DoubleSide,
      depthWrite: false,
    }),
  )
  cone.rotation.x = -Math.PI / 2
  cone.position.set(pos.x + 0.46, pos.y + 0.04, pos.z + 0.43 + coneLength / 2)
  dynamicGroup.add(cone)

  const coneTip = new THREE.Mesh(
    new THREE.SphereGeometry(0.035, 16, 12),
    new THREE.MeshBasicMaterial({ color: 0xffd166, transparent: true, opacity: 0.95, blending: THREE.AdditiveBlending }),
  )
  coneTip.position.set(pos.x + 0.46, pos.y + 0.04, pos.z + 0.43)
  dynamicGroup.add(coneTip)

  for (const dx of [-0.26, 0.14]) {
    const vent = new THREE.Mesh(
      new THREE.BoxGeometry(0.18, 0.018, 0.035),
      new THREE.MeshBasicMaterial({ color: 0x7ee8ff, transparent: true, opacity: 0.65 }),
    )
    vent.position.set(pos.x + dx, pos.y + 0.08, pos.z + 0.31)
    dynamicGroup.add(vent)
  }
}

function createMagnifierModel(pos: THREE.Vector3) {
  // 默认 Torus / Circle 位于 x-y 平面，正好像一只面向镜头的放大镜。
  // 这样手柄可以直接从圆环右下方接出去，不会再出现“柄和圈没连上”的问题。
  const ringMat = new THREE.MeshBasicMaterial({ color: 0x7ee8ff, transparent: true, opacity: 0.92 })
  const ring = new THREE.Mesh(new THREE.TorusGeometry(0.4, 0.028, 18, 72), ringMat)
  ring.position.copy(pos)
  dynamicGroup.add(ring)

  const glass = new THREE.Mesh(
    new THREE.CircleGeometry(0.36, 64),
    new THREE.MeshBasicMaterial({ color: 0x8feeff, transparent: true, opacity: 0.24, side: THREE.DoubleSide }),
  )
  glass.position.copy(pos)
  dynamicGroup.add(glass)

  const start = new THREE.Vector3(pos.x + 0.28, pos.y - 0.28, pos.z)
  const end = new THREE.Vector3(pos.x + 0.92, pos.y - 0.92, pos.z)
  addTube(dynamicGroup, start, end, 0.045, 0xffd166, 0.96)

  const gripEnd = new THREE.Vector3(pos.x + 1.1, pos.y - 1.1, pos.z)
  addTube(dynamicGroup, end, gripEnd, 0.064, 0x9b6a2f, 0.98)

  const joint = new THREE.Mesh(new THREE.SphereGeometry(0.072, 20, 16), new THREE.MeshBasicMaterial({ color: 0xffd166 }))
  joint.position.copy(start)
  dynamicGroup.add(joint)

  const shine = new THREE.Mesh(
    new THREE.RingGeometry(0.2, 0.23, 36),
    new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.18, side: THREE.DoubleSide }),
  )
  shine.position.set(pos.x - 0.07, pos.y + 0.07, pos.z + 0.002)
  dynamicGroup.add(shine)
}

function createScreen(x: number, compressed = false) {
  const screen = new THREE.Mesh(
    new THREE.BoxGeometry(0.06, 2.55, 1.45),
    new THREE.MeshPhysicalMaterial({ color: 0xffffff, transparent: true, opacity: 0.23, roughness: 0.25, metalness: 0 }),
  )
  screen.position.set(x, 0.75, 0)
  screenGroup.add(screen)
  labelGroup.add(
    makeTextSprite(compressed ? '光屏 / 承接实像\n位置已压缩显示' : '光屏 / 承接实像', '#ffffff', new THREE.Vector3(x, 2.55, 1.16), 0.72, 40),
  )
}

function createScreenProjection(x: number, imageTopY: number, score: number) {
  // 光屏寻像挑战：在光屏上显示“投影蜡烛”。
  // score 越高越清晰；偏离真实像距时，用半透明光斑和低透明度蜡烛表示模糊。
  const safeScore = clamp(score, 0, 100)
  const opacity = 0.18 + (safeScore / 100) * 0.52
  const blurOpacity = 0.2 + (1 - safeScore / 100) * 0.34
  const clampedTopY = clamp(imageTopY, -2.05, 2.05)
  const blurHeight = Math.max(0.62, Math.min(2.55, Math.abs(clampedTopY) + 0.62))

  const blurPatch = new THREE.Mesh(
    new THREE.PlaneGeometry(0.045, blurHeight, 1, 1),
    new THREE.MeshBasicMaterial({
      color: safeScore >= 80 ? 0x7ee8ff : 0xffd166,
      transparent: true,
      opacity: blurOpacity,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
      side: THREE.DoubleSide,
    }),
  )
  blurPatch.rotation.y = Math.PI / 2
  blurPatch.position.set(x - 0.045, clampedTopY * 0.48, 0.012)
  screenGroup.add(blurPatch)

  createCandleModel(screenGroup, {
    x: x - 0.075,
    topY: clampedTopY,
    bodyColor: 0xfff1c2,
    bodyAccentColor: safeScore >= 80 ? 0x34d9ff : 0xffd166,
    flameColor: 0xff9f43,
    glowColor: safeScore >= 80 ? 0x34d9ff : 0xffd166,
    opacity,
    radiusScale: 0.82,
    withLight: false,
    ghost: safeScore < 70,
  })

  const statusText = safeScore >= 92 ? '清晰成像' : safeScore >= 55 ? '略微模糊' : '明显模糊'
  labelGroup.add(
    makeTextSprite(
      `${statusText}\n清晰度 ${Math.round(safeScore)}%`,
      safeScore >= 80 ? '#7ee8ff' : '#ffd166',
      new THREE.Vector3(x, 2.92, 0),
      0.58,
      34,
    ),
  )
}

function clipTowardScene(start: THREE.Vector3, target: THREE.Vector3) {
  // 只截取场景内可见的一段延长线，不改变真实像距数值，也不把虚像压缩回场景。
  const dx = target.x - start.x
  const dy = target.y - start.y
  let t = 1

  if (dx > 0) t = Math.min(t, (MAX_X - start.x) / dx)
  if (dx < 0) t = Math.min(t, (MIN_X - start.x) / dx)
  if (dy > 0) t = Math.min(t, (MAX_DISPLAY_IMAGE_Y - start.y) / dy)
  if (dy < 0) t = Math.min(t, (-MAX_DISPLAY_IMAGE_Y - start.y) / dy)

  t = clamp(t, 0, 1)
  return new THREE.Vector3(start.x + dx * t, start.y + dy * t, start.z + (target.z - start.z) * t)
}

function createRays(m: OpticalMetrics, objectX: number, objectY: number, imageX: number, imageY: number, virtualOutOfRange = false) {
  const obj = new THREE.Vector3(objectX, objectY, 0)
  const lensAtObjY = new THREE.Vector3(0, objectY, 0)
  const center = new THREE.Vector3(0, 0, 0)

  if (!Number.isFinite(m.v)) {
    addRay(obj, lensAtObjY, 'yellow')
    addRay(lensAtObjY, new THREE.Vector3(MAX_X, objectY, 0), 'yellow')

    const slope = (0 - objectY) / (0 - objectX)
    addRay(obj, new THREE.Vector3(MAX_X, slope * MAX_X, 0), 'cyan')
    return
  }

  const img = new THREE.Vector3(imageX, imageY, 0)
  const exitX = MAX_X

  if (m.real) {
    addRay(obj, lensAtObjY, 'yellow')
    addRay(lensAtObjY, img, 'yellow')
    addRay(obj, img, 'cyan')

    const leftF = new THREE.Vector3(-cmToX(m.f), 0, 0)
    const yAtLens = yOnLine(obj, leftF, 0)
    const lensAtFocal = new THREE.Vector3(0, yAtLens, 0)
    addRay(obj, lensAtFocal, 'green')
    addRay(lensAtFocal, img, 'green')
    return
  }

  const yParallelExit = yOnLine(lensAtObjY, img, exitX)
  addRay(obj, lensAtObjY, 'yellow')
  addRay(lensAtObjY, new THREE.Vector3(exitX, yParallelExit, 0), 'yellow')
  if (!virtualOutOfRange) addDashedRay(lensAtObjY, img, 'red')

  const yCenterExit = yOnLine(center, img, exitX)
  addRay(obj, new THREE.Vector3(exitX, yCenterExit, 0), 'cyan')
  if (!virtualOutOfRange) addDashedRay(center, img, 'red')

  const leftF = new THREE.Vector3(-cmToX(m.f), 0, 0)
  const yAtLens = yOnLine(obj, leftF, 0)
  const lensAtFocal = new THREE.Vector3(0, yAtLens, 0)
  const yFocalExit = yOnLine(lensAtFocal, img, exitX)
  addRay(obj, lensAtFocal, 'green')
  addRay(lensAtFocal, new THREE.Vector3(exitX, yFocalExit, 0), 'green')
  if (!virtualOutOfRange) addDashedRay(lensAtFocal, img, 'red')
}

function yOnLine(a: THREE.Vector3, b: THREE.Vector3, x: number) {
  return a.y + ((b.y - a.y) / (b.x - a.x)) * (x - a.x)
}

function addRay(start: THREE.Vector3, end: THREE.Vector3, style: RayStyle) {
  const colorMap: Record<RayStyle, number> = {
    yellow: 0xffd166,
    cyan: 0x34d9ff,
    green: 0x64f4ac,
    red: 0xff6b7a,
  }
  const color = colorMap[style]
  addTube(rayGroup, start, end, 0.015, color, 0.9)
  addConeArrow(rayGroup, start, end, color)
}

function addDashedRay(start: THREE.Vector3, end: THREE.Vector3, style: RayStyle) {
  const color = style === 'red' ? 0xff6b7a : 0xffffff
  const points = [start, end]
  const geo = new THREE.BufferGeometry().setFromPoints(points)
  const mat = new THREE.LineDashedMaterial({ color, transparent: true, opacity: 0.72, dashSize: 0.12, gapSize: 0.08 })
  const line = new THREE.Line(geo, mat)
  line.computeLineDistances()
  rayGroup.add(line)
}

function addArrow(group: THREE.Group, start: THREE.Vector3, end: THREE.Vector3, color: number) {
  addTube(group, start, end, 0.035, color, 0.95)
  addConeArrow(group, start, end, color, 0.09, 0.2)
}

function addTube(group: THREE.Group, start: THREE.Vector3, end: THREE.Vector3, radius: number, color: number, opacity = 1) {
  const dir = new THREE.Vector3().subVectors(end, start)
  const len = dir.length()
  if (len < 0.0001) return

  const geo = new THREE.CylinderGeometry(radius, radius, len, 12)
  const mat = new THREE.MeshBasicMaterial({ color, transparent: opacity < 1, opacity })
  const mesh = new THREE.Mesh(geo, mat)
  mesh.position.copy(start).add(end).multiplyScalar(0.5)
  mesh.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), dir.normalize())
  group.add(mesh)
}

function addConeArrow(group: THREE.Group, start: THREE.Vector3, end: THREE.Vector3, color: number, radius = 0.055, height = 0.14) {
  const dir = new THREE.Vector3().subVectors(end, start)
  if (dir.lengthSq() < 0.0001) return
  dir.normalize()
  const cone = new THREE.Mesh(new THREE.ConeGeometry(radius, height, 16), new THREE.MeshBasicMaterial({ color }))
  cone.position.copy(end).addScaledVector(dir, -height * 0.45)
  cone.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), dir)
  group.add(cone)
}

function makeTextSprite(text: string, color: string, position: THREE.Vector3, size = 0.36, fontSize = 20) {
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')!
  const lines = text.split('\n')
  canvas.width = 512
  canvas.height = Math.max(128, lines.length * fontSize * 2.55)
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  ctx.font = `900 ${fontSize}px Microsoft YaHei, Arial`
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.lineWidth = Math.max(3, Math.round(fontSize * 0.2))
  ctx.strokeStyle = 'rgba(0,0,0,0.78)'
  ctx.fillStyle = color
  lines.forEach((line, index) => {
    const y = canvas.height / 2 + (index - (lines.length - 1) / 2) * fontSize * 1.45
    ctx.strokeText(line, canvas.width / 2, y)
    ctx.fillText(line, canvas.width / 2, y)
  })
  const texture = new THREE.CanvasTexture(canvas)
  texture.colorSpace = THREE.SRGBColorSpace
  const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: texture, transparent: true, depthWrite: false, depthTest: false }))
  sprite.position.copy(position)
  sprite.renderOrder = 99
  sprite.scale.set(size * 3.15, size * (canvas.height / canvas.width) * 3.15, 1)
  return sprite
}

function setCameraView(view: CameraView) {
  cameraView.value = view
  if (!camera || !controls) return
  if (view === 'top') {
    // 俯视也拉近，重点看光路关系，不再从很高处看整个光具座。
    camera.position.set(0, 10.2, 0.04)
    controls.target.set(0, 0.12, 0)
  } else {
    // 标准视角进一步拉近，主体占画面更大。
    camera.position.set(4.8, 3.7, 9.2)
    controls.target.set(0, 0.72, 0)
  }
  controls.update()
}

function resizeRenderer() {
  if (!canvasWrapRef.value || !renderer || !camera) return
  const rect = canvasWrapRef.value.getBoundingClientRect()
  const width = Math.max(1, rect.width)
  const height = Math.max(1, rect.height)
  camera.aspect = width / height
  camera.updateProjectionMatrix()
  renderer.setSize(width, height, false)
}

function animate(now: number) {
  const dt = Math.min(0.04, (now - lastTime) / 1000)
  lastTime = now

  // 不做持续自动演示：物距、焦距、物高都交给老师/学生手动拖动。
  // 这样不会在 OrbitControls 转动视角时反复重建场景，画面更稳、更适合课堂讲解。

  // 凸透镜保持静止。透镜旋转会让学生误以为透镜本身在运动，也会增加画面干扰。
  controls?.update()
  renderer?.render(scene, camera)
  frameId = requestAnimationFrame(animate)
}

function clearGroup(group: THREE.Group) {
  group.traverse(obj => {
    const mesh = obj as THREE.Mesh
    if (mesh.geometry) mesh.geometry.dispose()
    const mat = mesh.material as THREE.Material | THREE.Material[] | undefined
    if (Array.isArray(mat)) mat.forEach(m => m.dispose())
    else mat?.dispose()
  })
  group.clear()
}

function scheduleDynamicSceneUpdate() {
  if (dynamicUpdateFrame) return
  dynamicUpdateFrame = requestAnimationFrame(() => {
    dynamicUpdateFrame = 0
    updateDynamicScene()
  })
}

watch([metrics, showRays, showScreen, showLabels, focusChallenge, appMode, () => state.screenCm], () => {
  nextTick(scheduleDynamicSceneUpdate)
})

watch(
  metrics,
  value => {
    if (!focusChallenge.value && value.real && Number.isFinite(value.v)) {
      state.screenCm = clamp(value.v, -BENCH_LIMIT_CM, BENCH_LIMIT_CM)
    }
  },
  { immediate: true },
)

onMounted(() => {
  nextTick(initThree)
})

onBeforeUnmount(() => {
  if (frameId) cancelAnimationFrame(frameId)
  if (dynamicUpdateFrame) cancelAnimationFrame(dynamicUpdateFrame)
  window.removeEventListener('resize', resizeRenderer)
  resizeObserver?.disconnect()
  controls?.dispose()
  if (renderer) {
    renderer.dispose()
    renderer.domElement?.parentElement?.removeChild(renderer.domElement)
  }
  if (rootGroup) clearGroup(rootGroup)
})
</script>

<style scoped lang="scss">
:global(html),
:global(body),
:global(#app) {
  margin: 0;
  min-height: 100%;
}

.lens-page {
  --bg: #07111f;
  --panel: rgba(10, 24, 42, 0.78);
  --panel-strong: rgba(13, 32, 54, 0.9);
  --line: rgba(126, 232, 255, 0.2);
  --text: #eaf7ff;
  --muted: #9db8cf;
  --cyan: #34d9ff;
  --cyan2: #7ee8ff;
  --yellow: #ffd166;
  --green: #64f4ac;
  --red: #ff6b7a;

  width: 100vw;
  height: 100vh;
  padding: 16px;
  display: grid;
  grid-template-rows: 68px 1fr;
  gap: 12px;
  color: var(--text);
  font-family: 'Microsoft YaHei', 'PingFang SC', Arial, sans-serif;
  background:
    radial-gradient(circle at 18% 12%, rgba(52, 217, 255, 0.17), transparent 30%),
    radial-gradient(circle at 84% 20%, rgba(255, 209, 102, 0.12), transparent 32%), linear-gradient(135deg, #06111f 0%, #0b1b2f 48%, #07101d 100%);
  overflow: hidden;
  position: relative;
  box-sizing: border-box;

  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }
}

.bg-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(126, 232, 255, 0.055) 1px, transparent 1px), linear-gradient(90deg, rgba(126, 232, 255, 0.055) 1px, transparent 1px);
  background-size: 40px 40px;
  mask-image: radial-gradient(circle at center, black, transparent 76%);
  pointer-events: none;
}

.scan-line {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, transparent, rgba(126, 232, 255, 0.045), transparent);
  transform: translateY(-100%);
  animation: scan 8s linear infinite;
  pointer-events: none;
}

@keyframes scan {
  to {
    transform: translateY(100%);
  }
}

.topbar,
.panel,
.stage-card {
  position: relative;
  z-index: 1;
  border: 1px solid var(--line);
  background: linear-gradient(180deg, rgba(13, 34, 58, 0.86), rgba(7, 16, 29, 0.82));
  box-shadow:
    0 18px 48px rgba(0, 0, 0, 0.36),
    inset 0 0 28px rgba(52, 217, 255, 0.04);
  backdrop-filter: blur(16px);
}

.topbar {
  border-radius: 20px;
  padding: 12px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;

  .logo {
    width: 42px;
    height: 42px;
    border-radius: 15px;
    background:
      radial-gradient(circle at 50% 50%, #fff7ca 0 16%, #ffd166 17% 31%, rgba(255, 209, 102, 0.18) 32% 60%, transparent 61%),
      linear-gradient(135deg, rgba(52, 217, 255, 0.25), rgba(255, 209, 102, 0.2));
    border: 1px solid rgba(255, 209, 102, 0.45);
    box-shadow: 0 0 22px rgba(255, 209, 102, 0.25);
  }

  .eyebrow {
    font-size: 10px;
    letter-spacing: 0.22em;
    color: var(--cyan2);
    opacity: 0.9;
  }

  h1 {
    margin: 2px 0 0;
    font-size: 22px;
    line-height: 1;
  }
}

.top-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: flex-end;
  align-items: center;
  max-width: 760px;
}

button {
  min-height: 34px;
  border: 1px solid rgba(126, 232, 255, 0.22);
  color: var(--text);
  background: rgba(11, 28, 48, 0.86);
  border-radius: 11px;
  padding: 7px 10px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 800;
  transition: 0.18s ease;

  &:hover {
    transform: translateY(-1px);
    border-color: rgba(126, 232, 255, 0.55);
    box-shadow: 0 0 18px rgba(52, 217, 255, 0.12);
  }

  &.active {
    color: #082033;
    border-color: transparent;
    background: linear-gradient(135deg, #7ee8ff, #ffd166);
    box-shadow: 0 0 18px rgba(126, 232, 255, 0.24);
  }

  &.ghost:not(.active) {
    background: rgba(6, 17, 31, 0.58);
  }
}

.guide-mask {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: grid;
  place-items: center;
  padding: 24px;
  background:
    radial-gradient(circle at 22% 18%, rgba(52, 217, 255, 0.22), transparent 34%),
    radial-gradient(circle at 78% 72%, rgba(255, 209, 102, 0.14), transparent 35%), rgba(2, 7, 15, 0.72);
  backdrop-filter: blur(18px);
}

.guide-modal {
  width: min(1080px, 94vw);
  max-height: min(820px, 90vh);
  display: grid;
  grid-template-rows: auto 1fr auto;
  overflow: hidden;
  border-radius: 26px;
  border: 1px solid rgba(126, 232, 255, 0.28);
  color: #eaf7ff;
  background:
    linear-gradient(180deg, rgba(14, 35, 60, 0.96), rgba(5, 13, 25, 0.96)),
    radial-gradient(circle at 35% 0%, rgba(52, 217, 255, 0.14), transparent 38%);
  box-shadow:
    0 34px 90px rgba(0, 0, 0, 0.58),
    inset 0 0 36px rgba(52, 217, 255, 0.08);
}

.guide-header {
  display: flex;
  justify-content: space-between;
  gap: 18px;
  padding: 22px 24px 18px;
  border-bottom: 1px solid rgba(126, 232, 255, 0.16);
  background: linear-gradient(135deg, rgba(52, 217, 255, 0.12), rgba(255, 209, 102, 0.07));

  .guide-kicker {
    margin-bottom: 7px;
    color: #7ee8ff;
    font-size: 11px;
    font-weight: 900;
    letter-spacing: 0.22em;
  }

  h2 {
    margin: 0;
    font-size: 24px;
    line-height: 1.2;
  }

  p {
    margin: 9px 0 0;
    color: rgba(226, 232, 240, 0.78);
    font-size: 13px;
    line-height: 1.6;
  }
}

.guide-close {
  flex: 0 0 auto;
  width: 38px;
  height: 38px;
  min-height: 38px;
  padding: 0;
  display: grid;
  place-items: center;
  border-radius: 14px;
  font-size: 24px;
  line-height: 1;
  color: #fff5c2;
  background: rgba(255, 209, 102, 0.12);
  border-color: rgba(255, 209, 102, 0.28);
}

.guide-body {
  overflow: auto;
  padding: 18px 22px;
  scrollbar-width: thin;
  scrollbar-color: rgba(126, 232, 255, 0.45) transparent;
}

.guide-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.guide-card {
  border-radius: 18px;
  padding: 15px;
  border: 1px solid rgba(126, 232, 255, 0.16);
  background: linear-gradient(180deg, rgba(15, 35, 58, 0.75), rgba(4, 12, 22, 0.62));
  box-shadow: inset 0 0 18px rgba(52, 217, 255, 0.04);

  &.guide-main-card {
    margin-bottom: 12px;
    border-color: rgba(255, 209, 102, 0.24);
    background:
      radial-gradient(circle at 12% 0%, rgba(255, 209, 102, 0.12), transparent 36%),
      linear-gradient(180deg, rgba(18, 43, 68, 0.82), rgba(5, 14, 25, 0.66));
  }

  h3 {
    margin: 0 0 10px;
    color: #fff3b0;
    font-size: 15px;
  }

  ul,
  ol {
    margin: 0;
    padding-left: 19px;
    display: grid;
    gap: 8px;
  }

  li {
    color: #d7eefc;
    font-size: 12.5px;
    line-height: 1.65;
  }

  b {
    color: #7ee8ff;
  }
}

.guide-steps {
  counter-reset: guideStep;
  padding-left: 0 !important;
  list-style: none;

  li {
    position: relative;
    padding-left: 34px;

    &::before {
      counter-increment: guideStep;
      content: counter(guideStep);
      position: absolute;
      left: 0;
      top: 1px;
      width: 22px;
      height: 22px;
      display: grid;
      place-items: center;
      border-radius: 50%;
      color: #082033;
      background: linear-gradient(135deg, #7ee8ff, #ffd166);
      font-size: 12px;
      font-weight: 900;
    }
  }
}

.guide-footer {
  padding: 14px 22px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 14px;
  border-top: 1px solid rgba(126, 232, 255, 0.16);
  background: rgba(2, 8, 16, 0.5);

  span {
    color: rgba(226, 232, 240, 0.78);
    font-size: 12.5px;
    line-height: 1.5;
  }
}

.guide-start,
.guide-top-btn {
  color: #fff;
  border-color: transparent;
  background: linear-gradient(135deg, #7ee8ff, #ffd166);
  box-shadow: 0 0 18px rgba(126, 232, 255, 0.22);
}

@media (max-width: 760px) {
  .guide-grid {
    grid-template-columns: 1fr;
  }

  .guide-footer {
    align-items: stretch;
    flex-direction: column;
  }
}

.layout {
  min-height: 0;
  display: grid;
  grid-template-columns: 300px 1fr 330px;
  gap: 12px;
}

.panel {
  min-height: 0;
  overflow: hidden auto;
  border-radius: 22px;
  padding: 14px;
  scrollbar-width: thin;
  scrollbar-color: rgba(126, 232, 255, 0.45) transparent;
}

.panel-title {
  display: flex;
  align-items: center;
  gap: 9px;
  font-weight: 900;
  font-size: 16px;
  margin-bottom: 12px;

  span {
    width: 7px;
    height: 20px;
    border-radius: 999px;
    background: linear-gradient(180deg, var(--cyan), var(--yellow));
    box-shadow: 0 0 16px rgba(52, 217, 255, 0.45);
  }
}

.block,
.data-card,
.knowledge-card {
  padding: 12px;
  border: 1px solid rgba(126, 232, 255, 0.15);
  background: rgba(5, 15, 27, 0.45);
  border-radius: 16px;
  margin-bottom: 10px;

  h3 {
    margin: 0 0 9px;
    font-size: 14px;
    color: #dff8ff;
  }
}

.range-row {
  width: 100%;
  margin-top: 8px;
  margin-bottom: 12px;
}

:deep(.range-head) {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  width: 100%;
  margin-bottom: 8px;
}

:deep(.range-label) {
  flex: 1;
  min-width: 0;
  font-size: 12px;
  font-weight: 700;
  color: rgba(226, 232, 240, 0.92);
  line-height: 1.2;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

:deep(.range-value) {
  flex-shrink: 0;
  min-width: 58px;
  padding: 3px 8px;
  border-radius: 999px;
  text-align: center;
  font-size: 12px;
  font-weight: 900;
  line-height: 1.2;
  color: #ffd166;
  background: linear-gradient(135deg, rgba(255, 209, 102, 0.18), rgba(255, 176, 32, 0.08));
  border: 1px solid rgba(255, 209, 102, 0.35);
  box-shadow:
    0 0 12px rgba(255, 209, 102, 0.18),
    inset 0 0 10px rgba(255, 209, 102, 0.08);
}

:deep(.range-row) {
  width: 100%;
  margin-top: 8px;
}

:deep(.range-row .el-slider) {
  width: 100%;
  --el-slider-main-bg-color: #34d9ff;
  --el-slider-runway-bg-color: rgba(126, 232, 255, 0.16);
  --el-slider-stop-bg-color: rgba(255, 255, 255, 0.35);
  --el-slider-disabled-color: rgba(126, 232, 255, 0.25);
  --el-slider-border-radius: 999px;
  --el-slider-height: 4px;
  --el-slider-button-size: 14px;
  --el-slider-button-wrapper-size: 34px;
  --el-slider-button-wrapper-offset: -15px;
}

:deep(.range-row .el-slider__runway) {
  box-shadow: inset 0 0 8px rgba(52, 217, 255, 0.08);
}

:deep(.range-row .el-slider__bar) {
  background: linear-gradient(90deg, #34d9ff, #ffd166);
  box-shadow: 0 0 12px rgba(52, 217, 255, 0.35);
}

:deep(.range-row .el-slider__button) {
  border: 2px solid #ffd166;
  background: #07111f;
  box-shadow:
    0 0 0 4px rgba(255, 209, 102, 0.12),
    0 0 14px rgba(255, 209, 102, 0.35);
}

.preset-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 7px;
}

.layer-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 7px;

  button {
    width: 100%;
    justify-content: center;
  }
}

.control-block {
  border-color: rgba(255, 209, 102, 0.18);
  background: linear-gradient(135deg, rgba(52, 217, 255, 0.08), transparent 46%), rgba(5, 15, 27, 0.5);
}

.tip {
  color: var(--muted);
  font-size: 11px;
  line-height: 1.65;
  margin: 8px 0 0;
}

.task-list {
  margin: 0;
  padding-left: 18px;
  color: #dceffc;
  font-size: 12px;
  line-height: 1.7;
}

.stage-card {
  min-width: 0;
  min-height: 0;
  border-radius: 24px;
  overflow: hidden;
  padding: 12px;
  display: grid;
  grid-template-rows: 1fr 118px;
  gap: 10px;
}

.canvas-wrap {
  position: relative;
  min-height: 0;
  border-radius: 20px;
  overflow: hidden;
  border: 1px solid rgba(126, 232, 255, 0.16);
  background:
    radial-gradient(circle at 50% 38%, rgba(52, 217, 255, 0.12), transparent 38%),
    linear-gradient(180deg, rgba(6, 17, 31, 0.86), rgba(4, 10, 18, 0.96));

  canvas {
    display: block;
    width: 100%;
    height: 100%;
  }
}

.scene-title {
  position: absolute;
  left: 20px;
  top: 84px;
  z-index: 2;
  display: grid;
  gap: 4px;
  padding: 9px 11px;
  border-radius: 13px;
  background: rgba(0, 0, 0, 0.34);
  border: 1px solid rgba(126, 232, 255, 0.18);
  pointer-events: none;

  b {
    font-size: 14px;
    color: #fff3b0;
  }

  span {
    font-size: 11px;
    color: #c8e5f5;
  }
}

.legend-panel {
  position: absolute;
  right: 20px;
  top: 84px;
  z-index: 2;
  padding: 9px 11px;
  border-radius: 13px;
  background: rgba(0, 0, 0, 0.32);
  border: 1px solid rgba(126, 232, 255, 0.18);
  display: grid;
  gap: 6px;
  font-size: 11px;
  color: #c9e8f8;
  pointer-events: none;

  .legend-title {
    color: #ffffff;
    font-weight: 900;
  }

  .ray {
    display: inline-block;
    width: 20px;
    height: 3px;
    border-radius: 999px;
    margin-right: 6px;
    vertical-align: middle;
  }

  .yellow {
    background: var(--yellow);
  }
  .cyan {
    background: var(--cyan);
  }
  .green {
    background: var(--green);
  }
  .red {
    background: repeating-linear-gradient(90deg, var(--red) 0 6px, transparent 6px 10px);
  }
}

.toast {
  position: absolute;
  right: 20px;
  bottom: 138px;
  z-index: 2;
  max-width: 400px;
  padding: 10px 12px;
  border-radius: 14px;
  background: rgba(0, 0, 0, 0.38);
  border: 1px solid rgba(255, 209, 102, 0.24);
  color: #fff3c7;
  font-size: 12px;
  line-height: 1.55;
  pointer-events: none;
}

.metric-strip {
  border-radius: 20px;
  border: 1px solid rgba(126, 232, 255, 0.15);
  background: rgba(5, 15, 27, 0.55);
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  padding: 10px;
}

.metric-card {
  border-radius: 14px;
  padding: 10px;
  background: rgba(13, 34, 58, 0.7);
  border: 1px solid rgba(126, 232, 255, 0.12);

  span {
    display: block;
    font-size: 11px;
    color: var(--muted);
    margin-bottom: 5px;
  }

  b {
    font-size: 17px;
    color: var(--cyan2);
    font-variant-numeric: tabular-nums;
  }

  &.active b {
    color: var(--yellow);
  }
}

.card-head {
  font-size: 13px;
  color: #ffffff;
  font-weight: 900;
  margin-bottom: 8px;
}

.big-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  padding: 8px 0;
  border-bottom: 1px dashed rgba(126, 232, 255, 0.12);

  span {
    color: var(--muted);
    font-size: 12px;
  }

  b {
    color: var(--yellow);
    font-size: 15px;
    font-variant-numeric: tabular-nums;
  }
}

.small-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  margin-top: 10px;

  div {
    padding: 8px;
    border-radius: 12px;
    background: rgba(15, 35, 58, 0.65);
  }

  span {
    display: block;
    color: var(--muted);
    font-size: 11px;
    margin-bottom: 4px;
  }

  b {
    color: var(--cyan2);
    font-size: 13px;
  }
}

.knowledge-card {
  h3 {
    font-size: 14px;
  }

  ul {
    margin: 0;
    padding-left: 17px;
    color: #d7eefc;
    font-size: 12px;
    line-height: 1.68;
  }

  b {
    color: var(--cyan2);
  }
}

.formula-item {
  display: grid;
  gap: 6px;
  margin-top: 8px;

  b {
    color: #ffffff;
    font-size: 12px;
  }

  code {
    display: block;
    padding: 8px;
    border-radius: 10px;
    background: rgba(0, 0, 0, 0.28);
    color: #fff3b0;
    font-family: Consolas, Monaco, monospace;
    font-size: 12px;
  }

  span {
    color: #d7eefc;
    font-size: 12px;
    line-height: 1.55;
  }

  &.strong {
    padding: 8px;
    border-radius: 12px;
    background: linear-gradient(135deg, rgba(52, 217, 255, 0.1), rgba(255, 209, 102, 0.08));
    border: 1px solid rgba(126, 232, 255, 0.13);
  }
}

.app-grid {
  display: grid;
  gap: 8px;

  div {
    padding: 9px 10px;
    border-radius: 12px;
    background: rgba(15, 35, 58, 0.72);
    border: 1px solid rgba(126, 232, 255, 0.1);
  }

  b {
    display: block;
    color: var(--yellow);
    font-size: 12px;
    margin-bottom: 4px;
  }

  span {
    color: #d7eefc;
    font-size: 12px;
    line-height: 1.5;
  }
}

.canvas-wrap::before {
  content: '';
  position: absolute;
  inset: 0;
  pointer-events: none;
  background:
    radial-gradient(circle at 50% 48%, rgba(52, 217, 255, 0.16), transparent 34%),
    linear-gradient(90deg, rgba(52, 217, 255, 0.08), transparent 18%, transparent 82%, rgba(255, 209, 102, 0.07));
  mix-blend-mode: screen;
}

.canvas-wrap::after {
  content: '';
  position: absolute;
  left: 18px;
  right: 18px;
  bottom: 18px;
  height: 2px;
  border-radius: 999px;
  background: linear-gradient(90deg, transparent, rgba(52, 217, 255, 0.72), rgba(255, 209, 102, 0.58), transparent);
  box-shadow: 0 0 18px rgba(52, 217, 255, 0.42);
  pointer-events: none;
}

.stage,
.panel,
.topbar {
  box-shadow:
    0 22px 70px rgba(0, 0, 0, 0.42),
    inset 0 0 34px rgba(52, 217, 255, 0.07),
    0 0 0 1px rgba(52, 217, 255, 0.05);
}

.card,
.block,
.metric {
  background:
    linear-gradient(180deg, rgba(13, 34, 58, 0.72), rgba(5, 15, 27, 0.56)),
    radial-gradient(circle at 10% 0%, rgba(52, 217, 255, 0.1), transparent 42%);
}

.mode-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  margin-bottom: 10px;
}

.challenge-row {
  display: grid;
  grid-template-columns: 1fr;
  gap: 8px;
  margin: 10px 0 2px;

  .wide {
    width: 100%;
  }

  .ghost-btn:not(.active) {
    color: rgba(226, 247, 255, 0.88);
    background: rgba(6, 17, 31, 0.58);
  }
}

.feature-summary-card {
  border-color: rgba(255, 209, 102, 0.22);
  background: linear-gradient(180deg, rgba(255, 209, 102, 0.08), rgba(5, 15, 27, 0.5));
}

.report-card {
  pre {
    margin: 0;
    white-space: pre-wrap;
    color: #dff8ff;
    font-size: 12px;
    line-height: 1.65;
    font-family: 'Microsoft YaHei', 'PingFang SC', Arial, sans-serif;
  }
}

.top-snapshot-btn.active,
.top-snapshot-btn {
  color: #07111f;
  border-color: transparent;
  background: linear-gradient(135deg, #7ee8ff, #ffd166) !important;
  box-shadow: 0 0 18px rgba(255, 209, 102, 0.22);
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.45;
  transform: none !important;
  box-shadow: none !important;
}

.log-card {
  border-color: rgba(126, 232, 255, 0.22);
}

.log-empty {
  color: var(--muted);
  font-size: 12px;
  line-height: 1.65;
}

.log-list {
  display: grid;
  gap: 9px;
  max-height: 360px;
  overflow: hidden auto;
  padding-right: 2px;
}

.log-item {
  padding: 9px;
  border-radius: 12px;
  background: rgba(15, 35, 58, 0.68);
  border: 1px solid rgba(126, 232, 255, 0.13);

  .log-head {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 8px;
    margin-bottom: 6px;

    b {
      color: var(--yellow);
      font-size: 12px;
    }

    span {
      flex-shrink: 0;
      color: var(--muted);
      font-size: 11px;
      font-variant-numeric: tabular-nums;
    }
  }

  p {
    margin: 4px 0 0;
    color: #d7eefc;
    font-size: 11.5px;
    line-height: 1.55;

    b {
      color: var(--cyan2);
    }
  }
}

.rule-table-card {
  border-color: rgba(100, 244, 172, 0.22);
}

.rule-table {
  display: grid;
  gap: 8px;
}

.rule-row {
  display: grid;
  grid-template-columns: 112px 1fr;
  gap: 10px;
  align-items: center;
  padding: 9px 10px;
  border-radius: 13px;
  background: rgba(15, 35, 58, 0.62);
  border: 1px solid rgba(126, 232, 255, 0.12);
  transition: 0.18s ease;

  .rule-left {
    display: grid;
    gap: 3px;

    b {
      color: var(--cyan2);
      font-size: 12px;
    }

    span {
      color: var(--muted);
      font-size: 10.5px;
      line-height: 1.35;
    }
  }

  .rule-right {
    color: #dff8ff;
    font-size: 11.5px;
    line-height: 1.5;
  }

  &.active {
    background: linear-gradient(135deg, rgba(255, 209, 102, 0.2), rgba(52, 217, 255, 0.08));
    border-color: rgba(255, 209, 102, 0.42);
    box-shadow: 0 0 16px rgba(255, 209, 102, 0.14);

    .rule-left b,
    .rule-right {
      color: var(--yellow);
    }
  }
}

.challenge-card-left,
.step-guide-card {
  border-color: rgba(255, 209, 102, 0.18);
}

.challenge-status {
  display: grid;
  gap: 5px;
  padding: 9px 10px;
  border-radius: 13px;
  background: rgba(15, 35, 58, 0.64);
  border: 1px solid rgba(126, 232, 255, 0.12);
  margin-bottom: 9px;

  b {
    color: var(--yellow);
    font-size: 12px;
  }

  span {
    color: var(--muted);
    font-size: 11.5px;
    line-height: 1.5;
  }
}

.challenge-actions,
.step-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  margin-top: 8px;

  .wide {
    width: 100%;
  }

  .ghost-btn:not(.active) {
    color: rgba(226, 247, 255, 0.88);
    background: rgba(6, 17, 31, 0.58);
  }
}

.step-actions {
  grid-template-columns: 1fr 1fr 1fr;
}

.choice-list {
  display: grid;
  gap: 7px;
  margin-top: 10px;

  button {
    display: grid;
    gap: 3px;
    text-align: left;
    padding: 9px 10px;

    b {
      font-size: 12px;
    }

    span {
      color: rgba(226, 247, 255, 0.74);
      font-size: 11px;
      line-height: 1.45;
    }

    &.correct {
      border-color: rgba(100, 244, 172, 0.58);
      background: linear-gradient(135deg, rgba(100, 244, 172, 0.2), rgba(52, 217, 255, 0.08));
    }

    &.wrong {
      border-color: rgba(255, 107, 122, 0.58);
      background: linear-gradient(135deg, rgba(255, 107, 122, 0.18), rgba(15, 35, 58, 0.72));
    }
  }
}

.challenge-result {
  margin-top: 10px;
  padding: 10px;
  border-radius: 13px;
  background: rgba(255, 107, 122, 0.12);
  border: 1px solid rgba(255, 107, 122, 0.3);

  b {
    display: block;
    color: #ff9aa6;
    font-size: 12px;
    margin-bottom: 4px;
  }

  span {
    color: #ffe3e7;
    font-size: 11.5px;
    line-height: 1.55;
  }

  &.pass {
    background: rgba(100, 244, 172, 0.12);
    border-color: rgba(100, 244, 172, 0.32);

    b {
      color: var(--green);
    }

    span {
      color: #dcfff0;
    }
  }
}

.step-progress {
  display: grid;
  gap: 5px;
  padding: 10px;
  border-radius: 13px;
  background: linear-gradient(135deg, rgba(52, 217, 255, 0.12), rgba(255, 209, 102, 0.08));
  border: 1px solid rgba(126, 232, 255, 0.14);

  span {
    color: var(--muted);
    font-size: 11px;
  }

  b {
    color: var(--yellow);
    font-size: 13px;
  }
}

.step-text {
  margin: 9px 0 0;
  color: #d7eefc;
  font-size: 12px;
  line-height: 1.6;
}

@media (max-width: 1200px) {
  .lens-page {
    height: auto;
    min-height: 100vh;
    overflow: auto;
  }

  .layout {
    grid-template-columns: 1fr;
  }

  .stage-card {
    height: 760px;
  }

  .metric-strip {
    grid-template-columns: 1fr 1fr;
  }
}
</style>
