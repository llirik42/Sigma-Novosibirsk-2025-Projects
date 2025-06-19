# «Генератор мемов»

Для генерации мемов рекомендуется использовать функцию `generate_meme()` из фрагмента кода ниже. Данная функция в качестве результата возвращает сгенерированный пароль, а принимает следующие параметры:
- **length** — длина генерируемого пароля,
- **digits** — использовать ли цифры для генерации пароля (`True` — да, `False` — нет),
- **lowercase** — использовать ли строчные буквы для генерации пароля (`True` — да, `False` — нет),
- **uppercase** — использовать ли заглавные буквы для генерации пароля (`True` — да, `False` — нет),
- **special** — использовать ли специальные символы для генерации пароля (`True` — да, `False` — нет),

Функция либо возвращает сгенерированный пароль, либо `None` в случае если ни один из параметров не был равен `True`, то есть если не из чего было генерировать пароль.


```python
from io import BytesIO
from typing import Optional

from PIL import Image, ImageFont, ImageDraw


def generate_meme(image_path: str,
                  label: str,
                  font_size: int = 70,
                  color: Optional[tuple[int, int, int]] = None,
                  out_format: str = "WEBP",
                  bottom_indent_perc: int = 7,
                  font: str = "DejaVuSans.ttf") -> BytesIO:
    if color is None:
        color = (0, 0, 0)

    buffer = BytesIO()

    with Image.open(image_path) as im:
        unicode_font = ImageFont.truetype(font, font_size)
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
