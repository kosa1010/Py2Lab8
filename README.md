# Tworzenie aplikacji webowej(Flask, Flask SQLAlchemy)
Flask to mikroframework webowy napisany w języku Python, który pozwala na szybkie tworzenie aplikacji internetowych (webowych). Został zaprojektowany tak, aby był lekki, elastyczny i prosty w użyciu, a jednocześnie wystarczająco potężny, by budować złożone aplikacje.

## Główne cechy Flask:
- Minimalizm: nie narzuca struktury projektu ani nie wymaga użycia konkretnych narzędzi — możesz dobrać je samodzielnie.
- Routing URL: łatwe przypisywanie funkcji do konkretnych adresów URL.
- Wsparcie dla szablonów HTML (np. Jinja2).
- Wbudowany serwer deweloperski: idealny do testowania aplikacji lokalnie.
- Łatwa integracja z bazami danych (np. SQLite, PostgreSQL, MySQL przez SQLAlchemy).
- Rozszerzalność: duża liczba dostępnych rozszerzeń, np. do autoryzacji, formularzy, REST API, itp.

### W pierwszej kolejności należy zainstalować zaleności
```Python
pip install flask flask_sqlalchemy
```

### Struktura tworzonego prokektu:
```Python
crud-app/
│
├── app.py
├── models.py
├── templates/
│   ├── index.html
│   ├── authors.html
│   ├── books.html
│   └── form.html
```

### Konfiguracja aplikacji (app.py)
```Python
from flask import Flask, render_template, request, redirect, url_for
from models import db, Author, Book

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///crud.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db.init_app(app)
```
W projekcie wykorzyatna zostanie baza danych SQLite

### Modele danych (models.py)
Modele danych to reprezentacje struktur danych używanych w aplikacji — zazwyczaj opisują, jakie dane są przechowywane oraz jakie relacje zachodzą między nimi. W kontekście aplikacji webowych (np. przy użyciu Flask + SQLAlchemy) model danych często odpowiada tabeli w bazie danych. W tworzonym projekcie wykorzyatmy 2 modele danych reprezentujące dwie tabele połączone ze sobą relacją jeden do wielu których struktura przedstawiona została na poniższym diagramie ERD.


