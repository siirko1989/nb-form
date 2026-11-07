<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Внесення Залишків НБ</title>
    <script src="https://telegram.org"></script>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: var(--tg-theme-bg-color, #ffffff);
            color: var(--tg-theme-text-color, #222222);
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }
        h2 {
            text-align: center;
            margin-bottom: 20px;
            color: var(--tg-theme-hint-color, #777777);
            font-size: 18px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            font-size: 14px;
        }
        select, input {
            width: 100%;
            padding: 12px;
            border: 1px solid var(--tg-theme-hint-color, #ccc);
            border-radius: 8px;
            background-color: var(--tg-theme-secondary-bg-color, #f5f5f5);
            color: var(--tg-theme-text-color, #222222);
            font-size: 16px;
            box-sizing: border-box;
        }
        .checkbox-group {
            display: flex;
            align-items: center;
            margin-top: 10px;
            background: var(--tg-theme-secondary-bg-color, #f5f5f5);
            padding: 10px;
            border-radius: 8px;
        }
        .checkbox-group input {
            width: auto;
            margin-right: 10px;
        }
        #send_btn {
            width: 100%;
            padding: 15px;
            background-color: var(--tg-theme-button-color, #2481cc);
            color: var(--tg-theme-button-text-color, #ffffff);
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            margin-top: 20px;
        }
        /* Стилі для зони копіювання */
        #result_box {
            display: none;
            margin-top: 20px;
            background: #e8f4fd;
            border: 2px dashed #2481cc;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
            cursor: pointer;
        }
        .copy-hint {
            font-size: 12px;
            color: #2481cc;
            margin-top: 5px;
            font-weight: bold;
        }
        pre {
            text-align: left;
            margin: 10px 0 0 0;
            white-space: pre-wrap;
            font-family: monospace;
            background: #ffffff;
            padding: 10px;
            border-radius: 5px;
            border: 1px solid #cbdce9;
        }
    </style>
</head>
<body>

    <h2>Заповнення залишків палива</h2>

    <div id="main_form">
        <div class="form-group">
            <label for="base">Оберіть нафтобазу:</label>
            <select id="base">
                <option value="Ольшаниця">Ольшаниця</option>
                <option value="Чоп">Чоп</option>
                <option value="Скибнево">Скибнево</option>
                <option value="Овидиополь">Овидиополь</option>
            </select>
        </div>

        <div class="checkbox-group">
            <input type="checkbox" id="no_changes" onchange="toggleFuelInputs()">
            <label for="no_changes">Без змін (залишити як було)</label>
        </div>

        <div id="fuel_fields">
            <div class="form-group" style="margin-top: 15px;">
                <label for="a95">А-95 (вага в кг):</label>
                <input type="number" id="a95" placeholder="Наприклад: 193114" min="0" inputmode="numeric">
            </div>

            <div class="form-group">
                <label for="dt">ДП / ДТ (вага в кг):</label>
                <input type="number" id="dt" placeholder="Наприклад: 22540" min="0" inputmode="numeric">
            </div>
        </div>

        <button id="send_btn" onclick="generateReport()">Сформувати звіт для групи</button>
    </div>

    <!-- Блок з результатом копіювання -->
    <div id="result_box" onclick="copyToClipboard()">
        <span style="font-weight: bold; color: #222;">Звіт сформовано!</span>
        <div class="copy-hint"> Натисніть сюди, щоб СКОПІЮВАТИ текст</div>
        <pre id="report_text"></pre>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();

        function toggleFuelInputs() {
            const noChanges = document.getElementById('no_changes').checked;
            document.getElementById('fuel_fields').style.opacity = noChanges ? '0.3' : '1';
            document.getElementById('a95').disabled = noChanges;
            document.getElementById('dt').disabled = noChanges;
        }

        let globalTextToCopy = "";

        function generateReport() {
            const base = document.getElementById('base').value;
            const noChanges = document.getElementById('no_changes').checked;
            
            const today = new Date();
            const dd = String(today.getDate()).padStart(2, '0');
            const mm = String(today.getMonth() + 1).padStart(2, '0');
            const yyyy = today.getFullYear();
            const dateStr = `${dd}.${mm}.${yyyy}`;

            let reportText = `Залишки на НБ ${base} ${dateStr}:\n`;

            if (noChanges) {
                reportText += "Без змін";
            } else {
                const a95 = document.getElementById('a95').value.trim();
                const dt = document.getElementById('dt').value.trim();

                if (!a95 && !dt) {
                    alert("Будь ласка, введіть вагу палива або оберіть 'Без змін'!");
                    return;
                }

                if (a95) reportText += `⛽ А 95- ${a95} кг\n`;
                if (dt) reportText += `🛢️ ДП- ${dt} кг`;
            }

            globalTextToCopy = reportText;
            
            // Показуємо плашку з результатом
            document.getElementById('report_text').innerText = reportText;
            document.getElementById('main_form').style.display = 'none';
            document.getElementById('result_box').style.display = 'block';
            
            // Автоматично пробуємо скопіювати відразу
            copyToClipboard();
        }

        function copyToClipboard() {
            const el = document.createElement('textarea');
            el.value = globalTextToCopy;
            document.body.appendChild(el);
            el.select();
            document.execCommand('copy');
            document.body.removeChild(el);
            
            // Сповіщаємо користувача через вібро або спливаюче вікно Телеграму
            if (tg.showAlert) {
                tg.showAlert("Текст звіту успішно скопійовано! Закрийте форму та просто вставте його у вашу робочу групу.");
            } else {
                alert("Скопійовано! Вставте текст у групу.");
            }
            tg.close();
        }
    </script>
</body>
</html>
