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
![image](https://github.com/user-attachments/assets/4247af0c-f9bd-462e-aeb3-c40d54dd4b42)

Definicja modeli odwzwierciedlających powyższe tabele wygląda następująco:
```Python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Author(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    books = db.relationship('Book', backref='author', cascade="all, delete", lazy=True)

class Book(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    author_id = db.Column(db.Integer, db.ForeignKey('author.id'), nullable=False)

```

### Definicja endpointów
Endpointy (czyli punkty końcowe) to adresy URL, które aplikacja webowa (np. napisana we Flasku) udostępnia na zewnątrz, aby inne systemy (lub przeglądarka) mogły się z nią komunikować.
Endpoint to konkretny adres + metoda HTTP, który odpowiada na zapytanie. To właśnie pod tym adresem znajduje się jakaś funkcja aplikacji – np. pokazanie danych, zapisanie formularza, usunięcie rekordu itd.

Do pliku app.py dodajemy poniższe definicje endpointów (wyświetlanie wszystkich ayutorów, dodawanie autora):
```Python
@app.route('/')
def index():
    authors = Author.query.all()
    return render_template('authors.html', authors=authors)

@app.route('/add_author', methods=['POST'])
def add_author():
    name = request.form['name']
    new_author = Author(name=name)
    db.session.add(new_author)
    db.session.commit()
    return redirect(url_for('index'))
```
