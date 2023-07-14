<template>
  <div class="config">
    <li v-for="(item, index) in items" :key="index">
      {{ parentMessage }} - {{ index }} - {{ item.message }}
    </li>
    <!-- 使用特殊的 $event 变量 -->
    <el-button @click="warn('Form cannot be submitted yet.', $event)">Submit</el-button>

    <!-- 使用内联箭头函数 -->
    <el-button @click="(event) => warn('Form cannot be submitted yet.', event)">Submit</el-button>

    <p>Message is: {{ message }}</p>
    <el-input label-width="140px" style="width: 240px" v-model="message" placeholder="edit me" />

    <p>
      Ask a yes/no question:
      <el-input v-model="question" />
    </p>
    <p>{{ answer }}</p>
    <el-drawer v-model="drawer" title="I'm outer Drawer" size="30%">
      <template #header>
        <h4>set title by slot</h4>
      </template>
      <template #default>
        <div>
          <el-radio v-model="radio" label="Option 1" size="large">Option 1</el-radio>
          <el-radio v-model="radio" label="Option 2" size="large">Option 2</el-radio>
        </div>
      </template>
      <template #footer>
        <div style="flex: auto">
          <el-button @click="cancelClick">cancel</el-button>
          <el-button type="primary" @click="confirmClick">confirm</el-button>
        </div>
      </template>
    </el-drawer>
    <el-button @click="drawer = true"> open </el-button>

    <el-button @click="show = !show">控制动态效果</el-button>

    <!-- 测试动态效果 -->
    <Transition name="bounce" appear>
      <p v-if="show" style="margin-top: 20px; text-align: center">
        Hello here is some bouncy text!
      </p>
    </Transition>
    <!-- 测试动态效果 -->

    <!-- 深层动画 -->
    <Transition duration="550" name="nested" appear>
      <div v-if="show" class="outer">
        <div class="inner">Hello</div>
      </div>
    </Transition>
    <!-- 深层动画 -->
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { onMounted } from 'vue'
import { ElMessageBox } from 'element-plus'
onMounted(() => {
  console.log(`the component is now mounted.`)
})
const drawer = ref(false)
const message = ref('')
const parentMessage = ref('Parent')
const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
const radio = ref('Option 1')
function warn(message, event) {
  // 这里可以访问原生事件
  if (event) {
    event.preventDefault()
  }
  alert(message)
}
function cancelClick() {
  drawer.value = false
}
function confirmClick() {
  ElMessageBox.confirm(`Are you confirm to chose ${radio.value} ?`)
    .then(() => {
      drawer.value = false
    })
    .catch(() => {
      // catch error
    })
}
// 测试侦听
const question = ref('')
const answer = ref('Questions usually contain a question mark. ;-)')
// 可以直接侦听一个 ref
watch(question, async (newQuestion, oldQuestion) => {
  console.log(oldQuestion)
  if (newQuestion.indexOf('?') > -1) {
    answer.value = 'Thinking...'
    try {
      const res = await fetch('https://yesno.wtf/api')
      answer.value = (await res.json()).answer
    } catch (error) {
      answer.value = 'Error! Could not reach the API. ' + error
    }
  }
})

const show = ref(true)
</script>

<style>
@media (min-width: 1024px) {
  .config {
    padding-top: 100px;
    min-height: 100vh;
    align-items: center;
  }
}

.bounce-enter-active {
  animation: bounce-in 0.5s;
}
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
    transform: translateX(150px);
    opacity: 0;
  }
  50% {
    transform: scale(1.25);
  }
  100% {
    transform: scale(1);
  }
}

.outer,
.inner {
  background: #eee;
  padding: 30px;
  min-height: 100px;
}

.inner {
  background: #ccc;
}

.nested-enter-active,
.nested-leave-active {
  transition: all 0.3s ease-in-out;
}
/* delay leave of parent element */
.nested-leave-active {
  transition-delay: 0.25s;
}

.nested-enter-from,
.nested-leave-to {
  transform: translateY(30px);
  opacity: 0;
}

/* we can also transition nested elements using nested selectors */
.nested-enter-active .inner,
.nested-leave-active .inner {
  transition: all 0.3s ease-in-out;
}
/* delay enter of nested element */
.nested-enter-active .inner {
  transition-delay: 0.25s;
}

.nested-enter-from .inner,
.nested-leave-to .inner {
  transform: translateX(30px);
  /*
  	Hack around a Chrome 96 bug in handling nested opacity transitions.
    This is not needed in other browsers or Chrome 99+ where the bug
    has been fixed.
  */
  opacity: 0.001;
}
</style>
