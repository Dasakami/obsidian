Django Admin — встроенный интерфейс для **управления моделями**, создание CRUD без написания HTML/Views.

---

## 1️⃣ Подключение приложения admin

В `settings.py` убедись, что установлено:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # твои приложения
]
````

---

## 2️⃣ Регистрация моделей

```python
from django.contrib import admin
from .models import Author, Book

admin.site.register(Author)
admin.site.register(Book)
```

- Базовая регистрация моделей
    
- Теперь их можно редактировать через `/admin`
    

---

## 3️⃣ Кастомизация отображения моделей

```python
@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    list_display = ('name', 'email', 'birth_date')  # поля для списка
    search_fields = ('name', 'email')               # поиск
    list_filter = ('birth_date',)                   # фильтры
    ordering = ('name',)                             # сортировка
    list_per_page = 20                               # количество на странице
```

- `list_display` — столбцы в списке
    
- `search_fields` — поиск по полям
    
- `list_filter` — боковые фильтры
    
- `ordering` — сортировка по умолчанию
    
- `list_per_page` — пагинация
    

---

## 4️⃣ Inline-формы (связанные модели)

```python
from django.contrib import admin
from .models import Author, Book

class BookInline(admin.TabularInline):  # или StackedInline
    model = Book
    extra = 1  # количество пустых форм

@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    inlines = [BookInline]
```

- Позволяет редактировать связанные объекты **в одном интерфейсе**
    
- `TabularInline` — компактная таблица
    
- `StackedInline` — вертикальные блоки
    

---

## 5️⃣ Кастомизация форм в админке

```python
from django import forms
from django.contrib import admin
from .models import Author

class AuthorForm(forms.ModelForm):
    class Meta:
        model = Author
        fields = '__all__'

    def clean_email(self):
        email = self.cleaned_data['email']
        if not email.endswith('@example.com'):
            raise forms.ValidationError('Email должен быть example.com')
        return email

@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    form = AuthorForm
```

- Можно добавлять **валидацию**, кастомные поля, виджеты
    
- Подходит для сложной логики ввода данных
    

---

## 6️⃣ Кастомные действия (Actions)

```python
@admin.action(description='Отметить книги как опубликованные')
def make_published(modeladmin, request, queryset):
    queryset.update(is_published=True)

@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    actions = [make_published]
```

- Действия применяются к выделенным объектам
    
- Можно обновлять поля, удалять или делать любые операции
    

---

## 7️⃣ Кастомизация отображения полей

```python
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'display_categories')

    def display_categories(self, obj):
        return ", ".join([c.name for c in obj.categories.all()])
    display_categories.short_description = 'Категории'
```

- Позволяет показывать вычисляемые или связанные поля
    
- `short_description` — заголовок столбца
    

---

## 8️⃣ Поля только для чтения

```python
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    readonly_fields = ('created_at', 'updated_at')
```

- Нельзя редактировать в админке
    
- Полезно для автоматических полей, вроде `auto_now` или `auto_now_add`
    

---

## 9️⃣ Кастомизация интерфейса формы

```python
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    fieldsets = (
        ('Основное', {'fields': ('title', 'author')}),
        ('Дополнительно', {'fields': ('categories', 'pages')}),
    )
```

- `fieldsets` — делит форму на блоки
    
- Можно добавлять заголовки и классы CSS
    

---

## 🔟 Поиск, фильтры и сортировка

- `search_fields` — поиск по полям
    
- `list_filter` — боковые фильтры по категориям, датам, Boolean
    
- `ordering` — сортировка по умолчанию
    
- Можно комбинировать все параметры для удобства работы
    

---

## 11️⃣ Inline для ManyToMany через `through`

```python
class BookAuthorInline(admin.TabularInline):
    model = Book.authors.through
    extra = 1

@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    inlines = [BookAuthorInline]
```

- Если есть `ManyToManyField` с промежуточной моделью (`through`)
    
- Можно редактировать связи напрямую в админке
    

---

## 12️⃣ Best practices

- Используй `@admin.register(Model)` вместо `admin.site.register`
    
- Настраивай `list_display`, `search_fields`, `list_filter` для удобства
    
- Для сложных связей используй `Inline` формы
    
- Кастомные actions упрощают массовые операции
    
- Поля только для чтения для авто-данных (`auto_now`, `slug`, `created_at`)
    
- Для `ManyToMany` лучше отдельный inline с `through` моделью
    
- Минимизируй тяжелые вычисления в методах `list_display` — это влияет на скорость
    

---

💡 **Итог:**

- Django Admin позволяет быстро управлять моделями
    
- Поддерживает кастомизацию списка, формы, inline, действия, фильтры, поиск
    
- Можно добавлять валидацию, readonly-поля и кастомные методы
    
- Оптимизация админки важна для производительности
    

