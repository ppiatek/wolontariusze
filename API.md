## Wprowadzenie

API serwisu dla wolontariuszy ŚDM Kraków 2016 jest oparte o styl REST. Posiada
łatwo przewidywalne ścieżki dostępu zorientowane na zasoby danych. Wykorzystuje
dobrze znane cechy protokołu HTTP takie jak kody błędów i metody HTTP w celu
możliwie jak najlepszej kompatybilności z istniejącymi bibliotekami będącymi
klientami protokołu HTTP. Każda odpowiedź serwera jest zwracana w formacie
JSON. Wszystkie przykłady w dokumentacji korzystają z narzędzia `curl`.

Podstawowa ścieżka dostępu do API to: `https://wolontariusze.krakow2016.com/api/v2/`.

## Autentykacja

Wszystkie zapytania do API muszą być nadane bezpiecznym protokołem HTTPS.
Wszystkie nieszyfrowane zapytania HTTP zwrócą błąd. Wszystkie nieautoryzowane
zapytania do API również zwrócą błąd.

## Błędy

API serwisu wolontariuszy korzysta z konwencjonalnych kodów zwrotnych HTTP do
oznajmienia sukcesu lub porażki zapytania. W praktyce oznacza to, że kody z
zakresu 2xx oznaczają sukces, kody z zakresu 4xx oznaczają błąd spowodowany
błędnie wprowadzonymi danymi (np. brak wymaganego parametru), a kody z zakresu
5xx oznaczają błędy po stronie serwera.

### Atrybuty

| Identyfikator | Opis                                   |
| ---           | ---                                    |
| `status`      | Zawsze `"error"`.                      |
| `type`        | Typ błędu. Np. `authentication_error`. |
| `message`     | Słowny opis błędu.                     |

## OAuth

API serwisu wolontariuszy do autoryzacji zapytań API używa standardu OAuth 2.0.

**Ważne:** aby móc korzystać z protokołu autoryzacji OAuth 2.0, dostawca
aplikacji musi najpierw uzyskać dane uwierzytelniające w postaci:
identyfikatora klienta (`client_id`) oraz sekretu klienta (`client_secret`).

API serwisu wolontariuszy wspiera dwa przypadki użycia:

* Scenariusz *server-side* wspiera aplikacje będące w stanie trwale i bezpiecznie przechowywać dane.
* Scenariusz *client-side* wspiera aplikacje JavaScript działające w środowisku przeglądarki.

### Server-side

Ten scenariusz zaczyna się kiedy użytkownik wykonuje akcję, która wymaga
autoryzacji. Aplikacja przekierowuje użytkownika na adres serwisu wolontariuszy,
który zawiera w sobie parametry zapytania definiujące typ dostępu którego
wymaga aplikacja.

Serwis wolontariuszy kieruje autentykacją użytkownika i jego zgodą, a następnie
zwraca kod dostępu (authorization code). Aplikacja używa kodu dostępu oraz
swojego identyfikatora i sekretu klienta do uzyskania tokenu dostępu (access
token), który może być później używany do autoryzacji zapytań API serwisu
wolontariuszy wykonywanych w imieniu tego użytkownika.

Cały proces składa się z następujących kroków:

**1. Żądanie tokenu dostępu**  
Kiedy użytkownik po raz pierwszy próbuje wykonać akcję wymagającą autoryzacji
API, należy przekierować go pod adres:
`https://wolontariusze.krakow2016.com/api/v2/dialog/authorize`. Poniższa
tabelka opisuje parametry zapytania które należy (lub można) zawrzeć w adresie
URL. Wszystkie wartości danych w zapytaniu muszą być poprawnie zakodowane przy
pomocy kodów ucieczki URL.

| Parametr        | Komentarz |
| ---             | ---       |
| `client_id`     | Wymagany. |
| `redirect_uri`  | Wymagany. |
| `response_type` | Wymagany. |
| `scope`         | Wymagany. |

