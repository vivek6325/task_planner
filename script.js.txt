const calendar = document.getElementById('calendar');
const monthYear = document.getElementById('month-year');
const taskPopup = document.getElementById('task-popup');
const popupDate = document.getElementById('popup-date');
const taskList = document.getElementById('task-list');
const taskForm = document.getElementById('task-form');
const taskInput = document.getElementById('task-input');
const closePopupBtn = document.getElementById('close-popup');
const prevMonthBtn = document.getElementById('prev-month');
const nextMonthBtn = document.getElementById('next-month');

let currentDate = new Date();
let selectedDate = null;
let tasks = JSON.parse(localStorage.getItem('tasks')) || {};

function formatDate(date) {
    return date.toISOString().split('T')[0]; // YYYY-MM-DD
}

function renderCalendar() {
    calendar.innerHTML = '';

    const year = currentDate.getFullYear();
    const month = currentDate.getMonth();

    // Show current month and year
    const options = { month: 'long', year: 'numeric' };
    monthYear.textContent = currentDate.toLocaleDateString(undefined, options);

    const firstDay = new Date(year, month, 1).getDay();
    const daysInMonth = new Date(year, month + 1, 0).getDate();

    // Empty cells before 1st day
    for (let i = 0; i < firstDay; i++) {
        const emptyCell = document.createElement('div');
        calendar.appendChild(emptyCell);
    }

    // Day cells
    for (let day = 1; day <= daysInMonth; day++) {
        const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
        const dayDiv = document.createElement('div');
        dayDiv.classList.add('day');
        dayDiv.textContent = day;

        if (tasks[dateStr] && tasks[dateStr].length > 0) {
            dayDiv.classList.add('has-task');
        }

        dayDiv.addEventListener('click', () => openTaskPopup(dateStr));
        calendar.appendChild(dayDiv);
    }
}

function openTaskPopup(dateStr) {
    selectedDate = dateStr;
    popupDate.textContent = `Tasks for ${dateStr}`;
    taskList.innerHTML = '';

    const dayTasks = tasks[dateStr] || [];
    dayTasks.forEach((task, index) => {
        const li = document.createElement('li');
        li.textContent = task;
        li.title = 'Click to delete';
        li.addEventListener('click', () => {
            tasks[dateStr].splice(index, 1);
            if (tasks[dateStr].length === 0) delete tasks[dateStr];
            saveTasks();
            renderCalendar();
            openTaskPopup(dateStr);
        });
        taskList.appendChild(li);
    });

    taskPopup.classList.remove('hidden');
}

taskForm.addEventListener('submit', e => {
    e.preventDefault();
    const newTask = taskInput.value.trim();
    if (!newTask || !selectedDate) return;

    if (!tasks[selectedDate]) tasks[selectedDate] = [];
    tasks[selectedDate].push(newTask);
    taskInput.value = '';

    saveTasks();
    renderCalendar();
    openTaskPopup(selectedDate);
});

function saveTasks() {
    localStorage.setItem('tasks', JSON.stringify(tasks));
}

closePopupBtn.addEventListener('click', () => {
    taskPopup.classList.add('hidden');
});

prevMonthBtn.addEventListener('click', () => {
    currentDate.setMonth(currentDate.getMonth() - 1);
    renderCalendar();
});

nextMonthBtn.addEventListener('click', () => {
    currentDate.setMonth(currentDate.getMonth() + 1);
    renderCalendar();
});

window.onload = () => {
    renderCalendar();
};
