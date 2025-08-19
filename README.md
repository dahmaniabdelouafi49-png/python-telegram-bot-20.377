import logging
import datetime
import json
import os
import random
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import (
    Application,
    CommandHandler,
    CallbackQueryHandler,
    ContextTypes,
)

# 🔑 التوكن الخاص بالبوت
TOKEN = "7653495962:AAF_mpEDM_2i1YTU94OjGTvwRfat_K9MJeY"

# 📢 رابط قناة البوت
BOT_CHANNEL = "https://t.me/xxcb12xx/9"

# 🗂️ ملف تخزين المستخدمين
DATA_FILE = "users.json"

# ⏰ وقت الهدايا
daily_gift = {"points": 200, "hours": 24}
weekly_gift = {"points": 350, "days": 7}

# 📌 تهيئة اللوجز
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)

# 🛠️ تحميل / حفظ البيانات
def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    return {"users": {}, "funded_channels": []}

def save_data():
    with open(DATA_FILE, "w") as f:
        json.dump(users_data, f, indent=4)

# 🗂️ تحميل البيانات عند بداية التشغيل
users_data = load_data()
if "users" not in users_data:
    users_data["users"] = {}
if "funded_channels" not in users_data:
    users_data["funded_channels"] = []

# 🏠 إنشاء حساب للمستخدم لو مش موجود
def ensure_user(user_id: str):
    if user_id not in users_data["users"]:
        users_data["users"][user_id] = {
            "points": 0,
            "last_daily": None,
            "last_weekly": None,
            "orders": [],
        }
        save_data()

# 🏠 /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = str(update.effective_user.id)
    ensure_user(user_id)

    keyboard = [
        [InlineKeyboardButton("🎁 الهدية اليومية", callback_data="daily")],
        [InlineKeyboardButton("📦 الهدية الأسبوعية", callback_data="weekly")],
        [InlineKeyboardButton("🎡 عجلة الحظ", callback_data="wheel")],
        [InlineKeyboardButton("🛠️ الخدمات المجانية", callback_data="services")],
        [InlineKeyboardButton("📋 طلباتي", callback_data="orders")],
        [InlineKeyboardButton("🎟️ استخدام كود", callback_data="use_code")],
        [InlineKeyboardButton("💰 تجميع نقاط", callback_data="collect_points")],
        [InlineKeyboardButton("➕ نقاط ذاتية", callback_data="self_points")],
        [InlineKeyboardButton("📢 تمويل قناتي", callback_data="fund_channel")],
        [InlineKeyboardButton("📊 إحصائيات البوت", callback_data="bot_stats")],
        [InlineKeyboardButton("📺 قناة البوت", url=BOT_CHANNEL)],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text(
        "👋 مرحبا بك في البوت!\nاختر من القائمة:", reply_markup=reply_markup
    )

# 🎁 الهدية اليومية
async def daily_gift_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = str(query.from_user.id)
    ensure_user(user_id)
    user = users_data["users"][user_id]

    now = datetime.datetime.now()
    if user["last_daily"] and (now - datetime.datetime.fromisoformat(user["last_daily"])).total_seconds() < daily_gift["hours"] * 3600:
        remaining = daily_gift["hours"] * 3600 - (now - datetime.datetime.fromisoformat(user["last_daily"])).total_seconds()
        hours, minutes = divmod(int(remaining // 60), 60)
        await query.edit_message_text(
            f"⏳ انتظر {hours} ساعة و {minutes} دقيقة للحصول على هديتك اليومية."
        )
        return

    user["points"] += daily_gift["points"]
    user["last_daily"] = now.isoformat()
    save_data()
    await query.edit_message_text(
        f"✅ حصلت على {daily_gift['points']} نقطة كهديتك اليومية! 🎁\nرصيدك الحالي: {user['points']} نقطة."
    )

# 📦 الهدية الأسبوعية
async def weekly_gift_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = str(query.from_user.id)
    ensure_user(user_id)
    user = users_data["users"][user_id]

    now = datetime.datetime.now()
    if user["last_weekly"] and (now - datetime.datetime.fromisoformat(user["last_weekly"])).days < weekly_gift["days"]:
        remaining_days = weekly_gift["days"] - (now - datetime.datetime.fromisoformat(user["last_weekly"])).days
        await query.edit_message_text(
            f"⏳ انتظر {remaining_days} يوم للحصول على هديتك الأسبوعية."
        )
        return

    user["points"] += weekly_gift["points"]
    user["last_weekly"] = now.isoformat()
    save_data()
    await query.edit_message_text(
        f"✅ حصلت على {weekly_gift['points']} نقطة كهديتك الأسبوعية! 🎁\nرصيدك الحالي: {user['points']} نقطة."
    )

# ➕ النقاط الذاتية
async def self_points_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    keyboard = [
        [InlineKeyboardButton("➕ 1", callback_data="sp_1"),
         InlineKeyboardButton("➕ 3", callback_data="sp_3")],
        [InlineKeyboardButton("➕ 5", callback_data="sp_5"),
         InlineKeyboardButton("➕ 10", callback_data="sp_10")],
        [InlineKeyboardButton("➕ 15", callback_data="sp_15"),
         InlineKeyboardButton("➕ 30", callback_data="sp_30")],
        [InlineKeyboardButton("➕ 35", callback_data="sp_35")],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await query.edit_message_text("اختر عدد النقاط التي تريد إضافتها:", reply_markup=reply_markup)

# 🎯 تنفيذ إضافة النقاط الذاتية
async def add_self_points(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = str(query.from_user.id)
    ensure_user(user_id)
    user = users_data["users"][user_id]

    points_to_add = int(query.data.split("_")[1])  # يأخذ الرقم من callback مثل sp_10
    user["points"] += points_to_add
    save_data()

    await query.edit_message_text(
        f"✅ تمت إضافة {points_to_add} نقطة لرصيدك.\nرصيدك الحالي: {user['points']} نقطة."
    )

# 🎡 عجلة الحظ (عشوائية)
async def wheel_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user_id = str(query.from_user.id)
    ensure_user(user_id)
    user = users_data["users"][user_id]

    reward = random.choice([10, 20, 30, 50, 100, 200])
    user["points"] += reward
    save_data()

    await query.edit_message_text(
        f"🎡 دارت عجلة الحظ...\n✅ ربحت {reward} نقطة!\nرصيدك الحالي: {user['points']} نقطة."
    )

# 📊 إحصائيات البوت
async def bot_stats_func(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    total_users = len(users_data["users"])
    total_channels = len(users_data["funded_channels"])

    await query.edit_message_text(
        f"📊 إحصائيات البوت:\n\n👥 عدد المستخدمين: {total_users}\n📢 عدد القنوات/المجموعات الممولة: {total_channels}"
    )

# 🏃‍♂️ التشغيل
def main():
    application = Application.builder().token(TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(daily_gift_func, pattern="^daily$"))
    application.add_handler(CallbackQueryHandler(weekly_gift_func, pattern="^weekly$"))
    application.add_handler(CallbackQueryHandler(wheel_func, pattern="^wheel$"))
    application.add_handler(CallbackQueryHandler(self_points_func, pattern="^self_points$"))
    application.add_handler(CallbackQueryHandler(add_self_points, pattern="^sp_"))
    application.add_handler(CallbackQueryHandler(bot_stats_func, pattern="^bot_stats$"))

    application.run_polling()

if __name__ == "__main__":
    main()
    
