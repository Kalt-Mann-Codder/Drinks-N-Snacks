<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>School Snacks & Drinks</title>
<style>
body{font-family:Arial,sans-serif;margin:0;padding:0;background:#f4f4f9;color:#222;}
header{background:#ff6f61;color:#fff;padding:15px;text-align:center;}
header input{padding:5px;width:80%;margin-top:10px;border-radius:5px;border:none;}
nav a{color:white;text-decoration:none;margin:0 10px;}
main{padding:15px;}
section{background:white;padding:15px;border-radius:10px;margin-bottom:15px;box-shadow:0 5px 15px rgba(0,0,0,0.1);}
.product,.orderCard,.wishlistCard,.donationCard{display:block;border:1px solid #ddd;border-radius:10px;padding:10px;margin:10px 0;width:100%;max-width:400px;transition:transform 0.2s;background:#fff;}
.product:hover,.orderCard:hover{transform:scale(1.02);box-shadow:0 5px 15px rgba(0,0,0,0.2);}
button{padding:5px 10px;margin-top:5px;cursor:pointer;background:#ff6f61;color:white;border:none;border-radius:5px;}
input[type=number],input[type=text],select{padding:5px;margin:5px 0;width:100%;border-radius:5px;border:1px solid #ccc;}
.adminPanel{display:none;background:#fff3cd;padding:15px;border-radius:10px;box-shadow:0 5px 15px rgba(0,0,0,0.2);}
footer{text-align:center;padding:10px;background:#eee;}
h3{margin-top:15px;}
</style>
<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-database-compat.js"></script>
</head>
<body>

<header>
<h1>School Snacks & Drinks</h1>
<input type="text" id="searchBar" placeholder="Search or type secret code...">
<nav>
<a href="#productsSection">Products</a> |
<a href="#wishlistSection">Wish Notes</a> |
<a href="#donationSection">Donations</a> |
<a href="#ordersSection">My Orders</a>
</nav>
</header>

<main>
<section id="userSection">
<h2>Enter Your Details</h2>
<input type="text" id="userName" placeholder="Your Name in App">
<input type="text" id="userPhone" placeholder="Phone Number">
<input type="text" id="userGrade" placeholder="Grade">
<select id="userCurriculum">
<option value="">Select Curriculum</option>
<option value="IGCSE">IGCSE</option>
<option value="Cambridge">Cambridge</option>
<option value="National">National</option>
</select>
</section>

<section id="productsSection">
<h2>Products</h2>
<div class="product" data-name="Salsa" data-price="25">
<p>Salsa - 25 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addCart">Add to Cart</button>
</div>
<div class="product" data-name="Naks" data-price="10">
<p>Naks - 10 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addCart">Add to Cart</button>
</div>
<div class="product" data-name="Red Bull" data-price="50">
<p>Red Bull - 50 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addCart">Add to Cart</button>
</div>
<div class="product" data-name="Fizz" data-price="25">
<p>Fizz - 25 MZN</p>
<input type="number" min="1" value="1" class="quantity">
<button class="addCart">Add to Cart</button>
</div>
</section>

<section id="cartSection">
<h2>Cart</h2>
<ul id="cartList"></ul>
<p>Total: <span id="cartTotal">0</span> MZN</p>
<button id="placeOrder">Place Order</button>
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
<h2>Donations</h2>
<input type="text" id="donorName" placeholder="Your Name">
<input type="text" id="donorPhone" placeholder="Phone Number">
<input type="number" id="donationAmount" placeholder="Amount">
<button id="donateBtn">Donate</button>
<div id="donationListUser"></div>
<p>e-Mola: 875510022 | M-Pesa: 855458144</p>
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
// Firebase Config
const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT.firebaseio.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// Admin credentials
const ADMIN_USERNAME="Kalt";
const ADMIN_PASSWORD='A7v>H("h"?Td';
const SECRET_CODE="secret page 101";

// Cart
let cart = [];
let removedOrders = [];

// Helper validations
function validName(name){return /^[A-Za-z ]+$/.test(name);}
function validPhone(phone){return /^8[2-7]\d{7}$/.test(phone);}
function validCurriculum(cur){return ["IGCSE","Cambridge","National"].includes(cur);}

// CART
function renderCart(){
  const list = document.getElementById("cartList");
  list.innerHTML = "";
  let total = 0;
  cart.forEach(i=>{
    list.innerHTML += `<li>${i.name} x ${i.qty} = ${i.price*i.qty} MZN</li>`;
    total += i.price*i.qty;
  });
  document.getElementById("cartTotal").innerText = total;
}

// Add to cart
document.querySelectorAll(".addCart").forEach(btn=>{
  btn.addEventListener("click",e=>{
    const productDiv=e.target.parentElement;
    const name=productDiv.dataset.name;
    const price=parseInt(productDiv.dataset.price);
    const qty=parseInt(productDiv.querySelector(".quantity").value);
    const existing=cart.find(p=>p.name===name);
    if(existing) existing.qty+=qty;
    else cart.push({name,price,qty});
    renderCart();
  });
});

// PLACE ORDER
document.getElementById("placeOrder").addEventListener("click",()=>{
  const userName=document.getElementById("userName").value.trim();
  const userPhone=document.getElementById("userPhone").value.trim();
  const userGrade=document.getElementById("userGrade").value.trim();
  const userCurriculum=document.getElementById("userCurriculum").value;
  if(!validName(userName)||!validPhone(userPhone)||!validCurriculum(userCurriculum)||!userGrade){alert("Fill all fields correctly"); return;}
  if(cart.length===0){alert("Add items to cart"); return;}
  const order = {
    userName,userPhone,userGrade,userCurriculum,
    products: cart,
    total: cart.reduce((sum,p)=>sum+p.price*p.qty,0),
    date: Date.now()
  };
  db.ref("orders").push(order,err=>{
    if(err) alert("Error placing order");
    else{
      alert("Order placed!");
      cart=[];
      renderCart();
      renderUserOrders();
      renderAdminOrders();
    }
  });
});

// USER ORDERS
function renderUserOrders(){
  const myOrdersDiv=document.getElementById("myOrders");
  myOrdersDiv.innerHTML="";
  const userName=document.getElementById("userName").value.trim();
  const userPhone=document.getElementById("userPhone").value.trim();
  if(!validName(userName)||!validPhone(userPhone)){myOrdersDiv.innerHTML="Enter valid name & phone"; return;}
  db.ref("orders").once("value",snap=>{
    myOrdersDiv.innerHTML="";
    snap.forEach(child=>{
      const o=child.val();
      o.key=child.key;
      if(o.userName===userName && o.userPhone===userPhone){
        const div=document.createElement("div");
        div.className="orderCard";
        div.innerHTML=`<strong>Order #${o.key}</strong><br>${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}<br>Total: ${o.total} MZN
        <button onclick="cancelUserOrder('${o.key}')">Cancel</button>`;
        myOrdersDiv.appendChild(div);
      }
    });
  });
}
window.cancelUserOrder=function(key){
  if(confirm("Cancel this order?")){
    db.ref("orders/"+key).remove(err=>{
      if(!err){alert("Order canceled"); renderUserOrders(); renderAdminOrders();}
    });
  }
}

// ADMIN PANEL
document.getElementById("searchBar").addEventListener("keydown",e=>{
  if(e.key==="Enter" && e.target.value===SECRET_CODE){
    let u=prompt("Username"); let p=prompt("Password");
    if(u===ADMIN_USERNAME && p===ADMIN_PASSWORD){
      document.getElementById("adminPanel").style.display="block";
      renderAdminOrders();
      renderAdminWishlist();
      renderAdminDonations();
    }else alert("Wrong credentials!");
    e.target.value="";
  }
});

// ADMIN RENDER ORDERS
function renderAdminOrders(){
  const container=document.getElementById("adminOrders");
  container.innerHTML="";
  db.ref("orders").once("value",snap=>{
    snap.forEach(child=>{
      const o=child.val();
      const div=document.createElement("div");
      div.className="orderCard";
      div.innerHTML=`<strong>Order #${child.key}</strong> ${o.userName} (${o.userPhone}, Grade ${o.userGrade}, ${o.userCurriculum})<br>
      ${o.products.map(p=>`${p.qty} x ${p.name} = ${p.qty*p.price} MZN`).join("<br>")}
      <br>Total: ${o.total} MZN
      <button onclick="cancelAdminOrder('${child.key}')">Cancel</button>`;
      container.appendChild(div);
    });
  });
}
window.cancelAdminOrder=function(key){
  if(confirm("Cancel this order?")) db.ref("orders/"+key).remove(err=>{if(!err){alert("Order canceled"); renderUserOrders(); renderAdminOrders();}});
}

// TODO: Similar Firebase logic can be added for Wishlist & Donations
</script>
</body>
</html>
