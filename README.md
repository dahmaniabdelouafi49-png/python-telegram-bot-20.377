import logging
import os
import asyncio
from flask import Flask, request
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, ContextTypes

# التوكن من متغير البيئة
TOKEN = os.getenv("7653495962:AAF_mpEDM_2i1YTU94OjGTvwRfat_K9MJeY")

# إعداد التسجيل
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    level=logging.INFO
)

# إنشاء تطبيق البوت
application = Application.builder().token(TOKEN).build()

# أوامر البوت
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [InlineKeyboardButton("🎁 الهدية اليومية", callback_data="daily")],
        [InlineKeyboardButton("🎁 الهدية الأسبوعية", callback_data="weekly")],
        [InlineKeyboardButton("🎡 عجلة الحظ", callback_data="wheel")],
        [InlineKeyboardButton("🆓 الخدمات المجانية", callback_data="services")],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("مرحبًا بك في بوت النقاط 🎉", reply_markup=reply_markup)

application.add_handler(CommandHandler("start", start))

# Flask app (مطلوب من Vercel)
flask_app = Flask(Telegram )

@flask_app.route("/")
def home():
    return "✅ البوت شغال عبر Vercel"

@flask_app.route("/webhook", methods=["POST"])
def webhook():
    update = Update.de_json(request.get_json(force=True), application.bot)
    asyncio.run(application.process_update(update))
    return "ok"

# Vercel يحتاج متغير اسمه app
app = flask_app
