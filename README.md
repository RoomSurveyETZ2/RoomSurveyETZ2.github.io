<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Sondage - Lorraine</title>

<style>

body{
    margin:0;
    font-family: Arial, sans-serif;
    
    background: 
        linear-gradient(rgba(0,0,0,0.55), rgba(0,0,0,0.55)),
        url("metz2.jpg");
        
    background-size: cover;
    background-position: center;
    background-attachment: fixed;
}

.container{
    max-width:800px;
    margin:40px auto;
    background:white;
    padding:40px;
    border-radius:20px;
    box-shadow:0 15px 40px rgba(0,0,0,0.3);
}

h1{
    text-align:center;
    color:#b11226;
}

.option{
    margin:15px 0;
}

.confirm-btn{
    margin-top:25px;
    width:100%;
    padding:15px;
    border:none;
    border-radius:12px;
    background:#b11226;
    color:white;
    font-size:16px;
    font-weight:bold;
    cursor:pointer;
}

.confirm-btn:hover{
    background:#8c0f1e;
}

.result-bar{
    margin-top:8px;
    height:25px;
    background:#eee;
    border-radius:20px;
    overflow:hidden;
}

.result-fill{
    height:100%;
    width:0%;
    background:#b11226;
    text-align:right;
    padding-right:10px;
    color:white;
    font-weight:bold;
    line-height:25px;
    transition: width 1s ease;
}

.floating-img{
    position: fixed;
    bottom: 20px;
    right: 20px;
    width: 120px;
    z-index: 10;
}

</style>
</head>

<body>

<div class="container">
<h1>Quels noms préfères-tu pour les salles de formations ?</h1>

<div id="optionsContainer"></div>

<button id="confirmVote" class="confirm-btn">
Confirmer le vote
</button>

</div>

<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getFirestore, doc, getDoc, setDoc, updateDoc, increment } 
from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyAFZCwpU4RbuSsGicKRsPY6wfjiroJWyk4",
  authDomain: "survey-rooms.firebaseapp.com",
  projectId: "survey-rooms",
  storageBucket: "survey-rooms.firebasestorage.app",
  messagingSenderId: "955956214197",
  appId: "1:955956214197:web:0d4daed320fa993af481fc"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

const options = [
"Graoully",
"Verdun",
"Robert Schuman",
"Stanislas",
"Mirabelle",
"Cathédrale Saint-Étienne",
"Mont Saint-Quentin",
"Les Vosges",
"La Moselle",
"Madeleine de Commercy"
];

const voteDoc = doc(db, "survey", "results");

const container = document.getElementById("optionsContainer");
const confirmBtn = document.getElementById("confirmVote");

async function loadVotes() {
  const docSnap = await getDoc(voteDoc);

  if (!docSnap.exists()) {
    const initialData = {};
    options.forEach(name => initialData[name] = 0);
    await setDoc(voteDoc, initialData);
    return initialData;
  }

  return docSnap.data();
}

async function vote(selectedOptions) {
  const updates = {};
  selectedOptions.forEach(name => {
    updates[name] = increment(1);
  });
  await updateDoc(voteDoc, updates);
}

function showVotingUI() {
  options.forEach((option,index)=>{
    const div = document.createElement("div");
    div.className = "option";

    const checkbox = document.createElement("input");
    checkbox.type = "checkbox";
    checkbox.id = "opt-"+index;

    const label = document.createElement("label");
    label.htmlFor = "opt-"+index;
    label.textContent = " " + option;

    div.appendChild(checkbox);
    div.appendChild(label);
    container.appendChild(div);
  });
}

function showResults(data, alreadyVoted = false) {
  container.innerHTML = "";
  confirmBtn.style.display = "none";

  if(alreadyVoted){
    const msg = document.createElement("div");
    msg.style.marginBottom = "20px";
    msg.style.fontWeight = "bold";
    msg.style.color = "#b11226";
    msg.textContent = "Vous avez déjà voté ! Voici les résultats :";
    container.appendChild(msg);
  }

  const total = Object.values(data).reduce((a,b)=>a+b,0);

  options.forEach(option=>{
    const div = document.createElement("div");
    div.className = "option";

    const label = document.createElement("div");
    label.textContent = option;

    const bar = document.createElement("div");
    bar.className = "result-bar";

    const fill = document.createElement("div");
    fill.className = "result-fill";

    const percent = total === 0 ? 0 :
      Math.round((data[option] / total) * 100);

    fill.textContent = percent + "%";

    bar.appendChild(fill);
    div.appendChild(label);
    div.appendChild(bar);
    container.appendChild(div);

    setTimeout(()=>{
      fill.style.width = percent + "%";
    },100);
  });
}

confirmBtn.addEventListener("click", async function(){

  const checked = document.querySelectorAll("input[type=checkbox]:checked");

  if(checked.length === 0){
    alert("Veuillez sélectionner au moins une option.");
    return;
  }

  const selectedNames = [];

  checked.forEach(box=>{
    const index = parseInt(box.id.split("-")[1]);
    selectedNames.push(options[index]);
  });

  await vote(selectedNames);

  // MARQUER L'UTILISATEUR COMME AYANT VOTÉ
  localStorage.setItem("hasVoted", "true");

  const updatedData = await loadVotes();
  showResults(updatedData);
});

// Initialisation
const initialData = await loadVotes();

// Vérifier si l'utilisateur a déjà voté
const hasVoted = localStorage.getItem("hasVoted");

if(hasVoted){
  // Afficher message + résultats si déjà voté
  showResults(initialData, true);
} else {
  // Sinon, afficher le formulaire
  showVotingUI();
}

</script>

<img src="croix.png" class="floating-img">

</body>
</html>
