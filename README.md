<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>School Drinks & Snacks</title>
<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
/* General Styles */
body {margin:0; font-family: Arial, sans-serif; background:#f4f4f9; color:#222;}
header {background:#ff6f61; color:#fff; padding:20px; text-align:center;}
header input {padding:10px; width:90%; max-width:500px; margin-top:10px; border-radius:5px; border:none;}
nav a {color:white; text-decoration:none; margin:0 10px;}
main {padding:20px; max-width:1200px; margin:auto;}
section {background:white; padding:20px; border-radius:10px; margin-bottom:20px; box-shadow:0 5px 15px rgba(0,0,0,0.1);}
.product, .orderCard, .donationCard, .wishlistCard {display:flex; justify-content:space-between; align-items:center; border:1px solid #ddd; border-radius:10px; padding:10px; margin:10px 0; flex-wrap:wrap;}
.product:hover, .orderCard:hover {box-shadow:0 5px 15px rgba(0,0,0,0.2);}
button {padding:8px 15px; margin:5px; cursor:pointer; background:#ff6f61; color:white; border:none; border-radius:5px;}
button:hover {background:#e65550;}
input, select {padding:8px; margin:5px 0; border-radius:5px; border:1px solid #ccc; width:100%; max-width:300px;}
.adminPanel {display:none; background:#fff3cd; padding:20px; border-radius:10px; box-shadow:0 5px 15px rgba(0,0,0,0.2);}
h2,h3 {margin-top:15px;}
@media(max-width:600px){
  .product, .orderCard, .donationCard, .wishlistCard {flex-direction:column; align-items:flex-start;}
  header input {width:95%;}
}
</style>
</head>
<body>

<header>
<h1>School Drinks & Snacks</h1>
<input type="text" id="searchBar" placeholder="Search or type secret code...">
<nav>
<a href="#productsSection">Products</a> |
<a href="#wishlistSection">Wish Notes</a> |
<a href="#donationSection">Donation</a> |
<a href="#ordersSection">My Orders</a>
</nav>
</header>

<main>
<!-- User Info -->
<section id="userSection">
<h2>Enter Your Details</h2>
<input type="text" id="userName" placeholder="Your Name in App">
<input type="text" id="userPhone" placeholder="Phone Number (9 digits, start 8)">
<input type="text" id="userGrade" placeholder="Grade">
<select id="userCurriculum">
  <option value="">Select Curriculum</option>
  <option value="National">National</option>
  <option value="Cambridge">Cambridge</option>
  <option value="IGCSE">IGCSE</option>
</select>
</section>

<!-- Products -->
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
</section>

<!-- Wishlist -->
<section id="wishlistSection">
<h2>Wish Notes</h2>
<input type="text" id="wishName" placeholder="Your Name">
<input type="text" id="wishPhone" placeholder="Phone Number">
<input type="text" id="wishText" placeholder="What do you want?">
<button id="addWish">Add Wish</button>
<div id="wishList"></div>
</section>

<!-- Donations -->
<section id="donationSection">
<h2>Donation</h2>
<input type="text" id="donorName" placeholder="Your Name">
<input type="text" id="donorPhone" placeholder="Phone Number">
<input type="number" id="donationAmount" placeholder="Amount MZN">
<button id="donateBtn">Donate</button>
<div id="donationListUser"></div>
<p>e-Mola: 875510022 | M-Pesa: 855458144</p>
</section>

<!-- Orders -->
<section id="ordersSection">
<h2>My Orders</h2>
<div id="myOrders"></div>
</section>

<!-- Admin Panel -->
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
</main>

<footer>
<p>&copy; 2026 School Drinks & Snacks</p>
</footer>

<script>
// ----- Firebase Config -----
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT.firebaseio.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ----- Admin & Secret -----
const ADMIN_USERNAME="Kalt", ADMIN_PASSWORD='A7v>H("h"?Td', SECRET_CODE="secret page 101";
let removedOrders=[];

// ----- Utility Validation -----
function validName(name){return /^[A-Za-z ]+$/.test(name);}
function validPhone(phone){return /^8[2-7]\d{7}$/.test(phone);}
function validCurriculum(cur){return ["IGCSE","Cambridge","National"].includes(cur);}

// ----- Orders -----
function addOrder(product){
  const nameInApp=document.getElementById("userName").value.trim();
  const phone=document.getElementById("userPhone").value.trim();
  const grade=document.getElementById("userGrade").value.trim();
  const curriculum=document.getElementById("userCurriculum").value;
  if(!validName(nameInApp)||!validPhone(phone)||!grade||!validCurriculum(curriculum)){
    alert("Enter valid user details"); return;
  }
  const qty=parseInt(product.querySelector(".quantity").value);
  const name=product.dataset.name;
  const price=parseInt(product.dataset.price);
  const orderId=Date.now();
  const orderData={id:orderId,nameInApp,phone,grade,curriculum,products:[{name,qty,price}],total:qty*price,date:Date.now()};
  db.ref("orders/"+orderId).set(orderData);
  alert(`Added ${qty} x ${name}`);
  renderUserOrders();
  renderAdminOrders();
}

document.querySelectorAll(".addOrder").forEach(btn=>{
  btn.addEventListener("click", e=>addOrder(e.target.parentElement));
});

// ----- Render User Orders -----
function renderUserOrders(){
  const container=document.getElementById("myOrders"); container.innerHTML="";
  const name=document.getElementById("userName").value.trim();
  const phone=document.getElementById("userPhone").value.trim();
  if(!validName(name)||!validPhone(phone)){container.innerHTML="Enter valid Name & Phone to see orders."; return;}
  db.ref("orders").orderByChild("phone").equalTo(phone).once("value", snapshot=>{
    const ordersData=snapshot.val();
    container.innerHTML="";
    if(!ordersData){container.innerHTML="No orders yet."; return;}
    Object.values(ordersData).forEach(o=>{
      if(o.nameInApp!==name) return;
      let div=document.createElement("div"); div.className="orderCard";
      div.innerHTML=`<strong>Order #${o.id}</strong><br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN
      <button onclick="cancelUserOrder('${o.id}')">Cancel</button>`;
      container.appendChild(div);
    });
  });
}

window.cancelUserOrder=function(id){
  db.ref("orders/"+id).remove();
  renderUserOrders();
  renderAdminOrders();
  alert("Order canceled!");
}

// ----- Wishlist -----
function renderWish(){const container=document.getElementById("wishList"); container.innerHTML="";
db.ref("wishlist").once("value", snap=>{
  const data=snap.val();
  if(!data) return;
  Object.values(data).forEach(w=>{
    const div=document.createElement("div"); div.className="wishlistCard"; div.textContent=`${w.wishText} by ${w.name} (${w.phone})`; container.appendChild(div);
  });
});
}
renderWish();

document.getElementById("addWish")?.addEventListener("click", ()=>{
  const name=document.getElementById("wishName").value.trim();
  const phone=document.getElementById("wishPhone").value.trim();
  const text=document.getElementById("wishText").value.trim();
  if(!validName(name)||!validPhone(phone)||!text){alert("Enter valid info"); return;}
  const wishId=Date.now();
  db.ref("wishlist/"+wishId).set({id:wishId,name,phone,wishText:text});
  alert("Wish added!");
  renderWish();
  renderAdminWishlist();
});

// ----- Donations -----
function renderUserDonations(){const container=document.getElementById("donationListUser"); container.innerHTML="";
const name=document.getElementById("donorName").value.trim();
const phone=document.getElementById("donorPhone").value.trim();
db.ref("donations").orderByChild("phone").equalTo(phone).once("value", snap=>{
  const data=snap.val(); if(!data) return;
  Object.values(data).forEach(d=>{
    if(d.name!==name) return;
    const div=document.createElement("div"); div.className="donationCard"; div.textContent=`${d.amt} MZN`; container.appendChild(div);
  });
});
}
document.getElementById("donateBtn")?.addEventListener("click", ()=>{
  const name=document.getElementById("donorName").value.trim();
  const phone=document.getElementById("donorPhone").value.trim();
  const amt=parseInt(document.getElementById("donationAmount").value);
  if(!validName(name)||!validPhone(phone)||!amt){alert("Enter valid info"); return;}
  const donationId=Date.now();
  db.ref("donations/"+donationId).set({id:donationId,name,phone,amt,date:Date.now()});
  alert(`Thanks for donating ${amt} MZN!`);
  renderUserDonations();
  renderAdminDonations();
});

// ----- Secret Admin Panel -----
document.getElementById("searchBar")?.addEventListener("keydown", e=>{
  if(e.key==="Enter" && e.target.value===SECRET_CODE){
    const u=prompt("Admin Username"); const p=prompt("Admin Password");
    if(u===ADMIN_USERNAME && p===ADMIN_PASSWORD){
      document.getElementById("adminPanel").style.display="block";
      renderAdminOrders(); renderAdminWishlist(); renderAdminDonations();
    }else alert("Wrong credentials!");
    e.target.value="";
  }
});

// ----- Admin Render Functions -----
function renderAdminOrders(){
  const container=document.getElementById("adminOrders"); container.innerHTML="";
  db.ref("orders").once("value", snap=>{
    const data=snap.val();
    if(!data) return;
    Object.values(data).forEach(o=>{
      const div=document.createElement("div"); div.className="orderCard";
      div.innerHTML=`<strong>Order #${o.id}</strong> ${o.nameInApp} (${o.phone}, Grade ${o.grade}, ${o.curriculum})<br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN <button onclick="cancelOrder('${o.id}')">Cancel</button>`;
      container.appendChild(div);
    });
  });
}

window.cancelOrder=function(id){
  removedOrders.push(id);
  db.ref("orders/"+id).remove();
  renderAdminOrders();
  renderUserOrders();
  alert("Order canceled!");
}

function renderAdminWishlist(){
  const container=document.getElementById("adminWishlist"); container.innerHTML="";
  db.ref("wishlist").once("value", snap=>{
    const data=snap.val();
    if(!data) return;
    Object.values(data).forEach(w=>{
      const div=document.createElement("div"); div.className="wishlistCard"; div.textContent=`${w.wishText} by ${w.name} (${w.phone})`; container.appendChild(div);
    });
  });
}

function renderAdminDonations(){
  const container=document.getElementById("adminDonations"); container.innerHTML="";
  db.ref("donations").once("value", snap=>{
    const data=snap.val(); if(!data) return;
    Object.values(data).forEach(d=>{
      const div=document.createElement("div"); div.className="donationCard"; div.textContent=`${d.amt} MZN by ${d.name} (${d.phone})`; container.appendChild(div);
    });
  });
}

// Undo & Delete History
document.getElementById("undoCancel")?.addEventListener("click", ()=>{
  if(removedOrders.length===0){alert("Nothing to undo."); return;}
  alert("Undo works only manually with Firebase DB");
});

document.getElementById("deleteHistory")?.addEventListener("click", ()=>{
  if(confirm("Delete all data?")){
    db.ref().remove();
    alert("All history deleted!");
    renderAdminOrders(); renderAdminWishlist(); renderAdminDonations(); renderWish(); renderUserOrders(); renderUserDonations();
  }
});
</script>
</body>
</html>

