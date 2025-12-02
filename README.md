<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Japan Explorer PRO – By Muchan_official</title>

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
/* ==========================
   GLOBAL STYLES
========================== */
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f4f4f4;
    transition: background 0.3s, color 0.3s;
}
.dark body, .dark {
    background: #111;
    color: #eee;
}

header {
    background: #222;
    color: white;
    padding: 16px;
    text-align: center;
    font-size: 22px;
    font-weight: bold;
    position: sticky;
    top:0;
    z-index: 5000;
}
.dark header { background:#000; }

#map {
    height: 60vh;
    width: 100%;
}

/* ==========================
   UI + Cards
========================== */
.container {
    padding: 15px;
}

input, select {
    width: 100%;
    padding: 10px;
    border-radius: 8px;
    border: 1px solid #ccc;
    margin-bottom: 14px;
    font-size: 16px;
}
.dark input, .dark select {
    background:#222;
    color:white;
    border:1px solid #444;
}

.place-card {
    background: white;
    border-radius: 12px;
    padding: 15px;
    margin-bottom: 14px;
    box-shadow: 0 3px 8px rgba(0,0,0,0.12);
}
.dark .place-card {
    background:#1a1a1a;
    box-shadow:none;
}

.place-title {
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 6px;
}

.category {
    color: #777;
    margin-bottom: 8px;
}
.dark .category {
    color:#aaa;
}

