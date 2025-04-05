<script>
  import { page } from "$app/stores";
  import TaskItem from "$lib/components/TaskItem.svelte";
  import { onMount } from "svelte";

  let listId = "";
  let tasks = [];
  let selectedTaskIds = new Set();
  let ready = false;

  function getStorageKey() {
    return `todolist-${listId}`;
  }

  onMount(() => {
    const unsubscribe = page.subscribe(({ params }) => {
      if (params.id && params.id.trim() !== "") {
        listId = params.id;

        if (typeof localStorage !== 'undefined') {
          const key = getStorageKey();
          const raw = localStorage.getItem(key);

          try {
            const parsed = JSON.parse(raw);
            tasks = Array.isArray(parsed) ? parsed : [];
          } catch (e) {
            console.error("Erreur JSON corrompu :", e);
            tasks = [];
          }
        }

        ready = true;
      }
    });

    return unsubscribe;
  });

  function saveTasks() {
    if (!ready || !listId || typeof localStorage === 'undefined') return;

    try {
      localStorage.setItem(getStorageKey(), JSON.stringify(tasks));
    } catch (e) {
      console.error("Erreur de sauvegarde localStorage :", e);
    }
  }

  function findTaskById(list, id) {
    for (let task of list) {
      if (task.id === id) return task;
      const found = findTaskById(task.children, id);
      if (found) return found;
    }
    return null;
  }

  function findParentTask(list, id, parent = null) {
    for (let task of list) {
      if (task.id === id) return parent;
      const found = findParentTask(task.children, id, task);
      if (found) return found;
    }
    return null;
  }

  function deleteTaskById(list, ids) {
    return list.filter(task => {
      if (ids.has(task.id)) return false;
      task.children = deleteTaskById(task.children, ids);
      return true;
    });
  }

  function duplicateTask(task) {
    return {
      ...task,
      id: Date.now() + Math.random(),
      text: task.text + " (copie)",
      children: task.children.map(duplicateTask)
    };
  }

  function toggleCheck(task) {
    task.completed = !task.completed;
    if (selectedTaskIds.has(task.id)) {
      selectedTaskIds.delete(task.id);
    } else {
      selectedTaskIds.add(task.id);
    }
    selectedTaskIds = new Set(selectedTaskIds);
    tasks = [...tasks]; // trigger reactive update
    saveTasks();
  }

  function selectTask(id) {
    if (selectedTaskIds.has(id)) {
      selectedTaskIds.delete(id);
    } else {
      selectedTaskIds.add(id);
    }
    selectedTaskIds = new Set(selectedTaskIds);
  }

  function addSubtaskTo(task) {
    const subtask = {
      id: Date.now(),
      text: "Nouvelle sous-tâche",
      completed: false,
      children: []
    };
    task.children = [...task.children, subtask];
    tasks = [...tasks]; // ensure reactivity
    saveTasks();
  }

  function handleAdd() {
    const task = {
      id: Date.now(),
      text: selectedTaskIds.size ? "Nouvelle sous-tâche" : "Nouvelle tâche",
      completed: false,
      children: []
    };

    if (selectedTaskIds.size) {
      for (const id of selectedTaskIds) {
        const parent = findTaskById(tasks, id);
        if (parent) {
          parent.children = [...parent.children, task];
        }
      }
    } else {
      tasks = [...tasks, task];
    }

    tasks = [...tasks]; // trigger reactivity
    saveTasks();
  }

  function handleEdit() {
    if (selectedTaskIds.size !== 1) {
      alert("Sélectionne une seule tâche.");
      return;
    }
    const id = [...selectedTaskIds][0];
    const task = findTaskById(tasks, id);
    if (task) {
      const text = prompt("Modifier :", task.text);
      if (text) {
        task.text = text;
        tasks = [...tasks]; // trigger reactivity
        saveTasks();
      }
    }
  }

  function handleDelete() {
    tasks = deleteTaskById(tasks, selectedTaskIds);
    selectedTaskIds = new Set();
    tasks = [...tasks]; // trigger reactivity
    saveTasks();
  }

  function handleDuplicate() {
    const copies = [];

    for (const id of selectedTaskIds) {
      const task = findTaskById(tasks, id);
      const parent = findParentTask(tasks, id);
      if (task) {
        const clone = duplicateTask(task);
        if (parent) {
          parent.children = [...parent.children, clone];
        } else {
          copies.push(clone);
        }
      }
    }

    if (copies.length) {
      tasks = [...tasks, ...copies];
    }

    selectedTaskIds = new Set();
    tasks = [...tasks]; // trigger reactivity
    saveTasks();
  }
</script>

<style>
  .container {
    padding: 2rem;
    max-width: 600px;
    margin: auto;
  }

  .buttons {
    display: flex;
    gap: 10px;
    margin-bottom: 1rem;
  }

  .task-list {
    background: #fff;
    border-radius: 8px;
    padding: 1rem;
    border: 1px solid #ddd;
  }

  button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  h1 {
    font-size: 1.8rem;
    margin-bottom: 1rem;
  }
</style>

<div class="container">
  <a href="/">← Retour</a>
  <h1>Ma liste</h1>

  <div class="buttons">
    <button on:click={handleAdd} disabled={!ready}>Add</button>
    <button on:click={handleEdit} disabled={selectedTaskIds.size !== 1 || !ready}>Edit</button>
    <button on:click={handleDuplicate} disabled={selectedTaskIds.size === 0 || !ready}>Duplicate</button>
    <button on:click={handleDelete} disabled={selectedTaskIds.size === 0 || !ready}>Delete</button>
  </div>

  <div class="task-list">
    {#if ready}
      {#each tasks as task (task.id)}
        <TaskItem
          {task}
          level={0}
          {selectedTaskIds}
          {selectTask}
          {toggleCheck}
          {addSubtaskTo}
        />
      {/each}
    {/if}
  </div>
</div>
