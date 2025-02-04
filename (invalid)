import telebot
from telebot import types
import random
from threading import Timer

# Bot Token
TOKEN = '7912106541:AAHZI3rwpZCbGXt508FqaY9kE-gdIsZFNU8'
bot = telebot.TeleBot(TOKEN)

# Kullanıcı bakiyeleri ve bahisler
user_balances = {}
user_names = {}
bets = {}
selected_bet_amount = {}  # Kullanıcıların seçtiği bahis miktarı
active_games = set()  # Aktif oyunları takip etmek için
manual_bet_users = {}  # Manuel giriş yapan kullanıcıları takip etmek için
bet_message_ids = {}  # Bahis butonlarının mesaj kimliklerini takip etmek için

# Rulet görselleri klasör yolu
roulette_images_folder = '/storage/emulated/0/Rulet/'

# Rulet sayılarının renkleri
roulette_colors = {
    0: 'green', 1: 'red', 2: 'black', 3: 'red', 4: 'black', 5: 'red', 6: 'black', 
    7: 'red', 8: 'black', 9: 'red', 10: 'black', 11: 'black', 12: 'red', 13: 'black', 
    14: 'red', 15: 'black', 16: 'red', 17: 'black', 18: 'red', 19: 'red', 20: 'black', 
    21: 'red', 22: 'black', 23: 'red', 24: 'black', 25: 'red', 26: 'black', 27: 'red', 
    28: 'black', 29: 'black', 30: 'red', 31: 'black', 32: 'red', 33: 'black', 34: 'red', 
    35: 'black', 36: 'red'
}

# Bakiye formatlama
def format_amount(amount):
    suffixes = {10**12: 'T', 10**9: 'B', 10**6: 'M', 10**3: 'k'}
    for divisor, suffix in suffixes.items():
        if amount >= divisor:
            return f"{amount // divisor}{suffix} DTC 💰"
    return f"{amount} DTC 💰"

# Bahis miktarı butonları
def create_bet_amount_buttons():
    markup = types.InlineKeyboardMarkup(row_width=2)
    btn_5000 = types.InlineKeyboardButton("5000 DTC 💰", callback_data='amount_5000')
    btn_10000 = types.InlineKeyboardButton("10000 DTC 💰", callback_data='amount_10000')
    btn_1B = types.InlineKeyboardButton("1B DTC 💰", callback_data='amount_1000000000')
    btn_10B = types.InlineKeyboardButton("10B DTC 💰", callback_data='amount_10000000000')
    btn_manual = types.InlineKeyboardButton("Manuel Giriş 💬", callback_data='amount_manual')
    markup.add(btn_5000, btn_10000, btn_1B, btn_10B)
    markup.add(btn_manual)
    return markup

# Bahis butonları
def create_bet_buttons():
    markup = types.InlineKeyboardMarkup(row_width=3)
    btn_red = types.InlineKeyboardButton("🔴 Kırmızı", callback_data='bet_red')
    btn_black = types.InlineKeyboardButton("⚫ Siyah", callback_data='bet_black')
    btn_green = types.InlineKeyboardButton("🟢 Yeşil", callback_data='bet_green')
    btn_even = types.InlineKeyboardButton("Çift", callback_data='bet_even')
    btn_odd = types.InlineKeyboardButton("Tek", callback_data='bet_odd')
    btn_first12 = types.InlineKeyboardButton("1-12", callback_data='bet_first12')
    btn_second12 = types.InlineKeyboardButton("13-24", callback_data='bet_second12')
    btn_third12 = types.InlineKeyboardButton("25-36", callback_data='bet_third12')
    btn_numbers = types.InlineKeyboardButton("Sayı Seç", callback_data='bet_numbers')
    markup.add(btn_red, btn_black, btn_green)
    markup.add(btn_even, btn_odd)
    markup.add(btn_first12, btn_second12, btn_third12)
    markup.add(btn_numbers)
    return markup

# Sayı seçme butonları
def create_number_buttons():
    markup = types.InlineKeyboardMarkup(row_width=6)
    buttons = [types.InlineKeyboardButton(str(i), callback_data=f'bet_number_{i}') for i in range(37)]
    markup.add(*buttons)
    btn_back = types.InlineKeyboardButton("⬅️ Geri", callback_data='bet_back')
    markup.add(btn_back)
    return markup

# Kullanıcı adını alma fonksiyonu
def get_username(user_id):
    return user_names.get(user_id, f"ID-{user_id}")

# Kullanıcı adını değiştirme komutu
@bot.message_handler(commands=['changename'])
def change_name(message):
    chat_id = message.chat.id
    user_id = message.from_user.id
    try:
        new_name = message.text.split(maxsplit=1)[1]
        user_names[user_id] = new_name
        bot.send_message(chat_id, f"İsminiz '{new_name}' olarak değiştirildi.")
    except IndexError:
        bot.send_message(chat_id, "Lütfen yeni isminizi belirtin. Kullanım: /changename [yeni isim]")

