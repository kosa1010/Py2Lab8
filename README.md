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

### Konfiguracja aplikacji (`app.py`)
```Python
from flask import Flask, render_template, request, redirect, url_for
from models import db, Author, Book

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///crud.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db.init_app(app)
```
W projekcie wykorzyatna zostanie baza danych SQLite

### Modele danych (`models.py`)
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

Do pliku `app.py` dodajemy poniższe definicje endpointów:
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

@app.route('/author/<int:author_id>')
def author_detail(author_id):
    author = Author.query.get_or_404(author_id)
    return render_template('books.html', author=author)

@app.route('/add_book/<int:author_id>', methods=['POST'])
def add_book(author_id):
    title = request.form['title']
    new_book = Book(title=title, author_id=author_id)
    db.session.add(new_book)
    db.session.commit()
    return redirect(url_for('author_detail', author_id=author_id))

@app.route('/delete_book/<int:book_id>')
def delete_book(book_id):
    book = Book.query.get_or_404(book_id)
    author_id = book.author_id
    db.session.delete(book)
    db.session.commit()
    return redirect(url_for('author_detail', author_id=author_id))

```
### Szablony
Szablony (ang. templates) w kontekście aplikacji webowych, to pliki HTML z dodatkami, które pozwalają dynamicznie generować zawartość strony w zależności od danych z aplikacji.
We Flasku szablony są obsługiwane przez silnik Jinja2 — dzięki niemu możesz np. wstawić dane z Pythona do HTML-a, tworzyć pętle, warunki, dziedziczyć układy stron itp.
Szablony umieszczamy w katalogu templates
Szablon authors.html
```HTML
<!doctype html>
<html lang="pl">
<head>
    <meta charset="utf-8">
    <title>Autorzy</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
<div class="container mt-5">
    <h1 class="mb-4">📚 Lista autorów</h1>
    <div class="row">
        <ul class="list-group mb-4">
            {% for author in authors %}
            <li class="list-group-item d-flex justify-content-between align-items-center">
                <div class="col col-md-10">
                    <a href="{{ url_for('author_detail', author_id=author.id) }}">{{ author.name }}</a>
                </div>
                <div class="col">
                    <span class="badge bg-primary rounded-pill">{{ author.books|length }} książek</span>
                    <a href="{{ url_for('delete_author', author_id=author.id) }}"><button class="btn btn-danger" type="submit" >Usuń</button></a>
                </div>
            </li>
            {% else %}
            <li class="list-group-item text-muted">Brak autorów w bazie.</li>
            {% endfor %}
        </ul>
    </div>
    <form method="post" action="{{ url_for('add_author') }}" class="card p-3 shadow-sm">
        <h5 class="mb-3">Dodaj nowego autora</h5>
        <div class="mb-3">
            <input type="text" name="name" class="form-control" placeholder="Imię i nazwisko autora" required>
        </div>
        <button type="submit" class="btn btn-success">Dodaj</button>
    </form>
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```
