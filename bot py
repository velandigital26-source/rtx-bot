import logging, requests
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, ContextTypes

BOT_TOKEN = "9271244050:AAHX50mK87jczinv-8rf6o8oOVtHA-oYXTc"
ADMIN_ID = 0
FIREBASE_URL = "https://rp-admin06-default-rtdb.firebaseio.com"

logging.basicConfig(level=logging.INFO)

def fb_get(path):
    try:
        r = requests.get(f"{FIREBASE_URL}/{path}.json", timeout=15)
        if r.status_code == 200: return r.json()
        return None
    except: return None

def fb_set(path, data):
    try:
        r = requests.put(f"{FIREBASE_URL}/{path}.json", json=data, timeout=15)
        return r.status_code == 200
    except: return False

def fb_push(path, data):
    try:
        r = requests.post(f"{FIREBASE_URL}/{path}.json", json=data, timeout=15)
        if r.status_code == 200: return r.json()
        return None
    except: return None

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keys_data = fb_get("RTX_Keys") or {}
    stock_count = {}
    for k, v in keys_data.items():
        if isinstance(v, dict) and not v.get("sold"):
            name = f"{v.get('game','?')} {v.get('days','?')}D"
            stock_count[name] = stock_count.get(name, 0) + 1
    text = "🔥 *RTX RESELLING STORE* 🔥\n\n"
    if not stock_count:
        text += "😢 *Stock Empty da!*\nAdmin /addkey pannuven!\n\n"
    else:
        text += "📦 *AVAILABLE STOCK:*\n"
        for item, cnt in stock_count.items():
            text += f"• {item}: *{cnt} Keys*\n"
        text += "\n"
    text += "👇 Buy pannu da!"
    keyboard = [
        [InlineKeyboardButton("🛒 BUY NOW", callback_data="buy_menu")],
        [InlineKeyboardButton("💰 MY BALANCE", callback_data="my_bal")],
        [InlineKeyboardButton("📞 ADMIN", url="https://t.me/rtx_reselling_bot")]
    ]
    await update.message.reply_text(text, reply_markup=InlineKeyboardMarkup(keyboard), parse_mode="Markdown")

async def buttons(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    keys_data = fb_get("RTX_Keys") or {}
    if query.data == "buy_menu":
        games = {}
        for kid, info in keys_data.items():
            if isinstance(info, dict) and not info.get("sold"):
                g = f"{info.get('game')} {info.get('days')}D"
                games[g] = games.get(g, 0) + 1
        if not games:
            await query.edit_message_text("😢 Stock illa da!")
            return
        keyboard = []
        for gname, cnt in games.items():
            keyboard.append([InlineKeyboardButton(f"{gname} ({cnt})", callback_data=f"buy_{gname}")])
        await query.edit_message_text("👇 *Game Select Pannu da:*", reply_markup=InlineKeyboardMarkup(keyboard), parse_mode="Markdown")
    elif query.data.startswith("buy_"):
        game_name = query.data.replace("buy_", "")
        found_id, found_key = None, None
        for k_id, info in keys_data.items():
            if not isinstance(info, dict): continue
            g = f"{info.get('game')} {info.get('days')}D"
            if g == game_name and not info.get("sold"):
                found_id = k_id
                found_key = info.get("key")
                break
        if not found_key:
            await query.edit_message_text("❌ Stock thindhuchu da!")
            return
        fb_set(f"RTX_Keys/{found_id}/sold", True)
        fb_set(f"RTX_Keys/{found_id}/sold_to", str(query.from_user.id))
        await query.edit_message_text(f"✅ *SUCCESS DA!*\n\nGame: *{game_name}*\nKey: `{found_key}`\n\nCopy panniko! 🔥", parse_mode="Markdown")
    elif query.data == "my_bal":
        await query.edit_message_text("💰 Ippo free delivery da!")

async def addkey(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if len(context.args) < 3:
        await update.message.reply_text("Usage:\n/addkey GAME DAYS KEY\nEx: /addkey CARROM 30 RTX-ABC-123")
        return
    game = context.args[0].upper()
    days = context.args[1]
    key = " ".join(context.args[2:])
    data = {"game": game, "days": days, "key": key, "sold": False}
    res = fb_push("RTX_Keys", data)
    if res:
        await update.message.reply_text(f"✅ *Added!*\nGame: {game} {days}D\nKey: {key}", parse_mode="Markdown")
    else:
        await update.message.reply_text("❌ Firebase error! Rules check pannu!")

async def stock(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keys_data = fb_get("RTX_Keys") or {}
    total = len([k for k in keys_data.values() if isinstance(k, dict) and not k.get("sold")])
    await update.message.reply_text(f"📦 Total Stock: {total}")

def main():
    app = Application.builder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("addkey", addkey))
    app.add_handler(CommandHandler("stock", stock))
    app.add_handler(CallbackQueryHandler(buttons))
    print("BOT LIVE")
    app.run_polling()

if __name__ == "__main__":
    main()
