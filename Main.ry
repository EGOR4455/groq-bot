import os
import requests
from telegram import Update
from telegram.ext import Application, MessageHandler, CommandHandler, filters, ContextTypes

TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")
GROQ_KEY = os.getenv("GROQ_KEY")

GROQ_URL = "https://api.groq.com/openai/v1/chat/completions"
MODEL = "llama-3.3-70b-versatile"
SYSTEM_PROMPT = "Ты дружелюбный ИИ-ассистент. Отвечай на русском. Будь кратким и полезным."

user_histories = {}

def ask_ai(messages):
    r = requests.post(
        GROQ_URL,
        headers={"Authorization": f"Bearer {GROQ_KEY}", "Content-Type": "application/json"},
        json={"model": MODEL, "messages": messages, "temperature": 0.7, "max_tokens": 2048},
        timeout=60
    )
    data = r.json()
    if "choices" not in data:
        return f"Ошибка: {data}"
    return data["choices"][0]["message"]["content"]

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    name = update.effective_user.first_name
    await update.message.reply_text(
        f"Привет, {name}! 👋\n\n"
        f"Я ИИ на Llama 3.3 70B. Пиши что угодно!\n\n"
        f"/clear — очистить память"
    )

async def clear(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_histories.pop(update.effective_user.id, None)
    await update.message.reply_text("🧹 Память очищена!")

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    text = update.message.text
    if not text:
        return
    
    if user_id not in user_histories:
        user_histories[user_id] = [{"role": "system", "content": SYSTEM_PROMPT}]
    
    user_histories[user_id].append({"role": "user", "content": text})
    
    if len(user_histories[user_id]) > 21:
        user_histories[user_id] = [user_histories[user_id][0]] + user_histories[user_id][-20:]
    
    await update.message.chat.send_action("typing")
    
    try:
        answer = ask_ai(user_histories[user_id])
        user_histories[user_id].append({"role": "assistant", "content": answer})
        
        if len(answer) <= 4000:
            await update.message.reply_text(answer)
        else:
            for i in range(0, len(answer), 4000):
                await update.message.reply_text(answer[i:i+4000])
    except Exception as e:
        await update.message.reply_text(f"❌ Ошибка: {e}")

def main():
    app = Application.builder().token(TELEGRAM_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("clear", clear))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    print("✅ Бот запущен!")
    app.run_polling()

if __name__ == "__main__":
    main()
