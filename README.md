<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>FixLink - Service Repair Platform</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      font-family: Arial, sans-serif;
    }

    body {
      background: #f4f6f9;
      color: #222;
    }

    header {
      background: #1f2937;
      color: white;
      padding: 18px;
      text-align: center;
      font-size: 28px;
      font-weight: bold;
    }

    .container {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
      padding: 20px;
    }

    .card {
      background: white;
      border-radius: 14px;
      padding: 20px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }

    h2 {
      margin-bottom: 15px;
      color: #111827;
    }

    input, textarea, select {
      width: 100%;
      padding: 10px;
      margin-top: 8px;
      margin-bottom: 15px;
      border: 1px solid #ccc;
      border-radius: 8px;
    }

    button {
      padding: 10px 18px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
      transition: 0.2s;
    }

    .primary-btn {
      background: #2563eb;
      color: white;
    }

    .accept-btn {
      background: #16a34a;
      color: white;
      margin-top: 10px;
    }

    .complete-btn {
      background: #7c3aed;
      color: white;
      margin-top: 10px;
    }

    .danger-btn {
      background: #dc2626;
      color: white;
      margin-top: 10px;
    }

    button:hover {
      opacity: 0.9;
    }

    .request-card {
      border: 1px solid #ddd;
      border-radius: 12px;
      padding: 15px;
      margin-bottom: 15px;
      background: #fafafa;
    }

    .status {
      display: inline-block;
      padding: 5px 10px;
      border-radius: 20px;
      font-size: 13px;
      margin-top: 8px;
      font-weight: bold;
    }

    .pending {
      background: #facc15;
      color: #000;
    }

    .accepted {
      background: #60a5fa;
      color: white;
    }

    .completed {
      background: #22c55e;
      color: white;
    }

    .fraud {
      background: #ef4444;
      color: white;
    }

    img {
      width: 100%;
      max-height: 180px;
      object-fit: cover;
      border-radius: 10px;
      margin-top: 10px;
    }

    .rating {
      color: orange;
      font-size: 18px;
      margin-top: 8px;
    }

    @media(max-width:900px){
      .container{
        grid-template-columns: 1fr;
      }
    }
  </style>
</head>
<body>

<header>
  FixLink - AI Based Repair Service Platform
</header>

<div class="container">

  <!-- CUSTOMER FORM -->
  <div class="card">
    <h2>Raise Repair Request</h2>

    <label>Customer Name</label>
    <input type="text" id="customerName" placeholder="Enter your name">

    <label>Service Type</label>
    <select id="serviceType">
      <option>Electrician</option>
      <option>Plumber</option>
      <option>Technician</option>
      <option>Carpenter</option>
      <option>AC Repair</option>
      <option>Cleaning</option>
    </select>

    <label>Describe Issue</label>
    <textarea id="issue" rows="5" placeholder="Describe your issue..."></textarea>

    <label>Upload Issue Photo</label>
    <input type="file" id="photoInput" accept="image/*">

    <label>Location</label>
    <input type="text" id="location" placeholder="Enter locality">

    <button class="primary-btn" onclick="addRequest()">Submit Request</button>
  </div>

  <!-- TECHNICIAN DASHBOARD -->
  <div class="card">
    <h2>Technician Dashboard</h2>
    <button class="primary-btn" style="margin-bottom:15px;" onclick="toggleHistory()">View History</button>
    <div id="historyContainer" style="display:none; margin-bottom:20px;"></div>
    <div id="requestsContainer"></div>
  </div>

</div>

<script>

let requests = JSON.parse(localStorage.getItem("requests")) || [];
let historyRequests = JSON.parse(localStorage.getItem("historyRequests")) || [];

function saveRequests(){
  localStorage.setItem("requests", JSON.stringify(requests));
  localStorage.setItem("historyRequests", JSON.stringify(historyRequests));
}

function addRequest(){

  const customerName = document.getElementById("customerName").value;
  const serviceType = document.getElementById("serviceType").value;
  const issue = document.getElementById("issue").value;
  const location = document.getElementById("location").value;
  const photoInput = document.getElementById("photoInput");

  if(customerName === "" || issue === "" || location === ""){
    alert("Please fill all fields");
    return;
  }

  let imageURL = "";

  if(photoInput.files[0]){
    imageURL = URL.createObjectURL(photoInput.files[0]);
  }

  const request = {
    id: Date.now(),
    customerName,
    serviceType,
    issue,
    location,
    imageURL,
    status: "Pending",
    rating: null,
    fraudReported: false
  };

  requests.unshift(request);
  saveRequests();
  renderRequests();

  document.getElementById("customerName").value = "";
  document.getElementById("issue").value = "";
  document.getElementById("location").value = "";
  document.getElementById("photoInput").value = "";

  alert("Repair Request Submitted Successfully");
}

