<script setup lang="ts">
import { ref } from "vue";
import FormGenerator, {
  type FormSchema,
  type FormData,
} from "./components/FormGenerator.vue";

const formSchema: FormSchema = {
  fields: [
    { type: "text", label: "Имя", model: "name", required: true },
    { type: "email", label: "Email", model: "email", required: true },
    {
      type: "password",
      label: "Пароль",
      model: "password",
      required: true,
      minLength: 6,
    },
    {
      type: "select",
      label: "Роль",
      model: "role",
      options: ["Админ", "Пользователь"],
      required: true,
    },
    {
      type: "checkbox",
      label: "Согласен с условиями",
      model: "terms",
      required: true,
    },
  ],
};

const formData = ref<FormData>({
  name: "",
  email: "",
  password: "",
  role: "",
  terms: false,
});

const lastSubmit = ref<FormData | null>(null);

function onSubmit(data: FormData) {
  lastSubmit.value = { ...data };
}
</script>

<template>
  <main class="page">
    <h1>FormGenerator</h1>
    <div class="grid">
      <FormGenerator
        :schema="formSchema"
        v-model="formData"
        @submit="onSubmit"
      />
      <aside>
        <h3>formData (реактивно)</h3>
        <pre>{{ formData }}</pre>
        <h3>Последний submit</h3>
        <pre>{{ lastSubmit ?? "—" }}</pre>
      </aside>
    </div>
  </main>
</template>

<style scoped>
.page {
  padding: 32px;
  font-family: system-ui, sans-serif;
  max-width: 900px;
  margin: 0 auto;
}
.grid {
  display: grid;
  grid-template-columns: 360px 1fr;
  gap: 48px;
  align-items: start;
}
@media (max-width: 700px) {
  .page { padding: 16px; }
  .grid { grid-template-columns: 1fr; gap: 24px; }
}
pre {
  background: #f4f4f4;
  padding: 10px;
  border-radius: 6px;
  font-size: 13px;
}
h1 {
  margin-top: 0;
}
h3 {
  margin-bottom: 4px;
}
</style>
