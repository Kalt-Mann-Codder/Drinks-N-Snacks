<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>School Snacks & Drinks</title>
<style>
body {font-family: Arial, sans-serif; margin:0; background:#f4f4f9; color:#222;}
header {background:#ff6f61; color:#fff; padding:20px; text-align:center;}
header input {padding:5px; width:60%; margin-top:10px; border-radius:5px; border:none;}
nav a {color:white; text-decoration:none; margin:0 10px;}
main {padding:20px;}
section {background:white; padding:20px; border-radius:10px; margin-bottom:20px; box-shadow:0 5px 15px rgba(0,0,0,0.1);}
.product, .orderCard, .donationCard, .wishlistCard {display:inline-block; border:1px solid #ddd; border-radius:10px; padding:10px; margin:10px; width:calc(100% - 40px); max-width:400px; text-align:left; transition: transform 0.2s; background:#fff;}
.product:hover, .orderCard:hover, .donationCard:hover, .wishlistCard:hover {transform:scale(1.02); box-shadow:0 5px 15px rgba(0,0,0,0.2);}
button {padding:5px 10px; margin-top:5px; cursor:pointer; background:#ff6f61; color:white; border:none; border-radius:5px;}
input[type=number], input[type=text] {padding:5px; margin:5px 0; width:100%;}
footer {text-align:center; padding:10px; background:#eee;}
.adminPanel {display:none; background:#fff3cd; padding:20px; border-radius:10px; box-shadow:0 5px 15px rgba(0,0,0,0.2);}
h3 {margin-top:20px;}
</style>
<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js"></script>
</head>
<body>
<header>
<h1>School Snacks & Drinks</h1>
<input type="text" id="searchBar" placeholder="Search or type secret code...">
<nav>
<a href="#productsSection">Products</a> |
<a href="#wishlistSection">Wish Notes</a> |
<a href="#donationSection">Donation</a> |
<a href="#paymentSection">My Orders</a>
</nav>
</header>

<main>
<section id="userSection">
<h2>Enter Your Details</h2>
<input type="text" id="userName" placeholder="Your Name in App">
<input type="text" id="userPhone" placeholder="Phone Number">
<input type="text" id="userGrade" placeholder="Grade">
<input type="text" id="userCurriculum" placeholder="Curriculum (IGCSE/Cambridge/National)">
</section>

<section id="productsSection">
<h2>Our Products</h2>
<div class="product" data-name="Salsa" data-price="25">
<p>Salsa - 25 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addOrder">Add to Order</button>
</div>
<div class="product" data-name="Senor Naks" data-price="10">
<p>Senor Naks - 10 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addOrder">Add to Order</button>
</div>
<div class="product" data-name="Red Bull" data-price="50">
<p>Red Bull - 50 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addOrder">Add to Order</button>
</div>
<div class="product" data-name="Fizz" data-price="25">
<p>Fizz - 25 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addOrder">Add to Order</button>
</div>
</section>

<section id="wishlistSection">
<h2>Wish Notes</h2>
<input type="text" id="wishName" placeholder="Your Name">
<input type="text" id="wishPhone" placeholder="Phone Number">
<input type="text" id="wishText" placeholder="What do you want?">
<button id="addWish">Add Wish</button>
<div id="wishList"></div>
</section>

<section id="donationSection">
<h2>Donation</h2>
<input type="text" id="donorName" placeholder="Your Name">
<input type="text" id="donorPhone" placeholder="Phone Number">
<input type="number" id="donationAmount" placeholder="Amount">
<button id="donateBtn">Donate</button>
<div id="donationListUser"></div>
<p>e-Mola: 87xxxxxxx | M-Pesa: 85xxxxxxx</p>
</section>

<section id="paymentSection">
<h2>My Orders</h2>
<div id="myOrders"></div>
</section>

<section class="adminPanel" id="adminPanel">
<h2>Admin Panel</h2>
<button id="deleteHistory">Delete All History</button>
<button id="undoCancel">Undo Last Cancel</button>
<h3>Orders</h3>
<div id="adminOrders"></div>
<h3>Wishlist</h3>
<div id="adminWishlist"></div>
<h3>Donations</h3>
<div id="adminDonations"></div>
</section>

<footer>
<p>&copy; 2026 School Snacks & Drinks</p>
</footer>

<script>
// Firebase config
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const ADMIN_USERNAME="Kalt", ADMIN_PASSWORD='A7v>H("h"?Td', SECRET_CODE="secret page 101";
let removedOrders=[];

// Validation
function validName(name){return /^[A-Za-z ]+$/.test(name);}
function validPhone(phone){return /^8[2-7]\d{7}$/.test(phone);}
function validCurriculum(cur){return ["IGCSE","Cambridge","National"].includes(cur);}

// Add Order
document.querySelectorAll(".addOrder").forEach(btn=>{
btn.addEventListener("click", e=>{
const nameInApp=document.getElementById("userName").value.trim();
const phone=document.getElementById("userPhone").value.trim();
const grade=document.getElementById("userGrade").value.trim();
const curriculum=document.getElementById("userCurriculum").value.trim();
if(!validName(nameInApp)||!validPhone(phone)||!validCurriculum(curriculum)||!grade){alert("Check your info"); return;}
const productDiv=e.target.parentElement;
const name=productDiv.dataset.name;
const price=parseInt(productDiv.dataset.price);
const qty=parseInt(productDiv.querySelector(".quantity").value);
const order={id:Date.now(), nameInApp,phone,grade,curriculum,products:[{name,qty,price}],total:qty*price,date:Date.now()};
db.collection("orders").add(order).then(()=>{alert("Order placed!"); renderUserOrders(); renderAdminOrders();});
});
});

// User Orders
function renderUserOrders(){
const container=document.getElementById("myOrders"); container.innerHTML="";
const name=document.getElementById("userName").value.trim();
const phone=document.getElementById("userPhone").value.trim();
if(!validName(name)||!validPhone(phone)){container.innerHTML="Enter valid Name & Phone."; return;}
db.collection("orders").where("nameInApp","==",name).where("phone","==",phone).get().then(snap=>{
if(snap.empty){container.innerHTML="No orders yet."; return;}
snap.forEach(doc=>{
const o=doc.data();
let div=document.createElement("div"); div.className="orderCard";
div.innerHTML=`<strong>Order #${o.id}</strong><br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN <button onclick="cancelUserOrder('${doc.id}')">Cancel</button>`;
container.appendChild(div);
});
});
}
window.cancelUserOrder=function(id){if(confirm("Cancel this order?")){db.collection("orders").doc(id).delete().then(()=>{alert("Order canceled"); renderUserOrders(); renderAdminOrders();});}}

// Wishlist
function renderWish(){const container=document.getElementById("wishList"); container.innerHTML=""; db.collection("wishlist").get().then(snap=>{snap.forEach(doc=>{const w=doc.data(); const div=document.createElement("div"); div.className="wishlistCard"; div.textContent=`${w.wishText} by ${w.name} (${w.phone})`; container.appendChild(div);});});}
document.getElementById("addWish").addEventListener("click", ()=>{const name=document.getElementById("wishName").value.trim(); const phone=document.getElementById("wishPhone").value.trim(); const text=document.getElementById("wishText").value.trim(); if(!validName(name)||!validPhone(phone)||!text){alert("Invalid info"); return;} db.collection("wishlist").add({name,phone,wishText:text,date:Date.now()}).then(()=>{alert("Wish added!"); renderWish(); renderAdminWishlist();});});
renderWish();

// Donations
function renderUserDonations(){const container=document.getElementById("donationListUser"); container.innerHTML=""; const name=document.getElementById("donorName").value.trim(); const phone=document.getElementById("donorPhone").value.trim(); if(!validName(name)||!validPhone(phone)) return; db.collection("donations").where("name","==",name).where("phone","==",phone).get().then(snap=>{snap.forEach(doc=>{const d=doc.data(); const div=document.createElement("div"); div.className="donationCard"; div.textContent=`${d.amt} MZN`; container.appendChild(div);});});}
document.getElementById("donateBtn").addEventListener("click", ()=>{const name=document.getElementById("donorName").value.trim(); const phone=document.getElementById("donorPhone").value.trim(); const amt=parseInt(document.getElementById("donationAmount").value); if(!validName(name)||!validPhone(phone)||!amt){alert("Invalid info"); return;} db.collection("donations").add({name,phone,amt,date:Date.now()}).then(()=>{alert(`Thanks for donating ${amt} MZN!`); renderUserDonations(); renderAdminDonations();});});
renderUserDonations();

// Secret Admin
document.getElementById("searchBar").addEventListener("keydown", e=>{if(e.key==="Enter" && e.target.value===SECRET_CODE){let u=prompt("Admin Username"); let p=prompt("Admin Password"); if(u===ADMIN_USERNAME && p===ADMIN_PASSWORD){document.getElementById("adminPanel").style.display="block"; renderAdminOrders(); renderAdminWishlist(); renderAdminDonations();}else alert("Wrong credentials"); e.target.value="";}});

// Admin Renders
function renderAdminOrders(){const container=document.getElementById("adminOrders"); container.innerHTML=""; db.collection("orders").orderBy("date","desc").get().then(snap=>{snap.forEach(doc=>{const o=doc.data(); let div=document.createElement("div"); div.className="orderCard"; div.innerHTML=`<strong>Order #${o.id}</strong> ${o.nameInApp} (${o.phone}, Grade ${o.grade}, ${o.curriculum})<br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN <button onclick="cancelOrder('${doc.id}')">Cancel</button>`; container.appendChild(div);});});}
window.cancelOrder=function(id){if(confirm("Cancel this order?")){db.collection("orders").doc(id).delete().then(()=>{alert("Order canceled"); renderAdminOrders(); renderUserOrders();});}}
function renderAdminWishlist(){const container=document.getElementById("adminWishlist"); container.innerHTML=""; db.collection("wishlist").orderBy("date","desc").get().then(snap=>{snap.forEach(doc=>{const w=doc.data(); const div=document.createElement("div"); div.className="wishlistCard"; div.textContent=`${w.wishText} by ${w.name} (${w.phone})`; container.appendChild(div);});});}
function renderAdminDonations(){const container=document.getElementById("adminDonations"); container.innerHTML=""; db.collection("donations").orderBy("date","desc").get().then(snap=>{snap.forEach(doc=>{const d=doc.data(); const div=document.createElement("div"); div.className="donationCard"; div.textContent=`${d.amt} MZN by ${d.name} (${d.phone})`; container.appendChild(div);});});}

// Delete & Undo (optional)
document.getElementById("deleteHistory").addEventListener("click", ()=>{if(confirm("Delete all data?")){["orders","wishlist","donations"].forEach(c=>{db.collection(c).get().then(snap=>{snap.forEach(doc=>db.collection(c).doc(doc.id).delete());});}); alert("All data deleted."); renderAdminOrders(); renderAdminWishlist(); renderAdminDonations();}});

</script>
</body>
</html>
