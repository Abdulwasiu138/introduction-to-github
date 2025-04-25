<!DOCTYPE html>
<html>
<head>
  <title>UNIOSUN FBAS Voting</title>

  <!-- 1) Firebase Compat Scripts -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

  <!-- 2) Chart.js Script -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    body {
      font-family: Arial, sans-serif;
      background: #eef;
      padding: 20px;
      text-align: center;
    }
    .candidate {
      border: 1px solid #ccc;
      padding: 20px;
      margin: 15px auto;
      border-radius: 8px;
      background: #fff;
      max-width: 320px;
    }
    button {
      background: #2E8B57;
      color: #fff;
      border: none;
      padding: 10px 20px;
      border-radius: 5px;
      cursor: pointer;
    }
    .count {
      font-size: 18px;
      margin-top: 10px;
    }
    #message {
      color: green;
      font-weight: bold;
      margin-top: 20px;
    }
    canvas {
      margin: 30px auto 0;
      background: #fff;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 10px #ccc;
      max-width: 500px;
    }
  </style>
</head>
<body>

  <h1>UNIOSUN FBAS Voting</h1>
  <h2>Faculty of Basic and Applied Sciences</h2>

  <div class="candidate">
    <h2>Candidate A</h2>
    <button onclick="vote('CandidateA')">Vote</button>
    <p class="count" id="CandidateA-count">Votes: 0</p>
  </div>

  <div class="candidate">
    <h2>Candidate B</h2>
    <button onclick="vote('CandidateB')">Vote</button>
    <p class="count" id="CandidateB-count">Votes: 0</p>
  </div>

  <p id="message"></p>

  <canvas id="voteChart" width="400" height="250"></canvas>

  <script>
    // --- Initialize Firebase (compat) ---
    const firebaseConfig = {
      apiKey: "AIzaSyBVQ7aH7cogF7AWLaSHNzRrvFl3AgAfxjQ",
      authDomain: "uniosun-voting-poll.firebaseapp.com",
      databaseURL: "https://uniosun-voting-poll-default-rtdb.firebaseio.com",
      projectId: "uniosun-voting-poll",
      storageBucket: "uniosun-voting-poll.firebasestorage.app",
      messagingSenderId: "249237157002",
      appId: "1:249237157002:web:db3df016ae66a697b1e973"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    // --- Voting function ---
    function vote(candidate) {
      if (localStorage.getItem("voted")) {
        document.getElementById("message").textContent = "You have already voted!";
        return;
      }
      const voteRef = db.ref("votes/" + candidate);
      voteRef.transaction(current => (current || 0) + 1);
      localStorage.setItem("voted", "true");
      document.getElementById("message").textContent = `Thank you for voting for ${candidate}!`;
    }
    window.vote = vote;  // expose to onclick

    // --- Set up Chart.js bar chart ---
    const ctx = document.getElementById("voteChart").getContext("2d");
    const voteChart = new Chart(ctx, {
      type: "bar",
      data: {
        labels: ["Candidate A", "Candidate B"],
        datasets: [{
          label: "Votes",
          data: [0, 0],
          backgroundColor: ["#2E8B57", "#4682B4"]
        }]
      },
      options: {
        responsive: true,
        plugins: { legend: { display: false } },
        scales: { y: { beginAtZero: true, precision: 0 } }
      }
    });

    // --- Subscribe to realtime updates ---
    function updateCounts() {
      ["CandidateA", "CandidateB"].forEach((cand, idx) => {
        db.ref("votes/" + cand).on("value", snap => {
          const count = snap.val() || 0;
          document.getElementById(`${cand}-count`).textContent = `Votes: ${count}`;
          voteChart.data.datasets[0].data[idx] = count;
          voteChart.update();
        });
      });
    }
    updateCounts();
  </script>

</body>
</html>
