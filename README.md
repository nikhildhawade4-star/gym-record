
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Gym Record</title>
<style>
body{margin:0;background:#101713;color:white;font-family:Arial;padding:20px}
h1{color:#50e69a}
.card{background:#1d2b23;padding:18px;margin:12px 0;border-radius:12px}
input{width:100%;box-sizing:border-box;padding:13px;margin:8px 0;background:#203329;color:white;border:1px solid #526b5a;border-radius:8px}
button{padding:13px;background:#39c982;color:#07140d;border:0;border-radius:8px;width:100%;margin:7px 0;font-weight:bold}
.hidden{display:none}
small{color:#aabbb0}
</style>
</head>
<body>

<h1>💪 GYM RECORD</h1>
<small>GYM MANAGEMENT SYSTEM</small>

<div id="login" class="card">
<h3>Owner Login</h3>
<input id="email" type="email" placeholder="Owner Email">
<input id="password" type="password" placeholder="Password">
<button id="loginBtn">Login</button>
<p id="message"></p>
</div>

<div id="dashboard" class="hidden">
<h2>Owner Dashboard</h2>
<div class="card">Total Members: <b id="total">0</b></div>

<div class="card">
<h3>Add New Member</h3>
<input id="name" placeholder="Member Name">
<input id="phone" placeholder="Mobile Number">
<input id="plan" placeholder="Membership Plan">
<input id="fee" type="number" placeholder="Membership Fee">
<input id="date" type="date">
<button id="saveBtn">Save Member</button>
</div>

<h3>Member List</h3>
<div id="members"></div>
<button id="logoutBtn">Logout</button>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/12.6.0/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/12.6.0/firebase-auth.js";
import { getFirestore, collection, addDoc, getDocs, query, orderBy } from "https://www.gstatic.com/firebasejs/12.6.0/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

const ownerUID = "YOUR_OWNER_UID";

const $ = id => document.getElementById(id);

$("loginBtn").onclick = async () => {
 try {
  await signInWithEmailAndPassword(auth,$("email").value,$("password").value);
  $("message").textContent = "";
 } catch(e) {
  $("message").textContent = e.message;
 }
};

onAuthStateChanged(auth, async user => {
 if(user && user.uid === ownerUID){
  $("login").classList.add("hidden");
  $("dashboard").classList.remove("hidden");
  await loadMembers();
 } else {
  $("dashboard").classList.add("hidden");
  $("login").classList.remove("hidden");
  if(user) await signOut(auth);
 }
});

$("saveBtn").onclick = async () => {
 const user=auth.currentUser;
 if(!user || user.uid !== ownerUID) return;
 const name=$("name").value.trim();
 const phone=$("phone").value.trim();
 const plan=$("plan").value.trim();
 const fee=Number($("fee").value);
 const date=$("date").value;
 if(!name || !phone || !plan || !fee || !date){
  alert("Please fill all details");return;
 }
 try {
  await addDoc(collection(db,"members"),{
   name,phone,plan,fee,joiningDate:date,
   createdAt:new Date().toISOString()
  });
  $("name").value="";
  $("phone").value="";
  $("plan").value="";
  $("fee").value="";
  $("date").value="";
  await loadMembers();
  alert("Member saved successfully!");
 } catch(e){alert(e.message);}
};

async function loadMembers(){
 const user=auth.currentUser;
 if(!user || user.uid!==ownerUID) return;
 const snap=await getDocs(query(collection(db,"members"),orderBy("createdAt","desc")));
 $("total").textContent=snap.size;
 $("members").replaceChildren();
 snap.forEach(doc=>{
  const m=doc.data();
  const div=document.createElement("div");
  div.className="card";
  const title=document.createElement("b");
  title.textContent=m.name;
  const details=document.createElement("p");
  details.textContent=m.phone+" | "+m.plan+" | ₹"+m.fee+" | "+m.joiningDate;
  div.append(title,details);
  $("members").append(div);
 });
}

$("logoutBtn").onclick=()=>signOut(auth);
</script>
</body>
</html>
