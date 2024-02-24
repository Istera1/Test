import imaplib
import email
from email.header import decode_header

# Функция для получения кодов из почтового ящика
def get_codes(username, password, mail_server):
    try:
        # Подключение к почтовому серверу
        mail = imaplib.IMAP4_SSL(mail_server)
        mail.login(username, password)
        mail.select("inbox")

        # Поиск сообщений с кодами
        result, data = mail.search(None, '(SUBJECT "Security Code")')
        
        codes = []

        if result == 'OK':
            for num in data[0].split():
                # Получение содержимого сообщения
                result, data = mail.fetch(num, '(RFC822)')
                raw_email = data[0][1]
                msg = email.message_from_bytes(raw_email)
                
                # Получение текста сообщения
                if msg.is_multipart():
                    for part in msg.walk():
                        content_type = part.get_content_type()
                        content_disposition = str(part.get("Content-Disposition"))
                        if "attachment" not in content_disposition:
                            body = part.get_payload(decode=True).decode()
                else:
                    body = msg.get_payload(decode=True).decode()
                
                # Добавление кода в список
                codes.append(body.strip())

        mail.close()
        mail.logout()

        return codes

    except Exception as e:
        print("Error:", str(e))
        return []

# Пример использования функции
username = "your_email@example.com"
password = "your_password"
mail_server = "imap.example.com"

codes = get_codes(username, password, mail_server)
print("Received codes:", codes)
