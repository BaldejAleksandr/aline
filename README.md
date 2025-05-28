# Объяснение кода DatePicker на Kivy

Этот код создает всплывающее окно для выбора даты (календарь) с возможностью:
- Перемещаться между месяцами и годами
- Выбирать конкретный день
- Возвращать выбранную дату в основную программу
- Подсвечивать текущий день и выбранную дату

## Принцип работы:
1. Создается всплывающее окно (Popup) с календарем
2. Пользователь выбирает дату с помощью кнопок и списков
3. При нажатии "ОК" выбранная дата передается в функцию обратного вызова (callback)
4. Окно закрывается

## Подробное объяснение каждой строки:

```python
from kivy.uix.gridlayout import GridLayout
from kivy.uix.button import Button
from kivy.uix.popup import Popup
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.spinner import Spinner
import calendar
from datetime import datetime, date
from kivy.metrics import dp, sp
```
**добавление библиотек**: эти строчки добавляют в код нужные функции, переменные и библиотеки для работы программы
---
```python
class SimpleDatePicker(Popup):
    def __init__(self, callback=None, **kwargs):
        super().__init__(**kwargs)
        self.callback = callback
        self.title = "Выберите дату"
        self.size_hint = (0.9, 0.8)
        self.current_date = datetime.now()
        self.selected_date = None

        self.build_calendar()
```
**Создание класса календаря**:
создаём шаблон календаря, где будут указаны некоторые характеристики и всякие функции
---
```python
    def build_calendar(self):
        main_layout = BoxLayout(orientation='vertical', spacing=10, padding=10)
```
**Создание основной заготовки окна, туда дорисовываем элементы**:
этот код делает пустое основное окно и настраиваем отступы и поля
---
```python
        # Хедер с месяцем и годом
        header_layout = BoxLayout(orientation='horizontal', size_hint_y=None, height=50)
```
**Шапка календаря**:
созадём окно, где будет отображаться дата
- Фиксированная высота 50 пикселей (`size_hint_y=None, height=50`)
---
```python
        # Кнопка предыдущий месяц
        prev_btn = Button(text='<', size_hint_x=0.2)
        prev_btn.bind(on_press=self.prev_month)
        header_layout.add_widget(prev_btn)
```
**Кнопка "<" для перехода к предыдущему месяцу**:
этот код делает кнопку <, которая перекидывает на предидущий месяц(вызывает `prev_month`), так же код добавлем кнопку в шапку

---

```python
        # Спиннер для месяца
        months = ['Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь',
                  'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь']
        self.month_spinner = Spinner(
            text=months[self.current_date.month - 1],
            values=months,
            size_hint_x=0.4
        )
        self.month_spinner.bind(text=self.on_month_change)
        header_layout.add_widget(self.month_spinner)
```
при нажатии на кнопку этот код выводит всплывающее окно с списком месяцев и можем выбрать, так же добавляем в шапку
---

```python
        # Спиннер для года
        current_year = self.current_date.year
        years = [str(y) for y in range(current_year - 10, current_year + 10)]
        self.year_spinner = Spinner(
            text=str(self.current_date.year),
            values=years,
            size_hint_x=0.3
        )
        self.year_spinner.bind(text=self.on_year_change)
        header_layout.add_widget(self.year_spinner)
```
такая же штука, что и выше, только с годами, так же добавляем в шапку
---

