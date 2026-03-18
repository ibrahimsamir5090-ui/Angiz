<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>أنجز | 𝐇𝐞𝐦𝐚</title>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"/>
    <style>
        :root {
            --bg-color: #fdfaf5;
            --main-camel: #b08968;
            --light-camel: #e6ccb2;
            --text-color: #5d4037;
            --white: #ffffff;
            --black: #000000;
        }

        body {
            margin: 0; font-family: 'Cairo', sans-serif;
            background-color: var(--bg-color); color: var(--text-color);
            padding-bottom: 90px;
        }

        /* شاشة العقاب القاتمة */
        #punishOverlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: var(--black); color: white; z-index: 9999;
            display: none; flex-direction: column; align-items: center;
            justify-content: center; text-align: center; padding: 20px;
        }

        .header {
            padding: 30px 20px; text-align: center; background: var(--white);
            border-bottom: 2px solid var(--light-camel);
        }

        .header h1 { margin: 0; font-weight: 700; color: var(--main-camel); font-size: 28px; }

        .container { padding: 20px; max-width: 450px; margin: auto; }

        .page { display: none; }
        .page.active { display: block; animation: fadeIn 0.4s; }

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        .task-card {
            background: var(--white); border-radius: 20px; padding: 18px;
            margin-bottom: 15px; display: flex; align-items: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.02); border: 1px solid #f0eadd;
        }

        .dot {
            width: 20px; height: 20px; border: 2px solid var(--main-camel);
            border-radius: 50%; margin-left: 15px; transition: 0.3s;
        }

        .dot.filled { background: var(--main-camel); box-shadow: inset 0 0 0 3px white; }

        .task-info { flex: 1; }
        .task-info span { font-size: 16px; font-weight: 600; display: block; }
        .task-info small { color: #bca08d; font-size: 11px; }

        .upload-btn {
            background: var(--light-camel); color: var(--text-color);
            border: none; padding: 8px 12px; border-radius: 10px;
            font-size: 12px; cursor: pointer; font-family: 'Cairo';
        }

        .nav-bar {
            position: fixed; bottom: 0; width: 100%; background: var(--white);
            display: flex; justify-content: space-around; padding: 12px 0;
            border-top: 1px solid var(--light-camel); box-shadow: 0 -5px 15px rgba(0,0,0,0.03);
        }

        .nav-item { cursor: pointer; color: #d6ccc2; text-align: center; transition: 0.3s; }
        .nav-item i { font-size: 22px; }
        .nav-item span { font-size: 10px; display: block; margin-top: 4px; }
        .nav-item.active { color: var(--main-camel); }

        .rights {
            text-align: center; margin: 30px 0; font-size: 22px;
            font-weight: bold; color: var(--main-camel); letter-spacing: 2px;
        }

        .add-box { display: flex; gap: 8px; margin-bottom: 25px; }
        .add-box input {
            flex: 1; padding: 12px 15px; border-radius: 15px; border: 1px solid var(--light-camel);
            font-family: 'Cairo'; outline: none; background: #fff;
        }
        .add-box button {
            background: var(--main-camel); color: white; border: none;
            padding: 0 18px; border-radius: 15px; cursor: pointer;
        }
    </style>
</head>
<body>

<div id="punishOverlay">
    <i class="fa-solid fa-lock" style="font-size: 50px; color: var(--main-camel);"></i>
    <h2 style="margin-top:20px">توقف! أنت في فترة عقاب</h2>
    <p id="punishReason"></p>
    <div id="countdown" style="font-size: 45px; font-weight: bold; margin: 15px 0;"></div>
    <p style="color: #888;">التزم بالعقوبة ولا تفتح تطبيقات التواصل</p>
</div>

<div class="header">
    <h1>أنـجـز</h1>
</div>

<div class="container">
    <div class="add-box">
        <input type="text" id="taskInput" placeholder="اكتب مهمتك الجديدة هنا...">
        <button onclick="addTask()"><i class="fa-solid fa-plus"></i></button>
    </div>

    <div id="day" class="page active"><h3><i class="fa-solid fa-sun"></i> مهام اليوم</h3><div class="list"></div></div>
    <div id="week" class="page"><h3><i class="fa-solid fa-calendar-week"></i> مهام الأسبوع</h3><div class="list"></div></div>
    <div id="month" class="page"><h3><i class="fa-solid fa-moon"></i> مهام الشهر</h3><div class="list"></div></div>
    <div id="extra" class="page"><h3><i class="fa-solid fa-bolt"></i> مهام جانبية</h3><div class="list"></div></div>

    <div class="rights">𝐇𝐞𝐦𝐚</div>
</div>

<div class="nav-bar">
    <div class="nav-item active" onclick="switchPage('day', this)"><i class="fa-solid fa-house"></i><span>يومي</span></div>
    <div class="nav-item" onclick="switchPage('week', this)"><i class="fa-solid fa-list-check"></i><span>أسبوعي</span></div>
    <div class="nav-item" onclick="switchPage('month', this)"><i class="fa-solid fa-calendar-days"></i><span>شهري</span></div>
    <div class="nav-item" onclick="switchPage('extra', this)"><i class="fa-solid fa-star"></i><span>جانبي</span></div>
</div>

<script>
    let tasks = JSON.parse(localStorage.getItem('Hema_App_Tasks')) || [];
    let currentTab = 'day';

    function switchPage(pageId, el) {
        document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
        document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
        document.getElementById(pageId).classList.add('active');
        el.classList.add('active');
        currentTab = pageId;
        render();
    }

    function addTask() {
        const input = document.getElementById('taskInput');
        if (!input.value) return;
        tasks.push({
            id: Date.now(),
            text: input.value,
            type: currentTab,
            done: false,
            date: new Date().toLocaleDateString('ar-EG'),
            deadline: getDeadline(currentTab)
        });
        input.value = '';
        save();
    }

    function getDeadline(type) {
        const d = new Date();
        if (type === 'day') d.setHours(d.getHours() + 24);
        else if (type === 'week') d.setDate(d.getDate() + 7);
        else if (type === 'month') d.setMonth(d.getMonth() + 1);
        else return null;
        return d.getTime();
    }

    function complete(id) {
        const f = document.createElement('input');
        f.type = 'file'; f.accept = 'image/*';
        f.onchange = () => {
            tasks.find(t => t.id === id).done = true;
            save();
            alert("تم التوثيق.. بطل يا هيمـا!");
        };
        f.click();
    }

    function render() {
        const list = document.querySelector(`#${currentTab} .list`);
        list.innerHTML = '';
        tasks.filter(t => t.type === currentTab).forEach(t => {
            list.innerHTML += `
                <div class="task-card">
                    <div class="dot ${t.done ? 'filled' : ''}"></div>
                    <div class="task-info">
                        <span>${t.text}</span>
                        <small>التاريخ: ${t.date}</small>
                    </div>
                    ${!t.done ? `<button class="upload-btn" onclick="complete(${t.id})"><i class="fa-solid fa-camera"></i> إثبات</button>` : ''}
                </div>
            `;
        });
        checkPunishment();
    }

    function checkPunishment() {
        const now = new Date().getTime();
        tasks.forEach(t => {
            if (!t.done && t.deadline && now > t.deadline) {
                let time = (t.type === 'day') ? 7200 : (t.type === 'week') ? 21600 : 86400;
                startLock(time, t.type);
            }
        });
    }

    function startLock(s, type) {
        const overlay = document.getElementById('punishOverlay');
        overlay.style.display = 'flex';
        let left = s;
        const timer = setInterval(() => {
            let h = Math.floor(left / 3600);
            let m = Math.floor((left % 3600) / 60);
            let sc = left % 60;
            document.getElementById('countdown').innerText = `${h}:${m}:${sc}`;
            document.getElementById('punishReason').innerText = `تأخرت في إنجاز المهمة ال${type === 'day' ? 'يومية' : type === 'week' ? 'أسبوعية' : 'شهرية'}`;
            left--;
            if (left < 0) { clearInterval(timer); overlay.style.display = 'none'; }
        }, 1000);
    }

    function save() { localStorage.setItem('Hema_App_Tasks', JSON.stringify(tasks)); render(); }
    render();
</script>
</body>
</html>
# Angiz
انجز مهامك وبطل كسل
