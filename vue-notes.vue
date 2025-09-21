<script setup>
// `script setup` is Vue's composition API (very easy to use)
// It allows you to declare reactive state, imports, functions, etc. directly
// without needing to return them — everything is automatically exposed to the template.
import { ref, onMounted, computed, watch } from "vue";

// `ref` is used to create reactive variables that can hold any value type,
// including deeply nested objects, arrays, or JavaScript built-in data structures like Map
const name = ref("John Doe");
const status = ref("active");
const tasks = ref(["Task One", "Task Two", "Task Three"]);
const newTask = ref("");

// `computed` creates reactive values derived from other reactive sources.
// Computed properties are cached until dependencies change.
const taskCount = computed(() => tasks.value.length);

// `watch` runs a callback whenever the watched source changes.
// Useful for side effects like logging, fetching data, or validation.
watch(status, (newVal, oldVal) => {
  console.log(`Status changed from ${oldVal} to ${newVal}`);
});

// we can declare functions that mutate refs in the same scope and expose them as methods alongside the state:
// Exposed methods can then be used as event handlers:
const toggleStatus = () => {
  if (status.value === "active") {
    status.value = "pending";
  } else if (status.value === "pending") {
    status.value = "inactive";
  } else {
    status.value = "active";
  }
};

const addTask = () => {
  if (newTask.value.trim() !== "") {
    tasks.value.push(newTask.value);
    newTask.value = "";
  }
};

const deleteTask = (index) => {
  tasks.value.splice(index, 1);
};

// `onMounted` is a lifecycle hook that runs code after the component is mounted to the DOM.
onMounted(async () => {
  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/todos");
    const data = await response.json();
    tasks.value = data.map((task) => task.title);
  } catch (error) {
    console.log("Error fetching tasks");
  }
});
</script>

<template>
  <!-- The most basic form of data binding is text interpolation using the "Mustache" syntax (double curly braces): -->
  <!-- The double mustaches interpret the data as plain text, not HTML. -->
  <h1>{{ name }}</h1>

  <!-- v-if directive conditionally renders elements based on reactive state. -->
  <p v-if="status === 'active'">User is active</p>
  <p v-else-if="status === 'pending'">User is pending</p>
  <p v-else>User is inactive</p>

  <!-- v-show is similar to v-if, but will always be rendered and remain in the DOM,
       it toggles the CSS display property (better for toggling frequently). -->
  <p v-show="status === 'inactive'">Inactive users are hidden with v-show</p>

  <!-- `@submit.prevent` is an event modifier.
       - `@` is shorthand for v-on.
       - `.prevent` prevents the default form submission behavior. -->
  <form @submit.prevent="addTask">
    <label for="newTask">Add Task</label>

    <!-- v-model creates two-way binding between input value and the reactive variable `newTask`. -->
    <input type="text" id="newTask" name="newTask" v-model="newTask" />

    <button type="submit">Submit</button>
  </form>

  <h3>Number of Tasks: {{ taskCount }}</h3>
  <ul>
    <!-- v-for loops through arrays/objects to render a list.
         The `(task, index)` syntax gives access to both item and index.
         The `:key` attribute provides a unique identifier for efficient DOM updates. -->
    <li
      v-for="(task, index) in tasks"
      :key="task"
      :class="{ important: index === 0 }"
      :style="{ color: index % 2 === 0 ? 'blue' : 'green' }"
    >
      <!-- `:class` dynamically binds classes based on conditions. -->
      <!-- `:style` dynamically binds inline styles. -->
      <span>
        {{ task }}
      </span>
      <!-- @click is shorthand for v-on:click — it listens for a click event. -->
      <button @click="deleteTask(index)">x</button>
    </li>
  </ul>
  <br />

  <!-- Another usage of event binding with @click to trigger a function. -->
  <button @click="toggleStatus">Change Status</button>

  <!-- v-bind (shorthand `:`) dynamically sets HTML attributes. -->
  <button :disabled="status === 'inactive'">Disabled if inactive</button>
</template>

<style scoped>
/* Scoped styles only apply to this component instance. */
.important {
  font-weight: bold;
}
</style>
