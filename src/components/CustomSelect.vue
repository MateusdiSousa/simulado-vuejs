<template>
  <div class="custom-select">
    <label v-if="label">{{ label }}</label>
    <select
      v-model="selectedValue"
      @change="onChange"
      class="select-input"
    >
      <option v-if="placeholder" disabled value="">{{ placeholder }}</option>
      <option
        v-for="item in items"
        :key="item[keyProperty]"
        :value="item"
      >
        {{ item[displayProperty] }}
      </option>
    </select>

    <p v-if="showSelected && selectedValue" class="selected-value">
      Você selecionou: {{ selectedValue[displayProperty] }}
    </p>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';

const props = defineProps({
  items: {
    type: Array,
    required: true
  },
  label: {
    type: String,
    default: ''
  },
  placeholder: {
    type: String,
    default: 'Selecione uma opção'
  },
  modelValue: {
    type: [Object, String, Number],
    default: null
  },
  displayProperty: {
    type: String,
    default: 'nome'
  },
  keyProperty: {
    type: String,
    default: 'id'
  },
  showSelected: {
    type: Boolean,
    default: true
  }
});

const emit = defineEmits(['update:modelValue', 'change']);

const selectedValue = ref(props.modelValue);

watch(() => props.modelValue, (newValue) => {
  selectedValue.value = newValue;
});

watch(selectedValue, (newValue) => {
  emit('update:modelValue', newValue);
});

const onChange = () => {
  emit('change', selectedValue.value);
};
</script>

<style scoped>
.custom-select {
  margin-bottom: 1rem;
}

.select-input {
  padding: 0.5rem;
  border-radius: 4px;
  border: 1px solid #ccc;
  min-width: 200px;
}

.selected-value {
  margin-top: 0.5rem;
  color: #666;
}
</style>