.btn {
    display: inline-block;
    margin-top: 8px;
    padding: 8px 12px;
    background: #0078ff;
    color: white;
    border-radius: 6px;
    text-decoration: none;
    font-size: 14px;
}
.btn-green { background:#00c851; }

.dark .btn { background:#005fcc; }
.dark .btn-green { background:#009944; }

/* ==========================
   AD BOX
========================== */
.ad-box {
    background: #fff4d6;
    border: 1px solid #ffd17a;
    padding: 15px;
    border-radius: 8px;
    margin: 15px 0;
    text-align: center;
    font-weight: bold;
}
.dark .ad-box {
    background:#332b00;
    border-color:#665500;
}

/* ==========================
   FOOTER
========================== */
footer {
    text-align: center;
    padding: 25px;
    color: #555;
    font-size: 14px;
}
.dark footer { color:#999; }

/* ==========================
   DARK MODE TOGGLE
========================== */
.dark-toggle {
    position: fixed;
    right: 15px;
    top: 90px;
    background:#222;
    color:white;
    padding:10px 12px;
    border-radius:50%;
    font-size:18px;
    cursor:pointer;
    z-index:3000;
}
.dark .dark-toggle {
    background:#444;
}
</style>
</head>
<body>

<header>
Japan Explorer PRO – <span style="color:#00c3ff">Muchan_official</span>
</header>

<div id="map"></div>

<!-- Dark Mode Button -->
<div class="dark-toggle" onclick="toggleDarkMode()">🌓</div>

<div class="container">

    <input id="searchBox" placeholder="Search places… (Tokyo, Pokémon, TeamLab etc.)"/>
    
    <select id="categoryFilter">
        <option value="all">Filter by Category</option>
        <option value="pokemon">Pokémon Centers</option>
        <option value="cards">Card Shops / Booster Boxes</option>
        <option value="food">Food</option>
        <option value="local">Local Spots</option>
        <option value="matcha">Matcha</option>
        <option value="mochi">Mochi Workshops</option>
        <option value="fun">Fun Activities</option>
        <option value="attraction">Attractions</option>
        <option value="culture">Culture</option>
        <option value="anime">Anime & Otaku Spots</option>
    </select>

    <!-- Admin Panel Trigger -->
    <button onclick="openAdmin()" style="width:100%; padding:12px; font-size:16px; border-radius:10px; background:#444; color:white;">
        Add New Place (Admin)
    </button>

    <div id="placeList"></div>
</div>

<footer>Made with ❤️ by Muchan_official</footer>

<!-- Leaflet -->
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
/* ============================================================
   MAP INITIALIZATION
============================================================ */
const map = L.map('map').setView([35.6762, 139.6503], 5);

L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 19,
}).addTo(map);

/* ============================================================
   DARK MODE SYSTEM
============================================================ */
function toggleDarkMode() {
    document.body.classList.toggle("dark");
}
/* ============================================================
   PART 2/3 — PLACE DATABASE (Pokémon, TeamLab, Food, Activities...)
   Add or edit places here. Each place object fields:
   id, name, lat, lng, city, prefecture, category, description,
   tags (array), image (optional), affiliate (optional), pokemon_center (bool), featured (bool)
============================================================ */

const PLACES = [
  /* ----------------------
     Pokémon Centers / Major Stores
     ---------------------- */
  { id:"poke_tokyo_dx", name:"Pokémon Center TOKYO DX & Pokémon Café", lat:35.6695, lng:139.7626, city:"Tokyo", prefecture:"Tokyo", category:"pokemon", description:"Large official Pokémon Center with exclusive goods and café. Popular for collectors and booster box hunters.", tags:["pokemon","shop","tokyo","booster"], image:"", affiliate:"", pokemon_center:true, featured:true },
  { id:"poke_shibuya", name:"Pokémon Center Shibuya", lat:35.6595, lng:139.7005, city:"Shibuya", prefecture:"Tokyo", category:"pokemon", description:"Pokémon Center located in Shibuya with seasonal items and events.", tags:["pokemon","shibuya"], image:"", affiliate:"", pokemon_center:true },
  { id:"poke_osaka", name:"Pokémon Center OSAKA", lat:34.7055, lng:135.4983, city:"Osaka", prefecture:"Osaka", category:"pokemon", description:"Popular Osaka location for official merch and occasional booster restocks.", tags:["pokemon","osaka"], image:"", affiliate:"", pokemon_center:true },
  { id:"poke_nagoya", name:"Pokémon Center NAGOYA", lat:35.1709, lng:136.8815, city:"Nagoya", prefecture:"Aichi", category:"pokemon", description:"Official Nagoya store with a steady collector crowd.", tags:["pokemon","nagoya"], image:"", affiliate:"", pokemon_center:true },
  { id:"poke_yokohama", name:"Pokémon Center Yokohama", lat:35.4660, lng:139.6227, city:"Yokohama", prefecture:"Kanagawa", category:"pokemon", description:"Pokémon Center in Yokohama with exclusive regional goods.", tags:["pokemon","yokohama"], image:"", affiliate:"", pokemon_center:true },
  { id:"poke_fukuoka", name:"Pokémon Center Fukuoka", lat:33.5904, lng:130.4017, city:"Fukuoka", prefecture:"Fukuoka", category:"pokemon", description:"Southern Japan Pokémon Center serving Kyushu collectors.", tags:["pokemon","fukuoka"], image:"", affiliate:"", pokemon_center:true },

  /* ----------------------
     Card & Booster Shops
     ---------------------- */
  { id:"akihabara_cards", name:"Akihabara Card Shops", lat:35.6984, lng:139.7730, city:"Akihabara", prefecture:"Tokyo", category:"cards", description:"Row of card shops in Akihabara offering singles, packs and booster boxes.", tags:["cards","akihabara","booster","tcg"], image:"", affiliate:"" },
  { id:"nakano_broadway", name:"Nakano Broadway Card Stores", lat:35.7074, lng:139.6646, city:"Nakano", prefecture:"Tokyo", category:"cards", description:"Multiple stores for vintage and new cards; can find rare boosters.", tags:["cards","nakano","collectibles"], image:"", affiliate:"" },
  { id:"nipponbashi_osaka", name:"Nipponbashi Card Street", lat:34.6669, lng:135.5012, city:"Nipponbashi", prefecture:"Osaka", category:"cards", description:"Osaka's famous area for hobby & card shops.", tags:["cards","osaka","nipponbashi"], image:"", affiliate:"" },

  /* ----------------------
     TeamLab + Digital Art
     ---------------------- */
  { id:"teamlab_planets", name:"teamLab Planets TOKYO", lat:35.6261, lng:139.7817, city:"Toyosu (Tokyo)", prefecture:"Tokyo", category:"fun", description:"Walk-through water-based immersive installations — very popular for visually stunning content.", tags:["teamlab","planets","experience","art"], image:"", affiliate:"", featured:true },
  { id:"teamlab_borderless", name:"teamLab Borderless (Mori Building Digital Art Museum)", lat:35.6269, lng:139.7769, city:"Odaiba (Tokyo)", prefecture:"Tokyo", category:"fun", description:"Large-scale interactive digital art exhibits. Iconic and highly shareable on social media.", tags:["teamlab","borderless","digital","odaiba"], image:"", affiliate:"", featured:true },

  /* ----------------------
     Theme Parks & Large Attractions
     ---------------------- */
  { id:"universal_osaka", name:"Universal Studios Japan", lat:34.6654, lng:135.4300, city:"Osaka", prefecture:"Osaka", category:"attraction", description:"Major theme park with seasonal events and big-name IP attractions.", tags:["universal","themepark","osaka"], image:"", affiliate:"" },
  { id:"tokyo_disneyland", name:"Tokyo Disneyland & DisneySea", lat:35.6329, lng:139.8805, city:"Urayasu", prefecture:"Chiba", category:"attraction", description:"Two parks: Disneyland and DisneySea. Booking in advance recommended.", tags:["disney","themepark","chiba"], image:"", affiliate:"" },

  /* ----------------------
     Museums, Culture, Experiences
     ---------------------- */
  { id:"ghibli_museum", name:"Ghibli Museum (Mitaka)", lat:35.6961, lng:139.5753, city:"Mitaka", prefecture:"Tokyo", category:"culture", description:"Studio Ghibli museum — ticketed entry and magical exhibits.", tags:["ghibli","museum","mitaka"], image:"", affiliate:"" },
  { id:"mori_museum", name:"Mori Art Museum / Roppongi Hills", lat:35.6586, lng:139.7454, city:"Roppongi", prefecture:"Tokyo", category:"culture", description:"Contemporary art museum with panoramic city views.", tags:["art","mori","roppongi"], image:"", affiliate:"" },
  { id:"team_lab_kyoto", name:"teamLab Kyoto (select exhibitions)", lat:35.0116, lng:135.7681, city:"Kyoto", prefecture:"Kyoto", category:"fun", description:"teamLab has rotating exhibits in Kyoto — check current schedule.", tags:["teamlab","kyoto","art"], image:"", affiliate:"" },

  /* ----------------------
     Food (big cities + local)
     ---------------------- */
  { id:"ichiran_shibuya", name:"Ichiran Ramen (Shibuya)", lat:35.6590, lng:139.6993, city:"Shibuya", prefecture:"Tokyo", category:"food", description:"Famous solo-booth ramen — great for video reactions.", tags:["ramen","shibuya","food"], image:"", affiliate:"" },
  { id:"sushi_counter_tokyo", name:"Small Omakase Sushi — Tokyo (local)", lat:35.6655, lng:139.7301, city:"Minato", prefecture:"Tokyo", category:"food", description:"Neighborhood sushi counter with intimate omakase—great short-form content.", tags:["sushi","omakase","tokyo"], image:"", affiliate:"" },
  { id:"dotonbori", name:"Dotonbori Food Street", lat:34.6677, lng:135.5012, city:"Osaka", prefecture:"Osaka", category:"food", description:"Bustling food street with takoyaki, okonomiyaki, and street eats.", tags:["osaka","streetfood","dotonbori"], image:"", affiliate:"" },

  /* ----------------------
     Mochi Workshops & Matcha ceremonies
     ---------------------- */
  { id:"nara_mochi", name:"DIY Mochi Workshop — Nara", lat:34.6851, lng:135.8050, city:"Nara", prefecture:"Nara", category:"mochi", description:"Hands-on mochi pounding & shaping workshop for visitors.", tags:["mochi","workshop","nara"], image:"", affiliate:"" },
  { id:"kyoto_matcha_ceremony", name:"Matcha Ceremony — Gion Kyoto", lat:35.0037, lng:135.7788, city:"Gion", prefecture:"Kyoto", category:"matcha", description:"Traditional tea ceremony experience in Kyoto's Gion district.", tags:["matcha","tea","kyoto"], image:"", affiliate:"" },

  /* ----------------------
     Anime / Otaku / Arcades
     ---------------------- */
  { id:"akihabara_arcades", name:"Akihabara Arcades & Shops", lat:35.6984, lng:139.7730, city:"Akihabara", prefecture:"Tokyo", category:"anime", description:"Arcades, shops, gachapon, and anime goods.", tags:["anime","akihabara","arcade"], image:"", affiliate:"" },
  { id:"nakano_broadway_otaku", name:"Nakano Broadway (Otaku Paradise)", lat:35.7074, lng:139.6646, city:"Nakano", prefecture:"Tokyo", category:"anime", description:"Collectibles, vintage anime goods, and specialty shops.", tags:["anime","nakano","collectibles"], image:"", affiliate:"" },

  /* ----------------------
     Nature / Onsen / Sightseeing
     ---------------------- */
  { id:"onsen_hakone", name:"Hakone Onsen Area", lat:35.2428, lng:139.1063, city:"Hakone", prefecture:"Kanagawa", category:"local", description:"Hot springs, ryokan stays and scenic views of Mt. Fuji on clear days.", tags:["onsen","hakone","nature"], image:"", affiliate:"" },
  { id:"itsukushima", name:"Itsukushima / Miyajima Torii", lat:34.2950, lng:132.3190, city:"Miyajima", prefecture:"Hiroshima", category:"sightseeing", description:"Iconic floating torii gate and local sweets like momiji manju.", tags:["island","sightseeing","hiroshima"], image:"", affiliate:"" },

  /* ----------------------
     Events / seasonal / featured
     ---------------------- */
  { id:"sapporo_snowfest", name:"Sapporo Snow Festival (seasonal)", lat:43.0642, lng:141.3469, city:"Sapporo", prefecture:"Hokkaido", category:"attraction", description:"Winter festival with many ice sculptures — seasonal; check dates.", tags:["sapporo","festival","seasonal"], image:"", affiliate:"" },

  /* ----------------------
     Local hidden gems
     ---------------------- */
  { id:"kanazawa_teahouse", name:"Hidden Tea House — Kanazawa", lat:36.5613, lng:136.6562, city:"Kanazawa", prefecture:"Ishikawa", category:"local", description:"Small tea shop with matcha & wagashi — great local vibe.", tags:["tea","local","kanazawa"], image:"", affiliate:"" },

  /* ----------------------
     Example featured / partner
     ---------------------- */
  { id:"featured_booster_shop", name:"Featured Booster Box Shop (Tokyo Partner)", lat:35.7070, lng:139.7745, city:"Tokyo", prefecture:"Tokyo", category:"cards", description:"Partner shop with verified stock updates — affiliate links available (set in admin).", tags:["cards","booster","partner"], image:"", affiliate:"", featured:true }
];

/* ============================================================
   PART 2 — build markers from PLACES
============================================================ */

let placeMarkers = []; // store marker refs

function clearPlaceMarkers(){
  placeMarkers.forEach(m => markerGroup.removeLayer(m));
  placeMarkers = [];
}

function addPlaceMarkers(list){
  // list: array of place objects
  list.forEach(p=>{
    try {
      if (!p || typeof p !== 'object') return;
      const lat = Number(p.lat), lng = Number(p.lng);
      if (!validNumber(lat) || !validNumber(lng)) return;
      const marker = L.marker([lat, lng], { title: p.name || '' });
      const popupHtml = `
        <div style="max-width:320px">
          ${p.image ? `<img src="${escapeHtml(p.image)}" style="width:100%;height:120px;object-fit:cover;border-radius:8px;margin-bottom:8px" onerror="this.style.display='none'">` : ""}
          <h3 style="margin:0 0 6px">${escapeHtml(p.name)}</h3>
          <div style="color:#6b7280;font-size:13px">${escapeHtml(p.city || '')} · ${escapeHtml(p.prefecture || '')}</div>
          <p style="margin-top:8px;color:#374151">${escapeHtml(p.description || '')}</p>
          <div style="display:flex;gap:8px;margin-top:8px;flex-wrap:wrap">
            <a href="https://www.google.com/maps/dir/?api=1&destination=${lat},${lng}" target="_blank" style="padding:8px 10px;border-radius:8px;background:white;border:1px solid #e5e7eb;text-decoration:none">Directions</a>
            <button class="shareBtn" data-id="${escapeHtml(p.id)}" style="padding:8px 10px;border-radius:8px;border:1px solid #e5e7eb;background:white;cursor:pointer">Share</button>
            ${p.affiliate ? `<a href="${escapeHtml(p.affiliate)}" target="_blank" style="padding:8px 10px;border-radius:8px;background:#00c851;color:white;text-decoration:none">Buy / Affiliate</a>` : ''}
          </div>
        </div>
      `;
      marker.bindPopup(popupHtml);
      marker.options.placeId = p.id;
      markerGroup.addLayer(marker);
      placeMarkers.push(marker);
    } catch(err){
      console.warn('Failed to add marker for', p && p.id, err);
    }
  });
}

// Initially add all places
addPlaceMarkers(PLACES);
places = PLACES.slice(); // global places state continues in PART 3 (for filters/admin)
applyFilters && applyFilters(); // if applyFilters defined later, it will run
/* ============================================================
   PART 3/3 — Filters, persistence, admin, CSV, sharing, final wiring
============================================================ */

/* ------------------------------
   Persistence: load/save from localStorage
   ------------------------------ */
const STORAGE_KEY = 'jp_explorer_pro_places_v1';
function saveToLocal(){
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(places));
  } catch(e){
    console.warn('Save failed', e);
  }
}
function loadFromLocal(){
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return null;
    const parsed = JSON.parse(raw);
    if (Array.isArray(parsed)) return parsed;
    return null;
  } catch(e){ return null; }
}

