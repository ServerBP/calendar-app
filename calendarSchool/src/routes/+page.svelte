<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import { tick } from 'svelte';

  // Create stores for Week A and Week B schedules
  const weekAStore = writable(JSON.parse(localStorage.getItem('weekA')) || {
    Monday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Tuesday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Wednesday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Thursday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Friday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time']
  });

  const weekBStore = writable(JSON.parse(localStorage.getItem('weekB')) || {
    Monday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Tuesday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Wednesday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Thursday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time'],
    Friday: ['Homeroom', 'Lesson 1', 'Lesson 2', 'Break', 'Lesson 3', 'Lesson 4', 'Lunch', 'Lesson 5', 'Club Time']
  });

  let weekA, weekB;
  weekAStore.subscribe(value => {
    weekA = value;
    localStorage.setItem('weekA', JSON.stringify(value));
  });
  weekBStore.subscribe(value => {
    weekB = value;
    localStorage.setItem('weekB', JSON.stringify(value));
  });

  let currentWeek = 'A';
  let currentSchedule = weekA;

  const lessonTimes = [
    '8:30-8:50', '8:50-9:50', '9:50-10:50', '10:50-11:10',
    '11:10-12:10', '12:10-13:10', '13:10-14:10', '14:10-15:10', '15:10-16:00'
  ];

  const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'];

  let upcomingLesson = '';
  let currentLesson = '';
  let editMode = false;

  let lessonProgressPercentage = 0;
  let intervalId;

  function updateTimeAndLessons() {
    const upcoming = getUpcomingLesson();
    upcomingLesson = upcoming.lesson;
    const upcomingElement = document.getElementById('upcoming-lesson');
    if (upcomingElement) {
      upcomingElement.style.boxShadow = `0 0 15px ${upcoming.glowColor}`;
    }

    const current = getCurrentLesson();
    currentLesson = current.lesson;
    const currentElement = document.getElementById('current-lesson');
    if (currentElement) {
      currentElement.style.boxShadow = `0 0 15px ${current.glowColor}`;
    }

    updateLessonProgress();
  }

  function switchWeek(week) {
    currentWeek = week;
    currentSchedule = week === 'A' ? weekA : weekB;
  }

  function getUpcomingLesson() {
    const now = new Date();
    const day = now.getDay() - 1; // 0 is Monday, -1 is Sunday
    const time = now.getHours() * 60 + now.getMinutes();

    if (day < 0 || day > 4) {
      return { lesson: 'No lessons today', glowColor: 'grey' };
    }

    for (let i = 0; i < lessonTimes.length; i++) {
      const [start, end] = lessonTimes[i].split('-').map(t => {
        const [h, m] = t.split(':').map(Number);
        return h * 60 + m;
      });

      if (time < start) {
        const minutesUntil = start - time;
        const glowColor = minutesUntil > 50 ? 'green' : 
                          minutesUntil > 15 ? 'yellow' : 'red';
        return {
          lesson: `${currentSchedule[days[day]][i]} (${lessonTimes[i]})`,
          glowColor
        };
      }
    }

    return { lesson: 'No more lessons today', glowColor: 'grey' };
  }

  function getCurrentLesson() {
    const now = new Date();
    const day = now.getDay() - 1; // 0 is Monday, -1 is Sunday
    const time = now.getHours() * 60 + now.getMinutes();

    if (day < 0 || day > 4) {
      return { lesson: 'No lessons today', glowColor: 'grey' };
    }

    for (let i = 0; i < lessonTimes.length; i++) {
      const [start, end] = lessonTimes[i].split('-').map(t => {
        const [h, m] = t.split(':').map(Number);
        return h * 60 + m;
      });

      if (time >= start && time < end) {
        const lesson = currentSchedule[days[day]][i];
        const glowColor = lesson === 'Break' || lesson === 'Lunch' ? 'grey' : 'blue';
        return { lesson: `${lesson} (${lessonTimes[i]})`, glowColor };
      }
    }

    return { lesson: 'No current lesson', glowColor: 'grey' };
  }


  function toggleEditMode() {
    if (editMode) {
      saveSchedule();
    }
    editMode = !editMode;
  }

  function toggleTodoComplete(todo) {
    updateTodo(todo.id, { category: todo.category === 'Completed' ? 'To Do' : 'Completed' });
  }

  function updateSchedule(day, index, value) {
    if (currentWeek === 'A') {
      weekA[day][index] = value;
      weekAStore.set(weekA);
    } else {
      weekB[day][index] = value;
      weekBStore.set(weekB);
    }
  }

  function saveSchedule() {
    localStorage.setItem('weekA', JSON.stringify(weekA));
    localStorage.setItem('weekB', JSON.stringify(weekB));
  }

  function copyTimetable() {
    const timetableData = JSON.stringify({ weekA, weekB });
    navigator.clipboard.writeText(timetableData).then(() => {
      alert('Timetable copied to clipboard!');
    }, (err) => {
      console.error('Could not copy text: ', err);
    });
  }

  async function loadTimetable() {
    try {
      const timetableData = await navigator.clipboard.readText();
      const { weekA: newWeekA, weekB: newWeekB } = JSON.parse(timetableData);
      weekAStore.set(newWeekA);
      weekBStore.set(newWeekB);
      location.reload(); // Refresh the page
    } catch (err) {
      console.error('Could not load timetable: ', err);
      alert('Failed to load timetable. Make sure you have a valid timetable data in your clipboard.');
    }
  }

  // New code for to-do list
  const todoStore = writable(JSON.parse(localStorage.getItem('todos')) || []);

  let todos;
  todoStore.subscribe(value => {
    todos = value;
    localStorage.setItem('todos', JSON.stringify(value));
  });

  function addTodo(category) {
    const newTodo = {
      id: Date.now(),
      text: '',
      category,
      labels: []
    };
    todoStore.update(todos => [...todos, newTodo]);
  }

  function updateTodo(id, updates) {
    todoStore.update(todos => 
      todos.map(todo => todo.id === id ? { ...todo, ...updates } : todo)
    );
  }

  function deleteTodo(id) {
    todoStore.update(todos => todos.filter(todo => todo.id !== id));
  }

  function addLabel(todoId, label) {
    todoStore.update(todos => 
      todos.map(todo => 
        todo.id === todoId 
          ? { ...todo, labels: [...todo.labels, label] }
          : todo
      )
    );
  }

  function removeLabel(todoId, label) {
    todoStore.update(todos => 
      todos.map(todo => 
        todo.id === todoId 
          ? { ...todo, labels: todo.labels.filter(l => l !== label) }
          : todo
      )
    );
  }

  let draggedTodo = null;
  let dragOverCategory = null;

  function handleDragStart(todo) {
    draggedTodo = todo;
  }

  function handleDragOver(category, e) {
    e.preventDefault();
    dragOverCategory = category;
  }

  function handleDrop(category) {
    if (draggedTodo && draggedTodo.category !== category) {
      updateTodo(draggedTodo.id, { category });
    }
    draggedTodo = null;
    dragOverCategory = null;
  }

  function updateLessonProgress() {
    const now = new Date();
    const time = now.getHours() * 60 + now.getMinutes();

    const currentLessonInfo = getCurrentLesson();
    if (currentLessonInfo.lesson === 'No current lesson' || currentLessonInfo.lesson.includes('Break') || currentLessonInfo.lesson.includes('Lunch')) {
      lessonProgressPercentage = 0;
      return;
    }

    const [, lessonTime] = currentLessonInfo.lesson.split('(');
    const [start, end] = lessonTime.slice(0, -1).split('-').map(t => {
      const [h, m] = t.split(':').map(Number);
      return h * 60 + m;
    });

    const totalDuration = end - start;
    const elapsed = time - start;
    lessonProgressPercentage = Math.min(100, Math.max(0, (elapsed / totalDuration) * 100));
  }

  function autoResizeTextarea(node) {
    function resize() {
      node.style.height = 'auto';
      node.style.height = node.scrollHeight + 'px';
    }
    
    node.addEventListener('input', resize);
    
    return {
      destroy() {
        node.removeEventListener('input', resize);
      }
    };
  }

  async function handleTextareaInput(event, todo) {
    updateTodo(todo.id, { text: event.target.value });
    await tick();
    event.target.style.height = 'auto';
    event.target.style.height = event.target.scrollHeight + 'px';
  }

  function handleVisibilityChange() {
    if (document.visibilityState === 'visible') {
      startClock();
    } else {
      clearInterval(intervalId);
    }
  }

  function startClock() {
    clearInterval(intervalId); // Clear any existing interval
    updateTimeAndLessons(); // Update immediately
    intervalId = setInterval(updateTimeAndLessons, 60000); // Then update every minute
  }

  function handleFocus() {
    startClock();
  }

  onMount(() => {
    setInterval(() => {
      const upcoming = getUpcomingLesson();
      upcomingLesson = upcoming.lesson;
      document.getElementById('upcoming-lesson').style.boxShadow = `0 0 15px ${upcoming.glowColor}`;

      const current = getCurrentLesson();
      currentLesson = current.lesson;
      document.getElementById('current-lesson').style.boxShadow = `0 0 15px ${current.glowColor}`;

      updateLessonProgress();
    }, 60000); // Update every minute
    const upcoming = getUpcomingLesson();
    upcomingLesson = upcoming.lesson;
    document.getElementById('upcoming-lesson').style.boxShadow = `0 0 15px ${upcoming.glowColor}`;

    const current = getCurrentLesson();
    currentLesson = current.lesson;
    document.getElementById('current-lesson').style.boxShadow = `0 0 15px ${current.glowColor}`;

    updateLessonProgress();

    startClock();

    // Add event listeners for visibility change and focus
    document.addEventListener('visibilitychange', handleVisibilityChange);
    window.addEventListener('focus', handleFocus);

    return () => {
      clearInterval(intervalId);
      document.removeEventListener('visibilitychange', handleVisibilityChange);
      window.removeEventListener('focus', handleFocus);
    };
  });