```python
        # Кнопка следующий месяц
        next_btn = Button(text='>', size_hint_x=0.2)
        next_btn.bind(on_press=self.next_month)
        header_layout.add_widget(next_btn)

        main_layout.add_widget(header_layout)
```
**создаёт кнопку для перехода к следующему месяцу**:
- Аналогична кнопке "<", но вызывает `next_month`
- Добавляется в шапку
-`main_layout.add_widget(header_layout)` - эта строчка добавляет всю заполненную шапку в основное окно
---
```python
        # Дни недели
        days_header = GridLayout(cols=7, size_hint_y=None, height=40)
        week_days = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']
        for day in week_days:
            label = Label(text=day, bold=True)
            days_header.add_widget(label)
        main_layout.add_widget(days_header)
```
**таблица с днями недели**: создаём 7 колонок и вписываем короткие названия дней недели, в конце добавляем таблицу в главное окно
---
```python
        # Календарная сетка
        self.calendar_grid = GridLayout(cols=7, spacing=2)
        self.update_calendar()
        main_layout.add_widget(self.calendar_grid)
```
**таблица календаря**:
добавляем в таблицу числа месяца и настраиваем отступы, за это отвечает функция `update_calendar()`, в конце добавляем на главное окно
---

```python
        # Кнопки управления
        button_layout = BoxLayout(orientation='horizontal', size_hint_y=None, height=50, spacing=10)

        today_btn = Button(text='Сегодня')
        today_btn.bind(on_press=self.select_today)
        button_layout.add_widget(today_btn)
```

создаём кнопку "Сегодня" - вызывает `select_today` для быстрого перехода к текущей дате
---
```python
        ok_btn = Button(text='ОК',
                        background_color=[1, 0.48, 0, 1],
                        color=[1, 1, 1, 1],
                        font_size=sp(18),
                        background_normal='',
                        background_down='',
                        size_hint=(0.5, 1),
                        halign="center",
                        valign="middle",
                        shorten=False,
                        max_lines=2,
                        )
        ok_btn.bind(on_press=self.on_ok)
        button_layout.add_widget(ok_btn)
```
**Кнопка "ОК"**:
создаём кнопку ОК, то что ввели в дату запоминает с помощью функции `on_ok`
---
```python
        cancel_btn = Button(
            text="Отменить",
            background_color=[0.902, 0, 0.110, 1],  # RGB: 230, 0, 28
            color=[1, 1, 1, 1],
            font_size=sp(15),
            size_hint=(0.5, 1),
            background_normal='',
            background_down='',
            halign="center",
            valign="middle",
            shorten=False,
            max_lines=2,
        )
        cancel_btn.bind(on_press=self.dismiss)
        button_layout.add_widget(cancel_btn)
        main_layout.add_widget(button_layout)

        self.content = main_layout
```
**Кнопка "Отменить"**:
создаём кнопку отменить, которая закрывает окно без выбора даты (`dismiss`)
- Добавляется в панель кнопок
- Вся панель с кнопками добавляется в основное окно и настраиваем, чтобы работало как всплывающее окно
---

```python
    def update_calendar(self):
        self.calendar_grid.clear_widgets()

        # Получаем календарь для текущего месяца
        cal = calendar.monthcalendar(self.current_date.year, self.current_date.month)

        for week in cal:
            for day in week:
                if day == 0:
                    # Пустая ячейка
                    btn = Button(text='', background_color=(0.3, 0.3, 0.3, 0.3))
                else:
                    btn = Button(text=str(day))
                    btn.day = day
                    btn.bind(on_press=self.on_day_select)

                    # Выделение сегодняшнего дня
                    today = date.today()
                    if (day == today.day and
                            self.current_date.month == today.month and
                            self.current_date.year == today.year):
                        btn.background_color = (0.2, 0.6, 0.8, 1)

                    # Выделение выбранного дня
                    if (self.selected_date and
                            day == self.selected_date.day and
                            self.current_date.month == self.selected_date.month and
                            self.current_date.year == self.selected_date.year):
                        btn.background_color = (0.8, 0.2, 0.2, 1)

                self.calendar_grid.add_widget(btn)
```
**Обновление календаря**:
фунцкия для обновления календаря, сначала очищает то что было, потом получает новые данные и рисует их
---
Вот простое объяснение всех функций:

1. **Выбор дня (on_day_select)**
```python
def on_day_select(self, button):
    self.selected_date = date(self.current_date.year, self.current_date.month, button.day)
    self.update_calendar()
```
- Когда ты нажимаешь на число в календаре
- Запоминает выбранную дату (год и месяц остаются теми же, день берется с нажатой кнопки)
- Обновляет календарь, чтобы подсветить выбранный день