/* Replace default places with saved if present */
const saved = loadFromLocal();
if (saved && Array.isArray(saved) && saved.length) {
  places = saved;
} else {
  // places already set to PLACES in PART 2; keep as-is and save initial copy
  saveToLocal();
}

/* ------------------------------
   Marker group (cluster) and reinitialize
   ------------------------------ */
const markerGroup = L.markerClusterGroup({chunkedLoading:true});
map.addLayer(markerGroup);

// Clear any markers that may have been added earlier in PART 2
clearPlaceMarkers();
addPlaceMarkers(places);

/* ------------------------------
   Filters & List rendering
   ------------------------------ */
function populateFiltersUI(){
  // Build category options dynamically (in case admin added)
  const cats = Array.from(new Set(places.map(p=>p.category||'local'))).sort();
  const catSel = document.getElementById('categoryFilter');
  // preserve selected
  const cur = catSel.value || 'all';
  catSel.innerHTML = '<option value="all">Filter by Category</option>' + cats.map(c=>`<option value="${escapeHtml(c)}">${escapeHtml(c)}</option>`).join('');
  if (cats.includes(cur)) catSel.value = cur;
}
populateFiltersUI();

function renderPlaceList(list){
  const container = document.getElementById('placeList');
  container.innerHTML = '';
  if (!list || !list.length){
    container.innerHTML = '<div style="color:#777;padding:12px">No results. Try clearing filters.</div>';
    return;
  }
  list.forEach((p, idx)=>{
    const el = document.createElement('div');
    el.className = 'place-card';
    el.innerHTML = `
      <div class="place-title">${escapeHtml(p.name)}</div>
      <div class="category">${escapeHtml(p.city || '')} · ${escapeHtml(p.prefecture || '')} · ${escapeHtml(p.category || '')}</div>
      <div style="margin-top:8px">${escapeHtml(p.description || '')}</div>
    `;
    const actions = document.createElement('div');
    actions.style.marginTop = '10px';
    // View on map
    const viewBtn = document.createElement('button'); viewBtn.className = 'btn'; viewBtn.textContent = 'View on map';
    viewBtn.onclick = ()=> { map.setView([p.lat, p.lng], 15); setTimeout(()=> markerGroup.eachLayer(l => { if (l.options.placeId === p.id) l.openPopup(); }), 200); };
    // Directions
    const dir = document.createElement('a'); dir.className = 'btn'; dir.textContent = 'Directions'; dir.href = `https://www.google.com/maps/dir/?api=1&destination=${p.lat},${p.lng}`; dir.target = '_blank';
    // Share
    const share = document.createElement('button'); share.className = 'btn'; share.style.marginLeft='6px'; share.textContent = 'Share';
    share.onclick = ()=> copyShareCaption(p.id);
    actions.appendChild(viewBtn); actions.appendChild(dir); actions.appendChild(share);
    // Affiliate if exists
    if (p.affiliate){
      const aff = document.createElement('a'); aff.className = 'btn btn-green'; aff.textContent = 'Buy / Affiliate'; aff.href = p.affiliate; aff.target='_blank'; aff.style.marginLeft='6px';
      actions.appendChild(aff);
    }
    el.appendChild(actions);
    container.appendChild(el);

    // ad insertion
    if ((idx+1) % 12 === 0){
      const ad = document.createElement('div'); ad.className='ad-box'; ad.textContent='AD SLOT — Advertise here via Muchan_official';
      container.appendChild(ad);
    }
  });
}