</script>


<main>
  <div class="week-selector">
    <button class:active={currentWeek === 'A'} on:click={() => switchWeek('A')}>Week A</button>
    <button class:active={currentWeek === 'B'} on:click={() => switchWeek('B')}>Week B</button>
  </div>

  <div class="content">
    <div class="sidebar">
      <div id="upcoming-lesson" class="lesson-box">
        <h2>Upcoming Lesson:</h2>
        <p>{upcomingLesson}</p>
      </div>
      <div id="current-lesson" class="lesson-box">
        <h2>Current Lesson:</h2>
        <p>{currentLesson}</p>
        <div class="progress-bar">
          <div class="progress" style="width: {lessonProgressPercentage}%"></div>
        </div>
      </div>
    </div>

    <div class="timetable">
      <table>
        <thead>
          <tr>
            <th>Time</th>
            {#each days as day}
              <th>{day}</th>
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each lessonTimes as time, i}
            <tr>
              <td>{time}</td>
              {#each days as day}
                <td>
                  {#if editMode}
                    <input 
                      type="text" 
                      value={currentSchedule[day][i]} 
                      on:input={(e) => updateSchedule(day, i, e.target.value)}
                    />
                  {:else}
                    {currentSchedule[day][i]}
                  {/if}
                </td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>

  <div class="button-container">
    <button class="edit-button" on:click={toggleEditMode}>
      <span class="material-symbols-outlined">
        {editMode ? 'save' : 'edit'}
      </span>
      {editMode ? 'Save' : 'Edit'} Timetable
    </button>
    <button class="copy-button" on:click={copyTimetable}>
      <span class="material-symbols-outlined">copy_all</span>
      Copy Timetable
    </button>
    <button class="load-button" on:click={loadTimetable}>
      <span class="material-symbols-outlined">download</span>
      Load Timetable
    </button>
  </div>

  <div class="todo-container">
    {#each ['To Do', 'In Progress', 'Completed'] as category}
      <div 
        class="todo-category"
        on:dragover|preventDefault={(e) => handleDragOver(category, e)}
        on:drop|preventDefault={() => handleDrop(category)}
      >
        <h3>
          {category}
          <span class="material-symbols-outlined">
            {category === 'To Do' ? 'playlist_add' : category === 'In Progress' ? 'pending' : 'task_alt'}
          </span>
        </h3>
        {#each todos.filter(todo => todo.category === category) as todo (todo.id)}
          <div 
            class="todo-item"
            draggable="true"
            on:dragstart={() => handleDragStart(todo)}
          >
            <div class="todo-content">
              <button 
                class="todo-checkbox" 
                on:click={() => toggleTodoComplete(todo)}
              >
                <span class="material-symbols-outlined">
                  {todo.category === 'Completed' ? 'check_box' : 'check_box_outline_blank'}
                </span>
              </button>
              <textarea 
                bind:value={todo.text} 
                on:input={() => updateTodo(todo.id, { text: todo.text })}
                placeholder="Enter task..."
                rows="1"
              ></textarea>
            </div>
            <div class="todo-actions">
              <div class="todo-labels">
                {#each todo.labels as label}
                  <span class="todo-label">
                    {label}
                    <button on:click={() => removeLabel(todo.id, label)}>×</button>
                  </span>
                {/each}
                <input 
                  type="text" 
                  placeholder="Add label" 
                  on:keydown={(e) => {
                    if (e.key === 'Enter' && e.target.value) {
                      addLabel(todo.id, e.target.value);
                      e.target.value = '';
                    }
                  }}
                />
              </div>
              <button class="delete-todo" on:click={() => deleteTodo(todo.id)}>
                <span class="material-symbols-outlined">delete</span>
              </button>
            </div>
          </div>
        {/each}
        <button class="add-todo" on:click={() => addTodo(category)}>
          <span class="material-symbols-outlined">add</span>
          Add Task
        </button>
      </div>
    {/each}
  </div>
</main>

<style>
  :global(body) {
    background-color: #121212;
    color: #ffffff;
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
  }

  main {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }

  .week-selector {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
  }

  .week-selector button {
    background-color: #333;
    border: none;
    color: white;
    padding: 10px 20px;
    font-size: 18px;
    font-weight: bold;
    cursor: pointer;
    margin: 0 10px;
    transition: all 0.3s ease;
    border-radius: 25px;
  }

  .week-selector button.active {
    background-color: #555;
    box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
  }

  .content {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
  }

  .sidebar {
    width: 300px;
    margin-right: 20px;
  }

  .lesson-box {
    padding: 20px;
    background-color: #1e1e1e;
    border: 1px solid #444;
    border-radius: 10px;
    margin-bottom: 20px;
    transition: box-shadow 0.3s ease;
  }

  .timetable {
    flex-grow: 1;
    overflow-x: auto;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    table-layout: fixed;
  }

  th, td {
    border: 1px solid #444;
    padding: 10px;
    text-align: center;
    width: calc(100% / 6);
    word-wrap: break-word;
    overflow-wrap: break-word;
  }

  th {
    background-color: #333;
  }

  tr:nth-child(even) {
    background-color: #1e1e1e;
  }

  .edit-mode {
    margin-top: 20px;
    padding: 20px;
    background-color: #1e1e1e;
    border-radius: 10px;
  }

  .edit-day {
    margin-bottom: 10px;
  }

  .edit-day label {
    display: inline-block;
    width: 100px;
  }

  .edit-day input {
    width: calc(100% - 120px);
    padding: 5px;
    background-color: #333;
    color: white;
    border: 1px solid #444;
    border-radius: 5px;
  }

  .edit-mode button {
    margin-top: 20px;
    padding: 10px 20px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
  }

  .button-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 20px;
  }

  .edit-button, .copy-button, .load-button {
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 10px;
    padding: 10px 20px;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    transition: all 0.3s ease;
  }

  .edit-button {
    background-color: #2c3e50;
  }

  .copy-button {
    background-color: #34495e;
  }

  .load-button {
    background-color: #2c3e50;
  }

  .edit-button:hover, .copy-button:hover, .load-button:hover {
    box-shadow: 0 0 15px rgba(255, 255, 255, 0.3);
    opacity: 0.8;
  }

  .edit-button span, .copy-button span, .load-button span {
    margin-right: 0.6rem;
    font-size: 1.5rem;
  }

  .timetable td {
    position: relative;
  }

  .timetable td input {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    border: none;
    background: transparent;
    color: white;
    text-align: center;
    font-size: inherit;
    padding: 10px;
    box-sizing: border-box;
  }

  .timetable td input:focus {
    background-color: #2c3e50;
    outline: none;
  }

  .progress-bar {
    width: 100%;
    height: 10px;
    background-color: #1e1e1e;
    border-radius: 5px;
    overflow: hidden;
    margin-top: 10px;
  }

  .progress {
    height: 100%;
    background: linear-gradient(to right, #3498db, #2ecc71);
    transition: width 0.5s ease-in-out;
  }

  .todo-container {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
  }

  .todo-category {
    flex: 1;
    background-color: #1e1e1e;
    border-radius: 10px;
    padding: 20px;
    margin: 0 10px;
    min-height: 300px;
  }

  .todo-category h3 {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 15px;
  }

  .todo-item {
    background-color: #2c3e50;
    border-radius: 5px;
    padding: 10px;
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
    cursor: move;
  }

  .todo-content {
    display: flex;
    align-items: flex-start;
    margin-bottom: 10px;
  }

  .todo-checkbox {
    background: none;
    border: none;
    color: white;
    cursor: pointer;
    padding: 0;
    margin-right: 10px;
    transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
    flex-shrink: 0;
  }

  .todo-checkbox:hover {
    transform: scale(1.1);
    box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
  }

  .todo-item textarea {
    flex-grow: 1;
    background: transparent;
    border: none;
    color: white;
    resize: vertical;
    min-height: 24px;
    font-family: inherit;
    font-size: inherit;
    line-height: 1.5;
    overflow-y: hidden;
  }

  .todo-actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .todo-labels {
    display: flex;
    flex-wrap: wrap;
    flex-grow: 1;
    margin-right: 10px;
  }

  .todo-label {
    background-color: #34495e;
    color: white;
    border-radius: 3px;
    padding: 2px 5px;
    margin-right: 5px;
    margin-bottom: 5px;
    font-size: 12px;
    display: flex;
    align-items: center;
  }

  .todo-label button {
    background: none;
    border: none;
    color: white;
    margin-left: 5px;
    cursor: pointer;
    transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
  }

  .todo-label button:hover {
    transform: scale(1.1);
    box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
  }

  .delete-todo {
    background: none;
    border: none;
    color: #e74c3c;
    cursor: pointer;
    transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
    flex-shrink: 0;
  }

  .delete-todo:hover {
    transform: scale(1.1);
    box-shadow: 0 0 10px rgba(231, 76, 60, 0.5);
  }

  .add-todo {
    width: 100%;
    padding: 10px;
    background-color: #34495e;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
  }

  .add-todo:hover {
    transform: scale(1.05);
    box-shadow: 0 0 15px rgba(255, 255, 255, 0.3);
  }

  .add-todo span {
    margin-right: 5px;
  }
</style>