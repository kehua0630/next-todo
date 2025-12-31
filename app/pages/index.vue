<script setup lang="ts">
import { ref, computed, watch, onMounted } from "vue";

interface Todo {
  id: number;
  text: string;
  done: boolean;
  editing?: boolean;
}

const STORAGE_KEY = "nuxt-todos";

const todos = ref<Todo[]>([]);
const newTodo = ref("");
const filter = ref<"all" | "active" | "completed">("all");

/* ---------- LocalStorage ---------- */
onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    todos.value = JSON.parse(saved);
  }
});

watch(
  todos,
  (val) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(val));
  },
  { deep: true }
);

/* ---------- CRUD ---------- */
const addTodo = () => {
  if (!newTodo.value.trim()) return;

  todos.value.unshift({
    id: Date.now(),
    text: newTodo.value,
    done: false,
  });

  newTodo.value = "";
};

const removeTodo = (id: number) => {
  todos.value = todos.value.filter((t) => t.id !== id);
};

const startEdit = (todo: Todo) => {
  todo.editing = true;
};

const finishEdit = (todo: Todo) => {
  todo.editing = false;
  if (!todo.text.trim()) {
    removeTodo(todo.id);
  }
};

/* ---------- Filter ---------- */
const filteredTodos = computed(() => {
  if (filter.value === "active") {
    return todos.value.filter((t) => !t.done);
  }
  if (filter.value === "completed") {
    return todos.value.filter((t) => t.done);
  }
  return todos.value;
});

const remainingCount = computed(
  () => todos.value.filter((t) => !t.done).length
);
</script>

<template>
  <div class="min-h-screen flex justify-center bg-gray-100 p-6">
    <UCard class="w-full max-w-lg">
      <template #header>
        <h1 class="text-xl font-bold">📝 Todo List</h1>
      </template>

      <!-- 新增 -->
      <div class="flex gap-2 mb-4">
        <UInput
          v-model="newTodo"
          placeholder="新增待辦事項"
          @keyup.enter="addTodo"
        />
        <UButton color="primary" @click="addTodo"> 新增 </UButton>
      </div>

      <!-- 篩選 -->
      <div class="flex gap-2 mb-4">
        <UButton
          size="xs"
          :variant="filter === 'all' ? 'solid' : 'outline'"
          @click="filter = 'all'"
        >
          全部
        </UButton>
        <UButton
          size="xs"
          :variant="filter === 'active' ? 'solid' : 'outline'"
          @click="filter = 'active'"
        >
          未完成
        </UButton>
        <UButton
          size="xs"
          :variant="filter === 'completed' ? 'solid' : 'outline'"
          @click="filter = 'completed'"
        >
          已完成
        </UButton>
      </div>

      <!-- 清單 -->
      <ul class="space-y-2">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          class="flex items-center justify-between bg-white p-2 rounded shadow-sm"
        >
          <div class="flex items-center gap-2 flex-1">
            <input type="checkbox" v-model="todo.done" class="accent-primary" />

            <!-- 編輯 input -->
            <input
              v-if="todo.editing"
              v-model="todo.text"
              class="flex-1 border rounded px-2 py-1"
              @keyup.enter="finishEdit(todo)"
              @blur="finishEdit(todo)"
            />

            <!-- 顯示文字 -->
            <span
              v-else
              class="flex-1"
              :class="{ 'line-through text-gray-400': todo.done }"
            >
              {{ todo.text }}
            </span>
          </div>

          <!-- 操作按鈕 -->
          <div class="flex gap-1">
            <!-- ✏️ 編輯 -->
            <UButton
              icon="i-heroicons-pencil-square"
              color="gray"
              variant="ghost"
              @click="startEdit(todo)"
            />

            <!-- 🗑️ 刪除 -->
            <UButton
              icon="i-heroicons-trash"
              color="red"
              variant="ghost"
              @click="removeTodo(todo.id)"
            />
          </div>
        </li>
      </ul>

      <template #footer>
        <p class="text-sm text-gray-500">尚未完成 {{ remainingCount }} 筆</p>
      </template>
    </UCard>
  </div>
</template>
