<template>
  <div class="question-container">
    <div class="title"><b>{{ index + 1 }}：{{ question.title }}</b></div>
    <ul class="options">
      <li 
        v-for="(option, idx) in question.options" 
        :key="idx"
        class="option pointer"
        :class="{ selected: idx === question.selected }"
        @click="$emit('select', idx)"
      >
        {{ option.desc }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

const props = defineProps(['question', 'index'])
const emit = defineEmits(['select'])

const selectOption = (idx) => {
  emit('select', idx)
}
</script>

<style lang="stylus" scoped>
.question-container
  width: 100%;
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
.title
  padding-bottom: 10px;
  border-bottom: 2px solid #ddd;
  margin-bottom: 20px;
.options
  padding-left: 0;
  list-style: none;
.option
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
.option.selected
  background-color: rgb(50 121 255);
  color: #fff;
</style>