function getFilteredPlaces(){
  const q = (document.getElementById('searchBox').value || '').toLowerCase().trim();
  const cat = document.getElementById('categoryFilter').value || 'all';
  return places.filter(p=>{
    if (cat !== 'all' && p.category !== cat) return false;
    if (!q) return true;
    const hay = (p.name + ' ' + (p.city||'') + ' ' + (p.prefecture||'') + ' ' + (p.tags||[]).join(' ') + ' ' + (p.description||'')).toLowerCase();
    return hay.includes(q);
  });
}

function applyFilters(){
  const filtered = getFilteredPlaces();
  // rebuild map markers & list
  clearPlaceMarkers();
  addPlaceMarkers(filtered);
  renderPlaceList(filtered);
}

/* initial render */
applyFilters();

/* events */
document.getElementById('searchBox').addEventListener('input', ()=> applyFilters());
document.getElementById('categoryFilter').addEventListener('change', ()=> applyFilters());

/* ------------------------------
   Sharing & deep links
   ------------------------------ */
function copyShareCaption(id){
  const base = window.location.href.split('#')[0];
  const url = base + '#place=' + encodeURIComponent(id) + '&ref=muchan_official';
  const place = places.find(p=>p.id === id);
  const title = place ? place.name : 'Place';
  const caption = `${title}\nFound on ${TIKTOK_HANDLE}\nSee location: ${url}\n#Japan #Travel #${TIKTOK_HANDLE.replace('@','')}`;
  if (navigator.clipboard){
    navigator.clipboard.writeText(caption).then(()=> alert('Caption copied! Paste into TikTok description.'), ()=> prompt('Copy this caption:', caption));
  } else {
    prompt('Copy this caption:', caption);
  }
}

