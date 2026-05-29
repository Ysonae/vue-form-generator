<script setup lang="ts">
import { computed, reactive } from "vue";

interface BaseField {
  label: string;
  model: string;
  required?: boolean;
}
export interface TextField extends BaseField {
  type: "text" | "email" | "password";
  minLength?: number;
  pattern?: string;
  placeholder?: string;
}
export interface SelectField extends BaseField {
  type: "select";
  options: string[];
}
export interface CheckboxField extends BaseField {
  type: "checkbox";
}
export type Field = TextField | SelectField | CheckboxField;
export interface FormSchema {
  fields: Field[];
}
export type FormData = Record<string, string | boolean>;

function validateField(field: Field, value: unknown): string | null {
  if (field.type === "checkbox") {
    if (field.required && value !== true) return "Обязательное поле";
    return null;
  }
  if (field.type === "select") {
    if (field.required && (value == null || value === ""))
      return "Выберите значение";
    return null;
  }
  const str = typeof value === "string" ? value : "";
  if (field.required && str.trim() === "") return "Обязательное поле";
  if (!str) return null;
  if (field.type === "email" && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(str))
    return "Некорректный email";
  if (field.minLength != null && str.length < field.minLength)
    return `Минимум ${field.minLength} символов`;
  if (field.pattern && !new RegExp(field.pattern).test(str))
    return "Некорректный формат";
  return null;
}

const props = defineProps<{
  schema: FormSchema;
  modelValue: FormData;
}>();

const emit = defineEmits<{
  "update:modelValue": [value: FormData];
  submit: [value: FormData];
}>();

const touched = reactive<Record<string, boolean>>({});
let submitted = false;

function setValue(field: Field, value: string | boolean) {
  emit("update:modelValue", { ...props.modelValue, [field.model]: value });
}

const errors = computed<Record<string, string | null>>(() => {
  const result: Record<string, string | null> = {};
  for (const f of props.schema.fields) {
    result[f.model] = validateField(f, props.modelValue[f.model]);
  }
  return result;
});

const isValid = computed(() => Object.values(errors.value).every((e) => !e));

function showError(model: string): string | null {
  if (!touched[model] && !submitted) return null;
  return errors.value[model] ?? null;
}

function onSubmit() {
  submitted = true;
  for (const f of props.schema.fields) touched[f.model] = true;
  if (isValid.value) emit("submit", props.modelValue);
}
</script>

<template>
  <form class="fg" @submit.prevent="onSubmit" novalidate>
    <div v-for="field in schema.fields" :key="field.model" class="fg__row">
      <template v-if="field.type === 'checkbox'">
        <label class="fg__check">
          <input
            type="checkbox"
            :checked="modelValue[field.model] === true"
            @change="
              setValue(field, ($event.target as HTMLInputElement).checked)
            "
            @blur="touched[field.model] = true"
          />
          <span
            >{{ field.label
            }}<span v-if="field.required" class="fg__req">*</span></span
          >
        </label>
      </template>

      <template v-else>
        <label class="fg__label">
          {{ field.label }}<span v-if="field.required" class="fg__req">*</span>
        </label>

        <select
          v-if="field.type === 'select'"
          :value="modelValue[field.model] ?? ''"
          @change="setValue(field, ($event.target as HTMLSelectElement).value)"
          @blur="touched[field.model] = true"
        >
          <option value="" disabled>— выберите —</option>
          <option v-for="opt in field.options" :key="opt" :value="opt">
            {{ opt }}
          </option>
        </select>

        <input
          v-else
          :type="field.type"
          :value="modelValue[field.model] ?? ''"
          :placeholder="field.placeholder"
          @input="setValue(field, ($event.target as HTMLInputElement).value)"
          @blur="touched[field.model] = true"
        />
      </template>

      <div v-if="showError(field.model)" class="fg__error">
        {{ showError(field.model) }}
      </div>
    </div>

    <button type="submit" :disabled="!isValid">Отправить</button>
  </form>
</template>

<style scoped>
.fg {
  display: flex;
  flex-direction: column;
  gap: 14px;
  width: 100%;
}
.fg input:not([type="checkbox"]),
.fg select {
  width: 100%;
  box-sizing: border-box;
}
.fg__check input { width: auto; flex: 0 0 auto; }
.fg__row {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.fg__label {
  font-size: 14px;
  color: #333;
}
.fg__req {
  color: #d33;
  margin-left: 2px;
}
.fg__check {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}
.fg__error {
  color: #d33;
  font-size: 12px;
}
input[type="text"],
input[type="email"],
input[type="password"],
select {
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font: inherit;
}
select {
  appearance: none;
  -webkit-appearance: none;
  padding-right: 32px;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'><path fill='none' stroke='%23555' stroke-width='1.5' d='M1 1l5 5 5-5'/></svg>");
  background-repeat: no-repeat;
  background-position: right 10px center;
}
input:focus,
select:focus {
  outline: none;
  border-color: #4a90e2;
}
button {
  padding: 9px 14px;
  border: 0;
  border-radius: 6px;
  background: #4a90e2;
  color: #fff;
  font: inherit;
  cursor: pointer;
}
button:disabled {
  background: #aaa;
  cursor: not-allowed;
}
</style>
