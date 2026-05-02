<!DOCTYPE html>
<html>
<head>
  <title>Car Wash Booking</title>

  <!-- EmailJS -->
  <script src="https://cdn.jsdelivr.net/npm/emailjs-com@3/dist/email.min.js"></script>

  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js"></script>

  <style>
    body { font-family: Arial; text-align: center; }
    form { max-width: 400px; margin: auto; }
    input, select, button {
      width: 100%;
      padding: 10px;
      margin: 5px;
    }
    button {
      background: green;
      color: white;
      border: none;
    }
  </style>
</head>

<body>

<h1>🚗 Book a Car Wash</h1>

<form id="form">
  <input id="name" placeholder="Your Name" required>
  <input id="email" type="email" placeholder="Your Email" required>
  <input id="address" placeholder="Your Address" required>

  <input id="date" type="date" required>
  <select id="time"></select>

  <input id="brand" placeholder="Car Brand" required>
  <input id="type" placeholder="Car Type" required>
  <input id="color" placeholder="Car Color" required>

  <button type="submit">Book Now</button>
</form>

<script>
// 🔑 PASTE YOUR EMAILJS KEY
emailjs.init("YOUR_PUBLIC_KEY");

// 🔑 PASTE YOUR FIREBASE CONFIG
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const dateInput = document.getElementById("date");
const timeSelect = document.getElementById("time");

// Generate times
function generateTimes(bookedTimes = []) {
  timeSelect.innerHTML = "";

  for (let h = 8; h < 17; h++) {
    for (let m of [0, 25, 50]) {
      let time = `${h}:${m.toString().padStart(2,'0')}`;

      let option = document.createElement("option");
      option.value = time;
      option.textContent = time;

      if (bookedTimes.includes(time)) {
        option.disabled = true;
        option.textContent += " (Booked)";
      }

      timeSelect.appendChild(option);
    }
  }
}

// Load bookings from Firebase
async function loadBookings() {
  let date = dateInput.value;
  if (!date) return;

  let snapshot = await db.collection("bookings")
    .where("date", "==", date)
    .get();

  let bookedTimes = [];
  snapshot.forEach(doc => {
    bookedTimes.push(doc.data().time);
  });

  generateTimes(bookedTimes);
}

// Check weekend
dateInput.addEventListener("change", async () => {
  let day = new Date(dateInput.value).getDay();

  if (day !== 0 && day !== 6) {
    alert("Only Saturday & Sunday allowed");
    dateInput.value = "";
    return;
  }

  await loadBookings();
});

// Default times
generateTimes();

// Submit booking
document.getElementById("form").addEventListener("submit", async e => {
  e.preventDefault();

  let name = document.getElementById("name").value;
  let email = document.getElementById("email").value;
  let address = document.getElementById("address").value;
  let date = dateInput.value;
  let time = timeSelect.value;
  let brand = document.getElementById("brand").value;
  let type = document.getElementById("type").value;
  let color = document.getElementById("color").value;

  let id = date + "_" + time;

  let doc = await db.collection("bookings").doc(id).get();

  if (doc.exists) {
    alert("This time is already booked!");
    return;
  }

  // Save booking
  await db.collection("bookings").doc(id).set({
    name, email, address, date, time, brand, type, color
  });

  // Send email
  emailjs.send("YOUR_SERVICE_ID", "YOUR_TEMPLATE_ID", {
    name,
    email,
    address,
    date,
    time,
    brand,
    type,
    color
  });

  alert("Booking confirmed 🚗");

  // Refresh times
  await loadBookings();
});
</script>

</body>
</html>
