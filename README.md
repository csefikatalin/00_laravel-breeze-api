 #  Authentikációs végpontok létrehozása Laravel Breeze API-val

1. Laravel projekt létrehozása

    composer create-project laravel/laravel laravel-breeze-api

    php artisan key:generate --ansi

Ha most inicializálod a reposytorit könnyebb nyomon követni, mit változtat meg a projektben a breeze csomag.  git init

2. Telepítsük a breeze csomagot, majd készítsük el az API végpontokat

    composer require laravel/breeze --dev
    php artisan breeze:install api

## .env fájl configurálása

- adatbázis név és hozzáférések
- A backend és a frontend URL-jét. 

A Laravel applikációnk most backendként csupán  API kiszolgálóként fog működni, ezért backend és a frontend alkalmazásunk nem azonos domainen fog futni. Hiszen a backend a 8000-es, a frontend pedig a 3000-es proton lesz elérhető. Szerencsére a domain név azonos a két alkalmazás esetén. Ezket az elérési utakat be kell állítani a .env fájlban. 

    APP_URL=http://localhost:8000
    FRONTEND_URL=http://localhost:3000

Biztonsági okokból érdemes hozzáadni az alábbi válotozókat is az .env fájlhoz: 

    SANCTUM_STATEFUL_DOMAINS=localhost:3000
    SESSION_DOMAIN=localhost

A **SESSION_DOMAIN** beállítja a munkamenet cookie tartományát, ami segít megelőzni a  CSRF-támadásokat, vagy a munkamenetek eltérítését. Biztosítja, hogy csak a megadott domainről legenek elérhetőek az API kiszolgáló végpontjai. 
Ha nem adunk meg tartományt, a Laravel az alkalmazás aktuális tartományát használja.

A **SANCTUM_STATEFUL_DOMAINS**  azon tartományok listáját határozza meg, melyekről engedélyezzük  az állapotmegőrző kérések küldését a Laravel Sanctum által védett  végpontokhoz. Ez is segít megelőzni CSRF-támadásokat.

### Sütik (cookie) használata

    SESSION_DRIVER=cookie

A **SESSION_DRIVER=cookie** azt jelenti, hogy a Laravel sütiket használ a munkamenet adatok tárolásához. Amikor egy felhasználó bejelentkezik az alkalmazásába, a program egy sütiben tárolja a munkamenet adatait a böngészőben. Ez a süti minden további kérés során automatikusan visszaküldésre kerül a kiszolgálóhoz, lehetővé téve a Laravel számára a felhasználó munkamenetadatainak lekérését és az állapotuk megőrzését a kérések között.

## Laravel auth végpontjai
 A laravel Breeze létrehozta az alábbi végpontokat a routes/auth.php fájlban: 
 - /register
 - /login
 - /forgot-password
 - /reset-password
 - /verify-email/{id}/{hash}
 - /email/verification-notification
 - /logout

 Frontend oldalon a bejelentkzeés és a regsiztrációs során ezekhez a végpontokat hívhatjuk meg. 

 Az api.php-ban pedig a sanctum segítségével végezhetjük el a felhasználó azonosítását. 

## Változások a configurációs fájlokban

### config/app.php

### config/cors.php
Módosult az engedélezett útvonalak lista, illetve az engedélyezett origins lista. 
Valamint a supports_credentials tulajdonság beállításával engedélyezzük a cross site (nem azonos domainről érkező) kérések esetén a a kérés tartalmazza a sütiket és aegyéb hitelesítési adatokat. 

Amikor kéréseket küldünk különböző tartományok vagy aldoménok között, a böngésző biztonsági okokból blokkolhatja a kéréseket. Ez a Same-Origin Policy (Azonos Eredetű Politika), amely korlátozza, hogy egy tartományból származó weboldal hogyan léphet kapcsolatba más tartományokból származó erőforrásokkal.

A keresztszerver kérések engedélyezéséhez használhatjuk a Cross-Origin Resource Sharing (CORS) fejléceket, amelyek lehetővé teszik, hogy egy weboldal hozzáférjen erőforrásokhoz egy másik tartományból. A Laravel Sanctum esetében az **EnsureFrontendRequestsAreStateful** middleware állítja be  a szükséges CORS fejléceket.

    'paths' => ['*'],
    'allowed_methods' => ['*'],
    'allowed_origins' => [env('FRONTEND_URL', 'http:/localhost:3000')],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => true,

A 'supports_credentials' => true beállítási lehetőség azt jelzi, hogy a keresztszerver kérések tartalmazzák a sütiket és egyéb hitelesítési adatokat. Ha ez a beállítás false-ra van állítva, akkor a CORS fejlécek nem tartalmaznak hitelesítési adatokat, és a böngésző nem fog sütiket vagy egyéb hitelesítési információkat küldeni a kéréssel.

### config/sanctum.php 

Itt használja fel a SANCTUM_STATEFUL_DOMAINS értéket. 