2. **Предыдущий месяц (prev_month)**
```python
def prev_month(self, button):
    if self.current_date.month == 1:  # Если сейчас январь
        self.current_date = self.current_date.replace(year=self.current_date.year - 1, month=12)  # Переходим в декабрь прошлого года
    else:
        self.current_date = self.current_date.replace(month=self.current_date.month - 1)  # Просто предыдущий месяц
    self.update_spinners()  # Обновляем списки выбора
    self.update_calendar()  # Показываем новый месяц
```
- Переключает календарь на месяц назад
- Если был январь - переходит на декабрь предыдущего года
- Обновляет отображение

3. **Следующий месяц (next_month)**
```python
def next_month(self, button):
    if self.current_date.month == 12:  # Если сейчас декабрь
        self.current_date = self.current_date.replace(year=self.current_date.year + 1, month=1)  # Переходим в январь следующего года
    else:
        self.current_date = self.current_date.replace(month=self.current_date.month + 1)  # Просто следующий месяц
    self.update_spinners()
    self.update_calendar()
```
- Переключает календарь на месяц вперед
- Если был декабрь - переходит на январь следующего года
- Обновляет отображение

4. **Изменение месяца (on_month_change)**
```python
def on_month_change(self, spinner, text):
    months = ['Январь', 'Февраль', ..., 'Декабрь']
    month_num = months.index(text) + 1  # Получаем номер месяца (1-12)
    self.current_date = self.current_date.replace(month=month_num)  # Устанавливаем новый месяц
    self.update_calendar()  # Обновляем календарь
```
- Срабатывает при выборе месяца из списка
- Находит номер выбранного месяца
- Устанавливает этот месяц
- Показывает календарь для нового месяца

5. **Изменение года (on_year_change)**
```python
def on_year_change(self, spinner, text):
    year = int(text)  # Преобразуем текст в число (2023 → 2023)
    self.current_date = self.current_date.replace(year=year)  # Устанавливаем новый год
    self.update_calendar()  # Обновляем календарь
```
- Срабатывает при выборе года из списка
- Устанавливает выбранный год
- Показывает календарь для нового года

6. **Обновление списков (update_spinners)**
```python
def update_spinners(self):
    months = ['Январь', 'Февраль', ..., 'Декабрь']
    self.month_spinner.text = months[self.current_date.month - 1]  # Показываем текущий месяц
    self.year_spinner.text = str(self.current_date.year)  # Показываем текущий год
```
- Обновляет отображение в списках выбора
- Показывает актуальные месяц и год

7. **Сегодня (select_today)**
```python
def select_today(self, button):
    today = date.today()  # Получаем сегодняшнюю дату
    self.current_date = datetime(today.year, today.month, today.day)  # Устанавливаем текущую дату
    self.selected_date = today  # Выбираем сегодняшний день
    self.update_spinners()  # Обновляем списки
    self.update_calendar()  # Обновляем календарь
```
- Быстрый переход к сегодняшней дате
- Устанавливает и выбирает текущий день
- Обновляет отображение

8. **ОК (on_ok)**
```python
def on_ok(self, button):
    if self.selected_date and self.callback:  # Если дата выбрана и есть функция обработки
        self.callback(self.selected_date)  # Передаем выбранную дату в основную программу
    self.dismiss()  # Закрываем календарь
```
- Нажатие кнопки "ОК"
- Если дата выбрана - передает ее в программу
- Закрывает окно календаря

## Особенности работы:
1. Календарь автоматически показывает текущий месяц и год
2. Можно выбирать дату кликом по числу
3. Можно переключать месяцы кнопками или списками
4. Текущая дата и выбранная дата подсвечиваются разными цветами
5. При нажатии "ОК" выбранная дата передается в callback-функцию
6. При нажатии "Отменить" окно просто закрывается
