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
}
.success {
    margin-top: 15px;
    color: green;
    font-weight: bold;
}
</style>
</head>

<body>

<div class="container">


</div>

<h1>Daglig Arbetsrapport</h1>

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

<script>
(function(){
    emailjs.init("8QT8MtJ4IdJNiRo4A");
})();

document.getElementById("dagbokForm").addEventListener("submit", function(e){
    e.preventDefault();

    const btn = document.querySelector("button");
    btn.disabled = true;
    btn.innerHTML = "Skickar...";

    // Hämta tider
    const start = document.querySelector('[name="starttid"]').value;
    const end = document.querySelector('[name="sluttid"]').value;
    const rast = parseInt(document.querySelector('[name="rast"]').value) || 0;

    let arbetstid = "";

    if(start && end){
        const startTime = new Date(`1970-01-01T${start}:00`);
        const endTime = new Date(`1970-01-01T${end}:00`);
        let diff = (endTime - startTime) / 1000 / 60; // minuter
        diff -= rast;
        arbetstid = (diff/60).toFixed(2) + " timmar";
    }

    // Lägg till beräknad arbetstid som hidden field
    let hidden = document.createElement("input");
    hidden.type = "hidden";
    hidden.name = "arbetstid";
    hidden.value = arbetstid;
    this.appendChild(hidden);

    emailjs.sendForm(
        "service_eehhitu",
        "template_5xvghx2",
        this
    ).then(function(){
  
fetch("DIN_WEBAPP_URL_HÄR", {
    method: "POST",
    body: JSON.stringify(Object.fromEntries(new FormData(this))),
});
        document.getElementById("successMsg").innerHTML =
        "✅ Rapporten är skickad till kontoret och en kopia till dig.";

        document.getElementById("dagbokForm").reset();
        btn.disabled = false;
        btn.innerHTML = "Skicka rapport";

    }, function(error){
        alert("Något gick fel. Försök igen.");
        console.error(error);
        btn.disabled = false;
        btn.innerHTML = "Skicka rapport";
    });
});
</script>
