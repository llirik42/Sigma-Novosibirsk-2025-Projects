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

### Работа с изображениями

Бот должен использовать изображения, сохранённые в директории на диске (например, можно создать папку `memes` прямо в проект PyCharm).

Если бот позволяет пользователям загружать свои собственные изображения, то он должен сохранять их все либо в другую общую папку (например, `downloads`), либо создавать под каждого пользователя свою собственную папку (тогда папку можно именовать как `user-$id`, например: `user-619700989`).

Для работы с изображениями можно использовать следующий фрагмент кода:

```python
import os
from pathlib import Path
from typing import Optional

import aiofiles
from aiogram.client.session import aiohttp



async def download_photo(
        downloads_folder_path: str,
        file_path: str,
        session: aiohttp.ClientSession,
) -> str:
    __ensure_directory_exists(downloads_folder_path)

    p = Path(file_path)
    path = f"{downloads_folder_path}/{p.name}"

    async with session.get(file_path) as resp:
        if resp.status == 200:
            f = await aiofiles.open(path, mode='wb')
            await f.write(await resp.read())
            await f.close()

    return path


def get_meme_list(meme_folder_path: str) -> str:
    __ensure_directory_exists(meme_folder_path)

    result = ""

    for f in os.listdir(meme_folder_path):
        p = Path(f)
        result += f"{p.stem}\n\n"

    return result


def find_meme(meme_folder_path: str, meme_name: str) -> Optional[str]:
    __ensure_directory_exists(meme_folder_path)

    for f in os.listdir(meme_folder_path):
        p = Path(f)
        if p.stem.lower() == meme_name.lower():
            return f"{meme_folder_path}/{f}"

    return None


def __ensure_directory_exists(path: str) -> None:
    if not os.path.exists(path):
        os.makedirs(path)
```

Из данного фрагмента кода нужно использовать следующие функции:
- `download_photo()` — скачать изображение пользователя,
- `get_meme_list()` — получение списка всех доступных мемов из папки,
- `find_meme()` — поиск мема по названию (либо возвращается путь к файлу, либо `None`).

Для скачивания изображения пользователя нужно сделать 2 вещи. Во-первых, в файле `bot.py` необходимо создать объект `session`, чтобы потом его везде использовать:

```python
dp = Dispatcher()
...

async def main() -> None:
    ...
dp["session"] = aiohttp.ClientSession(
    base_url=f"https://api.telegram.org/file/bot{BOT_TOKEN}/",
)
```

> Установить библиотеку можно с помощью `pip install aiohttp`

Во-вторых, чтобы скачать изображение пользователя из его сообщения, нужно сделать следующее:

```python
photo = message.photo[-1].file_id

photo_file = await bot.get_file(photo)

output_path = await download_photo(
    file_path=photo_file.file_path,
    session=...,
    downloads_folder_path=...,
)
```

### Генерация мемов

Для генерации мемов рекомендуется использовать функцию `generate_meme()` из фрагмента кода ниже. Данная функция в качестве результата возвращает изображение, а принимает следующие параметры:
- **image_path** — путь к изображению (например, "./memes/meme-1.jpg"),
- **label** — текст мема,
- **font_family** — шрифт текста (можно использовать фрагмент кода ниже для просмотра всех поддерживаем системой шрифтов). P.S. Не все шрифты поддерживают русский язык(
- **font_size** — размер текста в каких-то условных единицах,
- **color** — цвет текста, состоит из трёх компонент (red, green, blue), каждая со значением от 0 до 255.
  - **(0, 0, 0)** — чёрный,
  - **(255, 255, 255)** — белый,
  - **(255, 0, 0)** — красный,
  - и т.д.
- **bottom_indent_perc** — отступ текста снизу в условных единицах, 
- **out_format** — формат итогового изображения.

```python
from io import BytesIO

from PIL import Image, ImageFont, ImageDraw


def generate_meme(
        image_path: str,
        label: str,
        font_family: str = "DejaVuSans.ttf",
        font_size: int = 55,
        color: tuple[int, int, int] = None,
        bottom_indent_perc: int = 7,
        out_format: str = "WEBP",
) -> BytesIO:

    if color is None:
        color = (0, 0, 0)

    buffer = BytesIO()

    with Image.open(image_path) as im:
        unicode_font = ImageFont.truetype(font_family, font_size)
        xy = (im.size[0] // 2, im.size[1] - bottom_indent_perc * im.size[1] // 100)
        ImageDraw.Draw(im).text(
            xy=xy,
            text=label,
            fill=color,
            font=unicode_font,
            anchor="mm",
        )
        im.save(fp=buffer, format=out_format)

    buffer.seek(0)

    return buffer
```
> Установить библиотеку можно с помощью `pip install pillow`



```python
from matplotlib import font_manager


for n in font_manager.get_font_names():
    print(n)
```
> Установить библиотеку можно с помощью `pip install matplotlib`


Чтобы отправить пользователь сгенерированный мем можно использовать следующий фрагмент кода:

```python
from aiogram.types import BufferedInputFile


meme = generate_meme(...)

await message.answer_photo(
        BufferedInputFile(
            meme.read(),
            filename="придумать название",
        ),
    )
```

### Предполагаемый состав команды и роли

Всё зависит от уровня сложности (или уровень сложности от состава команды — пока не решил). Предполагаемые роли для уровня *Advanced+*:
- 1 человек занимается функцией для генерацией мемов (проверка её работоспособности, подкручивание параметров и т.д.),
- 1 человек занимается работой с изображениями (скачивание, получения списка изображений и т.д.),
- 2 человека занимаются обработчиками.
