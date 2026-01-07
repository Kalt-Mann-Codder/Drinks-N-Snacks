<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>School Snacks & gggggg Drinks</title>
<style>
body {font-family: Arial, sans-serif; margin:0; background:#f4f4f9; color:#222;}
header {background:#ff6f61; color:#fff; padding:20px; text-align:center;}
header input {padding:5px; width:60%; margin-top:10px; border-radius:5px; border:none;}
nav a {color:white; text-decoration:none; margin:0 10px;}
main {padding:20px;}
section {background:white; padding:20px; border-radius:10px; margin-bottom:20px; box-shadow:0 5px 15px rgba(0,0,0,0.1);}
.product, .orderCard, .donationCard, .wishlistCard {display:block; border:1px solid #ddd; border-radius:10px; padding:10px; margin:10px 0; width:100%; max-width:400px; text-align:left; transition: transform 0.2s; background:#fff;}
.product:hover, .orderCard:hover {transform:scale(1.02); box-shadow:0 5px 15px rgba(0,0,0,0.2);}
button {padding:5px 10px; margin-top:5px; cursor:pointer; background:#ff6f61; color:white; border:none; border-radius:5px;}
input[type=number], input[type=text], select {padding:5px; margin:5px 0; width:100%;}
footer {text-align:center; padding:10px; background:#eee;}
.adminPanel {display:none; background:#fff3cd; padding:20px; border-radius:10px; box-shadow:0 5px 15px rgba(0,0,0,0.2);}
h3 {margin-top:20px;}
</style>

<header>
<h1>School Snacks & Drinks</h1>
<input type="text" id="searchBar" placeholder="Search or type secret code...">
<nav>
<a href="#productsSection">Products</a> |
<a href="#wishlistSection">Wish Notes</a> |
<a href="#donationSection">Donation</a> |
<a href="#ordersSection">My Orders</a>
</nav>
</header>

<main>
<section id="userSection">
<h2>Your Details</h2>
<input type="text" id="userName" placeholder="Your Name (letters only)">
<input type="text" id="userPhone" placeholder="Phone Number (8XXXXXXXX)">
<input type="text" id="userGrade" placeholder="Grade">
<select id="userCurriculum">
  <option value="">Select Curriculum</option>
  <option value="National">National</option>
  <option value="Cambridge">Cambridge</option>
  <option value="IGCSE">IGCSE</option>
</select>
</section>

<section id="productsSection">
<h2>Products</h2>
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
<p>Payments via e-Mola: 875510022 | M-Pesa: 855458144</p>
</section>

<section id="ordersSection">
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
const ADMIN_USERNAME="Kalt";
const ADMIN_PASSWORD='A7v>H("h"?Td';
const SECRET_CODE="secret page 101";
const WEEK_MS=7*24*60*60*1000;

let orders=JSON.parse(localStorage.getItem("orders")||"[]").filter(o=>Date.now()-o.date<WEEK_MS);
let wishlist=JSON.parse(localStorage.getItem("wishlist")||"[]");
let donations=JSON.parse(localStorage.getItem("donations")||"[]");
let removedOrders=[];

function saveAll(){localStorage.setItem("orders",JSON.stringify(orders)); localStorage.setItem("wishlist",JSON.stringify(wishlist)); localStorage.setItem("donations",JSON.stringify(donations));}

// Validation
function validName(n){return /^[A-Za-z ]+$/.test(n);}
function validPhone(p){return /^8[2-7]\d{7}$/.test(p);}
function validCurriculum(c){return ["National","Cambridge","IGCSE"].includes(c);}

// Add order
document.querySelectorAll(".addOrder").forEach(btn=>{
btn.addEventListener("click", e=>{
const nameInApp=document.getElementById("userName").value.trim();
const phone=document.getElementById("userPhone").value.trim();
const grade=document.getElementById("userGrade").value.trim();
const curriculum=document.getElementById("userCurriculum").value;
if(!validName(nameInApp)){alert("Invalid Name"); return;}
if(!validPhone(phone)){alert("Invalid Phone"); return;}
if(!grade){alert("Enter Grade"); return;}
if(!validCurriculum(curriculum)){alert("Select valid Curriculum"); return;}

const div=e.target.parentElement;
const productName=div.dataset.name;
const price=parseInt(div.dataset.price);
const qty=parseInt(div.querySelector(".quantity").value);
const order={id:Date.now(), nameInApp, phone, grade, curriculum, products:[{name:productName, qty, price}], total:qty*price, date:Date.now()};
orders.push(order); saveAll(); alert(`Added ${qty} x ${productName}`); renderUserOrders(); renderAdminOrders();
});
});

// Render user orders
function renderUserOrders(){
const container=document.getElementById("myOrders"); container.innerHTML="";
const name=document.getElementById("userName").value.trim();
const phone=document.getElementById("userPhone").value.trim();
if(!validName(name)||!validPhone(phone)){container.innerHTML="Enter valid Name & Phone to see orders."; return;}
const userOrders=orders.filter(o=>o.nameInApp===name && o.phone===phone);
if(userOrders.length===0){container.innerHTML="No orders yet."; return;}
userOrders.forEach(o=>{
let div=document.createElement("div"); div.className="orderCard";
div.innerHTML=`<strong>Order #${o.id}</strong><br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN
<button onclick="cancelUserOrder(${o.id})">Cancel</button>`;
container.appendChild(div);
});
}

// Cancel user order
window.cancelUserOrder=function(id){
const name=document.getElementById("userName").value.trim();
const phone=document.getElementById("userPhone").value.trim();
const index=orders.findIndex(o=>o.id===id && o.nameInApp===name && o.phone===phone);
if(index>-1){removedOrders.push(orders[index]); orders.splice(index,1); saveAll(); renderUserOrders(); renderAdminOrders(); alert("Order canceled");}
}

// Wishlist
function renderWish(){const container=document.getElementById("wishList"); container.innerHTML=""; wishlist.forEach(w=>{let div=document.createElement("div"); div.className="wishlistCard"; div.textContent=`${w.wishText} by ${w.name} (${w.phone})`; container.appendChild(div);});}
renderWish();
document.getElementById("addWish").addEventListener("click",()=>{
const name=document.getElementById("wishName").value.trim();
const phone=document.getElementById("wishPhone").value.trim();
const text=document.getElementById("wishText").value.trim();
if(!validName(name)||!validPhone(phone)||!text){alert("Enter valid info"); return;}
if(wishlist.some(w=>w.name===name && w.phone===phone)){alert("Already added"); return;}
wishlist.push({name, phone, wishText:text}); saveAll(); renderWish(); renderAdminWishlist(); alert("Wish added");
});

// Donations
function renderUserDonations(){
const container=document.getElementById("donationListUser"); container.innerHTML="";
const name=document.getElementById("donorName").value.trim(); const phone=document.getElementById("donorPhone").value.trim();
donations.filter(d=>d.name===name && d.phone===phone).forEach(d=>{let div=document.createElement("div"); div.className="donationCard"; div.textContent=`${d.amt} MZN`; container.appendChild(div);});
}
renderUserDonations();
document.getElementById("donateBtn").addEventListener("click",()=>{
const name=document.getElementById("donorName").value.trim();
const phone=document.getElementById("donorPhone").value.trim();
const amt=document.getElementById("donationAmount").value.trim();
if(!validName(name)||!validPhone(phone)||!amt){alert("Enter valid info"); return;}
donations.push({name, phone, amt, date:Date.now()}); saveAll(); renderUserDonations(); renderAdminDonations(); alert(`Thanks for donating ${amt} MZN!`);
});

// Admin Panel
document.getElementById("searchBar").addEventListener("keydown", e=>{
if(e.key==="Enter" && e.target.value===SECRET_CODE){
let u=prompt("Username"); let p=prompt("Password");
if(u===ADMIN_USERNAME && p===ADMIN_PASSWORD){
document.getElementById("adminPanel").style.display="block"; renderAdminOrders(); renderAdminWishlist(); renderAdminDonations();}
else alert("Wrong credentials!");
e.target.value="";
}
});

// Admin renders
function renderAdminOrders(){const container=document.getElementById("adminOrders"); container.innerHTML=""; orders.forEach(o=>{let div=document.createElement("div"); div.className="orderCard"; div.innerHTML=`<strong>Order #${o.id}</strong> ${o.nameInApp} (${o.phone}, Grade ${o.grade}, ${o.curriculum})<br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN <button onclick="cancelOrder(${o.id})">Cancel</button>`; container.appendChild(div);});}
window.cancelOrder=function(id){const index=orders.findIndex(o=>o.id===id); if(index>-1){removedOrders.push(orders[index]); orders.splice(index,1); saveAll(); renderAdminOrders(); renderUserOrders(); alert("Order canceled");}}

// Admin Wishlist
function renderAdminWishlist(){const container=document.getElementById("adminWishlist"); container.innerHTML=""; wishlist.forEach(w=>{let div=document.createElement("div"); div.className="wishlistCard"; div.textContent=`${w.wishText} by ${w.name} (${w.phone})`; container.appendChild(div);});}
renderAdminWishlist();

// Admin Donations
function renderAdminDonations(){const container=document.getElementById("adminDonations"); container.innerHTML=""; donations.forEach(d=>{let div=document.createElement("div"); div.className="donationCard"; div.textContent=`${d.amt} MZN by ${d.name} (${d.phone})`; container.appendChild(div);});}
renderAdminDonations();

// Undo cancel
document.getElementById("undoCancel").addEventListener("click",()=>{
if(removedOrders.length===0){alert("Nothing to undo"); return;}
orders.push(removedOrders.pop()); saveAll(); renderAdminOrders(); renderUserOrders(); alert("Undo done");
});

// Delete history
document.getElementById("deleteHistory").addEventListener("click",()=>{
if(confirm("Delete all orders, wishlist, donations?")){orders=[]; wishlist=[]; donations=[]; removedOrders=[]; saveAll(); renderAdminOrders(); renderAdminWishlist(); renderAdminDonations(); renderWish(); renderUserOrders(); renderUserDonations(); alert("All history deleted");}
});

</script>
</body>
</html>
