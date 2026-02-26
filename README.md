<!DOCTYPE html>
<html lang="sv">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Daglig Arbetsrapport</title>

<!-- EmailJS -->
<script src="https://cdn.jsdelivr.net/npm/emailjs-com@3/dist/email.min.js"></script>

<style>
body {
    font-family: Arial, sans-serif;
    background: #f4f6f8;
    padding: 15px;
}
.container {
    background: white;
    padding: 20px;
    border-radius: 10px;
    max-width: 600px;
    margin: auto;
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
}
.logo {
    text-align: center;
    margin-bottom: 15px;
}
.logo img {
    max-width: 180px;
    height: auto;
    display: inline-block;
}
h1 {
    text-align: center;
    font-size: 20px;
}
label {
    font-weight: bold;
    display: block;
    margin-top: 15px;
}
input, textarea {
    width: 100%;
    padding: 10px;
    margin-top: 5px;
    border-radius: 6px;
    border: 1px solid #ccc;
    font-size: 16px;
}
textarea { min-height: 80px; }
button {
    margin-top: 20px;
    width: 100%;
    padding: 14px;
    background-color: #1f4e79;
    color: white;
    border: none;
    border-radius: 6px;
    font-size: 16px;
    cursor: pointer;
}
button:disabled {
    background-color: #7a99b5;
    cursor: not-allowed;
}
.success {
    margin-top: 15px;
    color: green;
    font-weight: bold;
    text-align: center;
}

/* Lösenordsskydd */
#dagbokContainer { display: none; }
</style>
</head>

<body>

<div class="container">

    <div class="logo">
        <img src="images/logga.png" alt="Företagslogotyp">
    </div>

    <h1>Daglig Arbetsrapport</h1>

    <!-- Lösenordsskydd -->
    <label>Lösenord:</label>
    <input type="password" id="accessPass" placeholder="Ange lösenord">
    <button id="checkPass">Öppna dagbok</button>

    <div id="dagbokContainer">
        <form id="dagbokForm">

            <label>Datum</label>
            <input type="date" name="datum" required>

            <label>Namn på arbetstagare</label>
            <input type="text" name="namn" required>

            <label>Arbetarens e-post (för kopia)</label>
            <input type="email" name="worker_email" required>

            <label>Antal resurs/person</label>
            <input type="number" name="antal_resurs">

            <label>Arbetsordernummer</label>
            <input type="text" name="ordernummer">

            <label>Arbetsplats / Adress</label>
            <input type="text" name="plats">

            <label>Starttid</label>
            <input type="time" name="starttid">

            <label>Sluttid</label>
            <input type="time" name="sluttid">

            <label>Rast (minuter)</label>
            <input type="number" name="rast">

            <label>Fordon / Maskin</label>
            <input type="text" name="fordon">

            <label>Utfört arbete</label>
            <textarea name="arbete"></textarea>

            <label>Avvikelser / Noteringar</label>
            <textarea name="avvikelser"></textarea>

            <button type="submit">Skicka rapport</button>
            <div class="success" id="successMsg"></div>

        </form>
    </div>
</div>

<script>
(function(){
    emailjs.init("8QT8MtJ4IdJNiRo4A"); // Din public key
})();

// Lösenordsskydd
document.getElementById("checkPass").addEventListener("click", function(){
    const pass = document.getElementById("accessPass").value;
    if(pass === "BYGG123"){ // Ändra till ditt lösenord
        document.getElementById("dagbokContainer").style.display = "block";
        this.style.display = "none";
        document.getElementById("accessPass").style.display = "none";
    } else {
        alert("Fel lösenord!");
    }
});

// Skicka formulär
document.getElementById("dagbokForm").addEventListener("submit", function(e){
    e.preventDefault();
    const btn = this.querySelector("button");
    btn.disabled = true;
    btn.innerHTML = "Skickar...";

    // Beräkna arbetstid
    const start = this.querySelector('[name="starttid"]').value;
    const end = this.querySelector('[name="sluttid"]').value;
    const rast = parseInt(this.querySelector('[name="rast"]').value) || 0;
    let arbetstid = "";
    if(start && end){
        const startTime = new Date(`1970-01-01T${start}:00`);
        const endTime = new Date(`1970-01-01T${end}:00`);
        let diff = (endTime - startTime) / 1000 / 60 - rast;
        arbetstid = (diff/60).toFixed(2) + " timmar";
    }

    // Lägg till arbetstid som hidden
    let hidden = document.createElement("input");
    hidden.type = "hidden";
    hidden.name = "arbetstid";
    hidden.value = arbetstid;
    this.appendChild(hidden);

    // Skicka med EmailJS
    emailjs.sendForm(
        "service_eehhitu", // Ditt Service ID
        "template_5xvghx2", // Ditt Template ID
        this
    ).then(() => {
        // Skicka till Google Apps Script
        fetch("DIN_WEBAPP_URL_HÄR", {
            method: "POST",
            body: JSON.stringify(Object.fromEntries(new FormData(this))),
        });

        document.getElementById("successMsg").innerHTML =
            "✅ Rapporten skickad, loggad och PDF skapad.";
        this.reset();
        btn.disabled = false;
        btn.innerHTML = "Skicka rapport";
    }, (error) => {
        alert("Något gick fel. Försök igen.");
        console.error(error);
        btn.disabled = false;
        btn.innerHTML = "Skicka rapport";
    });
});
</script>
</body>
</html>
