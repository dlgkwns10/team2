# team2
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>할 일 캘린더</title>

  <style>
    body {
      font-family: sans-serif;
      margin: 20px;
      display: flex;
      gap: 30px;
    }

    .calendar {
      width: 420px;
    }

    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 5px;
    }

    .day-name,
    .date {
      border: 1px solid #ddd;
      padding: 10px;
      text-align: center;
    }

    .day-name {
      background: #f0f0f0;
      font-weight: bold;
    }

    .date {
      height: 70px;
      cursor: pointer;
      position: relative;
    }

    .date:hover {
      background: #f9f9f9;
    }

    .selected {
      border: 2px solid #007bff;
    }

    .done-mark {
      position: absolute;
      bottom: 5px;
      right: 5px;
      font-size: 18px;
    }

    .todo-panel {
      width: 300px;
    }

    ul {
      padding: 0;
      list-style: none;
    }

    li {
      margin-bottom: 10px;
    }

    input[type="text"] {
      width: 70%;
      padding: 5px;
    }

    button {
      padding: 5px 10px;
    }
  </style>
</head>
<body>

  <div class="calendar">
    <div class="header">
      <button onclick="changeMonth(-1)">◀</button>
      <h2 id="monthYear"></h2>
      <button onclick="changeMonth(1)">▶</button>
    </div>

    <div class="grid" id="calendarGrid"></div>
  </div>

  <div class="todo-panel">
    <h2 id="selectedDateTitle">날짜 선택</h2>

    <input type="text" id="todoInput" placeholder="할 일 입력">
    <button onclick="addTodo()">추가</button>

    <ul id="todoList"></ul>
  </div>

<script>
  let currentDate = new Date();
  let selectedDate = null;

  let todos = JSON.parse(localStorage.getItem("todos")) || {};

  const calendarGrid = document.getElementById("calendarGrid");
  const monthYear = document.getElementById("monthYear");
  const todoList = document.getElementById("todoList");
  const selectedDateTitle = document.getElementById("selectedDateTitle");

  const dayNames = ["일", "월", "화", "수", "목", "금", "토"];

  function saveTodos() {
    localStorage.setItem("todos", JSON.stringify(todos));
  }

  function renderCalendar() {
    calendarGrid.innerHTML = "";

    dayNames.forEach(day => {
      const div = document.createElement("div");
      div.className = "day-name";
      div.innerText = day;
      calendarGrid.appendChild(div);
    });

    const year = currentDate.getFullYear();
    const month = currentDate.getMonth();

    monthYear.innerText = `${year}년 ${month + 1}월`;

    const firstDay = new Date(year, month, 1).getDay();
    const lastDate = new Date(year, month + 1, 0).getDate();

    for (let i = 0; i < firstDay; i++) {
      const empty = document.createElement("div");
      calendarGrid.appendChild(empty);
    }

    for (let date = 1; date <= lastDate; date++) {
      const div = document.createElement("div");
      div.className = "date";
      div.innerHTML = `<div>${date}</div>`;

      const dateKey = formatDate(year, month + 1, date);

      if (todos[dateKey] && todos[dateKey].length > 0) {
        const allDone = todos[dateKey].every(todo => todo.done);

        if (allDone) {
          const mark = document.createElement("div");
          mark.className = "done-mark";
          mark.innerText = "✅";
          div.appendChild(mark);
        }
      }

      div.onclick = () => selectDate(dateKey, div);

      calendarGrid.appendChild(div);
    }
  }

  function formatDate(year, month, date) {
    return `${year}-${String(month).padStart(2, "0")}-${String(date).padStart(2, "0")}`;
  }

  function selectDate(dateKey, element) {
    selectedDate = dateKey;

    document.querySelectorAll(".date").forEach(d => {
      d.classList.remove("selected");
    });

    element.classList.add("selected");

    selectedDateTitle.innerText = `${dateKey} 할 일`;

    renderTodos();
  }

  function renderTodos() {
    todoList.innerHTML = "";

    const list = todos[selectedDate] || [];

    list.forEach((todo, index) => {
      const li = document.createElement("li");

      li.innerHTML = `
        <input type="checkbox" ${todo.done ? "checked" : ""}
          onchange="toggleTodo(${index})">
        ${todo.text}
        <button onclick="deleteTodo(${index})">삭제</button>
      `;

      todoList.appendChild(li);
    });

    renderCalendar();
    saveTodos();
  }

  function addTodo() {
    const input = document.getElementById("todoInput");
    const text = input.value.trim();

    if (!selectedDate) {
      alert("날짜를 선택하세요.");
      return;
    }

    if (!text) return;

    if (!todos[selectedDate]) {
      todos[selectedDate] = [];
    }

    todos[selectedDate].push({
      text,
      done: false
    });

    input.value = "";

    renderTodos();
  }

  function toggleTodo(index) {
    todos[selectedDate][index].done =
      !todos[selectedDate][index].done;

    renderTodos();
  }

  function deleteTodo(index) {
    todos[selectedDate].splice(index, 1);

    renderTodos();
  }

  function changeMonth(offset) {
    currentDate.setMonth(currentDate.getMonth() + offset);
    renderCalendar();
  }

  renderCalendar();
</script>

</body>
</html>