**2. Zgoda użytkownika**  
W tym kroku użytkownik decyduje czy dać uprawnienie aplikacji do wykonywania
zapytań do API serwisu wolontariuszy w jego imieniu. Serwis wolontariuszy
wyświetla nazwę aplikacji żądającej uprawnień. Użytkownik może wybrać czy chce
aplikację upoważnić czy odrzucić.  Pytanie pojawia się zawsze, bez względu na
to czy zgoda została już wcześniej wydana.

**3. Obsługa odpowiedzi**  
Po podjęciu decyzji przez użytkownika, serwis wolontariuszy przekierowuje
użytkownika na adres `redirect_uri`, który został ustalony w kroku 1.

Jeżeli użytkownik wyraził zgodę na dostęp aplikacji, do adresu zostanie
dopisany parametr `code`. Wartość tego parametru to kod dostępu, który może
zostać wymieniony na token dostępu (opis w kroku 4).

**4. Wymiana kodu dostępu na token dostępu**  
Zakładając że użytkownik upoważnił Twoją aplikację do dostępu, należy teraz
wymienić kod dostępu na token dostępu. Aby tego dokonać należy wysłać zapytanie
POST na adres `https://wolontariusze.krakow2016.com/api/v2/oauth/token`
dołączając poniższe pary klucz-wartość w ciele zapytania:

| Parametr        | Komentarz                  |
| ---             | ---                        |
| `code`          | Kod dostępu.               |
| `client_id`     | Identyfikator klienta API. |
| `client_secret` | Sekret klienta API.        |
| `redirect_uri`  | Zwrotny adres URL.         |
| `grant_type`    | Zawsze `"*"`.              |

**5. Obsługa odpowiedzi i zapisanie danych**  
API serwisu wolontariuszy w odpowiedzi na zapytanie POST zwróci obiekt JSON z
tokenem dostępu.

```json
{
  "access_token": "YNmPUK9W72TVrxOlPlDysnimcIfX2qdKsDtPpXraczYGtuJ3CdjJn8ZYkos6Ckn7",
  "token_type": "Bearer"
}
```

### Client-side

*TODO*

### Wywoływanie zapytań do API serwisu wolontariuszy

Po uzyskaniu tokenu dostępu dla użytkownika, aplikacja może używać tokenu do
wysyłania w imieniu użytkownika autoryzowanych zapytań do API.

API serwisu wolontariuszy wymaga aby token dostępu był wysłany w nagłówku
`Authorization: Bearer`.

Używając narzędzia cURL, przykładowe zapytanie będzie wyglądało w następujący sposób:

```
curl -H "Authorization: Bearer ACCESS_TOKEN" -H "Content-Type: application/json" https://wolontariusze.krakow2016.com/api/v2/volunteers/e5725fc8-1837-4a32-823c-2f08c7a8b3a1
```

API zwróci kod HTTP 401 (Brak autoryzacji) jeżeli przesłane zapytanie będzie
zawierać nieprawidłowy albo wygały token dostępu lub będzie nieuprawnione z
innego powodu.

W kolejnych przykładach, dla większej przejrzystości, pomijamy nagłówki
`Authorization` i `Content-Type`, aczkolwiek należy pamiętać o tym że były one
obecne przy wysyłaniu przytoczonych poniżej przykładnowych zapytań.

## Wolontariusze

Wolontariusze są użytkownikami systemu. API umożliwia pobieranie i aktualizację
danych pojedynczych wolontariuszy, jak również listowanie wolontariuszy.
Przykłady użycia są zawarte w pliku:
<https://github.com/Krakow2016/wolontariusze/blob/master/spec/api_volunteer_spec.js>.

### Atrybuty

| Klucz                 | Pomijalny | Opis                                                    |
| ---                   | ---       | ---                                                     |
| `id`                  | Nie       |                                                         |
| `email`               | Nie       | Adres e-mail.                                           |
| `first_name`          | Nie       | Imię.                                                   |
| `last_name`           | Nie       | Nazwisko.                                               |
| `is_admin`            | Tak       | 'true' jeżeli użytkownik ma uprawnienia administratora. |
| `is_approved`         | Tak       | 'true' jeżeli użytkownik może logować się w systemie.   |
| `phone`               | Tak       | Numer telefonu kontaktowego. Np. `"+48 123456789"`.     |
| `profile_picture_url` | Tak       | Adres url do zdjęcia profilowego.                       |

