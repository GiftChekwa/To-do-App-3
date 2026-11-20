<script>
  import { onMount } from "svelte";

  // ⏰ Live clock
  let now = new Date();
  onMount(() => {
    const timer = setInterval(() => (now = new Date()), 1000);
    return () => clearInterval(timer);
  });

  // Turn a date into "2026-11-18" format
  function iso(d) {
    const y = d.getFullYear();
    const m = String(d.getMonth() + 1).padStart(2, "0");
    const day = String(d.getDate()).padStart(2, "0");
    return `${y}-${m}-${day}`;
  }
  function daysFromNow(n) {
    const d = new Date();
    d.setDate(d.getDate() + n);
    return iso(d);
  }

  // The form
  let newTask = "";
  let date = iso(new Date());
  let time = "09:00";

  // The list
  let tasks = [];

  function addTask() {
    if (newTask.trim() === "" || !date) return;
    tasks = [
      ...tasks,
      { id: Date.now(), text: newTask.trim(), done: false, date, time }
    ];
    newTask = "";
  }

  function toggleTask(id) {
    tasks = tasks.map((t) => (t.id === id ? { ...t, done: !t.done } : t));
  }

  function removeTask(id) {
    tasks = tasks.filter((t) => t.id !== id);
  }

  function when(task) {
    return new Date(`${task.date}T${task.time}`);
  }

  function formatWhen(task) {
    const d = when(task);
    const day = d.toLocaleDateString(undefined, {
      weekday: "short",
      month: "short",
      day: "numeric",
      year: "numeric"
    });
    const t = d.toLocaleTimeString([], { hour: "numeric", minute: "2-digit" });
    return `${day} • ${t}`;
  }

  function isOverdue(task, current) {
    return !task.done && when(task) < current;
  }

  $: sortedTasks = [...tasks].sort((a, b) => when(a) - when(b));
  $: remaining = tasks.filter((t) => !t.done).length;
</script>

