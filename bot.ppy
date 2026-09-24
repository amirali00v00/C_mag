import os
import asyncio
import random
from aiohttp import web, ClientSession

BOT_TOKEN = os.environ.get("BOT_TOKEN")
PIXABAY_KEY = os.environ.get("PIXABAY_KEY")
WEBHOOK_URL = os.environ.get("WEBHOOK_URL")


async def send_message(session, chat_id, text):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    await session.post(url, json={"chat_id": chat_id, "text": text})


async def search_and_send(chat_id, query):
    async with ClientSession() as session:
        try:
            url = (
                f"https://pixabay.com/api/?key={PIXABAY_KEY}"
                f"&q={query}&per_page=20&safesearch=true&image_type=photo"
            )
            async with session.get(url) as resp:
                data = await resp.json()

            if not data.get("hits"):
                await send_message(session, chat_id, "چیزی پیدا نکردم 😕")
                return

            chosen = random.choice(data["hits"])
            image_url = chosen["webformatURL"]

            send_url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendPhoto"
            payload = {
                "chat_id": chat_id,
                "photo": image_url,
                "caption": f"نتیجه برای: {query}",
            }
            async with session.post(send_url, json=payload) as resp:
                if resp.status != 200:
                    print(await resp.text())
                    await send_message(session, chat_id, "عکس پیدا شد ولی تلگرام نتونست بفرسته 😅")
        except Exception as e:
            print(f"Error: {e}")
            await send_message(session, chat_id, "یه خطایی پیش اومد.")


async def webhook_handler(request):
    try:
        update = await request.json()
        if "message" in update and "text" in update["message"]:
            chat_id = update["message"]["chat"]["id"]
            user_text = update["message"]["text"]
            asyncio.create_task(search_and_send(chat_id, user_text))
        return web.Response(status=200)
    except Exception as e:
        print(f"Webhook error: {e}")
        return web.Response(status=500)


async def on_startup(app):
    if WEBHOOK_URL:
        async with ClientSession() as session:
            set_url = f"https://api.telegram.org/bot{BOT_TOKEN}/setWebhook"
            await session.post(set_url, json={"url": f"{WEBHOOK_URL}/webhook"})
            print("Webhook set!")


app = web.Application()
app.router.add_post("/webhook", webhook_handler)
app.router.add_get("/healthcheck", lambda r: web.Response(text="OK"))
app.on_startup.append(on_startup)


if __name__ == "__main__":
    web.run_app(app, port=int(os.environ.get("PORT", 8080)))
