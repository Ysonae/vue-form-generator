# FormGenerator

Компонент-генератор формы по JSON-схеме. Vue 3 + TS + Vite.

## Запуск

npm install
npm run dev

## Использование

<FormGenerator :schema="formSchema" v-model="formData" @submit="onSubmit" />

Поддерживаемые типы полей: `text`, `email`, `password`, `select`, `checkbox`
Валидация: `required`, `minLength`, `pattern`, формат email

## Что можно доработать под прод

- Расширить типы значений (number, date, file, мультиселект, вложенные объекты)
- Кросс-полевая валидация (например, `password === confirmPassword`)
- Асинхронная валидация с состоянием `pending` (проверка email на занятость и тд)
- Режимы валидации (`onBlur` / `onChange` / `onSubmit`) и дебаунс
- A11y: связка `label`+`input` через `for`/`id`, `aria-invalid`, `aria-describedby`
- Тесты на `validateField`
- Для реального прода - взять готовое: VeeValidate / FormKit / Vuelidate
