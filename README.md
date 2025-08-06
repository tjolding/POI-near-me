# POI-near-me
Point of interest near me
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Nearby Wikipedia POIs</title>
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet/dist/leaflet.css"
  />
  <style>
    #map {
      height: 100vh;
      width: 100%;
    }
    .control {
      position: absolute;
      top: 10px;
      left: 10px;
      background: white;
      padding: 8px;
      z-index: 1000;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div class="control">
    <label for="radius">Radius:</label>
    <select id="radius">
      <option value="1">1 mile</option>
      <option value="5">5 miles</option>
      <option value="20" selected>20 miles</option>
    </select>
    <button onclick="getLocation()">Refresh</button>
  </div>
  <div id="map"></div>

  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script>
    let map;

    function initMap(lat, lon) {
      if (map) {
        map.setView([lat, lon], 12);
        return;
      }

      map = L.map('map').setView([lat, lon], 12);

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors'
      }).addTo(map);

      L.marker([lat, lon]).addTo(map)
        .bindPopup("You are here")
        .openPopup();
    }

    function getLocation() {
      if (!navigator.geolocation) {
        alert("Geolocation is not supported by your browser");
        return;
      }

      navigator.geolocation.getCurrentPosition(
        position => {
          const lat = position.coords.latitude;
          const lon = position.coords.longitude;
          initMap(lat, lon);
          const miles = parseFloat(document.getElementById('radius').value);
          const radiusInKm = miles * 1.60934;
          getWikipediaPOIs(lat, lon, radiusInKm);
        },
        () => {
          alert("Unable to retrieve your location");
        }
      );
    }

    function getWikipediaPOIs(lat, lon, radiusKm) {
      const url = `https://secure.geonames.org/findNearbyWikipediaJSON?lat=${lat}&lng=${lon}&radius=${radiusKm}&maxRows=50&username=demo`;

      // Replace "demo" with your registered username from geonames.org
      fetch(url)
        .then(res => res.json())
        .then(data => {
          if (!data.geonames) {
            alert("No nearby Wikipedia entries found.");
            return;
          }

          data.geonames.forEach(place => {
            L.marker([place.lat, place.lng])
              .addTo(map)
              .bindPopup(`<strong>${place.title}</strong><br>${place.summary}<br><a href="https://${place.wikipediaUrl}" target="_blank">Read more</a>`);
          });
        })
        .catch(err => {
          console.error(err);
          alert("Error fetching Wikipedia data");
        });
    }

    // Load on page load
    window.onload = getLocation;
  </script>
</body>
</html>
