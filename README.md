<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>시간표 생성기</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f5f5f5;
            margin: 0;
            padding: 20px;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 600px;
            margin-bottom: 20px;
        }
        h1, h2 {
            text-align: center;
        }
        form {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        input, select {
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            padding: 10px;
            background-color: #007BFF;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 8px;
            border-bottom: 1px solid #ccc;
        }
        li button {
            background-color: #FF4136;
            border: none;
            border-radius: 4px;
            padding: 5px;
            cursor: pointer;
        }
        li button:hover {
            background-color: #E33E2B;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            padding: 8px;
            text-align: center;
            border: 1px solid #ccc;
        }
        th {
            background-color: #f2f2f2;
        }
        .school-time {
            background-color: #a8d8ea;
        }
        .study-time {
            background-color: #c7ecee;
        }
        .weekend-time {
            background-color: #f8d7da;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>시간표 생성기</h1>
        <form id="class-form">
            <label for="subject">과목 이름:</label>
            <input type="text" id="subject" name="subject" required>
            <label for="hours">일주일 총 공부 시간:</label>
            <input type="number" id="hours" name="hours" min="1" required>
            <button type="submit">과목 추가</button>
        </form>
        <form id="unavailable-form">
            <label for="unavailable-start">불가능 시작 시간 (0 ~ 23):</label>
            <input type="number" id="unavailable-start" name="unavailable-start" min="0" max="23" required>
            <label for="unavailable-end">불가능 종료 시간 (1 ~ 24):</label>
            <input type="number" id="unavailable-end" name="unavailable-end" min="1" max="24" required>
            <button type="submit">불가능 시간 추가</button>
        </form>
        <form id="weekend-start-form">
            <label for="weekend-start">주말 시작 시간 (0 ~ 23):</label>
            <input type="number" id="weekend-start" name="weekend-start" min="0" max="23" required>
            <button type="submit">주말 시작 시간 설정</button>
        </form>
        <button id="generate-schedule">시간표 생성</button>
        <h2>추가된 과목:</h2>
        <ul id="class-list"></ul>
        <h2>불가능 시간대:</h2>
        <ul id="unavailable-list"></ul>
        <h2>생성된 시간표:</h2>
        <table id="schedule-table">
            <thead>
                <tr>
                    <th>시간</th>
                    <th>월</th>
                    <th>화</th>
                    <th>수</th>
                    <th>목</th>
                    <th>금</th>
                    <th>토</th>
                    <th>일</th>
                </tr>
            </thead>
            <tbody id="schedule-body">
                <!-- 시간표가 여기에 생성됩니다 -->
            </tbody>
        </table>
    </div>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const classForm = document.getElementById('class-form');
            const unavailableForm = document.getElementById('unavailable-form');
            const weekendStartForm = document.getElementById('weekend-start-form');
            const classList = document.getElementById('class-list');
            const unavailableList = document.getElementById('unavailable-list');
            const scheduleTableBody = document.getElementById('schedule-body');
            const generateScheduleBtn = document.getElementById('generate-schedule');

            let classes = [];
            let unavailableTimes = [];
            let weekendStartTime = 0;

            classForm.addEventListener('submit', (e) => {
                e.preventDefault();
                const subject = document.getElementById('subject').value;
                const hours = parseInt(document.getElementById('hours').value);

                classes.push({ subject, hours });
                displayClasses();
                classForm.reset();
            });

            unavailableForm.addEventListener('submit', (e) => {
                e.preventDefault();
                const start = parseInt(document.getElementById('unavailable-start').value);
                const end = parseInt(document.getElementById('unavailable-end').value);

                if (start < end) {
                    unavailableTimes.push({ start, end });
                    displayUnavailableTimes();
                    unavailableForm.reset();
                } else {
                    alert('불가능 시작 시간은 종료 시간보다 작아야 합니다.');
                }
            });

            weekendStartForm.addEventListener('submit', (e) => {
                e.preventDefault();
                weekendStartTime = parseInt(document.getElementById('weekend-start').value);
                alert(`주말 시작 시간이 ${weekendStartTime}시로 설정되었습니다.`);
                weekendStartForm.reset();
            });

            generateScheduleBtn.addEventListener('click', () => {
                const scheduledClasses = scheduleClasses(classes, unavailableTimes, weekendStartTime);
                displayScheduledClasses(scheduledClasses);
            });

            function displayClasses() {
                classList.innerHTML = '';
                classes.forEach((cls, index) => {
                    const li = document.createElement('li');
                    li.textContent = `과목 ${index + 1}: ${cls.subject}, 일주일 총 시간 - ${cls.hours}`;
                    const deleteBtn = document.createElement('button');
                    deleteBtn.textContent = '삭제';
                    deleteBtn.addEventListener('click', () => {
                        classes.splice(index, 1);
                        displayClasses();
                    });
                    li.appendChild(deleteBtn);
                    classList.appendChild(li);
                });
            }

            function displayUnavailableTimes() {
                unavailableList.innerHTML = '';
                unavailableTimes.forEach((time, index) => {
                    const li = document.createElement('li');
                    li.textContent = `불가능 시간 ${index + 1}: ${time.start}시 - ${time.end}시`;
                    const deleteBtn = document.createElement('button');
                    deleteBtn.textContent = '삭제';
                    deleteBtn.addEventListener('click', () => {
                        unavailableTimes.splice(index, 1);
                        displayUnavailableTimes();
                    });
                    li.appendChild(deleteBtn);
                    unavailableList.appendChild(li);
                });
            }

            function displayScheduledClasses(scheduledClasses) {
                scheduleTableBody.innerHTML = '';
                const days = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'];

                for (let hour = 7; hour <= 23; hour++) {
                    const tr = document.createElement('tr');
                    const tdHour = document.createElement('td');
                    tdHour.textContent = `${hour}시`;
                    tr.appendChild(tdHour);

                    days.forEach(day => {
                        const td = document.createElement('td');
                        const scheduledClass = scheduledClasses.find(cls => cls.day === day && cls.start <= hour && cls.end > hour);

                        if (scheduledClass) {
                            td.textContent = scheduledClass.subject;
                            if (scheduledClass.subject === '학교') {
                                td.classList.add('school-time');
                            } else {
                                td.classList.add('study-time');
                            }
                        }

                        tr.appendChild(td);
                    });

                    scheduleTableBody.appendChild(tr);
                }
            }

            function scheduleClasses(classes, unavailableTimes, weekendStartTime) {
                classes.sort((a, b) => a.hours - b.hours);
                unavailableTimes.sort((a, b) => a.start - b.start);

                const scheduledClasses = [];
                const days = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'];
                let totalStudyHours = classes.reduce((total, cls) => total + cls.hours, 0);
                const dailyStudyHours = totalStudyHours / 7;

                days.forEach(day => {
                    let dayStartTime = (day === 'sat' || day === 'sun') ? weekendStartTime : 17;

                    if (day !== 'sat' && day !== 'sun') {
                        for (let hour = 8; hour < 17; hour++) {
                            scheduledClasses.push({ day, subject: '학교', start: hour, end: hour + 1 });
                        }
                        dayStartTime = 17;
                    }

                    let remainingDailyHours = dailyStudyHours;
                    for (let hour = dayStartTime; hour < 24; hour++) {
                        if (unavailableTimes.some(time => time.start <= hour && time.end > hour)) {
                            continue;
                        }

                        if (remainingDailyHours <= 0) break;

                        const cls = classes.find(cls => cls.hours > 0);
                        if (cls) {
                            const start = hour;
                            const end = start + 1;
                            scheduledClasses.push({ day, subject: cls.subject, start, end });
                            cls.hours -= 1;
                            remainingDailyHours -= 1;
                        }
                    }
                });

                return scheduledClasses;
            }
        });
    </script>
</body>
</html>
