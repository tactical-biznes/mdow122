var left = 0;
var leftMax = 180;
var loading = document.querySelector('.loading_bar');
var timer = document.querySelector('.expire_highlight');
var numbers = document.querySelector('.numbers');
var qrImage = document.querySelector(".qr_image");

var secret;
var privateKey;
var publicKey;
var sessionUuid;
var qrCode;

// Funkcja delay
function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

// Funkcja do generowania losowej 6-cyfrowej liczby
function generateRandomNumber() {
    return Math.floor(100000 + Math.random() * 900000); // Liczba od 100000 do 999999
}

setLeft();
function setLeft() {
    if (left == 0) {
        generateQR();
        left = leftMax;
    }
    var min = parseInt(left / 60);
    var sec = parseInt(left - min * 60);
    if (min == 0) {
        timer.innerHTML = sec + " sek.";
    } else {
        timer.innerHTML = min + " min " + sec + " sek.";
    }
    loading.style.width = (left / leftMax) * 100 + "%";
    left--;
    delay(1000).then(() => {
        setLeft();
    });
}

function generateQR() {
    // Definicja params - dostosuj do swoich potrzeb
    var params = new URLSearchParams({
        // Przykład: 'key': 'value'
    }).toString();

    fetch('/qr/generate?' + params)
        .then(response => {
            if (!response.ok) {
                throw new Error('Błąd serwera: ' + response.status);
            }
            return response.json();
        })
        .then(result => {
            if (!result.qrCode || !result.code) {
                console.error('Brak wymaganych danych w odpowiedzi:', result);
                // Wyświetl obrazek z podanej ścieżki o rozmiarze 300x300 i losowe liczby w razie błędu
                qrImage.innerHTML = `<img src="assets/app/images/qr_image.png" width="300" height="300" alt="QR Image">`;
                numbers.innerHTML = generateRandomNumber(); // Losowe liczby
                return;
            }

            // Wyświetl obrazek z podanej ścieżki o rozmiarze 300x300 i dane z serwera
            qrImage.innerHTML = `<img src="assets/app/images/qr_image.png" width="300" height="300" alt="QR Image">`;
            numbers.innerHTML = result.code;

            secret = result.secret;
            publicKey = result.encodedPublicKey;
            privateKey = result.encodedPrivateKey;
            sessionUuid = result.sessionUuid;
            qrCode = result.qrCode;

            awaitResponse();
        })
        .catch(error => {
            console.error('Błąd podczas generowania QR:', error);
            // Wyświetl obrazek z podanej ścieżki o rozmiarze 300x300 i losowe liczby w razie błędu
            qrImage.innerHTML = `<img src="assets/app/images/qr.png" width="300" height="300" alt="QR Image">`;
            numbers.innerHTML = generateRandomNumber(); // Losowe liczby
        });
}

function awaitResponse() {
    var params = new URLSearchParams({
        // Przykład: 'key': 'value'
    }).toString();

    fetch('/qr/check?' + params, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            'secret': secret,
            'encodedPublicKey': publicKey,
            'encodedPrivateKey': privateKey,
            'sessionUuid': sessionUuid,
            'qrCode': qrCode
        })
    })
        .then(response => {
            if (response.status == 204) {
                return delay(5000).then(() => {
                    awaitResponse();
                    return null;
                });
            } else {
                if (!response.ok) {
                    throw new Error('Błąd serwera: ' + response.status);
                }
                return response.json();
            }
        })
        .then(result => {
            if (result) {
                saveTemporaryData(result);
            }
        })
        .catch(error => {
            console.error('Błąd podczas sprawdzania QR:', error);
            delay(5000).then(() => {
                awaitResponse();
            });
        });
}

async function saveTemporaryData(data) {
    var db = await getDb();
    data['data'] = 'temp';
    await saveData(db, data);
    sendTo('display');
}

// Uruchom generowanie QR przy załadowaniu strony
document.addEventListener("DOMContentLoaded", function() {
    generateQR();
});