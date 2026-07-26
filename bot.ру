import telebot
from telebot import types
import sqlite3
import json
import random
from datetime import datetime, timedelta

# --- КОНФИГУРАЦИЯ ---
TOKEN = '8840651866:AAH0B2y3imMGfQyFkDXbjKDt77H2rCWc2oo'
ADMIN_ID = 1107252462
PROVIDER_TOKEN = 'ТВОЙ_PROVIDER_TOKEN'

bot = telebot.TeleBot(TOKEN)

# --- БАЗА ДАННЫХ ---
def init_db():
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users (
                    user_id INTEGER PRIMARY KEY,
                    username TEXT,
                    level INTEGER DEFAULT 1,
                    premium INTEGER DEFAULT 0,
                    premium_until TEXT,
                    current_lesson INTEGER DEFAULT 1,
                    score INTEGER DEFAULT 0,
                    passed_lessons TEXT DEFAULT '[]'
                )''')
    conn.commit()
    conn.close()

def add_user(user_id, username):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('INSERT OR IGNORE INTO users (user_id, username) VALUES (?, ?)', (user_id, username))
    conn.commit()
    conn.close()

def update_premium(user_id, days=0):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    until = (datetime.now() + timedelta(days=days)).isoformat() if days > 0 else None
    c.execute('UPDATE users SET premium = 1, premium_until = ? WHERE user_id = ?', (until, user_id))
    conn.commit()
    conn.close()

def is_premium(user_id):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('SELECT premium, premium_until FROM users WHERE user_id = ?', (user_id,))
    result = c.fetchone()
    conn.close()
    if not result:
        return False
    premium, until = result
    if premium == 0:
        return False
    if until and datetime.now() > datetime.fromisoformat(until):
        return False
    return True

def get_user_lesson(user_id):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('SELECT current_lesson FROM users WHERE user_id = ?', (user_id,))
    result = c.fetchone()
    conn.close()
    return result[0] if result else 1

def update_lesson(user_id, lesson):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('UPDATE users SET current_lesson = ? WHERE user_id = ?', (lesson, user_id))
    conn.commit()
    conn.close()

def update_score(user_id, points):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('UPDATE users SET score = score + ? WHERE user_id = ?', (points, user_id))
    conn.commit()
    conn.close()

def get_score(user_id):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('SELECT score FROM users WHERE user_id = ?', (user_id,))
    result = c.fetchone()
    conn.close()
    return result[0] if result else 0

def get_passed_lessons(user_id):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('SELECT passed_lessons FROM users WHERE user_id = ?', (user_id,))
    result = c.fetchone()
    conn.close()
    if result and result[0]:
        return json.loads(result[0])
    return []

def add_passed_lesson(user_id, lesson_id):
    passed = get_passed_lessons(user_id)
    if lesson_id not in passed:
        passed.append(lesson_id)
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('UPDATE users SET passed_lessons = ? WHERE user_id = ?', (json.dumps(passed), user_id))
    conn.commit()
    conn.close()

def get_level(user_id):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('SELECT level FROM users WHERE user_id = ?', (user_id,))
    result = c.fetchone()
    conn.close()
    return result[0] if result else 1

def set_level(user_id, level):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('UPDATE users SET level = ? WHERE user_id = ?', (level, user_id))
    conn.commit()
    conn.close()

def get_status(score):
    if score >= 70:
        return '👑 Душа Грузии'
    elif score >= 55:
        return '🏔️ Местный житель'
    elif score >= 40:
        return '🍽️ Знаток хинкали'
    elif score >= 25:
        return '🍷 Гость Грузии'
    else:
        return '🇬🇪 Турист'

# --- СКИДКИ ---
def get_discount(user_id):
    passed = get_passed_lessons(user_id)
    total_lessons = 25
    score = get_score(user_id)

    if len(passed) == total_lessons:
        if score >= 60:
            return 0.50
        else:
            return 0.00
    elif len(passed) > total_lessons:
        return 0.15
    else:
        return 0.00

# --- УРОКИ (25 шт) ---
LESSONS = {
    1: {'title': 'Буква ა', 'content': 'ა — как "а" в "арбуз".', 'test': [{'type': 'choice', 'question': 'Как читается ა?', 'options': ['а', 'б', 'в'], 'answer': 0}]},
    2: {'title': 'Буква ბ', 'content': 'ბ — как "б" в "банан".', 'test': [{'type': 'choice', 'question': 'Как читается ბ?', 'options': ['б', 'п', 'м'], 'answer': 0}]},
    3: {'title': 'Буква გ', 'content': 'გ — как "г" в "гора".', 'test': [{'type': 'choice', 'question': 'Как читается გ?', 'options': ['г', 'к', 'х'], 'answer': 0}]},
    4: {'title': 'Буква დ', 'content': 'დ — как "д" в "дом".', 'test': [{'type': 'choice', 'question': 'Как читается დ?', 'options': ['д', 'т', 'н'], 'answer': 0}]},
    5: {'title': 'Буква ე', 'content': 'ე — как "э" в "это".', 'test': [{'type': 'choice', 'question': 'Как читается ე?', 'options': ['э', 'е', 'и'], 'answer': 0}]},
    6: {'title': 'Слова: Приветствие', 'content': 'გამარჯობა — Здравствуйте', 'test': [{'type': 'choice', 'question': 'Перевод: გამარჯობა', 'options': ['Здравствуйте', 'Спасибо', 'Пока'], 'answer': 0}]},
    7: {'title': 'Слова: Благодарность', 'content': 'მადლობა — Спасибо', 'test': [{'type': 'choice', 'question': 'Перевод: მადლობა', 'options': ['Спасибо', 'Пожалуйста', 'Привет'], 'answer': 0}]},
    8: {'title': 'Слова: До свидания', 'content': 'ნახვამდის — До свидания', 'test': [{'type': 'choice', 'question': 'Перевод: ნახვამდის', 'options': ['До свидания', 'Привет', 'Спасибо'], 'answer': 0}]},
    9: {'title': 'Слова: Да / Нет', 'content': 'კი — Да, არა — Нет', 'test': [{'type': 'choice', 'question': 'Что значит "კი"?', 'options': ['Да', 'Нет', 'Может быть'], 'answer': 0}]},
    10: {'title': 'Слова: Еда', 'content': 'პური — Хлеб, ხორცი — Мясо', 'test': [{'type': 'choice', 'question': 'Перевод: პური', 'options': ['Хлеб', 'Мясо', 'Сыр'], 'answer': 0}]},
    11: {'title': 'Фразы: Как дела?', 'content': 'როგორ ხარ? — Как дела?', 'test': [{'type': 'choice', 'question': 'Перевод: როგორ ხარ?', 'options': ['Как дела?', 'Как тебя зовут?', 'Сколько стоит?'], 'answer': 0}]},
    12: {'title': 'Фразы: Меня зовут', 'content': 'მე მქვია... — Меня зовут...', 'test': [{'type': 'choice', 'question': 'Перевод: მე მქვია', 'options': ['Меня зовут', 'Как тебя зовут?', 'Я есть'], 'answer': 0}]},
    13: {'title': 'Фразы: В кафе', 'content': 'მინდა ყავა. — Я хочу кофе.', 'test': [{'type': 'choice', 'question': 'Перевод: მინდა ყავა.', 'options': ['Я хочу кофе', 'Я люблю кофе', 'Где кофе?'], 'answer': 0}]},
    14: {'title': 'Фразы: Счёт', 'content': 'ანგარიში, გთხოვთ. — Счёт, пожалуйста.', 'test': [{'type': 'choice', 'question': 'Перевод: ანგარიში, გთხოვთ.', 'options': ['Счёт, пожалуйста', 'Вода, пожалуйста', 'Спасибо'], 'answer': 0}]},
    15: {'title': 'Фразы: Где?', 'content': 'სად არის...? — Где находится...?', 'test': [{'type': 'choice', 'question': 'Перевод: სად არის?', 'options': ['Где находится?', 'Что это?', 'Сколько стоит?'], 'answer': 0}]},
    16: {'title': 'Диалог: Знакомство', 'content': '- გამარჯობა, მე ვარ ანა. — Здравствуйте, я Анна.\n- მე ვარ თამარ. — Я Тамар.', 'test': [{'type': 'choice', 'question': 'Как представиться?', 'options': ['მე ვარ...', 'რა გქვია?', 'ნახვამდის'], 'answer': 0}]},
    17: {'title': 'Диалог: В магазине', 'content': '— ეს რა ღირს? — Сколько это стоит?', 'test': [{'type': 'choice', 'question': 'Как спросить цену?', 'options': ['ეს რა ღირს?', 'რა გქვია?', 'სად არის?'], 'answer': 0}]},
    18: {'title': 'Диалог: В такси', 'content': '— მივდივარ აეროპორტში. — Я еду в аэропорт.', 'test': [{'type': 'choice', 'question': 'Как сказать "Я еду в аэропорт"?', 'options': ['მივდივარ აეროპორტში', 'მე მივდივარ', 'სად არის აეროპორტი?'], 'answer': 0}]},
    19: {'title': 'Диалог: В ресторане', 'content': '— მინდა ხინკალი. — Я хочу хинкали.', 'test': [{'type': 'choice', 'question': 'Как сказать "Я хочу хинкали"?', 'options': ['მინდა ხინკალი', 'მე მიყვარს ხინკალი', 'ეს არის ხინკალი'], 'answer': 0}]},
    20: {'title': 'Диалог: Время', 'content': '— რომელი საათია? — Который час?', 'test': [{'type': 'choice', 'question': 'Как спросить время?', 'options': ['რომელი საათია?', 'რა ღირს?', 'სად ხარ?'], 'answer': 0}]},
    21: {'title': 'Диалог: Погода', 'content': '— დღეს ცხელა. — Сегодня жарко.', 'test': [{'type': 'choice', 'question': 'Как сказать "Сегодня жарко"?', 'options': ['დღეს ცხელა', 'დღეს ცივა', 'დღეს კარგია'], 'answer': 0}]},
    22: {'title': 'Диалог: Семья', 'content': '— ეს არის ჩემი დედა. — Это моя мама.', 'test': [{'type': 'choice', 'question': 'Как сказать "Это моя мама"?', 'options': ['ეს არის ჩემი დედა', 'ეს არის ჩემი მამა', 'ეს არის ჩემი და'], 'answer': 0}]},
    23: {'title': 'Диалог: У врача', 'content': '— მაქვს თავის ტკივილი. — У меня болит голова.', 'test': [{'type': 'choice', 'question': 'Как сказать "У меня болит голова"?', 'options': ['მაქვს თავის ტკივილი', 'მაქვს კბილის ტკივილი', 'მაქვს ცხვირი'], 'answer': 0}]},
    24: {'title': 'Диалог: Разговор с другом', 'content': '— როგორ ხარ? — Как дела?\n— კარგად. — Хорошо.', 'test': [{'type': 'choice', 'question': 'Как ответить "хорошо"?', 'options': ['კარგად', 'ცუდად', 'კი'], 'answer': 0}]},
    25: {'title': 'Диалог: В гостях', 'content': '— გაგიმარჯოს! — Будем здоровы!', 'test': [{'type': 'choice', 'question': 'Как сказать "Будем здоровы!"?', 'options': ['გაგიმარჯოს!', 'ნახვამდის!', 'მადლობა!'], 'answer': 0}]}
}

# --- ПРЕМИУМ (5 уроков с диалогами по возрастам) ---
PREMIUM_LESSONS = {
    101: {'title': 'Диалог с дедушкой (60+)', 'content': '— გამარჯობა, შვილო! როგორ ხარ?\n— კარგად, ბატონო.\n— ხინკალი გინდა?\n— კი, მადლობა!', 'test': [{'type': 'choice', 'question': 'Как дедушка спросил "как дела"?', 'options': ['როგორ ხარ?', 'რა ღირს?', 'სად ხარ?'], 'answer': 0}]},
    102: {'title': 'Диалог с таксистом (40+)', 'content': '— სად მიდიხართ?\n— აეროპორტში.\n— 20 ლარი.\n— კარგი.', 'test': [{'type': 'choice', 'question': 'Как спросить "куда едете"?', 'options': ['სად მიდიხართ?', 'რა ღირს?', 'როგორ ხარ?'], 'answer': 0}]},
    103: {'title': 'Диалог с продавцом (30+)', 'content': '— ეს რა ღირს?\n— 10 ლარი.\n— აი, 10 ლარი.\n— მადლობა!', 'test': [{'type': 'choice', 'question': 'Как сказать "сколько стоит"?', 'options': ['რა ღირს?', 'რა არის?', 'როგორ?'], 'answer': 0}]},
    104: {'title': 'Диалог с подругой (20+)', 'content': '— როგორ ხარ?\n— კარგად. რა ახალი?\n— ყველაფერი კარგადაა.', 'test': [{'type': 'choice', 'question': 'Как спросить "что нового"?', 'options': ['რა ახალი?', 'რა ღირს?', 'როგორ?'], 'answer': 0}]},
    105: {'title': 'Диалог с подростком (10+)', 'content': '— გამარჯობა!\n— გამარჯობა!\n— წავიდეთ?\n— კი!', 'test': [{'type': 'choice', 'question': 'Как сказать "пойдём"?', 'options': ['წავიდეთ!', 'ნახვამდის!', 'მადლობა!'], 'answer': 0}]}
}

# --- КОМАНДЫ БОТА ---
@bot.message_handler(commands=['start'])
def start(message):
    user_id = message.chat.id
    username = message.from_user.username or 'unknown'
    add_user(user_id, username)

    markup = types.InlineKeyboardMarkup(row_width=1)
    markup.add(
        types.InlineKeyboardButton('1️⃣ Ничего не знаю', callback_data='level_1'),
        types.InlineKeyboardButton('2️⃣ Знаю пару слов', callback_data='level_2'),
        types.InlineKeyboardButton('3️⃣ Могу объясниться', callback_data='level_3'),
        types.InlineKeyboardButton('4️⃣ Говорю свободно', callback_data='level_4')
    )

    bot.send_message(
        user_id,
        '🇬🇪 *გამარჯობა, брат!*\n\n'
        'Я — твой гид по Грузии. Я помогу тебе заговорить по-грузински.\n\n'
        'Чтобы подобрать уроки, скажи:\n*Как у тебя с грузинским?*',
        reply_markup=markup,
        parse_mode='Markdown'
    )

@bot.callback_query_handler(func=lambda call: call.data.startswith('level_'))
def set_level_callback(call):
    user_id = call.message.chat.id
    level = int(call.data.split('_')[1])
    set_level(user_id, level)

    bot.answer_callback_query(call.id, f'Уровень {level} сохранён!')
    bot.send_message(user_id, f'✅ Отлично! Я буду подбирать уроки под твой уровень.')

    show_main_menu(call.message)

def show_main_menu(message):
    user_id = message.chat.id
    markup = types.InlineKeyboardMarkup(row_width=2)
    markup.add(
        types.InlineKeyboardButton('📚 Уроки', callback_data='lessons'),
        types.InlineKeyboardButton('📝 Словарь', callback_data='dictionary'),
        types.InlineKeyboardButton('🌟 Premium', callback_data='premium_info'),
        types.InlineKeyboardButton('📊 Прогресс', callback_data='progress')
    )
    bot.send_message(
        message.chat.id,
        'Выбери действие:',
        reply_markup=markup
    )

@bot.callback_query_handler(func=lambda call: True)
def handle_callback(call):
    user_id = call.message.chat.id

    if call.data == 'lessons':
        show_lesson_menu(call.message)
    elif call.data == 'dictionary':
        show_dictionary(call.message)
    elif call.data == 'premium_info':
        show_premium_info(call.message)
    elif call.data == 'progress':
        show_progress(call.message)
    elif call.data.startswith('lesson_'):
        lesson_id = int(call.data.split('_')[1])
        show_lesson(call.message, lesson_id)
    elif call.data.startswith('premium_'):
        lesson_id = int(call.data.split('_')[1])
        show_premium_lesson(call.message, lesson_id)
    elif call.data.startswith('quiz_ans_'):
        handle_choice_answer(call)
    elif call.data == 'buy_premium':
        send_invoice(call.message)

def show_lesson_menu(message):
    user_id = message.chat.id
    markup = types.InlineKeyboardMarkup(row_width=2)
    level = get_level(user_id)

    for lesson_id, lesson in LESSONS.items():
        btn = types.InlineKeyboardButton(f'📖 {lesson["title"]}', callback_data=f'lesson_{lesson_id}')
        markup.add(btn)

    if is_premium(user_id):
        for pid, lesson in PREMIUM_LESSONS.items():
            btn = types.InlineKeyboardButton(f'🌟 {lesson["title"]}', callback_data=f'premium_{pid}')
            markup.add(btn)
    else:
        markup.add(types.InlineKeyboardButton('🌟 Premium 🔒', callback_data='premium_info'))

    bot.send_message(
        message.chat.id,
        f'📚 *Твой уровень: {level}*\nВыбери урок:',
        reply_markup=markup,
        parse_mode='Markdown'
    )

def show_lesson(message, lesson_id):
    user_id = message.chat.id
    lesson = LESSONS.get(lesson_id)
    if not lesson:
        bot.send_message(user_id, '❌ Урок не найден.')
        return

    update_lesson(user_id, lesson_id)

    text = f'*{lesson["title"]}*\n━━━━━━━━━━━━━━━━━━━━━\n\n{lesson["content"]}'

    markup = types.InlineKeyboardMarkup(row_width=2)
    markup.add(
        types.InlineKeyboardButton('📝 Начать тест', callback_data=f'quiz_start_{lesson_id}')
    )

    bot.send_message(user_id, text, reply_markup=markup, parse_mode='Markdown')

def show_premium_lesson(message, lesson_id):
    user_id = message.chat.id
    if not is_premium(user_id):
        bot.send_message(user_id, '❌ Это премиум-урок. Оформи подписку, чтобы получить доступ!')
        return

    lesson = PREMIUM_LESSONS.get(lesson_id)
    if not lesson:
        bot.send_message(user_id, '❌ Урок не найден.')
        return

    text = f'*{lesson["title"]}*\n━━━━━━━━━━━━━━━━━━━━━\n\n{lesson["content"]}'

    markup = types.InlineKeyboardMarkup(row_width=2)
    markup.add(
        types.InlineKeyboardButton('📝 Начать тест', callback_data=f'quiz_premium_{lesson_id}')
    )

    bot.send_message(user_id, text, reply_markup=markup, parse_mode='Markdown')

@bot.callback_query_handler(func=lambda call: call.data.startswith('quiz_start_'))
def start_quiz(call):
    user_id = call.message.chat.id
    lesson_id = int(call.data.split('_')[2])
    lesson = LESSONS.get(lesson_id)
    if not lesson:
        return

    bot.send_message(user_id, '📝 *Начинаем тест!*', parse_mode='Markdown')
    user_quiz_data = {'lesson_id': lesson_id, 'questions': lesson['test'], 'current': 0, 'score': 0}
    ask_question(user_id, user_quiz_data)

@bot.callback_query_handler(func=lambda call: call.data.startswith('quiz_premium_'))
def start_premium_quiz(call):
    user_id = call.message.chat.id
    lesson_id = int(call.data.split('_')[2])
    lesson = PREMIUM_LESSONS.get(lesson_id)
    if not lesson:
        return

    bot.send_message(user_id, '📝 *Начинаем тест!*', parse_mode='Markdown')
    user_quiz_data = {'lesson_id': lesson_id, 'questions': lesson['test'], 'current': 0, 'score': 0}
    ask_question(user_id, user_quiz_data)

def ask_question(user_id, quiz_data):
    if quiz_data['current'] >= len(quiz_data['questions']):
        finish_quiz(user_id, quiz_data)
        return

    q = quiz_data['questions'][quiz_data['current']]
    if q['type'] == 'choice':
        markup = types.InlineKeyboardMarkup(row_width=1)
        for i, opt in enumerate(q['options']):
            markup.add(types.InlineKeyboardButton(opt, callback_data=f'quiz_ans_{i}_{quiz_data["current"]}'))
        bot.send_message(user_id, f'❓ {q["question"]}', reply_markup=markup)

def handle_choice_answer(call):
    user_id = call.message.chat.id
    bot.answer_callback_query(call.id, '✅ Ответ принят!')
    bot.send_message(user_id, '✅ Отлично! Переходим дальше.')

def finish_quiz(user_id, quiz_data):
    add_passed_lesson(user_id, quiz_data['lesson_id'])
    update_score(user_id, 5)
    bot.send_message(user_id, '🏁 *Тест завершён! Ты получил +5 хинкалей.*', parse_mode='Markdown')

# --- ПРЕМИУМ ИНФО ---
def show_premium_info(message):
    user_id = message.chat.id

    if is_premium(user_id):
        text = '🌟 *У вас есть Premium-доступ!*\n\n'
        markup = types.InlineKeyboardMarkup(row_width=1)
        for pid, lesson in PREMIUM_LESSONS.items():
            markup.add(types.InlineKeyboardButton(f'📖 {lesson["title"]}', callback_data=f'premium_{pid}'))
        bot.send_message(user_id, text, reply_markup=markup, parse_mode='Markdown')
        return

    discount = get_discount(user_id)
    price = 75
    discount_text = ''

    if discount == 0.50:
        discount_text = '🔥 *Ты прошёл курс на отлично! Скидка 50%!*\n💰 Цена: *37 Stars*'
        price = 37
    elif discount == 0.15:
        discount_text = '💪 *Ты перепрошёл уроки! Скидка 15%!*\n💰 Цена: *63 Stars*'
        price = 63
    else:
        discount_text = '💰 Цена: 75 Stars'

    text = f'''🌟 *Premium-доступ*

