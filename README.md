
## INSTALACJA 

Na początku musimy pobrać repozytorium na którym znajduje się nasze oprogramowanie WebGoat którego będziemy używać.

  https://github.com/WebGoat/WebGoat

Dodatkowo będziemy potrzebowali dockera 

  https://www.docker.com/products/docker-desktop/
  https://www.apachefriends.org/pl/download.html

Po instalacji wszystkich 3 rzeczy możemy przejść do części praktycznej naszego projektu. 
### URUCHOMIENIE WEBGOAT
```bash
  docker run -it -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 webgoat/webgoat
```
  W przypadku jakichś problemów z uruchomieniem kontenera możemy spróbować uruchomić go na innym porcie, należy zapamiętać ten port gdyż będzie on wykorzystywany  w dalszej części.


# ZADANIA 
### ZADANIE 1
1. Pierwszym krokiem jest utworzenie profilu użytkownika
![App Screenshot](https://snipboard.io/ejtuPR.jpg)

2. Następnie przechodzimy do zakładki Server-side Request Forgery i wybieramy Cross-Site Request Forgeries. Zalecamy zaponzać się z dodatkową treścią umieszczoną na pierwszych dwóch slajdach w celu łatwiejszego zrozumienia tematu.

3. Teraz przechodzimy do zakładki Basic Get CSRF Exercise, w której wykonamy pierwsze zadanie. W tym momencie musimy uruchomić serwer apache. Celem tego zadania jest uruchomienie poniższego formularza z zewnętrznego źródła podczas logowania. Odpowiedź będzie zawierać "flagę" (wartość liczbową) którą wpiszemy poniżej.
4. Tworzymy plik do którego musimy wkleić kod  
```html
<html>
	<body>
	<script>history.pushState('','', '/')</script>
	 <form action="http://localhost:{NUMER_PORTU}/WebGoat/csrf/basic-get-flag" method="POST">
		<input type="hidden" name="csrf" value="false" />
		<input type="hidden" name="submit" value="Submit&#32;Query"/>
		<input type="submit" value="Submit request" />
	 </form>
	</body>
</html>
```
Po uruchomieniu tego kodu powiniśmy otrzymać status, że zadanie zostało wykonane poprawnie. Dodatkowo zwrócona została wartość liczoba flagi, którą musimy wkleić do formularza.
<br>![App Screenshot](https://snipboard.io/CZyGlD.jpg)

### ZADANIE 2
1. Na początku możemy włączyć konsolę [f12] i zaobserwować zapytania wysyłane poprzez formularz z opiniami. Gdy dodamy opinię powiniśmy zauważyć wysłane zapytanie metodą POST. 
2. W tym przypadku również będziemy tworzyć plik z naszym kodem. Będzie potrzebne nam:
- post url request
- content type
- parametry 
```html
<html>
	<form action=http://localhost:{NUMER_PORTU}/WebGoat/csrf/review method=post enctype='application/x-www-form-urlencoded; charset=UTF-8'>
		<input name='pierwszy_parametr' value='dowolna_wartosc' type='hidden'> 
		<input name='drugi_parametr' value='dowolna_wartosc' type='hidden'> 
		<input name='trzeci_parametr' value='wartosc_odczytana' type='hidden'> 
		<input type=submit value="Submit">
	</form>
</html>
```
	<input name='' value='totalnielosowe' type='hidden'> - sprawia że ofiara nie jest świadoma ataku który jest przeprowadzany.
3. Po wykonaniu powyższego kodu i odświeżenia głównej strony jesteśmy w stanie zobaczyć dwie opinie.

### ZADANIE 3
1. Na początku możemy włączyć konsolę [f12] i zaobserwować zapytanie wysyłane poprzez wiadomość którą uzupełniliśmy.

2. Warto zwrócić uwagę tutaj na content type.
![App Screenshot](https://snipboard.io/9OYCsH.jpg)

3. W przypadku zapytania POST i użycia content-type innego niż 3 wyżej wymienione zapytanie nie jest już wtedy proste, jest uznawane jako "preflighted request". 

4. W zapytaniu preflighted request wykonywane są dodatkowe sprawdzenia bezpieczeństwa przez serwer w celu sprawdzenia czy na prawdę jesteśmy autoryzowanym użytkownikiem do wykonania zapytania. Dlatego chcemy unikać tego typu zapytań. 

5. Zmienimy nasze zapytanie aby wykorzystywało "text/plain" zamiast "application/json"
```html
<html>
	<title>JSON CSRF POC</title>
	<center> 
	<h1>JSON CSRF POC </h1>
	<form action=http://localhost:{NUMER_PORTU}/WebGoat/csrf/feedback/message method=post enctype ="application/json">
	<input name='{"name":"WebGoat","email":"webgoat@webgoat.org","content":"Webgoat tekst","ignore_me":"'value = 'test"}' type=hidden>
	<input type=submit value="Submit">
	</form>
	</center>
</html>
```

6. Próbujemy zamaskować typ z json aby był traktowany jako zwykły tekst, dlatego używamy takiego sposobu przekazania parametrów. Parametr ignore_me jest używany w celu ominięcia problemu w którym jest to traktowane jako json.

### ZADANIE 4
1. Uruchamiamy oprogramowanie Burp Suite Community Edition, ustawiamy serwer proxy w ustawieniach windowsa i otwieramy odpowiednią przeglądarke (dokładna instrukcja konfiguracji na Upelu przedmiotu, zakładka "Zajęcia laboratoryjne numer 2")

2. Przechodzimy do zakładki (A1) Broken access control i wybieramy Hijack a session.

3. W HTTP history musimy odszukać żądanie które zawiera ciasteczka <code>hijack_cookie</code> i przesłać je do sequencera
   <br> ![App Screenshot](https://snipboard.io/LsNziA.jpg)

4. W sequencerze za pomocą live capture generujemy kilkadziesiąt/kilkaset tokenów i zapisujemy je do pliku tekstowego

5. Możemy zauważyć, że klucze są podzielone na dwie części, a ich wartości nie są losowe - pierwsza część jest czymś w rodzaju numeru użytkownika, a druga przypomina timestamp.
   <br> ![App Screenshot](https://snipboard.io/OLf5ph.jpg)

6. Żeby przeprowadzić atak brutalny należy przesłać zapytanie z ciasteczkiem <code>hijack_cookie</code> do intrudera i metodą prób i błędów próbować znaleźć token używany przez innego użytkownika.
Jeśli w zapytaniu nie przesyłane jest ciasteczko <code>hijack_cookie</code>, to trzeba jeszcze raz wysłać formularz.
Do znalezienia klucza potrzeba troche szczęścia, raczej nie znajdziecie go przy pierwszej próbie ataku.