/* deep link on load */
function handleHash(){
  const h = location.hash.slice(1);
  if (!h) return;
  const params = new URLSearchParams(h.replace(/\?/,''));
  const pid = params.get('place');
  if (pid){
    const p = places.find(x=>x.id === pid);
    if (p && validNumber(Number(p.lat)) && validNumber(Number(p.lng))){
      map.setView([p.lat, p.lng], 15);
      setTimeout(()=> markerGroup.eachLayer(l => { if (l.options.placeId === pid) l.openPopup(); }), 400);
    } else {
      console.warn('Place not found for hash', pid);
    }
  }
}
handleHash();

/* ------------------------------
   Admin (simple local-password + JSON editor & CSV export/import)
   ------------------------------ */
function openAdmin(){
  let pass = localStorage.getItem('jp_admin_pass_v1');
  if (!pass){
    const newPass = prompt('No admin password set. Create an admin password (saved locally in your browser).');
    if (!newPass) return alert('Admin setup cancelled.');
    localStorage.setItem('jp_admin_pass_v1', btoa(newPass));
    return alert('Password saved locally. Press Admin again to open admin tools.');
  }
  const entered = prompt('Enter admin password');
  if (!entered) return;
  if (btoa(entered) !== localStorage.getItem('jp_admin_pass_v1')) return alert('Incorrect password.');
  // show admin menu
  const action = prompt('Admin menu: type one of:\n"add" - add new place\n"edit" - edit full JSON\n"import" - paste JSON array to append\n"csv" - export CSV\n"reset" - reset to sample (danger)\n"export" - download JSON\n"delete" - delete a place by id\n(cancel to exit)');
  if (!action) return;
  if (action === 'add'){
    const json = prompt('Paste single place JSON object (example: {"name":"X","lat":35,"lng":139,"city":"Tokyo","prefecture":"Tokyo","category":"food","description":"...","id":"optional_id"})');
    if (!json) return;
    try {
      const obj = JSON.parse(json);
      if (!obj.id) obj.id = uid();
      if (!validNumber(Number(obj.lat)) || !validNumber(Number(obj.lng))) return alert('Invalid lat/lng');
      places.unshift(obj);
      saveToLocal(); populateFiltersUI(); applyFilters();
      alert('Added: ' + obj.name);
    } catch(e){ alert('Invalid JSON: ' + e.message); }
    return;
  }
  if (action === 'edit'){
    const edited = prompt('Edit full JSON array for all places. Be careful!', JSON.stringify(places, null, 2));
    if (!edited) return;
    try {
      const arr = JSON.parse(edited);
      if (!Array.isArray(arr)) return alert('Must be JSON array.');
      // minimal validation
      for (let i=0;i<arr.length;i++){
        const it = arr[i];
        if (!it.id) it.id = uid();
        if (!validNumber(Number(it.lat)) || !validNumber(Number(it.lng))) return alert('Invalid coords at index ' + i);
      }
      places = arr;
      saveToLocal(); populateFiltersUI(); applyFilters();
      alert('Saved changes.');
    } catch(e){ alert('Invalid JSON: ' + e.message); }
    return;
  }
  if (action === 'import'){
    const pasted = prompt('Paste a JSON array to append to current places');
    if (!pasted) return;
    try {
      const arr = JSON.parse(pasted);
      if (!Array.isArray(arr)) return alert('Must be an array.');
      let added = 0;
      arr.forEach(it=>{
        if (!it.id) it.id = uid();
        if (!validNumber(Number(it.lat)) || !validNumber(Number(it.lng))) return;
        places.push(it); added++;
      });
      if (added) { saveToLocal(); populateFiltersUI(); applyFilters(); alert('Imported ' + added + ' places'); }
      else alert('No valid places found in import.');
    } catch(e){ alert('Invalid JSON: ' + e.message); }
    return;
  }
  if (action === 'csv'){
    // export CSV
    const hdrs = ['id','name','lat','lng','city','prefecture','category','description','tags','affiliate','pokemon_center','featured','image','url'];
    const rows = places.map(p=> hdrs.map(h=>{
      const v = p[h];
      if (Array.isArray(v)) return '"'+v.join('|')+'"';
      return '"'+String(v||'').replace(/"/g,'""')+'"';
    }).join(','));
    const csv = hdrs.join(',') + '\n' + rows.join('\n');
    download('places_export.csv', csv);
    return;
  }
  if (action === 'reset'){
    if (!confirm('Reset to built-in sample places? This will overwrite your current list.')) return;
    places = PLACES.slice();
    saveToLocal(); populateFiltersUI(); applyFilters();
    alert('Reset to sample data.');
    return;
  }
  if (action === 'export'){
    download('places_export.json', JSON.stringify(places, null, 2));
    return;
  }
  if (action === 'delete'){
    const id = prompt('Enter id of place to delete (use copy of id from list)');
    if (!id) return;
    const idx = places.findIndex(x=>x.id === id);
    if (idx === -1) return alert('Id not found.');
    if (!confirm('Delete "' + places[idx].name + '"?')) return;
    places.splice(idx,1);
    saveToLocal(); populateFiltersUI(); applyFilters();
    alert('Deleted.');
    return;
  }
  alert('Unknown action or cancelled.');
}

/* ------------------------------
   Utility: CSV download helper (already used)
   ------------------------------ */
function download(filename, text){
  const blob = new Blob([text], {type:'text/plain;charset=utf-8'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = filename; a.click(); URL.revokeObjectURL(url);
}

/* ------------------------------
   Periodic auto-save and final touches
   ------------------------------ */
setInterval(()=> saveToLocal(), 60_000); // every minute

// Ensure UI populated
populateFiltersUI();
applyFilters();

// Expose debug functions
window.JP_EXPLORER = {
  getPlaces: ()=> JSON.parse(JSON.stringify(places)),
  setPlaces: (arr)=> { if (!Array.isArray(arr)) throw new Error('Array required'); places = arr; saveToLocal(); populateFiltersUI(); applyFilters(); }
};

console.log('Japan Explorer PRO initialized. Muchan_official');