💎 *Что даёт Premium:*
• 5 диалогов с разными возрастами
• Реальные сценарии: такси, магазин, кафе
• Сленг и живая речь

{discount_text}'''

    markup = types.InlineKeyboardMarkup(row_width=1)
    markup.add(types.InlineKeyboardButton(f'💳 Купить за {price} Stars', callback_data='buy_premium'))
    bot.send_message(user_id, text, reply_markup=markup, parse_mode='Markdown')

def send_invoice(message):
    user_id = message.chat.id
    if is_premium(user_id):
        bot.send_message(user_id, 'У тебя уже есть Premium-доступ 🎉')
        return

    discount = get_discount(user_id)
    price = 75
    if discount == 0.50:
        price = 37
    elif discount == 0.15:
        price = 63

    bot.send_invoice(
        chat_id=user_id,
        title='Premium-доступ к грузинскому',
        description='5 диалогов с разными возрастами. Разговорная практика.',
        invoice_payload='premium_access',
        provider_token=PROVIDER_TOKEN,
        currency='XTR',
        prices=[{'label': 'Полный доступ', 'amount': price}]
    )

@bot.pre_checkout_query_handler(func=lambda query: True)
def process_pre_checkout_query(query):
    bot.answer_pre_checkout_query(query.id, ok=True)

@bot.message_handler(content_types=['successful_payment'])
def process_payment(message):
    user_id = message.chat.id
    update_premium(user_id, days=365)
    bot.send_message(user_id, '🎉 *Premium-доступ активирован навсегда!*\n\nТеперь тебе доступны диалоги с разными возрастами.', parse_mode='Markdown')

def show_dictionary(message):
    all_words = []
    for lesson in LESSONS.values():
        all_words.extend(lesson.get('words', []))
    text = '📚 *Словарь*\n\n'
    for w in all_words[:30]:
        text += f'• {w["ge"]} — {w["ru"]}\n'
    bot.send_message(message.chat.id, text, parse_mode='Markdown')

def show_progress(message):
    user_id = message.chat.id
    passed = get_passed_lessons(user_id)
    total = len(LESSONS)
    score = get_score(user_id)
    premium = is_premium(user_id)
    status = get_status(score)

    text = f'📊 *Твой прогресс*\n\n'
    text += f'📖 Пройдено уроков: {len(passed)}/{total}\n'
    text += f'🥟 Хинкалей: {score}\n'
    text += f'🏆 Статус: *{status}*\n'
    text += f'🌟 Premium: {"✅ Да" if premium else "❌ Нет"}'
    bot.send_message(message.chat.id, text, parse_mode='Markdown')

if __name__ == '__main__':
    init_db()
    print('🇬🇪 Бот запущен!')
    bot.infinity_polling()