<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

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
                          minutesUntil > 30 ? 'yellow' : 'red';
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

  onMount(() => {
    setInterval(() => {
      const upcoming = getUpcomingLesson();
      upcomingLesson = upcoming.lesson;
      document.getElementById('upcoming-lesson').style.boxShadow = `0 0 15px ${upcoming.glowColor}`;
      
      const current = getCurrentLesson();
      currentLesson = current.lesson;
      document.getElementById('current-lesson').style.boxShadow = `0 0 15px ${current.glowColor}`;
    }, 60000); // Update every minute
    const upcoming = getUpcomingLesson();
    upcomingLesson = upcoming.lesson;
    document.getElementById('upcoming-lesson').style.boxShadow = `0 0 15px ${upcoming.glowColor}`;
    
    const current = getCurrentLesson();
    currentLesson = current.lesson;
    document.getElementById('current-lesson').style.boxShadow = `0 0 15px ${current.glowColor}`;
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
      {editMode ? 'Save' : 'Edit'} Calendar
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
</style>