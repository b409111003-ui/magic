<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>社區散步友善地圖</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" crossorigin=""/>
  <style>
    #map { height: 100vh; width: 100%; }
    .custom-icon {
      font-size: 28px;
      text-align: center;
      line-height: 28px;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" crossorigin=""></script>
  <script src="https://unpkg.com/leaflet-routing-machine@latest/dist/leaflet-routing-machine.js"></script>

  <script>
    // 初始化地圖
    const map = L.map('map').setView([25.033964, 121.564468], 15);

    // OSM 圖磚
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap 貢獻者'
    }).addTo(map);

    // 定位使用者位置
    let userMarker;
    map.locate({ setView: true, maxZoom: 17 });

    map.on('locationfound', function(e) {
      if (userMarker) map.removeLayer(userMarker);
      userMarker = L.marker(e.latlng, {
        icon: L.divIcon({ className: 'custom-icon', html: '🧍' })
      }).addTo(map).bindPopup("你在這裡").openPopup();
    });

    map.on('locationerror', function() {
      alert("無法取得定位");
    });

    // 自訂 Icon
    function createIcon(emoji) {
      return L.divIcon({
        className: 'custom-icon',
        html: emoji,
        iconSize: [28, 28],
        iconAnchor: [14, 28]
      });
    }

    const icons = {
      park: createIcon("🌳"),
      water: createIcon("💧"),
      store: createIcon("🏪"),
      market: createIcon("🛒")
    };

    // 測試點資料
    const places = [
      { name: "崇德綠地", type: "park", coords: [25.0333, 121.5652] },
      { name: "社區飲水機", type: "water", coords: [25.0328, 121.5639] },
      { name: "7-Eleven 崇德門市", type: "store", coords: [25.0341, 121.5665] },
      { name: "信義傳統市場", type: "market", coords: [25.0319, 121.5649] }
    ];

    places.forEach(place => {
      L.marker(place.coords, { icon: icons[place.type] })
        .addTo(map)
        .bindPopup(`${place.name}`);
    });

    // OSRM 路線規劃 (走路)
    function routeTo(dest) {
      if (!userMarker) {
        alert("尚未取得使用者位置");
        return;
      }

      L.Routing.control({
        waypoints: [
          userMarker.getLatLng(),
          L.latLng(dest[0], dest[1])
        ],
        router: L.Routing.osrmv1({
          serviceUrl: "https://router.project-osrm.org/route/v1"
        }),
        lineOptions: {
          styles: [{ color: 'blue', weight: 5 }]
        },
        createMarker: () => null
      }).addTo(map);
    }

    // 點擊地圖標記時規劃路線
    places.forEach(place => {
      L.marker(place.coords, { icon: icons[place.type] })
        .addTo(map)
        .bindPopup(`${place.name} <br><button onclick="routeTo([${place.coords}])">導航</button>`);
    });
  </script>
</body>
</html>

