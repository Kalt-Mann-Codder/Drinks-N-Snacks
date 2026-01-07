<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>School Bus of Drinks</title>
<style>
body{font-family:sans-serif;background:#f0f2f5;margin:0;padding:0;}
.container{max-width:800px;margin:40px auto;padding:25px;background:#fff;border-radius:15px;box-shadow:0 8px 20px rgba(0,0,0,0.15);}
h1,h2,h3{text-align:center;color:#222;margin-bottom:15px;}
.products{display:flex;flex-wrap:wrap;justify-content:space-around;margin:25px 0;}
.product{padding:12px 18px;background:#e8e8e8;border-radius:8px;text-align:center;width:30%;margin-bottom:15px;}
.product button{margin-top:8px;padding:6px 12px;background:#007bff;color:#fff;border:none;border-radius:6px;cursor:pointer;}
.product button:hover{background:#0056b3;}
ul{list-style:none;padding:0;margin:10px 0;}
ul li{padding:8px 6px;background:#f9f9f9;border-radius:6px;margin-bottom:5px;box-shadow:0 2px 5px rgba(0,0,0,0.05);}
input[type=number], input[type=text], select{padding:6px;margin:5px 0;border-radius:6px;border:1px solid #ccc;width:100%;}
button#placeOrder{background:#28a745;color:#fff;padding:12px 25px;border:none;border-radius:6px;cursor:pointer;margin-top:10px;display:block;width:100%;}
button#placeOrder:hover{background:#218838;}
button{outline:none;}
button:disabled{background:#aaa;cursor:not-allowed;}
.admin-panel{display:none;margin-top:30px;}
</style>
<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js"></script>
</head>
<body>
<div class="container">
<h1>School Bus of Drinks</h1>

<div class="products">
  <div class="product" data-name="Salsa" data-price="25">
    Salsa - 25 MZN <button class="addCart">Add</button> 
    <button class="addWishlist">♥ Wishlist</button>
  </div>
  <div class="product" data-name="Senor Naks" data-price="10">
    Senor Naks - 10 MZN <button class="addCart">Add</button> 
    <button class="addWishlist">♥ Wishlist</button>
  </div>
  <div class="product" data-name="Red Bull" data-price="50">
    Red Bull - 50 MZN <button class="addCart">Add</button> 
    <button class="addWishlist">♥ Wishlist</button>
  </div>
  <div class="product" data-name="Fizz" data-price="25">
    Fizz - 25 MZN <button class="addCart">Add</button> 
    <button class="addWishlist">♥ Wishlist</button>
  </div>
</div>

<h2>Your Order</h2>
<ul id="orderList"></ul>
<p>Total: <span id="total">0</span> MZN</p>

<h3>Enter Your Details</h3>
<label>Name: <input type="text" id="userName" placeholder="Letters only"></label><br>
<label>Number: <input type="text" id="userNumber" placeholder="9 digits, starts with 8"></label><br>
<label>Curriculum: 
  <select id="userCorresponding">
    <option value="">--Select--</option>
    <option value="National">National</option>
    <option value="Cambridge">Cambridge</option>
    <option value="IGCSE">IGCSE</option>
  </select>
</label><br>
<label>Donation: <input type="number" id="donation" min="0" value="0"></label><br>
<label>Note: <input type="text" id="note" placeholder="Special instructions"></label><br>
<button id="placeOrder">Place Order</button>
<p id="message"></p>

<h2>Your Wishlist</h2>
<ul id="wishlistList"></ul>

<h2 class="admin-trigger">Admin Panel</h2>
<input type="text" id="adminCode" placeholder="Type secret page code">
<div class="admin-panel">
<h2>Admin Panel</h2>
<label>Username: <input type="text" id="adminUser"></label><br>
<label>Password: <input type="password" id="adminPass"></label><br>
<button id="loginAdmin">Login</button>
<div id="adminOrders"></div>
</div>
</div>

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

let cart = [];
let wishlist = JSON.parse(localStorage.getItem('wishlist') || '[]');

function renderCart(){
  const list = document.getElementById('orderList');
  list.innerHTML = '';
  let total = 0;
  cart.forEach(i=>{
    list.innerHTML += `<li>${i.name} x ${i.quantity} = ${i.price*i.quantity} MZN</li>`;
    total += i.price*i.quantity;
  });
  document.getElementById('total').innerText = total;
}

function renderWishlist(){
  const listEl = document.getElementById('wishlistList');
  listEl.innerHTML = '';
  wishlist.forEach(i=>{
    listEl.innerHTML += `<li>${i.name} - ${i.price} MZN <button onclick="moveToCart('${i.name}')">Add to Cart</button></li>`;
  });
}

function moveToCart(name){
  const item = wishlist.find(i=>i.name===name);
  if(item){
    cart.push({...item, quantity:1});
    wishlist = wishlist.filter(i=>i.name!==name);
    localStorage.setItem('wishlist', JSON.stringify(wishlist));
    renderCart();
    renderWishlist();
  }
}

document.querySelectorAll('.addCart').forEach(btn=>{
  btn.addEventListener('click', e=>{
    const prod = e.target.parentElement;
    const name = prod.dataset.name;
    const price = parseFloat(prod.dataset.price);
    const item = cart.find(i=>i.name===name);
    if(item) item.quantity++;
    else cart.push({name, price, quantity:1});
    renderCart();
  });
});

document.querySelectorAll('.addWishlist').forEach(btn=>{
  btn.addEventListener('click', e=>{
    const prod = e.target.parentElement;
    const name = prod.dataset.name;
    const price = parseFloat(prod.dataset.price);
    if(!wishlist.find(i=>i.name===name)){
      wishlist.push({name, price});
      localStorage.setItem('wishlist', JSON.stringify(wishlist));
      renderWishlist();
    }
  });
});

function validateInputs(name, number, curriculum){
  if(!/^[a-zA-Z\s]+$/.test(name)) return 'Name must have letters only';
  if(!/^8[2-7]\d{7}$/.test(number)) return 'Number must start with 8, second 2-7, total 9 digits';
  if(!["National","Cambridge","IGCSE"].includes(curriculum)) return 'Select a valid curriculum';
  return null;
}

document.getElementById('placeOrder').addEventListener('click', ()=>{
  const userName = document.getElementById('userName').value.trim();
  const userNumber = document.getElementById('userNumber').value.trim();
  const userCorresponding = document.getElementById('userCorresponding').value;
  const validation = validateInputs(userName,userNumber,userCorresponding);
  if(validation){
    alert(validation);
    return;
  }
  if(cart.length===0) return alert('Add items first');

  const donation = parseFloat(document.getElementById('donation').value) || 0;
  const note = document.getElementById('note').value || '';
  const total = cart.reduce((sum,i)=>sum+i.price*i.quantity,0);

  const order = {items:cart, donation, note, total, timestamp: Date.now(), userName, userNumber, userCorresponding};
  db.collection('orders').add(order).then(()=>{
    document.getElementById('message').innerText = 'Order placed successfully!';
    cart = [];
    renderCart();
    document.getElementById('donation').value=0;
    document.getElementById('note').value='';
  }).catch(err=>console.error(err));
});

// Admin Panel
const adminPanel = document.querySelector('.admin-panel');
document.getElementById('adminCode').addEventListener('input', e=>{
  if(e.target.value==='secret page 101') adminPanel.style.display='block';
});

const ADMIN_USER = 'Kalt';
const ADMIN_PASS = 'A7v>H("h"?Td';

document.getElementById('loginAdmin').addEventListener('click', ()=>{
  const user = document.getElementById('adminUser').value;
  const pass = document.getElementById('adminPass').value;
  if(user===ADMIN_USER && pass===ADMIN_PASS){
    alert('Admin logged in');
    loadAdminOrders();
  } else alert('Invalid credentials');
});

function loadAdminOrders(){
  const container = document.getElementById('adminOrders');
  container.innerHTML = '<h3>Orders:</h3>';
  db.collection('orders').orderBy('timestamp','desc').get().then(snapshot=>{
    snapshot.forEach(doc=>{
      const o = doc.data();
      const items = o.items.map(i=>`${i.name} x${i.quantity}`).join(', ');
      container.innerHTML += `<p><b>${o.userName}</b> (${o.userNumber}, ${o.userCorresponding}) | Items: ${items} | Donation: ${o.donation} | Total: ${o.total} MZN | ${new Date(o.timestamp).toLocaleString()}</p>`;
    });
  });
}

renderCart();
renderWishlist();
</script>
</body>
</html>