<main>
  <div class="card">
    <!-- Clock -->
    <div class="clock">
      <div class="time">
        {now.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit", second: "2-digit" })}
      </div>
      <div class="date">
        {now.toLocaleDateString(undefined, {
          weekday: "long",
          year: "numeric",
          month: "long",
          day: "numeric"
        })}
      </div>
    </div>

    <h1>My To-Do List</h1>

    <!-- Add form -->
    <div class="form">
      <input
        class="text-input"
        bind:value={newTask}
        placeholder="What do you need to do?"
        on:keydown={(e) => e.key === "Enter" && addTask()}
      />

      <div class="row">
        <label>
          <span>Date</span>
          <input type="date" bind:value={date} />
        </label>

        <label>
          <span>Time</span>
          <input type="time" bind:value={time} />
        </label>
      </div>

      <!-- Quick buttons -->
      <div class="quick">
        <button type="button" on:click={() => (date = daysFromNow(0))}>Today</button>
        <button type="button" on:click={() => (date = daysFromNow(1))}>Tomorrow</button>
        <button type="button" on:click={() => (date = daysFromNow(7))}>Next week</button>
      </div>

      <button class="add" on:click={addTask}>＋ Add Task</button>
    </div>

    <!-- Task list -->
    <ul>
      {#each sortedTasks as task (task.id)}
        <li class:done={task.done} class:overdue={isOverdue(task, now)}>
          <button
            class="check"
            on:click={() => toggleTask(task.id)}
            aria-label="Mark as done"
          >
            {task.done ? "✅" : "⭕"}
          </button>

          <div class="info">
            <span class="task-text">{task.text}</span>
            <span class="task-when">
              {isOverdue(task, now) ? "⚠️ Overdue • " : "🕒 "}{formatWhen(task)}
            </span>
          </div>

          <button
            class="remove"
            on:click={() => removeTask(task.id)}
            aria-label="Delete task"
          >
            🗑️
          </button>
        </li>
      {/each}
    </ul>

    {#if tasks.length === 0}
      <p class="empty">Nothing to do yet. Add something! 🎈</p>
    {:else}
      <p class="count">
        {remaining} {remaining === 1 ? "task" : "tasks"} left
      </p>
    {/if}
  </div>
</main>

<style>
  /* 🍋 Lemon background (!important so nothing can cover it) */
  :global(html),
  :global(body) {
    margin: 0;
    min-height: 100vh;
    font-family: "Segoe UI", system-ui, -apple-system, sans-serif;
    background: #fff3a3 !important;
    background-image: none !important;
  }

  main {
    display: flex;
    justify-content: center;
    padding: 40px 16px;
  }

  .card {
    width: 100%;
    max-width: 480px;
    background: #f8efb2;
    border-radius: 24px;
    padding: 28px;
    box-shadow: 0 20px 50px rgba(0, 0, 0, 0.25);
  }

  /* Clock */
  .clock {
    text-align: center;
    padding: 16px;
    border-radius: 18px;
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    margin-bottom: 20px;
  }
  .time {
    font-size: 2.4rem;
    font-weight: 700;
    letter-spacing: 2px;
  }
  .date {
    opacity: 0.9;
    margin-top: 4px;
  }

  h1 {
    text-align: center;
    color: #3b2f63;
    margin: 0 0 18px;
    font-size: 1.6rem;
  }

  /* Form */
  .form {
    display: flex;
    flex-direction: column;
    gap: 12px;
    margin-bottom: 20px;
  }

  input {
    font-family: inherit;
    font-size: 1rem;
    padding: 12px 14px;
    border: 2px solid #e6e1f5;
    border-radius: 12px;
    background: #faf9ff;
    color: #3b2f63;
    outline: none;
    transition: border-color 0.2s, box-shadow 0.2s;
    box-sizing: border-box;
    width: 100%;
  }
  input:focus {
    border-color: #764ba2;
    box-shadow: 0 0 0 4px rgba(118, 75, 162, 0.15);
  }

  .row {
    display: flex;
    gap: 12px;
  }
  label {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 6px;
  }
  label span {
    font-size: 0.85rem;
    font-weight: 600;
    color: #6b5b95;
  }

  button {
    font-family: inherit;
    cursor: pointer;
    border: none;
  }

  /* Quick date buttons */
  .quick {
    display: flex;
    gap: 8px;
  }
  .quick button {
    flex: 1;
    padding: 8px;
    border-radius: 999px;
    background: #efeaff;
    color: #6b5b95;
    font-weight: 600;
    font-size: 0.85rem;
    transition: background 0.2s, transform 0.15s;
  }
  .quick button:hover {
    background: #764ba2;
    color: white;
    transform: translateY(-1px);
  }

  .add {
    padding: 14px;
    border-radius: 12px;
    font-size: 1.05rem;
    font-weight: 700;
    color: white;
    background: linear-gradient(135deg, #667eea, #764ba2);
    box-shadow: 0 6px 16px rgba(118, 75, 162, 0.4);
    transition: transform 0.15s, box-shadow 0.15s;
  }
  .add:hover {
    transform: translateY(-2px);
    box-shadow: 0 10px 20px rgba(118, 75, 162, 0.5);
  }
  .add:active {
    transform: translateY(0);
  }

  /* List */
  ul {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  li {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 14px;
    background: white;
    border-radius: 14px;
    border-left: 5px solid #667eea;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
    transition: transform 0.15s;
  }
  li:hover {
    transform: translateX(4px);
  }
  li.overdue {
    border-left-color: #e74c3c;
    background: #fff5f5;
  }
  li.done {
    border-left-color: #2ecc71;
    opacity: 0.65;
  }

  .info {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 4px;
    text-align: left;
  }
  .task-text {
    font-weight: 600;
    color: #3b2f63;
    word-break: break-word;
  }
  li.done .task-text {
    text-decoration: line-through;
    color: #999;
  }
  .task-when {
    font-size: 0.8rem;
    color: #8a7fb0;
  }
  li.overdue .task-when {
    color: #e74c3c;
    font-weight: 600;
  }

  .check,
  .remove {
    background: none;
    font-size: 1.3rem;
    padding: 4px;
    border-radius: 8px;
    transition: transform 0.15s;
  }
  .check:hover,
  .remove:hover {
    transform: scale(1.2);
  }

  .empty,
  .count {
    text-align: center;
    color: #8a7fb0;
    margin: 18px 0 0;
  }
  .count {
    font-weight: 600;
  }

  @media (max-width: 420px) {
    .row {
      flex-direction: column;
    }
    .time {
      font-size: 2rem;
    }
  }
</style>