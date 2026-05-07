
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Wish Upload</title>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-storage-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore-compat.js"></script>

<style>

body{
    margin:0;
    background:#0f172a;
    color:white;
    font-family:Arial;
    text-align:center;
}

.box{
    width:90%;
    max-width:420px;
    margin:auto;
    margin-top:30px;
    background:#1e293b;
    padding:20px;
    border-radius:20px;
}

input,button{
    width:100%;
    padding:12px;
    margin-top:12px;
    border:none;
    border-radius:10px;
    font-size:16px;
}

input{
    background:#334155;
    color:white;
}

button{
    background:#38bdf8;
    font-weight:bold;
}

video{
    width:100%;
    border-radius:15px;
    margin-top:10px;
}

.card{
    background:#334155;
    padding:10px;
    margin-top:20px;
    border-radius:15px;
}

.timer{
    color:#22c55e;
    font-size:20px;
    margin-top:10px;
}

</style>
</head>

<body>

<div class="box">

<h1>✨ Wish Upload ✨</h1>

<input type="text" id="wish" placeholder="Enter wish">

<input type="file"
id="video"
accept="video/*"
capture="environment">

<button onclick="uploadWish()">Upload Wish</button>

<div id="status"></div>

<div id="posts"></div>

</div>

<script>

// =======================
// FIREBASE CONFIG
// =======================

const firebaseConfig = {

apiKey: "PASTE_API_KEY",
authDomain: "YOUR_PROJECT.firebaseapp.com",
projectId: "YOUR_PROJECT",
storageBucket: "YOUR_PROJECT.appspot.com",
messagingSenderId: "123456",
appId: "APP_ID"

};

firebase.initializeApp(firebaseConfig);

const db = firebase.firestore();
const storage = firebase.storage();


// =======================
// UPLOAD FUNCTION
// =======================

async function uploadWish(){

    let wish = document.getElementById("wish").value;
    let file = document.getElementById("video").files[0];

    if(!wish || !file){
        alert("Wish aur video upload karo");
        return;
    }

    document.getElementById("status").innerHTML =
    "Uploading...";

    // file upload
    let fileName = Date.now() + "_" + file.name;

    let storageRef =
    storage.ref("videos/" + fileName);

    await storageRef.put(file);

    let videoURL =
    await storageRef.getDownloadURL();

    // upload time
    let uploadTime = Date.now();

    // delete after 2 days
    let deleteTime =
    uploadTime + (2 * 24 * 60 * 60 * 1000);

    // firestore save
    await db.collection("wishes").add({

        wish: wish,
        video: videoURL,
        uploadTime: uploadTime,
        deleteTime: deleteTime

    });

    document.getElementById("status").innerHTML =
    "✅ Uploaded Successfully";

    loadWishes();
}


// =======================
// LOAD POSTS
// =======================

async function loadWishes(){

    let posts = document.getElementById("posts");

    posts.innerHTML = "";

    let data =
    await db.collection("wishes").get();

    data.forEach(doc => {

        let item = doc.data();

        // auto delete after 2 days
        if(Date.now() > item.deleteTime){
            return;
        }

        let div = document.createElement("div");

        div.className = "card";

        div.innerHTML = `

        <h3>${item.wish}</h3>

        <video controls>
            <source src="${item.video}">
        </video>

        <div class="timer"
        id="timer_${doc.id}">
        </div>

        `;

        posts.appendChild(div);

        startCountdown(
            item.uploadTime,
            doc.id
        );

    });

}


// =======================
// 24 HOUR COUNTDOWN
// =======================

function startCountdown(uploadTime,id){

    let endTime =
    uploadTime + (24 * 60 * 60 * 1000);

    let timer =
    document.getElementById("timer_" + id);

    let x = setInterval(function(){

        let now = Date.now();

        let distance = endTime - now;

        if(distance < 0){

            timer.innerHTML =
            "⏰ Countdown Finished";

            clearInterval(x);

            return;
        }

        let hours =
        Math.floor((distance%(1000*60*60*24))
        /(1000*60*60));

        let minutes =
        Math.floor((distance%(1000*60*60))
        /(1000*60));

        let seconds =
        Math.floor((distance%(1000*60))
        /1000);

        timer.innerHTML =
        "⏳ " +
        hours + "h " +
        minutes + "m " +
        seconds + "s";

    },1000);

}

loadWishes();

</script>

</body>
</html>
