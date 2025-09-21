Django Forms — это инструмент для работы с **HTML-формами**, валидации данных и взаимодействия с пользователем.  
Forms позволяют **принимать данные**, **проверять их корректность** и **сохранять в модели**.

---

## 1️⃣ Базовые формы (forms.Form)

```python
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100, label='Имя')
    email = forms.EmailField(label='Email')
    message = forms.CharField(widget=forms.Textarea, label='Сообщение')
````

- `forms.CharField` → текстовое поле
    
- `forms.EmailField` → поле с проверкой email
    
- `widget=forms.Textarea` → текстовое поле с многострочным вводом
    
- `label` → название поля в шаблоне
    

---

## 2️⃣ Поля форм (Field types)

| Поле                                       | Описание             |
| ------------------------------------------ | -------------------- |
| `CharField`                                | Текстовое поле       |
| `EmailField`                               | Email с проверкой    |
| `IntegerField`                             | Целое число          |
| `FloatField`                               | Дробное число        |
| `DecimalField(max_digits, decimal_places)` | Десятичное число     |
| `BooleanField`                             | True/False           |
| `NullBooleanField`                         | True/False/None      |
| `DateField`                                | Дата                 |
| `DateTimeField`                            | Дата и время         |
| `TimeField`                                | Время                |
| `ChoiceField(choices=…)`                   | Выбор из списка      |
| `MultipleChoiceField`                      | Множественный выбор  |
| `FileField`                                | Загрузка файлов      |
| `ImageField`                               | Загрузка изображений |
| `SlugField`                                | SEO-friendly текст   |

---

## 3️⃣ Виджеты (Widgets)

Widgets отвечают за **отображение поля в HTML**:

```python
from django.forms import TextInput, EmailInput, NumberInput, Select, CheckboxInput, FileInput

class MyForm(forms.Form):
    name = forms.CharField(widget=TextInput(attrs={'class':'form-control'}))
    email = forms.EmailField(widget=EmailInput(attrs={'placeholder':'Введите email'}))
    age = forms.IntegerField(widget=NumberInput(attrs={'min':0, 'max':100}))
    gender = forms.ChoiceField(choices=[('M','Муж'),('F','Жен')], widget=Select)
    agree = forms.BooleanField(widget=CheckboxInput)
```

- `attrs` — позволяет добавлять HTML-атрибуты (класс, id, placeholder)
    

---

## 4️⃣ Валидация данных

### 🔹 Встроенные валидаторы

```python
from django.core.validators import MinValueValidator, MaxValueValidator, RegexValidator

age = forms.IntegerField(validators=[MinValueValidator(0), MaxValueValidator(120)])
phone = forms.CharField(validators=[RegexValidator(r'^\+7\d{10}$', 'Некорректный телефон')])
```

### 🔹 Кастомные методы clean_*

```python
class RegistrationForm(forms.Form):
    username = forms.CharField(max_length=50)
    password1 = forms.CharField(widget=forms.PasswordInput)
    password2 = forms.CharField(widget=forms.PasswordInput)

    def clean_password2(self):
        p1 = self.cleaned_data.get('password1')
        p2 = self.cleaned_data.get('password2')
        if p1 != p2:
            raise forms.ValidationError('Пароли не совпадают')
        return p2
```

### 🔹 Общая валидация формы

```python
def clean(self):
    cleaned_data = super().clean()
    start = cleaned_data.get('start_date')
    end = cleaned_data.get('end_date')
    if start and end and start > end:
        raise forms.ValidationError('Дата начала не может быть позже даты конца')
```

---

## 5️⃣ ModelForm — формы на основе моделей

```python
from django.forms import ModelForm
from .models import Author

class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = ['name', 'email', 'birth_date']  # или '__all__'
        labels = {'name':'Имя автора'}
        widgets = {'birth_date': forms.DateInput(attrs={'type':'date'})}
```

- Автоматически создаёт поля формы на основе модели
    
- Позволяет использовать **валидацию модели**
    
- Можно переопределять поля и виджеты
    

---

## 6️⃣ Файлы в формах

```python
class UploadForm(forms.Form):
    file = forms.FileField()
    image = forms.ImageField(required=False)
```

- Для работы с файлами в views:
    

```python
if request.method == 'POST':
    form = UploadForm(request.POST, request.FILES)
    if form.is_valid():
        handle_file(form.cleaned_data['file'])
```

- Не забывай добавлять `enctype="multipart/form-data"` в шаблон HTML
    

---

## 7️⃣ InlineFormSet — формы для связанных моделей

```python
from django.forms import inlineformset_factory
from .models import Author, Book

BookFormSet = inlineformset_factory(Author, Book, fields=('title','pages'), extra=1)
```

- Используется для редактирования связанных объектов **в одной форме**
    
- `extra` — количество пустых форм для добавления
    

---

## 8️⃣ Пользовательские поля и виджеты

```python
class ColorField(forms.CharField):
    def to_python(self, value):
        return value.lower()  # всегда преобразуем к нижнему регистру

class ColorWidget(forms.TextInput):
    input_type = 'color'

class ColorForm(forms.Form):
    color = ColorField(widget=ColorWidget)
```

---

## 9️⃣ FormHelper и crispy-forms (опционально)

- Для красивого отображения форм
    
- Позволяет управлять **классами, кнопками и сетками**
    

```python
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit

class ContactForm(forms.Form):
    name = forms.CharField()
    email = forms.EmailField()

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.form_method = 'post'
        self.helper.add_input(Submit('submit', 'Отправить'))
```

---

## 🔟 Работа с формами в views

```python
from django.shortcuts import render, redirect
from .forms import ContactForm

def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST, request.FILES)
        if form.is_valid():
            # обработка данных
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            return redirect('success')
    else:
        form = ContactForm()
    return render(request, 'contact.html', {'form': form})
```

- `cleaned_data` — очищенные и валидированные данные
    
- `form.is_valid()` — запускает всю валидацию формы
    

---

## 11️⃣ Формы для пользовательской регистрации и аутентификации

```python
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm

# регистрация
class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        fields = UserCreationForm.Meta.fields + ('email',)

# вход
form = AuthenticationForm(request, data=request.POST)
```

---

## 12️⃣ Best practices

- Всегда проверяй `form.is_valid()` перед использованием `cleaned_data`
    
- Для ModelForm используй `fields` или `exclude`, не '**all**' без необходимости
    
- Добавляй кастомные валидаторы через `clean_` или встроенные валидаторы
    
- Для связанных объектов используй `inlineformset_factory`
    
- Для файлов всегда добавляй `request.FILES`
    
- Для красивого отображения форм используй `widgets` или библиотеки вроде `crispy-forms`
    

---

💡 **Итог:**

- Django Forms — мощный инструмент для работы с HTML-формами
    
- Поддерживает кастомные поля, виджеты, валидацию и ModelForms
    
- Можно работать с файлами, inline-формами, регистрацией и аутентификацией
    
- Валидация и чистка данных — основа безопасного приложения
    
