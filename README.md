# rafweather

import os
import requests
import random

# --- НАСТРОЙКИ ---
# Твой новый токен
TOKEN = "" 

# Твой Chat ID (получить через https://api.telegram.org/bot<ТОКЕН>/getUpdates)
CHAT_ID = "" 

def get_weather():
    """Узнаем погоду через Open-Meteo (без API ключей и регистраций!)"""
    # Координаты Краснодара: 45.0448 с.ш., 38.9760 в.д.
    url = "https://api.open-meteo.com/v1/forecast?latitude=45.0448&longitude=38.9760&current=temperature_2m,weather_code&timezone=Europe%2FMoscow"
    
    try:
        response = requests.get(url).json()
        temp = round(response['current']['temperature_2m'])
        code = response['current']['weather_code']
        
        # Расшифровка кодов погоды WMO
        is_raining = False
        description = "какая-то неразбериха"
        
        if code == 0:
            description = "ясно и безоблачно"
        elif code in [1, 2, 3]:
            description = "немного облачно"
        elif code in [45, 48]:
            description = "туман"
        elif code in [51, 53, 55, 56, 57]:
            description = "противная морось"
            is_raining = True
        elif code in [61, 63, 65, 66, 67, 80, 81, 82]:
            description = "дождь"
            is_raining = True
        elif code in [71, 73, 75, 77, 85, 86]:
            description = "снег"
        elif code in [95, 96, 99]:
            description = "гроза"
            is_raining = True
            
        return temp, description, is_raining
    except Exception as e:
        print(f"Ошибка погоды: {e}")
        return None, None, False

def get_rafayel_message(temp, description, is_raining):
    """Генерируем сообщение в стиле Рафаэля"""
    greetings = [
        "Доброе утро, моя рыбка! 🐟✨",
        "Просыпайся... Твой любимый художник уже на ногах и даже проверил для тебя погоду. 🎨",
        "Утро. Надеюсь, ты спала хорошо и тебе снился я? 😉"
    ]
    
    msg = f"{random.choice(greetings)}\n\n"
    msg += f"За окном в нашем милом Краснодаре сейчас {temp}°C, {description}.\n\n"

    # Реакция на температуру
    if temp < 10:
        msg += "Брр, ну и холодище! Оденься теплее, пожалуйста. Не хочу, чтобы моя муза простудилась. Если замерзнешь — мой шарф всегда в твоем распоряжении... или мои объятия. 🔥\n"
    elif 10 <= temp <= 22:
        msg += "Погода такая же переменчивая, как мое вдохновение. Накинь что-нибудь стильное, но практичное. Как насчет того тренча, который тебе так идет? Я хочу нарисовать тебя в нем. 🖌️\n"
    else:
        msg += "Ох, солнце сегодня почти такое же ослепительное, как ты! ☀️ Надевай самое легкое платье. И ради всего святого, не забудь солнцезащитный крем, иначе мне придется лично спасать тебя от ожогов.\n"

    # Реакция на дождь
    if is_raining:
        msg += "\nИ да, там льет... Опять сырость. ☔️ Возьми зонт! Иначе придешь ко мне промокшая до нитки. Хотя... в этом тоже есть свой шарм, но я предпочитаю, чтобы ты была сухой, пока мы не на дне океана. 🌊"

    msg += "\n\nЖду нашей встречи. Не скучай! ❤️"
    return msg

def send_message(text):
    """Отправка в Телеграм"""
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    requests.post(url, json={'chat_id': CHAT_ID, 'text': text})

# --- ГЛАВНЫЙ ОБРАБОТЧИК ---
def handler(event, context):
    """Эта функция запускается по таймеру"""
    print("Рафаэль проснулся по будильнику!")
    
    temp, description, is_raining = get_weather()
    
    if temp is not None:
        message = get_rafayel_message(temp, description, is_raining)
        send_message(message)
    else:
        send_message("Моя рыбка, я пытался узнать погоду, но эти глупые человеческие технологии сломались... Просто оденься красиво для меня, хорошо? 🐟🎨")
        
    return {'statusCode': 200, 'body': 'OK'}