# Bakiye sorgulama komutu
@bot.message_handler(commands=['balance'])
def check_balance(message):
    chat_id = message.chat.id
    user_id = message.from_user.id
    balance = user_balances.get(user_id, 10000000000)  # Varsayılan bakiye
    bot.send_message(chat_id, f"Bakiyeniz: {format_amount(balance)}")

# Rulet başlatma
@bot.message_handler(commands=['rulet'])
def start_rulet(message):
    global bets  # Global olarak deklarasyon
    chat_id = message.chat.id
    user_id = message.from_user.id

    if chat_id in active_games:
        bot.send_message(chat_id, "Şu anda zaten bir oyun açık. Lütfen sonuçların açıklanmasını bekleyin.")
        return

    active_games.add(chat_id)
    user_balances.setdefault(user_id, 10000000000)  # Varsayılan bakiye (10B DTC)
    selected_bet_amount[chat_id] = 5000  # Varsayılan bahis miktarı

    bot.send_message(chat_id, f"Rulet oyununa hoş geldiniz! Bakiyeniz: {format_amount(user_balances[user_id])}")
    bet_amount_message = bot.send_message(chat_id, "Bahis miktarınızı seçin:", reply_markup=create_bet_amount_buttons())
    bet_message = bot.send_message(chat_id, "Bahislerinizi yapabilirsiniz!", reply_markup=create_bet_buttons())
    
    bet_message_ids[chat_id] = [bet_amount_message.message_id, bet_message.message_id]
    
    try:
        image_path = f"{roulette_images_folder}rulet.çark.jpg"
        with open(image_path, 'rb') as image_file:
            bot.send_photo(chat_id, image_file)
    except FileNotFoundError:
        bot.send_message(chat_id, "Çark görseli bulunamadı.")
        active_games.remove(chat_id)
        return

    bets = {}  # Bahisleri temizle
    bot.send_message(chat_id, "Bahis yapmak için 25 saniyeniz var!")
    Timer(25, roulette_game, args=[chat_id]).start()

# Bahis işlemleri
@bot.callback_query_handler(func=lambda call: True)
def handle_bets(call):
    global bets  # Global olarak deklarasyon
    chat_id = call.message.chat.id
    user_id = call.from_user.id
    user_balances.setdefault(user_id, 10000000000)  # Varsayılan bakiye (10B DTC)
    selected_bet_amount.setdefault(chat_id, 5000)  # Varsayılan bahis miktarı

    if call.data.startswith('amount_'):
        if call.data == 'amount_manual':
            msg = bot.send_message(chat_id, "Lütfen bahis miktarını girin:")
            manual_bet_users[user_id] = chat_id
            bot.register_next_step_handler(msg, process_manual_bet_amount, chat_id, user_id)
        else:
            selected_bet_amount[chat_id] = int(call.data.split('_')[1])
            try:
                bot.answer_callback_query(call.id, f"Bahis miktarı {format_amount(selected_bet_amount[chat_id])} olarak ayarlandı.")
            except telebot.apihelper.ApiTelegramException as e:
                print(f"Error answering callback query: {e}")

    elif call.data.startswith('bet_'):
        bet_type = call.data.split('_')[1]

        if bet_type == 'numbers':
            bot.edit_message_reply_markup(chat_id, call.message.message_id, reply_markup=create_bet_buttons_with_numbers())
        elif bet_type == 'back':
            bot.edit_message_reply_markup(chat_id, call.message.message_id, reply_markup=create_bet_buttons())
        else:
            bet_amount = selected_bet_amount[chat_id]
            if bet_amount > user_balances[user_id]:
                bot.send_message(chat_id, "Yetersiz bakiye!")
                return

            user_balances[user_id] -= bet_amount
            symbol = ''
            if bet_type == 'red':
                symbol = '🔴'
            elif bet_type == 'black':
                symbol = '⚫'
            elif bet_type == 'green':
                symbol = '🟢'
            elif bet_type.startswith('number'):
                bet_number = bet_type.split('_')[1]
                symbol = f'{bet_number}'

            if bet_type.startswith('number_'):
                bet_number = bet_type.split('_')[1]
                bets.setdefault(user_id, []).append((bet_number, bet_amount))
                bot.send_message(chat_id, f"{symbol} number {bet_number} için {format_amount(bet_amount)} bahis yaptınız.")
            else:
                bets.setdefault(user_id, []).append((bet_type, bet_amount))
                bot.send_message(chat_id, f"{symbol} {bet_type.upper()} için {format_amount(bet_amount)} bahis yaptınız.")

def process_manual_bet_amount(message, chat_id, user_id):
    if user_id in manual_bet_users and manual_bet_users[user_id] == chat_id:
        try:
            bet_amount = int(message.text)
            if bet_amount <= 0:
                bot.send_message(chat_id, "Lütfen geçerli bir bahis miktarı girin.")
                return
            selected_bet_amount[chat_id] = bet_amount
            bot.send_message(chat_id, f"Bahis miktarı {format_amount(bet_amount)} olarak ayarlandı.")
        except ValueError:
            bot.send_message(chat_id, "Geçersiz miktar. Lütfen geçerli bir sayı girin.")
        finally:
            del manual_bet_users[user_id]  # İşlem tamamlandıktan sonra kullanıcıyı listeden çıkar

