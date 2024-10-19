from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler

# قائمة لتخزين الطلبات
requests = []

async def start(update: Update, context):
    await update.message.reply_text('مرحبا! أرسل طلباتك هنا.')

async def add_request(update: Update, context):
    request_text = ' '.join(context.args)
    requests.append({'text': request_text, 'status': 'غير جاهز'})
    await update.message.reply_text(f'تم إضافة الطلب: {request_text}')

async def mark_ready(update: Update, context):
    keyboard = []
    for idx, request in enumerate(requests):
        button = InlineKeyboardButton(request['text'], callback_data=str(idx))
        keyboard.append([button])
    
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text('اختر الطلب الذي تم تجهيزه:', reply_markup=reply_markup)

async def button_handler(update: Update, context):
    query = update.callback_query
    await query.answer()
    
    idx = int(query.data)
    requests[idx]['status'] = 'جاهز'
    await query.edit_message_text(text=f'تم تجهيز الطلب: {requests[idx]["text"]}')

async def list_requests(update: Update, context):
    if not requests:
        await update.message.reply_text('لا توجد طلبات.')
        return

    message = "الطلبات:\n"
    for request in requests:
        message += f'- {request["text"]}: {request["status"]}\n'
    await update.message.reply_text(message)

async def main():
    # استبدل التوكن بالتوكن الخاص بك
    application = Application.builder().token('YOUR_BOT_TOKEN').build()

    # تحضير المعالجين
    application.add_handler(CommandHandler('start', start))
    application.add_handler(CommandHandler('add', add_request))
    application.add_handler(CommandHandler('list', list_requests))
    application.add_handler(CommandHandler('mark_ready', mark_ready))
    application.add_handler(CallbackQueryHandler(button_handler))

    # بدء البوت
    await application.start()
    await application.updater.idle()

if __name__ == '__main__':
    import asyncio
    asyncio.run(main())