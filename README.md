PROJEKT: PROSTA STRONA + BAZA SQL (TECHNIKUM)


=================================================

1. CEL PROJEKTU
   
=================================================

Celem projektu jest stworzenie prostej strony internetowej, która:
- łączy się z bazą danych,
- pobiera i wyświetla dane z tabel,
- umożliwia filtrowanie i wyszukiwanie produktów.

Technologie:
- HTML
- PHP
- MySQL / MariaDB

=================================================

2. STRUKTURA PLIKÓW
   
=================================================
```
/sklep/
│
├── index.html
├── produkty.php
└── baza_sklep.sql
```
=================================================

3. BAZA DANYCH (baza_sklep.sql)
   
=================================================
```
CREATE DATABASE IF NOT EXISTS sklep
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_polish_ci;

USE sklep;

CREATE TABLE IF NOT EXISTS kategorie (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nazwa VARCHAR(100) NOT NULL
);

CREATE TABLE IF NOT EXISTS produkty (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nazwa VARCHAR(150) NOT NULL,
    opis TEXT,
    cena DECIMAL(10,2) NOT NULL,
    id_kategorii INT,
    data_dodania DATE NOT NULL,
    FOREIGN KEY (id_kategorii) REFERENCES kategorie(id)
);

INSERT INTO kategorie (nazwa) VALUES
('Laptopy'),
('Telefony'),
('Akcesoria');

INSERT INTO produkty (nazwa, opis, cena, id_kategorii, data_dodania) VALUES
('Laptop XYZ', '15 cali, 8GB RAM, 256GB SSD', 2999.99, 1, '2026-01-01'),
('Laptop Student', '14 cali, 8GB RAM, 512GB SSD', 2499.00, 1, '2026-01-05'),
('Smartfon ABC', 'Ekran 6.1 cala, 128GB', 1999.00, 2, '2026-01-02'),
('Smartfon Pro', 'Ekran 6.7 cala, 256GB', 2999.00, 2, '2026-01-03'),
('Mysz bezprzewodowa', 'Mysz optyczna USB', 79.99, 3, '2026-01-03'),
('Klawiatura mechaniczna', 'Podświetlana RGB', 199.99, 3, '2026-01-04');

=================================================
4. PLIK STARTOWY (index.html)
=================================================
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <title>Sklep - strona startowa</title>
</head>
<body>
    <h1>Witamy w prostym sklepie</h1>
    <p>Kliknij poniżej, aby przejść do listy produktów.</p>
    <p><a href="produkty.php">Przejdź do produktów</a></p>
</body>
</html>
```

=================================================

5. PHP (produkty.php)
   
=================================================
```
<?php
$host = "localhost";
$user = "root";
$pass = "";
$db   = "sklep";

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Błąd połączenia z bazą: " . $conn->connect_error);
}

$conn->set_charset("utf8mb4");

$sql_kategorie = "SELECT id, nazwa FROM kategorie ORDER BY nazwa";
$wynik_kategorie = $conn->query($sql_kategorie);

$wybrana_kategoria = isset($_GET['kategoria']) ? (int)$_GET['kategoria'] : 0;
$szukaj = isset($_GET['szukaj']) ? trim($_GET['szukaj']) : "";

$sql_produkty = "
    SELECT p.id, p.nazwa, p.opis, p.cena, p.data_dodania, k.nazwa AS kategoria
    FROM produkty p
    LEFT JOIN kategorie k ON p.id_kategorii = k.id
    WHERE 1=1
";

if ($wybrana_kategoria > 0) {
    $sql_produkty .= " AND p.id_kategorii = " . $wybrana_kategoria;
}

if ($szukaj !== "") {
    $szukaj_esc = $conn->real_escape_string($szukaj);
    $sql_produkty .= " AND p.nazwa LIKE '%" . $szukaj_esc . "%'";
}

$sql_produkty .= " ORDER BY p.cena ASC";
$wynik_produkty = $conn->query($sql_produkty);
?>
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <title>Lista produktów</title>
    <style>
        table, th, td { border: 1px solid #000; border-collapse: collapse; padding: 4px; }
        th { background-color: #eee; }
        form { margin-bottom: 20px; }
    </style>
</head>
<body>

<h1>Lista produktów</h1>
<p><a href="index.html">← Powrót</a></p>

<form method="get" action="produkty.php">
    <label>Kategoria:
        <select name="kategoria">
            <option value="0">-- wszystkie --</option>
<?php
if ($wynik_kategorie && $wynik_kategorie->num_rows > 0) {
    while ($kat = $wynik_kategorie->fetch_assoc()) {
        $selected = ($kat['id'] == $wybrana_kategoria) ? 'selected' : '';
        echo '<option value="' . $kat['id'] . '" ' . $selected . '>' .
             htmlspecialchars($kat['nazwa']) .
             '</option>';
    }
}
?>
        </select>
    </label>
    <label> Szukaj:
        <input type="text" name="szukaj" value="<?php echo htmlspecialchars($szukaj); ?>">
    </label>
    <button type="submit">Filtruj</button>
</form>

<table>
    <tr>
        <th>ID</th>
        <th>Nazwa</th>
        <th>Kategoria</th>
        <th>Opis</th>
        <th>Cena</th>
        <th>Data dodania</th>
    </tr>
<?php
if ($wynik_produkty && $wynik_produkty->num_rows > 0) {
    while ($row = $wynik_produkty->fetch_assoc()) {
        echo "<tr>";
        echo "<td>" . $row['id'] . "</td>";
        echo "<td>" . htmlspecialchars($row['nazwa']) . "</td>";
        echo "<td>" . htmlspecialchars($row['kategoria']) . "</td>";
        echo "<td>" . htmlspecialchars($row['opis']) . "</td>";
        echo "<td>" . number_format($row['cena'], 2, ',', ' ') . "</td>";
        echo "<td>" . $row['data_dodania'] . "</td>";
        echo "</tr>";
    }
} else {
    echo '<tr><td colspan="6">Brak produktów</td></tr>';
}
$conn->close();
?>
</table>

</body>
</html>
```

=================================================

6. JAK URUCHOMIĆ
   
=================================================
1. Skopiuj katalog "sklep" do katalogu serwera:
   - XAMPP -> htdocs
   - WAMP -> www
   - Linux -> /var/www/html/

2. Importuj plik baza_sklep.sql przez phpMyAdmin.

3. W pliku produkty.php zmień dane logowania:
   user = root
   pass = ""

4. Uruchom w przeglądarce:
   http://localhost/sklep/index.html

=================================================

7. ZADANIE DODATKOWE (na wyższą ocenę)
   
=================================================

Dodać możliwość SORTOWANIA produktów.


Kryteria sortowania do wyboru:
- po cenie rosnąco
- po cenie malejąco
- po nazwie alfabetycznie
- po dacie dodania (najnowsze pierwsze)


Wymagania:
- dopisać pole <select> sortowanie
- obsłużyć parametry GET
- zmieniać ORDER BY w SQL
- funkcja ma działać razem z filtrowaniem i wyszukiwaniem


#### Punktacja proponowana:
   +1 pkt - działa jeden typ sortowania
   +2 pkt - działa kilka typów sortowania
   +3 pkt - działa także z filtrowaniem i wyszukiwaniem
   +1 pkt - zapamiętanie wybranego sortowania w select
   MAX: +4 pkt do oceny projektu.

###


=================================================


### Prześlij dokument na maila  wykonanezadania100@gmail.com z wykonanym zadaniem lub link do gihub z repozytorium zadania.

=================================================