### Tworzenie obiektu wolontariusza

Pozwala administratorom na dodawanie nowych użytkowników do systemu.

**Wymagane uprawnienia:**

Ta ścieżka jest dostępna jedynie dla użytkowników którzy są administratorami w
systemie (flaga `is_admin` jest `true`). Zapytania użytkowników bez
odpowiednich uprawnień zwrócą błąd `403` (brak dostępu).

**Ścieżka:**  
```
POST https://wolontariusze.krakow2016.com/api/v2/volunteers/
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/volunteers/
```

**Przykładowa odpowiedź:**  
```
{}
```

### Pobieranie obiektu wolontariusza

**Ścieżka:**  
```
GET https://wolontariusze.krakow2016.com/api/v2/volunteers/:id
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/volunteers/e5725fc8-1837-4a32-823c-2f08c7a8b3a1
```

**Przykładowa odpowiedź:**  
```json
{
  "status": "success",
  "data": {
    "volunteer": {
      "id": "e5725fc8-1837-4a32-823c-2f08c7a8b3a1",
      "email": "faustyna@kowalska.pl",
      "first_name": "Faustyna",
      "last_name": "Kowalska",
      "phone": "+48123456789",
      "is_admin": true,
      "is_approved": true
    }
  }
}
```

### Aktualizacja obiektu wolontariusza

**Ścieżka:**  
```
POST https://wolontariusze.krakow2016.com/api/v2/volunteers/:id
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/volunteers/123
```

**Przykładowa odpowiedź:**  
```
{}
```

### Listowanie wolontariuszy

**Ścieżka:**  
```
GET https://wolontariusze.krakow2016.com/api/v2/volunteers
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/volunteers
```

**Przykładowa odpowiedź:**  
```
{}
```

## Aktywności i zadania

Zadania są dodatkową pracą której wolontariusze będą mogli się podjąć i zgłosić
się do niej. Aktywność jest klasą, od której wywodzą się zadania oraz inne
zdarzenia które wolontariusz będzie mógł samodzielnie zalogować w systemie. API
umożliwia tworzenie, usuwanie i aktualizację aktywności oraz pobieranie
pojedynczych aktywności jak i całej listy aktywności.
Przykłady użycia są zawarte w pliku:
<https://github.com/Krakow2016/wolontariusze/blob/master/spec/api_activity_spec.js>.

| Klucz         | Pomijalny | Opis                                                                       |
| ---           | ---       | ---                                                                        |
| `id`          | Nie       |                                                                            |
| `description` | Nie       | Opis aktywności.                                                           |
| `name`        | Nie       | Nazwa aktywności.                                                          |
| `volunteers`  | Nie       | Tablica wolontariuszy zgłoszonych do wykonania zadania.                    |
| `datetime`    | Tak       | Data i czas rozpoczęcia zadania.                                           |
| `is_urgent`   | Tak       | `true` dla zadań oznaczonych jako pilne.                                   |
| `lat_lon`     | Tak       | Współrzędne geograficzne miejsca wykonywania aktywności. Np. `[0.0, 0.0]`. |
| `limit`       | Tak       | Limit osób które mogą zgłosić się do zadania. Np. `10`.                    |
| `place`       | Tak       | Opis miejsca wykonywania zadania. Np. `"Sankruarium św. Jana Pawła II"`.   |

### Tworzenie obiektu aktywności

**Ścieżka:**  
```
POST https://wolontariusze.krakow2016.com/api/v2/activities
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/activities -d '{"name": "nazwa", "description": "opis"}'
```

**Przykładowa odpowiedź:**  
```json
{
    "status": "success",
    "data": {
        "activity": {
            "created_at": "2016-02-01T22:50:08.906Z",
            "description": "opis",
            "id": "0565ea98-86bf-4d5f-a3da-3236c8c3a876",
            "name": "nazwa",
            "user_id": "1"
        }
    }
}
```

