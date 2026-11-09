# testb
🚗 راهنمای اتصال دانگل Bluetooth ELM327 به ویندوز

راهنمای مرحله‌به‌مرحله اتصال دانگل Bluetooth ELM327 به کامپیوتر ویندوزی، شناسایی پورت COM، تست ارتباط با Python و راه‌اندازی پروژه Car Assistant با اطلاعات واقعی خودرو.

---

📋 فهرست مطالب

- "۱. پیش‌نیازها" (#۱-پیشنیازها)
- "۲. اتصال دانگل به خودرو" (#۲-اتصال-دانگل-به-خودرو)
- "۳. اتصال بلوتوث به ویندوز" (#۳-اتصال-بلوتوث-به-ویندوز)
- "۴. شناسایی پورت COM" (#۴-شناسایی-پورت-com)
- "۵. بررسی پورت با Python" (#۵-بررسی-پورت-با-python)
- "۶. تست ارتباط با ELM327" (#۶-تست-ارتباط-با-elm327)
- "۷. اتصال به پروژه اصلی" (#۷-اتصال-به-پروژه-اصلی)
- "۸. عیب‌یابی" (#۸-عیبیابی)
- "۹. چک‌لیست نهایی" (#۹-چکلیست-نهایی)

---

۱. پیش‌نیازها

برای راه‌اندازی این سیستم به موارد زیر نیاز داریم:

- دانگل Bluetooth ELM327
- خودروی مجهز به پشتیبانی OBD-II
- کامپیوتر دارای سیستم‌عامل Windows
- بلوتوث فعال در ویندوز
- نصب Python (پیشنهاد: Python 3.13)
- پروژه Car Assistant

۲. اتصال دانگل به خودرو

۱. محل درگاه OBD-II خودرو را پیدا کنید. این درگاه معمولاً زیر فرمان یا اطراف داشبورد قرار دارد.

۲. دانگل ELM327 را به درگاه OBD-II متصل کنید.

۳. سوئیچ خودرو را روی حالت ON قرار دهید.

۴. چراغ‌های روی دانگل را بررسی کنید.

«نکته: برای Pair کردن معمولاً روشن بودن سوئیچ کافی است. برای دریافت اطلاعات RPM باید موتور روشن باشد.»

۳. اتصال بلوتوث به ویندوز

۱. وارد تنظیمات Windows شوید.

۲. بخش "Bluetooth & Devices" را باز کنید.

۳. بلوتوث را روشن کنید.

۴. روی "Add Device" کلیک کنید.

۵. گزینه "Bluetooth" را انتخاب کنید.

۶. منتظر شناسایی دانگل بمانید.

نام دانگل ممکن است یکی از موارد زیر باشد:

OBDII
OBD2
ELM327
V-LINK
OBD Bluetooth

در صورت درخواست رمز، رمزهای رایج زیر را امتحان کنید:

1234
0000

پس از اتصال، وضعیت دستگاه باید به‌صورت Paired یا Connected نمایش داده شود.

۴. شناسایی پورت COM

پس از Pair شدن دانگل، باید شماره پورت سریال آن را پیدا کنیم.

۱. کلیدهای "Win + X" را فشار دهید.

۲. گزینه "Device Manager" را باز کنید.

۳. قسمت "Ports (COM & LPT)" را پیدا کنید.

۴. پورت مربوط به دانگل را بررسی کنید.

نمونه:

Standard Serial over Bluetooth link (COM5)
Standard Serial over Bluetooth link (COM6)

اگر دو پورت Incoming و Outgoing وجود داشت، معمولاً باید پورت Outgoing را انتخاب کنید.

مثلاً:

OBD_PORT=COM6

«شماره COM در هر کامپیوتر متفاوت است. مقدار COM6 صرفاً نمونه است.»

۵. بررسی پورت با Python

ابتدا وارد پوشه پروژه شوید و محیط مجازی را فعال کنید.

.\venv\Scripts\Activate.ps1

نصب کتابخانه PySerial:

pip install pyserial

نمایش پورت‌های سریال موجود:

python -c "from serial.tools import list_ports; [print(p.device, '|', p.description) for p in list_ports.comports()]"

خروجی نمونه:

COM3 | Bluetooth Device
COM6 | Standard Serial over Bluetooth link

شماره پورت خروجی دانگل را یادداشت کنید.

۶. تست ارتباط با ELM327

در ریشه پروژه یک فایل با نام زیر ایجاد کنید:

test_bluetooth_obd.py

کد زیر را داخل فایل قرار دهید:

import obd

PORT = "COM6"  # شماره پورت خود را وارد کنید

connection = None

try:
    print("Connecting to ELM327...")

    connection = obd.OBD(
        portstr=PORT,
        baudrate=38400,
        fast=False,
        timeout=10,
    )

    print("Connected:", connection.is_connected())
    print("Port:", connection.port_name())

    if connection.is_connected():

        commands = {
            "RPM": obd.commands.RPM,
            "Speed": obd.commands.SPEED,
            "Coolant": obd.commands.COOLANT_TEMP,
            "Engine Load": obd.commands.ENGINE_LOAD,
            "Voltage": obd.commands.CONTROL_MODULE_VOLTAGE,
        }

        for name, command in commands.items():
            response = connection.query(command)
            print(f"{name}: {response.value}")

    else:
        print("Could not connect to vehicle.")

except Exception as error:
    print("Error:", repr(error))

finally:
    if connection is not None:
        connection.close()
        print("Connection closed.")

توجه: مقدار "COM6" را با شماره پورت واقعی دانگل جایگزین کنید.

اجرای تست:

python test_bluetooth_obd.py

خروجی نمونه در صورت موفقیت:

Connecting to ELM327...
Connected: True
Port: COM6

RPM: 850.0 revolutions_per_minute
Speed: 0 kilometers_per_hour
Coolant: 83 degrees_Celsius
Engine Load: 12.5 percent
Voltage: 14.2 volts

Connection closed.

مقادیر بالا صرفاً نمونه هستند و اطلاعات واقعی به وضعیت موتور و پشتیبانی ECU بستگی دارند.

۷. اتصال به پروژه اصلی

پس از موفقیت تست مستقل، فایل ".env" پروژه را باز کنید.

مقدار قبلی شبیه‌ساز:

OBD_PORT=socket://127.0.0.1:35000

مقدار جدید برای دانگل بلوتوث:

OBD_PORT=COM6

سپس پروژه را اجرا کنید:

python main.py

کلاس فعلی "OBDClient" پروژه، پورت را از تنظیمات دریافت می‌کند؛ بنابراین برای تغییر نوع اتصال، نیازی به تغییر اساسی ساختار پروژه نیست.

نکته درباره Baudrate

در پیاده‌سازی فعلی "OBDClient" مقدار Baudrate برابر 38400 است.

اگر اتصال برقرار نشد، باید سرعت ارتباط سریال دانگل بررسی شود. بعضی دانگل‌ها از سرعت‌های دیگری مانند 9600، 19200، 57600 یا 115200 استفاده می‌کنند.

۸. عیب‌یابی

خطا یا وضعیت| علت احتمالی
Access is denied| پورت توسط برنامه دیگری اشغال شده است.
Port not found| شماره COM اشتباه است.
Unable to connect| دانگل Pair نشده یا پورت اشتباه است.
No response from adapter| مشکل ارتباط سریال یا Baudrate
Unable to connect to vehicle| مشکل ارتباط دانگل با ECU خودرو
Connected: False| ارتباط کامل برقرار نشده است.

نکات مهم

- نرم‌افزارهای دیاگ دیگر را هنگام تست ببندید.
- شماره COM را مجدداً بررسی کنید.
- در صورت نیاز Baudrate را بررسی کنید.
- سوئیچ خودرو و اتصال فیزیکی دانگل را بررسی کنید.
- مطمئن شوید خودرو از پروتکل OBD-II پشتیبانی می‌کند.
- اگر دانگل از نوع BLE باشد، ممکن است پورت COM ایجاد نشود و به روش اتصال متفاوتی نیاز داشته باشید.

۹. چک‌لیست نهایی

- [ ] اتصال دانگل به درگاه OBD-II خودرو
- [ ] Pair کردن دانگل با ویندوز
- [ ] شناسایی پورت COM خروجی
- [ ] نصب PySerial
- [ ] اجرای تست مستقل Python
- [ ] برقراری ارتباط با ELM327
- [ ] دریافت اطلاعات RPM
- [ ] دریافت اطلاعات Speed
- [ ] دریافت دمای آب موتور
- [ ] تنظیم فایل ".env"
- [ ] اجرای "main.py"
- [ ] بررسی عملکرد Detectorها
- [ ] بررسی ارسال هشدارهای تلگرام

---

⚠️ نکته مهم درباره پروژه Car Assistant

در اجرای قبلی پروژه روی ویندوز، خطای اعتبارسنجی Pydantic مربوط به فیلد "diagnostic_status" مشاهده شده بود.

بنابراین موفقیت اتصال سخت‌افزاری لزوماً به معنی اجرای بدون خطای کل پروژه نیست.

پیشنهاد می‌شود ابتدا ارتباط سخت‌افزاری را با فایل "test_bluetooth_obd.py" تأیید کنید و سپس مشکلات احتمالی لایه پردازش داده را بررسی کنید.

---

🔗 منابع

- "مستندات python-OBD" (https://python-obd.readthedocs.io/)
- "مستندات PySerial" (https://pyserial.readthedocs.io/)
- "راهنمای Bluetooth در Windows" (https://support.microsoft.com/windows)

---

Car Assistant Project

"Windows + Python + Bluetooth ELM327 + OBD-II"