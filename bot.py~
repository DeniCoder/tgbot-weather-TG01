from aiogram import Bot, Dispatcher, types
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton
from aiogram.filters import Command
import requests
import logging
from config import TOKEN_BOT, API_KEY  # Импорты токена и ключа API
import asyncio

# Включаем логирование (для отладки)
logging.basicConfig(level=logging.INFO)

# Инициализация бота и диспетчера
bot = Bot(token=TOKEN_BOT)
dp = Dispatcher()

# Создание клавиатуры
keyboard = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="Справка")],
        [KeyboardButton(text="Погода")]
    ],
    resize_keyboard=True
)

# Функция для получения погоды через API WeatherAPI
def get_weather(city: str):
    url = f"http://api.weatherapi.com/v1/current.json"
    params = {"key": API_KEY, "q": city, "lang": "ru"}
    try:
        response = requests.get(url, params=params)
        data = response.json()
        if "error" in data:
            return "Город не найден. Попробуйте снова."
        # Формируем текст ответа
        city_name = data["location"]["name"]
        country = data["location"]["country"]
        temp = data["current"]["temp_c"]
        condition = data["current"]["condition"]["text"]
        icon_url = f"http:{data['current']['condition']['icon']}"  # Ссылка на иконку
        humidity = data["current"]["humidity"]
        wind_kph = data["current"]["wind_kph"]

        return {
            "text": (
                f"🌍 Город: {city_name}, {country}\n"
                f"🌡 Температура: {temp}°C\n"
                f"💨 Ветер: {wind_kph} км/ч\n"
                f"💧 Влажность: {humidity}%\n"
                f"☁️ Условие: {condition}"
            ),
            "icon": icon_url,
        }
    except Exception as e:
        logging.exception("Ошибка при запросе погоды")
        return "Произошла ошибка при получении данных о погоде."

# Команда /start
@dp.message(Command("start"))
async def start_command(message: types.Message):
    await message.answer(
        "Привет! Я бот, который поможет узнать текущую погоду в любом городе. "
        "Просто отправьте мне название города, и я расскажу, какая там погода сейчас! "
        "Нажмите /help, чтобы узнать больше.",
        reply_markup=keyboard,
    )

# Команда /help
@dp.message(Command("help"))
async def help_command(message: types.Message):
    await message.answer(
        "Используйте следующие команды:\n"
        "/start - Начать работу с ботом\n"
        "/help - Справка о боте\n\n"
        "Также вы можете просто отправить название города, чтобы узнать его погоду."
    )

# Обработка сообщений с текстом (название города)
@dp.message()
async def get_weather_message(message: types.Message):
    city = message.text.strip()
    weather = get_weather(city)
    if isinstance(weather, str):
        # Если вернулась ошибка
        await message.answer(weather)
    else:
        # Если погода найдена, отправляем текст и картинку
        await bot.send_photo(message.chat.id, weather["icon"])
        await message.answer(weather["text"])

# Запуск бота
async def main():
    logging.info("Бот запущен!")
    try:
        await dp.start_polling(bot)
    finally:
        await bot.session.close()

if __name__ == "__main__":
    asyncio.run(main())