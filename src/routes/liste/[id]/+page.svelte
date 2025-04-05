<script>
  import { page } from "$app/stores";
  import TaskItem from "$lib/components/TaskItem.svelte";
  import { onMount } from "svelte";

  let listId = "";
  let tasks = [];
  let selectedTaskIds = new Set();

  // Génère la clé de stockage en fonction de l'ID de la liste
  function getStorageKey() {
    return `todolist-${listId}`;
  }

  // Charger les tâches à l’ouverture
  onMount(() => {
    const unsubscribe = page.subscribe(({ params }) => {
      listId = params.id;

      const data = localStorage.getItem(getStorageKey());
      if (data) {
        try {
          const parsed = JSON.parse(data);
          if (Array.isArray(parsed)) {
            tasks = parsed;
          }
        } catch (e) {
          console.error("Erreur de parsing des tâches :", e);
        }
      }
    });

    return unsubscribe;
  });

  // Sauvegarder dans localStorage
  function saveTasks() {
    localStorage.setItem(getStorageKey(), JSON.stringify(tasks));
  }

  // Fonctions de recherche
  function findTaskById(taskList, id) {
    for (const task of taskList) {
      if (task.id === id) return task;
      const found = findTaskById(task.children, id);
      if (found) return found;
    }
    return null;
  }

  function findParentTask(taskList, childId, parent = null) {
    for (const task of taskList) {
      if (task.id === childId) return parent;
      const found = findParentTask(task.children, childId, task);
      if (found) return found;
    }
    return null;
  }

  function deleteTaskById(taskList, ids) {
    return taskList.filter(task => {
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
      children: task.children.map(duplicateTask),
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
    saveTasks();
  }

  function selectTask(taskId) {
    if (selectedTaskIds.has(taskId)) {
      selectedTaskIds.delete(taskId);
    } else {
      selectedTaskIds.add(taskId);
    }
    selectedTaskIds = new Set(selectedTaskIds);
  }

  function addSubtaskTo(task) {
    const newSubtask = {
      id: Date.now(),
      text: "Nouvelle sous-tâche",
      completed: false,
      children: []
    };
    task.children = [...task.children, newSubtask];
    saveTasks();
  }

  function handleAdd() {
    const newTask = {
      id: Date.now(),
      text: selectedTaskIds.size ? "Nouvelle sous-tâche" : "Nouvelle tâche",
      completed: false,
      children: []
    };

    if (selectedTaskIds.size > 0) {
      for (const id of selectedTaskIds) {
        const parent = findTaskById(tasks, id);
        if (parent) {
          parent.children = [...parent.children, newTask];
        }
      }
    } else {
      tasks = [...tasks, newTask];
    }
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
      const newText = prompt("Modifier la tâche :", task.text);
      if (newText) {
        task.text = newText;
        saveTasks();
      }
    }
  }

  function handleDelete() {
    tasks = deleteTaskById(tasks, selectedTaskIds);
    selectedTaskIds = new Set();
    saveTasks();
  }

  function handleDuplicate() {
    const clones = [];

    for (const id of selectedTaskIds) {
      const task = findTaskById(tasks, id);
      const parent = findParentTask(tasks, id);
      if (task) {
        const clone = duplicateTask(task);
        if (parent) {
          parent.children = [...parent.children, clone];
        } else {
          clones.push(clone);
        }
      }
    }

    if (clones.length) {
      tasks = [...tasks, ...clones];
    }

    selectedTaskIds = new Set();
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
    <button on:click={handleAdd}>Add</button>
    <button on:click={handleEdit} disabled={selectedTaskIds.size !== 1}>Edit</button>
    <button on:click={handleDuplicate} disabled={selectedTaskIds.size === 0}>Duplicate</button>
    <button on:click={handleDelete} disabled={selectedTaskIds.size === 0}>Delete</button>
  </div>

  <div class="task-list">
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
  </div>
</div>
