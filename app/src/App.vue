<script setup lang="ts">
import { ref, onMounted, watch, computed, nextTick} from 'vue';
import { supabase } from './supabase';

const startFlg = ref(false);
const current_question = ref('');
const typeBox = ref('');
const current_question_counts = ref(0);
const question_counts = ref<number>(0);

interface Lang {
  id: number;
  language: string;
  already: boolean;
}

const langs = ref<Lang[]>([]);

const getLang = async () => {
  try {
    const { data: result, error } = await supabase.from('HTML').select('id, language, already');
    if (error) {
      console.error("Error fetching langs:", error);
      return;
    }
    langs.value = result || [];
  } catch (error) {
    console.error("An unexpected error occurred:", error);
  }
};
onMounted(getLang);

const gameStart = () => {
  current_question.value = langs.value[0].language;
  question_counts.value = langs.value.length;

  startFlg.value = true;
  nextTick(function () {
    const form = document.getElementById('typeForm');
    if (form != null) {
      form.focus();
    }
  })
}
const gaugeStyle = computed((): { width: string; backgroundColor: string } => {
  const rate: number = 100 / question_counts.value;
  const width = ref(rate * current_question_counts.value + '%');
  const color = ref('')
  if (current_question_counts.value == question_counts.value) {
    width.value = 100 + '%'
    color.value = '#03a9f4'
  } else {
    color.value = 'orange'
  }
  return {
    width: width.value,
    backgroundColor: color.value
  }
})
const styleObject = ref(gaugeStyle)
watch(typeBox, (e): void => {
  if (e == current_question.value) {
    langs.value[0].language.slice(0, 1);
    current_question_counts.value++;
    if(current_question_counts.value != question_counts.value){
      current_question.value = langs.value[current_question_counts.value].language;
    }
    typeBox.value = '';    
  }
})
</script>

<template>
  <div class="container">
    <div class="title">
      <h1>Typing Game</h1>
      <div class="marker"></div>
    </div>
    <button v-if="startFlg != true" class="startButton mb-20" @click="gameStart">Start</button>
    <div v-if="startFlg">
      <div v-if="current_question_counts != question_counts" class="question mb-20">{{ current_question }}</div>
      <div v-if="current_question_counts == question_counts" class="clear">Clear!</div>
      <div class="typeFormWrapper mb-20">
        <input id="typeForm" v-model="typeBox" type="text" class="typeForm" />
      </div>
      <div class="gaugeWrapper mb-20">
        <div v-bind:style="styleObject" class="gauge"></div>
      </div>
      <div>{{ current_question_counts }}/{{ question_counts }}</div>
    </div>
  </div>
</template>

<style scoped>
* {
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
}
.mb-20 {
  margin-bottom: 20px;
}
.container {
  width: 400px;
  text-align: center;
  margin: 0 auto;
  font-size: 30px;
}
.title {
  position: relative;
  font-size: 48px;
}
.marker {
  width: 100%;
  height: 38%;
  background-color: rgb(209, 209, 55);
  position: absolute;
  bottom: 5%;
  z-index: -1;
}
.startButton {
  background-color: black;
  color: #fff;
  padding: 4px 60px;
  border: none;
  outline: none;
  border-radius: 8px;
  cursor: pointer;
}
.startButton:hover {
  opacity: 0.7;
}
.question {
  color: gray;
}
.clear {
  color: rgb(121, 121, 187);
}
.typeForm {
  text-align: center;
  outline: none;
  border: none;
}
.typeFormWrapper {
  border-bottom: solid 1px gray;
}
.gauge {
  height: 12px;
  transition: all 0.3s ease;
}
.gaugeWrapper {
  border: 1px solid;
  height: 12px;
}
</style>
