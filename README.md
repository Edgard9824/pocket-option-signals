# pocket-option-signals
from flask import Flask, render_template, request, redirect
import threading
import time
import datetime
import random
import requests

app = Flask(name)

settings = {
    'telegram_token': '8472684949:AAG9Qd7bHt-7BRhNw-7x4nQp3a1Eq2Uoq8k',
    'telegram_chat_id': None,
    'pairs': ["EURUSD-OTC", "GBPUSD-OTC", "USDJPY-OTC", "AUDUSD-OTC", "NZDUSD-OTC", "EURJPY-OTC"],
    'use_rsi': True,
    'rsi_period': 14,
    'rsi_overbought': 70,
    'rsi_oversold': 30,
    'use_stoch': True,
    'stoch_k': 14,
    'stoch_d': 3,
    'stoch_slow': 3,
    'use_ema': True,
    'ema_fast': 50,
    'ema_slow': 200,
    'use_levels': True,
    'tf_seconds': 15,
}

bot_running = False
latest_signal = {}
signal_history = []

from bot_logic import generate_signal, send_telegram_signal

def run_bot():
    global bot_running, latest_signal, signal_history
    while bot_running:
        sig = generate_signal(settings)
        if sig:
            latest_signal = sig
            signal_history.insert(0, sig)
            send_telegram_signal(sig, settings)
        time.sleep(settings['tf_seconds'])

@app.route('/', methods=['GET', 'POST'])
def index():
    global bot_running
    if request.method == 'POST':
        action = request.form.get('action')
        if action == 'start':
            bot_running = True
            threading.Thread(target=run_bot).start()
        elif action == 'stop':
            bot_running = False
    return render_template('index.html',
                           running=bot_running,
                           latest=latest_signal,
                           history=signal_history)

@app.route('/setchat', methods=['POST'])
def set_chat():
    settings['telegram_chat_id'] = request.form.get('chatid')
    return redirect('/')

if name == 'main':
    app.run(host='0.0.0.0', port=8080)
    import random
import datetime
import requests

def generate_signal(settings):
    # Здесь вставляй реальную логику анализа: RSI, Stochastic, EMA, уровни
    # Пока — псевдосигнал:
    pair = random.choice(settings['pairs'])
    direction = random.choice(['up','down'])
    winrate = f"{random.randint(80,95)}%"
    return {
        'pair': pair,
        'direction': direction,
        'winrate': winrate,
        'time': datetime.datetime.now().strftime("%H:%M:%S")
    }

def send_telegram_signal(sig, settings):
    token = settings['telegram_token']
    cid = settings['telegram_chat_id']
    if not cid:
        return
    arrow = '🔺 ВВЕРХ' if sig['direction']=='up' else '🔻 ВНИЗ'
    message = (
        f"📢 СИГНАЛ OTC\n"
        f"Пара: {sig['pair']}\n"
        f"Направление: {arrow}\n"
        f"Winrate: {sig['winrate']}\n"
        f"⏳ Вход через {settings['tf_seconds']} сек"
    )
    url = f"https://api.telegram.org/bot{token}/sendMessage"
    try:
        requests.post(url, data={'chat_id': cid, 'text': message})
    except:
        pass
