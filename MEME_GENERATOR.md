# «Генератор мемов»

«Генератор мемов» — твой карманный мастер по производству смеха! Выбери картинку, добавь текст и вуаля! — у тебя мем, способный разбудить даже спящего кота! 😛 

💡 Идеально для:
- Унижений в групповом чате,
- Вечерних посиделок с ноутом и печеньками,
- Работы (если вы SMM-щик, конечно😏).

Создавай, смейся, делись.

## Функциональность

Бот должен генерировать мемы, используя изображения и текст. Для этого он берёт изображение и рисует нужный текст внизу этого изображения (пример ниже).

![meme.jpg](misc/meme_example.jpg)

**Основная функциональность:** бот должен поддерживать команду `/generate`, которая и генерирует мемы (в итоге бот присылает сообщение с изображением). Бот должен спросить у пользователя, какой текст вписать в мем.

***Уровни сложности***:
1. *Easy*. Бот рандомно выбирает изображение из своей базы данных изображений.
2. *Medium*. Бот позволяет пользователю выбрать изображение, которое будет использовано для генерации.
3. *Advanced*. Бот позволяет пользователю загружать свои собственные изображения, которые могут быть использованы для генерации.
4. *Advanced+*. Бот позволяет конфигурировать размер текста на изображении, отступ снизу, шрифт и цвет текста.

## Реализация

Бот должен использовать изображения, сохранённые в директории на диске (например, можно создать папку `memes` прямо в проект PyCharm).

Если бот позволяет пользователям загружать свои собственные изображения, то он должен сохранять их все либо в другую общую папку (например, `downloads`), либо создавать под каждого пользователя свою собственную папку (тогда папку можно именовать как `user-$id`, например: `user-619700989`).

### Рисование текста на изображении

```python
from io import BytesIO

from PIL import Image, ImageFont, ImageDraw

label = "Когда регистрация на Сигму"  # Сам текст
image_path = "../memes/cow.jpg"  # Путь к исходному изображению 
font_family = "DejaVuSans.ttf"  # Шрифт
font_size = 55  # Размер текста
color = (0, 0, 0)  # Цвет текста
bottom_indent = 7  # Отступ текста снизу в условных единицах
out_format: str = "WEBP"  # формат итогового изображения

# Итоговое изображение 
buffer = BytesIO()

# Открытие исходного изображения, рисование текста, показ итогового изображения и сохранение его в буфер
with Image.open(image_path) as im:
    unicode_font = ImageFont.truetype(font_family, font_size)
    xy = (im.size[0] // 2, im.size[1] - bottom_indent * im.size[1] // 100)
    ImageDraw.Draw(im).text(
        xy=xy,
        text=label,
        fill=color,
        font=unicode_font,
        anchor="mm",
    )
    im.show()
    im.save(fp=buffer, format=out_format)

# Подготовка изображения для отправки
buffer.seek(0)
```

> `pip install pillow`

> Цвет текста состоит из трёх компонент (red, green, blue), каждая со значением от 0 до 255.
> - **(0, 0, 0)** — чёрный,
> - **(255, 255, 255)** — белый,
> - **(255, 0, 0)** — красный, 
> и т.д.

### Отправка изображений

```python
from aiogram.types import Message, BufferedInputFile

async def handler(message: Message):
    buffer = ...  # Сгенерированный мем в формате BytesIO

    await message.answer_photo(
        BufferedInputFile(
            buffer.read(),
            filename="Какое-то название файла для Telegram",
        ),
    )

    buffer.close()
```


### Скачивание изображений из Telegram

```python
import aiofiles
from pathlib import Path

import aiohttp
from aiogram import Bot, F
from aiogram.types import Message, BufferedInputFile


@router.message(F.photo)
async def handler(message: Message, bot: Bot, session: aiohttp.ClientSession):
    downloads_folder_path = "../downloads"  # Путь к директории, куда нужно скачать изображение
    
    photo = message.photo[-1].file_id  # Изображение из сообщения
    photo_file = await bot.get_file(photo)  # Файл
    photo_path = photo_file.file_path  # Путь к изображению в Telegram (то, с чем Telegram работает "под капотом")
    
    p = Path(photo_path)
    path = f"{downloads_folder_path}/{p.name}"

    async with session.get(photo_path) as resp:
        if resp.status == 200:
            f = await aiofiles.open(path, mode='wb')
            await f.write(await resp.read())
            await f.close()

)
```

> `F.photo` в `@router.message` позволяет указать фильтр, то есть данный обработчик сработает лишь если сообщение будет содержать изображение

> `pip install aiohttp`

Для корректной работы нужно определить в `Dispatcher` значения для `bot` и `session` следующим образом:

```python
import aiohttp

dp = Dispatcher()

async def main() -> None:
    ...
    
    bot = Bot(token="TOKEN")

    ...
    
    dp["bot"] = bot
    dp["session"] = aiohttp.ClientSession(
        base_url=f"https://api.telegram.org/file/bot{"TOKEN"}/",
    )
    
    ...
    
    await dp.start_polling(bot)
```

### Получение содержимого директории

```python
import os
from pathlib import Path


directory_path = "../memes"  # Путь к директории

for f in os.listdir(directory_path):
    p = Path(f)
    print(p.name, p.stem)
```

### Получение списка всех доступных в системе шрифтов

```python
from matplotlib import font_manager


for n in font_manager.get_font_names():
    print(n)
```
> `pip install matplotlib`


## Предполагаемый состав команды и роли

Всё зависит от уровня сложности (или уровень сложности от состава команды — пока не решил). **Приблизительные** предполагаемые роли для уровня *Advanced+*:
- 1 человек занимается генерацией мемов,
- 1 человек занимается работой с изображениями,
- 2 человека занимаются обработчиками и всем остальным.