# Bahis ve sayı butonlarını birleştirme
def create_bet_buttons_with_numbers():
    markup = types.InlineKeyboardMarkup(row_width=6)
    bet_buttons = [
        types.InlineKeyboardButton("🔴 Kırmızı", callback_data='bet_red'),
        types.InlineKeyboardButton("⚫ Siyah", callback_data='bet_black'),
        types.InlineKeyboardButton("🟢 Yeşil", callback_data='bet_green'),
        types.InlineKeyboardButton("Çift", callback_data='bet_even'),
        types.InlineKeyboardButton("Tek", callback_data='bet_odd'),
        types.InlineKeyboardButton("1-12", callback_data='bet_first12'),
        types.InlineKeyboardButton("13-24", callback_data='bet_second12'),
        types.InlineKeyboardButton("25-36", callback_data='bet_third12')
    ]
    number_buttons = [types.InlineKeyboardButton(str(i), callback_data=f'bet_number_{i}') for i in range(37)]
    markup.add(*bet_buttons)
    markup.add(*number_buttons)
    markup.add(types.InlineKeyboardButton("⬅️ Geri", callback_data='bet_back'))
    return markup

# Rulet oyunu
def roulette_game(chat_id):
    global bets  # Global olarak deklarasyon
    result = random.randint(0, 36)

    # Bahisler kapandı mesajı ve görseli silme
    if chat_id in bet_message_ids:
        for message_id in bet_message_ids[chat_id]:
            bot.delete_message(chat_id, message_id)

    bot.send_message(chat_id, "Bahisler kapandı!")

    # Sonuç sembolü ve rengi
    result_color = roulette_colors[result]
    result_symbol = '🟢' if result_color == 'green' else '🔴' if result_color == 'red' else '⚫'

    # Sonuç görseli ve kazananları gönder
    winners = []
    for user_id, bets_list in bets.items():
        total_winnings = 0
        for bet_type, bet_amount in bets_list:
            if bet_type == str(result):  # Sayı bahsi
                winnings = bet_amount * 36
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'red' and result_color == 'red':  # Kırmızı bahis
                winnings = bet_amount * 2
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'black' and result_color == 'black':  # Siyah bahis
                winnings = bet_amount * 2
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'green' and result_color == 'green':  # Yeşil bahis
                winnings = bet_amount * 36
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'even' and result % 2 == 0 and result != 0:  # Çift sayı bahis
                winnings = bet_amount * 2
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'odd' and result % 2 != 0:  # Tek sayı bahis
                winnings = bet_amount * 2
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'first12' and 1 <= result <= 12:  # 1-12 bölgesi bahis
                winnings = bet_amount * 3
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'second12' and 13 <= result <= 24:  # 13-24 bölgesi bahis
                winnings = bet_amount * 3
                user_balances[user_id] += winnings
                total_winnings += winnings
            elif bet_type == 'third12' and 25 <= result <= 36:  # 25-36 bölgesi bahis
                winnings = bet_amount * 3
                user_balances[user_id] += winnings
                total_winnings += winnings

        if total_winnings > 0:
            winners.append(f"💲 {get_username(user_id)}: {format_amount(total_winnings)} kazandı") 

    try:
        image_path = f"{roulette_images_folder}rulet.{result}.jpg"
        with open(image_path, 'rb') as image_file:
            result_text = f"Rulet sonucu: {result} ({result_color}) {result_symbol}\n"
            if winners:
                result_text += "\nKazananlar:\n" + "\n".join(winners)
            bot.send_photo(chat_id, image_file, caption=result_text)
    except FileNotFoundError:
        result_text = f"Rulet sonucu: {result} ({result_color}) {result_symbol}\n"
        if winners:
            result_text += "\nKazananlar:\n" + "\n".join(winners)
        bot.send_message(chat_id, result_text)
    
    bets.clear()
    active_games.remove(chat_id)

# Para gönderme komutu
@bot.message_handler(commands=['moneys'])
def send_money(message):
    chat_id = message.chat.id
    user_id = message.from_user.id
    try:
        target_id, amount = map(int, message.text.split()[1:3])
        if user_balances[user_id] >= amount:
            user_balances[user_id] -= amount
            user_balances[target_id] = user_balances.get(target_id, 0) + amount
            bot.send_message(chat_id, f"{format_amount(amount)} başarıyla gönderildi.")
        else:
            bot.send_message(chat_id, "Yetersiz bakiye!")
    except ValueError:
        bot.send_message(chat_id, "Geçersiz komut. Kullanım: /moneys [ID] [miktar]")

# Yardım komutu
@bot.message_handler(commands=['help'])
def help_command(message):
    help_text = (
        "/changename [yeni isim] - Kullanıcı adınızı değiştirir.\n"
        "/balance - Bakiyenizi gösterir.\n"
        "/rulet - Rulet oyununu başlatır.\n"
        "/moneys [ID] [miktar] - Belirtilen ID'ye belirtilen miktarda para gönderir.\n"
    )
    bot.send_message(message.chat.id, help_text)

# Botu başlat
bot.polling()