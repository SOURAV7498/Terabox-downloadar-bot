import asyncio
from pyrogram import Client, filters
from pyrogram.types import Message, InlineKeyboardMarkup, InlineKeyboardButton
from pyrogram.errors import UserNotParticipant

# 🔧 CONFIGURATION
BOT_TOKEN = "7527412469:AAHhA4W4cpMcbdUd0IJ5CtbN-mvQWyUNHjE"
API_ID = 21140176  # Get this from https://my.telegram.org
API_HASH = "b081ec8da8cf5263a6593041c1ae2a3b"
CHANNEL_USERNAME = "Hindi_movie_zone_SB"  # Without @

# 📦 Initialize the bot
app = Client("terabox_bot", bot_token=BOT_TOKEN, api_id=API_ID, api_hash=API_HASH)

# 🛑 Check if user is member of the channel
async def is_subscribed(user_id):
    try:
        member = await app.get_chat_member(CHANNEL_USERNAME, user_id)
        return member.status in ("member", "administrator", "creator")
    except UserNotParticipant:
        return False
    except:
        return False

# 🎬 Fake link extractor (you must replace with real Terabox extractor)
def get_direct_links(terabox_url):
    return {
        "download_link": f"https://fake-download-link.com?src={terabox_url}",
        "stream_link": f"https://fake-stream-link.com?src={terabox_url}"
    }

# 🚪 Start Command
@app.on_message(filters.command("start"))
async def start_cmd(client, message: Message):
    user = message.from_user
    if not await is_subscribed(user.id):
        join_button = InlineKeyboardMarkup([
            [InlineKeyboardButton("📢 Join Channel", url=f"https://t.me/{CHANNEL_USERNAME}")]
        ])
        await message.reply(
            "🚫 To use this bot, you must join our channel first.",
            reply_markup=join_button
        )
        return

    await message.reply("✅ Welcome! Send me any Terabox video link to get the download and stream links.")

# 🔗 Handle Terabox Links
@app.on_message(filters.text & ~filters.command(["start"]))
async def handle_link(client, message: Message):
    user = message.from_user
    if not await is_subscribed(user.id):
        join_button = InlineKeyboardMarkup([
            [InlineKeyboardButton("📢 Join Channel", url=f"https://t.me/{CHANNEL_USERNAME}")]
        ])
        await message.reply(
            "🚫 To use this bot, you must join our channel first.",
            reply_markup=join_button
        )
        return

    url = message.text.strip()
    if "terabox" not in url:
        await message.reply("❌ Please send a valid Terabox video link.")
        return

    links = get_direct_links(url)

    reply_text = f"""
🎬 Terabox Video Link Processed

🔗 Direct Download: [Click Here]({links['download_link']})
▶️ Direct Stream: [Click Here]({links['stream_link']})

🕒 _This message will be deleted in 10 minutes, you can forward this message to any group or saved message._
"""
    sent_msg = await message.reply(reply_text, disable_web_page_preview=True)

    # ⏳ Schedule deletion in 10 minutes
    await asyncio.sleep(600)
    try:
        await sent_msg.delete()
    except:
        pass

# 🚀 Run the bot
app.run()