### Pobieranie obiektu aktywności

**Ścieżka:**  
```
GET https://wolontariusze.krakow2016.com/api/v2/activities/:id
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/activities/0565ea98-86bf-4d5f-a3da-3236c8c3a876
```

**Przykładowa odpowiedź:**  
```json
{
    "status": "success",
    "data": {
        "activity": {
            "id": "0565ea98-86bf-4d5f-a3da-3236c8c3a876",
            "name": "nazwa",
            "description": "opis",
            "created_at": "2016-02-01T22:50:08.906Z",
            "volunteers": []
        }
    }
}
```

### Aktualizacja obiektu aktywności

*Wymagane uprawnienia:* administrator.

**Ścieżka:**  
```
POST https://wolontariusze.krakow2016.com/api/v2/activities/:id
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/activities/0565ea98-86bf-4d5f-a3da-3236c8c3a876 -d '{"name": "nazwa", "description": "zmieniony opis"}'
```

**Przykładowa odpowiedź:**  
```json
{
    "status": "success",
    "data": {
        "activity": {
            "created_at": "2016-02-01T22:50:08.906Z",
            "description": "zmieniony opis",
            "id": "0565ea98-86bf-4d5f-a3da-3236c8c3a876",
            "name": "nazwa",
            "updated_at": "2016-02-02T11:47:28.162Z",
            "user_id": "1"
        }
    }
}
```

### Listowanie aktywności

**Ścieżka:**  
```
GET https://wolontariusze.krakow2016.com/api/v2/activities
```

**Przykładowe zapytanie:**  
```
$ curl https://wolontariusze.krakow2016.com/api/v2/activities
```

**Przykładowa odpowiedź:**  
```
{}
```

### Wysłanie zgłoszenia do aktywności

**Ścieżka:**  
```
POST https://wolontariusze.krakow2016.com/api/v2/activities/:id/join
```

**Przykładowe zapytanie:**  
```
$ curl -X POST https://wolontariusze.krakow2016.com/api/v2/activities/0565ea98-86bf-4d5f-a3da-3236c8c3a876/join
```

**Przykładowa odpowiedź:**  
```json
{
    "status": "success",
    "data": {
        "joint": {
            "activity_id": "0565ea98-86bf-4d5f-a3da-3236c8c3a876",
            "created_at": "2016-02-02T13:05:35.413Z",
            "id": "91186ae2-27f6-4f3b-aadf-4d9570557187",
            "user_id": "1"
        }
    }
}
```

### Cofanie zgłoszenia do aktywności

**Ścieżka:**  
```
POST https://wolontariusze.krakow2016.com/api/v2/activities/:id/leave
```

**Przykładowe zapytanie:**  
```
$ curl -X POST https://wolontariusze.krakow2016.com/api/v2/activities/0565ea98-86bf-4d5f-a3da-3236c8c3a876/leave
```

**Przykładowa odpowiedź:**  
```json
{
    "status": "success",
    "data": {
        "joint": {
            "activity_id": "0565ea98-86bf-4d5f-a3da-3236c8c3a876",
            "created_at": "2016-02-02T13:05:35.413Z",
            "id": "91186ae2-27f6-4f3b-aadf-4d9570557187",
            "is_canceled": true,
            "updated_at": "2016-02-02T13:06:24.705Z",
            "user_id": "1"
        }
    }
}
```

### Usunięcie obiektu aktywności

*Wymagane uprawnienia:* administrator.

**Ścieżka:**  
```
DELETE https://wolontariusze.krakow2016.com/api/v2/activities/:id
```

**Przykładowe zapytanie:**  
```
$ curl -X DELETE https://wolontariusze.krakow2016.com/api/v2/activities/0565ea98-86bf-4d5f-a3da-3236c8c3a876
```

**Przykładowa odpowiedź:**  
```
{
    "status": "success",
    "data": {}
}
```

## Baza noclegowa

*TODO*
