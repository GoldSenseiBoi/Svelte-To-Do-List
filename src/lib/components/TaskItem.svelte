<script>
    // ⛑️ Auto-import pour éviter le bug récursif "TaskItem is not defined"
    import TaskItem from "./TaskItem.svelte";
  
    export let task;
    export let level = 0;
    export let selectedTaskIds;
    export let selectTask;
    export let toggleCheck;
    export let addSubtaskTo;
  </script>
  
  <div class="task" style="margin-left: {level * 20}px">
    <div class="task-row">
      <input
        type="checkbox"
        checked={selectedTaskIds.has(task.id)}
        on:change={() => toggleCheck(task)}
      />
      <span
        class:selected={selectedTaskIds.has(task.id)}
        on:click={() => selectTask(task.id)}
      >
        {task.text}
      </span>
      <button on:click={() => addSubtaskTo(task)}>Sous-liste</button>
    </div>
  
    {#if task.children && task.children.length > 0}
      {#each task.children as child (child.id)}
        <TaskItem
          task={child}
          level={level + 1}
          {selectedTaskIds}
          {selectTask}
          {toggleCheck}
          {addSubtaskTo}
        />
      {/each}
    {/if}
  </div>
  
  <style>
    .task-row {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      border-bottom: 1px solid #eee;
      padding: 0.5rem 0;
    }
  
    .task-row span {
      flex: 1;
      cursor: pointer;
    }
  
    .task-row span.selected {
      font-weight: bold;
      text-decoration: underline;
    }
  </style>
  