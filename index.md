<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Мои заметки</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            max-width: 800px;
            margin: 40px auto;
            padding: 0 20px;
            background-color: #f9f9f9;
            color: #333;
            line-height: 1.6;
        }
        h1 {
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
        }
        .notes-list {
            list-style: none;
            padding: 0;
        }
        .note-item {
            display: flex;
            align-items: center;
            margin-bottom: 12px;
            font-size: 16px;
        }
        .tag {
            background-color: #e1ecf4;
            color: #39739d;
            padding: 2px 8px;
            border-radius: 4px;
            font-size: 12px;
            font-weight: bold;
            margin-right: 10px;
            min-width: 60px;
            text-align: center;
        }
        .date {
            color: #666;
            font-variant-numeric: tabular-nums;
            margin-right: 15px;
        }
        .title {
            font-weight: 500;
        }
        .title a {
            color: #0066cc;
            text-decoration: none;
        }
        .title a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>

    <h1>📝 Мои заметки</h1>

    <ul class="notes-list">
        <!-- Пример заметки №1 -->
        <li class="note-item">
            <span class="tag">HTML</span>
            <span class="date"></span>
            <span class="title">— Как создать структуру сайта</span>
        </li>
        
        <!-- Пример заметки №2 -->
        <li class="note-item">
            <span class="tag">CSS</span>
            <span class="date"></span>
            <span class="title">— Основы Flexbox для списков</span>
        </li>

        <!-- Пример заметки №3 -->
        <li class="note-item">
            <span class="tag">GitHub</span>
            <span class="date"></span>
            <span class="title">— Инструкция по деплою на Pages</span>
        </li>
    </ul>

</body>
</html>
