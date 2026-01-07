<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>School Snacks & Drinks</title>
<style>
/* ---- Global ---- */
body {margin:0;font-family: Arial,sans-serif;background:#f4f4f9;color:#222;}
header {background:#ff6f61;color:#fff;padding:20px;text-align:center;}
header input {padding:8px;width:60%;margin-top:10px;border-radius:5px;border:none;}
nav a {color:white;text-decoration:none;margin:0 10px;font-weight:bold;}
main {padding:20px;}
section {background:white;padding:20px;border-radius:10px;margin-bottom:20px;box-shadow:0 5px 15px rgba(0,0,0,0.1);}
.product, .orderCard, .donationCard, .wishlistCard {display:block;border:1px solid #ddd;border-radius:10px;padding:10px;margin:10px 0;transition: transform 0.2s;background:#fff;}
.product:hover, .orderCard:hover {transform:scale(1.02);box-shadow:0 5px 15px rgba(0,0,0,0.2);}
button {padding:6px 12px;margin-top:5px;cursor:pointer;background:#ff6f61;color:white;border:none;border-radius:5px;}
button:hover {background:#e65550;}
input[type=number], input[type=text], select {padding:6px;margin:5px 0;width:100%;border-radius:5px;border:1px solid #ccc;}
footer {text-align:center;padding:10px;background:#eee;}
.adminPanel {display:none;background:#fff3cd;padding:20px;border-radius:10px;box-shadow:0 5px 15px rgba(0,0,0,0.2);}
h3 {margin-top:20px;}
@media screen and (max-width:600px){header input{width:90%;} .product, .orderCard{width:100%;}}
</style>
<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
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
<div class="product" data-name="Naks" data-price="10">
<p>Naks - 10 MZN</p>
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
<!-- Add more products here -->
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
<p>e-Mola: 875510022 | M-Pesa: 855458144</p>
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
// ---- Firebase config ----
const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "YOUR_FIREBASE_AUTHDOMAIN",
  databaseURL: "YOUR_FIREBASE_DB_URL",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ---- Constants ----
const ADMIN_USERNAME="Kalt", ADMIN_PASSWORD='A7v>H("h"?Td', SECRET_CODE="secret page 101";
let removedOrders=[];

// ---- Validation ----
function validName(name){return /^[A-Za-z ]+$/.test(name);}
function validPhone(phone){return /^8[2-7]\d{7}$/.test(phone);}
function validCurriculum(cur){return ["IGCSE","Cambridge","National"].includes(cur);}

// ---- Add Order ----
document.querySelectorAll(".addOrder").forEach(btn=>{
btn.addEventListener("click", e=>{
const nameInApp=document.getElementById("userName").value.trim();
const phone=document.getElementById("userPhone").value.trim();
const grade=document.getElementById("userGrade").value.trim();
const curriculum=document.getElementById("userCurriculum").value.trim();
if(!validName(nameInApp)){alert("Invalid name."); return;}
if(!validPhone(phone)){alert("Phone must start 8(2-7) and have 9 digits."); return;}
if(!validCurriculum(curriculum)){alert("Curriculum must be IGCSE, Cambridge, or National."); return;}
if(!grade){alert("Enter your grade."); return;}
const productDiv=e.target.parentElement;
const name=productDiv.dataset.name;
const price=parseInt(productDiv.dataset.price);
const qty=parseInt(productDiv.querySelector(".quantity").value);
const id=Date.now();
const order={id,nameInApp,phone,grade,curriculum,products:[{name,qty,price}],total:qty*price,date:Date.now(),active:true};
db.ref("orders/"+id).set(order);
alert(`Added ${qty} x ${name} for ${nameInApp}`);
renderUserOrders();
renderAdminOrders();
});
});

// ---- User Orders ----
function renderUserOrders(){
const myOrdersContainer=document.getElementById("myOrders");
myOrdersContainer.innerHTML="";
const name=document.getElementById("userName").value.trim().toLowerCase();
const phone=document.getElementById("userPhone").value.trim();
if(!validName(name)||!validPhone(phone)){myOrdersContainer.innerHTML="Enter valid Name & Phone to see orders."; return;}
db.ref("orders").orderByChild("active").equalTo(true).once("value").then(snapshot=>{
snapshot.forEach(snap=>{
const o=snap.val();
if(o.nameInApp.toLowerCase()===name && o.phone===phone){
const div=document.createElement("div");
div.className="orderCard";
div.innerHTML=`<strong>Order #${o.id}</strong><br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN
<button onclick="cancelUserOrder('${o.id}')">Cancel</button>`;
myOrdersContainer.appendChild(div);
}
});
});
}
document.getElementById("userName").addEventListener("input", renderUserOrders);
document.getElementById("userPhone").addEventListener("input", renderUserOrders);

// ---- Cancel User Order ----
window.cancelUserOrder=function(id){
db.ref("orders/"+id).get().then(snap=>{
if(snap.exists()){removedOrders.push(snap.val()); db.ref("orders/"+id).remove(); renderUserOrders(); renderAdminOrders(); alert("Order canceled!");}
});
}

// ---- Wishlist ----
function renderWish(){
const container=document.getElementById("wishList"); container.innerHTML="";
db.ref("wishlist").once("value").then(snapshot=>{
snapshot.forEach(snap=>{
const w=snap.val();
container.innerHTML+=`<div class="wishlistCard">${w.wishText} by ${w.name} (${w.phone})</div>`;
});
});
}
renderWish();
document.getElementById("addWish")?.addEventListener("click", ()=>{
const name=document.getElementById("wishName").value.trim();
const phone=document.getElementById("wishPhone").value.trim();
const text=document.getElementById("wishText").value.trim();
if(!validName(name)||!validPhone(phone)||!text){alert("Enter valid info."); return;}
db.ref("wishlist").push({name,phone,wishText:text});
renderWish();
renderAdminWishlist();
alert("Wish added!");
});

// ---- Donations ----
function renderUserDonations(){
const container=document.getElementById("donationListUser"); container.innerHTML="";
const name=document.getElementById("donorName").value.trim();
const phone=document.getElementById("donorPhone").value.trim();
if(!validName(name)||!validPhone(phone)) return;
db.ref("donations").once("value").then(snapshot=>{
snapshot.forEach(snap=>{
const d=snap.val();
if(d.name===name && d.phone===phone){
container.innerHTML+=`<div class="donationCard">${d.amt} MZN</div>`;
}
});
});
}
renderUserDonations();

document.getElementById("donateBtn")?.addEventListener("click", ()=>{
const name=document.getElementById("donorName").value.trim();
const phone=document.getElementById("donorPhone").value.trim();
const amt=document.getElementById("donationAmount").value.trim();
if(!validName(name)||!validPhone(phone)||!amt){alert("Enter valid info."); return;}
db.ref("donations").push({name,phone,amt,date:Date.now()});
alert(`Thanks for donating ${amt} MZN!`);
renderUserDonations();
renderAdminDonations();
});

// ---- Secret Admin Panel ----
document.getElementById("searchBar")?.addEventListener("keydown", e=>{
if(e.key==="Enter" && e.target.value===SECRET_CODE){
let u=prompt("Admin Username"); let p=prompt("Admin Password");
if(u===ADMIN_USERNAME && p===ADMIN_PASSWORD){document.getElementById("adminPanel").style.display="block"; renderAdminOrders(); renderAdminWishlist(); renderAdminDonations();}
else alert("Wrong credentials!");
e.target.value="";
}
});

// ---- Admin renders ----
function renderAdminOrders(){
const container=document.getElementById("adminOrders"); container.innerHTML="";
db.ref("orders").once("value").then(snapshot=>{
snapshot.forEach(snap=>{
const o=snap.val();
let div=document.createElement("div"); div.className="orderCard";
div.innerHTML=`<strong>Order #${o.id}</strong> ${o.nameInApp} (${o.phone}, Grade ${o.grade}, ${o.curriculum})<br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN 
<button onclick="cancelOrder('${o.id}')">Cancel</button>`;
container.appendChild(div);
});
});
}
window.cancelOrder=function(id){db.ref("orders/"+id).get().then(snap=>{if(snap.exists()){removedOrders.push(snap.val()); db.ref("orders/"+id).remove(); renderAdminOrders(); renderUserOrders(); alert("Order canceled!");}});}

// Admin Wishlist
function renderAdminWishlist(){const container=document.getElementById("adminWishlist"); container.innerHTML="";
db.ref("wishlist").once("value").then(snapshot=>{snapshot.forEach(snap=>{const w=snap.val();container.innerHTML+=`<div class="wishlistCard">${w.wishText} by ${w.name} (${w.phone})</div>`;});});}

// Admin Donations
function renderAdminDonations(){const container=document.getElementById("adminDonations"); container.innerHTML="";
db.ref("donations").once("value").then(snapshot=>{snapshot.forEach(snap=>{const d=snap.val();container.innerHTML+=`<div class="donationCard">${d.amt} MZN by ${d.name} (${d.phone})</div>`;});});}

// Undo last cancel
document.getElementById("undoCancel")?.addEventListener("click", ()=>{
if(removedOrders.length===0){alert("Nothing to undo."); return;}
const last=removedOrders.pop();
db.ref("orders/"+last.id).set(last);
renderAdminOrders();
renderUserOrders();
alert("Undo done.");
});

// Delete all history
document.getElementById("deleteHistory")?.addEventListener("click", ()=>{
if(confirm("Delete all orders, wishlist, donations?")){
db.ref("orders").remove();
db.ref("wishlist").remove();
db.ref("donations").remove();
removedOrders=[];
renderAdminOrders(); renderAdminWishlist(); renderAdminDonations(); renderWish(); renderUserOrders(); renderUserDonations();
alert("All history deleted!");}
});
</script>
</body>
</html>