function renderRequests(){

  // Keep completed requests temporarily visible for rating submission

  const container = document.getElementById("requestsContainer");

  if(requests.length === 0){
    container.innerHTML = "<p>No repair requests available.</p>";
    return;
  }

  container.innerHTML = "";

  requests.forEach(request => {

    const card = document.createElement("div");
    card.className = "request-card";

    let statusClass = "pending";

    if(request.status === "Accepted"){
      statusClass = "accepted";
    }
    else if(request.status === "Completed"){
      statusClass = "completed";
    }

    if(request.fraudReported){
      statusClass = "fraud";
    }

    card.innerHTML = `
      <h3>${request.serviceType}</h3>
      <p><b>Customer:</b> ${request.customerName}</p>
      <p><b>Issue:</b> ${request.issue}</p>
      <p><b>Location:</b> ${request.location}</p>
      <span class="status ${statusClass}">${request.fraudReported ? 'Fraud Reported' : request.status}</span>
      ${request.imageURL ? `<img src="${request.imageURL}">` : ""}

      ${request.status === 'Pending' ?
      `<button class="accept-btn" onclick="acceptRequest(${request.id})">Accept Job</button>` : ""}

      ${request.status === 'Accepted' ?
      `<button class="complete-btn" onclick="completeRequest(${request.id})">Mark Completed</button>` : ""}

      ${request.status === 'Completed' && request.rating === null ?
      `
      <div style="margin-top:12px;">
        <label>Customer Rating</label>
        <select id="rating-${request.id}">
          <option value="">Select Rating</option>
          <option value="1">1</option>
          <option value="2">2</option>
          <option value="3">3</option>
          <option value="4">4</option>
          <option value="5">5</option>
        </select>

        <button class="complete-btn" onclick="submitRating(${request.id})">
          Submit Rating
        </button>

        <button class="danger-btn" onclick="reportFraud(${request.id})">
          Report Suspicious Behaviour
        </button>
      </div>
      ` : ""}

      ${request.rating ?
      `<div class="rating">⭐ Rating: ${request.rating}/5</div>` : ""}
    `;

    container.appendChild(card);
  });
}

function acceptRequest(id){

  requests = requests.map(request => {
    if(request.id === id){
      request.status = "Accepted";
    }
    return request;
  });

  saveRequests();
  renderRequests();
}

function completeRequest(id){

  requests = requests.map(request => {
    if(request.id === id){
      request.status = "Completed";
    }
    return request;
  });

  saveRequests();
  renderRequests();
}

function finalizeCompletedRequest(id){

  const completedRequest = requests.find(r => r.id === id);

  if(completedRequest){
    historyRequests.unshift({...completedRequest});
  }

  requests = requests.filter(request => request.id !== id);

  saveRequests();
  renderRequests();
}

function toggleHistory(){

  const historyContainer = document.getElementById("historyContainer");

  if(historyContainer.style.display === "none"){

    historyContainer.style.display = "block";

    if(historyRequests.length === 0){
      historyContainer.innerHTML = "<p>No completed history available.</p>";
      return;
    }

    let html = "<h3>Completed Service History</h3>";

    historyRequests.forEach(request => {
      html += `
      <div class="request-card">
        <h4>${request.serviceType}</h4>
        <p><b>Customer:</b> ${request.customerName}</p>
        <p><b>Issue:</b> ${request.issue}</p>
        <p><b>Location:</b> ${request.location}</p>
        <span class="status completed">Completed</span>
        ${request.rating ? `<div class="rating">⭐ Rating: ${request.rating}/5</div>` : ""}
      </div>
      `;
    });

    historyContainer.innerHTML = html;

  }else{
    historyContainer.style.display = "none";
  }
}

function submitRating(id){

  const ratingValue = document.getElementById(`rating-${id}`).value;

  if(ratingValue === ""){
    alert("Please select rating");
    return;
  }

  requests = requests.map(request => {
    if(request.id === id){
      request.rating = ratingValue;
    }
    return request;
  });

  const completedRequest = requests.find(r => r.id === id);

  if(completedRequest){
    historyRequests.unshift({...completedRequest});
  }

  requests = requests.filter(request => request.id !== id);

  saveRequests();
  renderRequests();

  alert("Rating Submitted Successfully");
}

function reportFraud(id){

  requests = requests.map(request => {
    if(request.id === id){
      request.fraudReported = true;
    }
    return request;
  });

  saveRequests();
  renderRequests();

  alert("Complaint Registered Against Technician");
}

renderRequests();

</script>

</body>
</html>
