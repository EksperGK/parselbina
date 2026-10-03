<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ParselCBS — Kadastro & Ulusal Koordinat Analiz Sistemi</title>

  <!-- Leaflet Harita Kütüphanesi -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <!-- Proj4js (Projeksiyon ve Koordinat Dönüşüm Motoru) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/proj4js/2.9.2/proj4.js"></script>

  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }

    body, html {
      width: 100%;
      height: 100%;
      overflow: hidden;
      background: #0f172a;
      color: #f8fafc;
    }

    /* 1. Üst Tek İnce Yatay Bar */
    #top-bar {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      height: 52px;
      background: rgba(15, 23, 42, 0.94);
      backdrop-filter: blur(12px);
      border-bottom: 1px solid rgba(255, 255, 255, 0.1);
      display: flex;
      align-items: center;
      padding: 0 16px;
      gap: 10px;
      z-index: 1000;
      box-shadow: 0 4px 20px rgba(0, 0, 0, 0.35);
    }

    .brand-tag {
      font-weight: 700;
      font-size: 14px;
      letter-spacing: 0.5px;
      color: #38bdf8;
      display: flex;
      align-items: center;
      gap: 6px;
      margin-right: 6px;
      white-space: nowrap;
    }

    .form-group {
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .form-group label {
      font-size: 11px;
      text-transform: uppercase;
      font-weight: 600;
      color: #94a3b8;
      white-space: nowrap;
    }

    select, input {
      background: #1e293b;
      border: 1px solid #334155;
      color: #f1f5f9;
      height: 32px;
      padding: 0 8px;
      border-radius: 6px;
      font-size: 12px;
      outline: none;
      transition: all 0.2s ease;
    }

    select:focus, input:focus {
      border-color: #38bdf8;
      box-shadow: 0 0 0 2px rgba(56, 189, 248, 0.2);
    }

    select {
      min-width: 110px;
      cursor: pointer;
    }

    input[type="number"] {
      width: 75px;
      -moz-appearance: textfield;
    }
    input::-webkit-outer-spin-button,
    input::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }

    button.btn-primary {
      background: linear-gradient(135deg, #0284c7 0%, #0369a1 100%);
      border: 1px solid #38bdf8;
      color: #ffffff;
      height: 32px;
      padding: 0 16px;
      border-radius: 6px;
      font-size: 12px;
      font-weight: 600;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 6px;
      transition: all 0.2s;
      white-space: nowrap;
    }

    button.btn-primary:hover {
      background: linear-gradient(135deg, #0369a1 0%, #075985 100%);
      box-shadow: 0 0 10px rgba(56, 189, 248, 0.3);
    }

    button.btn-secondary {
      background: #334155;
      border: 1px solid #475569;
      color: #e2e8f0;
      height: 32px;
      padding: 0 10px;
      border-radius: 6px;
      font-size: 11px;
      cursor: pointer;
      transition: background 0.2s;
      white-space: nowrap;
    }

    button.btn-secondary:hover {
      background: #475569;
    }

    .separator {
      width: 1px;
      height: 24px;
      background: #334155;
      margin: 0 4px;
    }

    /* Harita Alanı */
    #map {
      position: absolute;
      top: 52px;
      left: 0;
      right: 0;
      bottom: 0;
      z-index: 1;
    }

    /* Sağ Alt Bilgi Kartı (HUD / Koordinat Kartı) */
    #info-panel {
      position: absolute;
      bottom: 24px;
      right: 24px;
      width: 360px;
      max-height: calc(100vh - 100px);
      background: rgba(15, 23, 42, 0.95);
      backdrop-filter: blur(14px);
      border: 1px solid rgba(255, 255, 255, 0.15);
      border-radius: 12px;
      z-index: 1000;
      padding: 16px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
      display: none;
      flex-direction: column;
      gap: 12px;
      overflow-y: auto;
    }

    .panel-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 1px solid #334155;
      padding-bottom: 8px;
    }

    .panel-title {
      font-size: 14px;
      font-weight: 700;
      color: #38bdf8;
    }

    .badge {
      background: #0369a1;
      color: #e0f2fe;
      padding: 2px 6px;
      border-radius: 4px;
      font-size: 10px;
      font-weight: 600;
    }

    .info-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 8px;
      font-size: 11px;
    }

    .info-box {
      background: #1e293b;
      padding: 6px 8px;
      border-radius: 6px;
      border: 1px solid #334155;
    }

    .info-label {
      color: #94a3b8;
      font-size: 10px;
      display: block;
      margin-bottom: 2px;
    }

    .info-value {
      font-weight: 600;
      color: #f8fafc;
    }

    .coord-table-wrap {
      max-height: 160px;
      overflow-y: auto;
      border: 1px solid #334155;
      border-radius: 6px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 10px;
      text-align: right;
    }

    th, td {
      padding: 4px 6px;
      border-bottom: 1px solid #1e293b;
    }

    th {
      background: #1e293b;
      color: #94a3b8;
      font-weight: 600;
      position: sticky;
      top: 0;
    }

    td {
      font-family: "Courier New", Courier, monospace;
      color: #cbd5e1;
    }

    td:first-child, th:first-child {
      text-align: center;
    }

    #loading-spinner {
      display: none;
      position: absolute;
      top: 64px;
      left: 50%;
      transform: translateX(-50%);
      background: #0284c7;
      color: white;
      padding: 6px 16px;
      border-radius: 20px;
      font-size: 12px;
      font-weight: 600;
      z-index: 2000;
      box-shadow: 0 4px 12px rgba(0,0,0,0.3);
    }
  </style>
</head>
<body>

  <!-- 1. ÜST TEK İNCE YATAY SATIR -->
  <header id="top-bar">
    <div class="brand-tag">
      ParselCBS
    </div>

    <!-- İl -->
    <div class="form-group">
      <label for="select-il">İl</label>
      <select id="select-il">
        <option value="">İl Seçiniz...</option>
      </select>
    </div>

    <!-- İlçe -->
    <div class="form-group">
      <label for="select-ilce">İlçe</label>
      <select id="select-ilce" disabled>
        <option value="">Önce İl</option>
      </select>
    </div>

    <!-- Mahalle -->
    <div class="form-group">
      <label for="select-mahalle">Mahalle</label>
      <select id="select-mahalle" disabled>
        <option value="">Önce İlçe</option>
      </select>
    </div>

    <!-- Ada -->
    <div class="form-group">
      <label for="input-ada">Ada</label>
      <input type="number" id="input-ada" placeholder="Ada No" />
    </div>

    <!-- Parsel -->
    <div class="form-group">
      <label for="input-parsel">Parsel</label>
      <input type="number" id="input-parsel" placeholder="Parsel" />
    </div>

    <!-- Sorgula Butonu -->
    <button class="btn-primary" id="btn-sorgula" onclick="executeParcelQuery()">
      Sorgula
    </button>

    <div class="separator"></div>

    <!-- Pratik Hızlı Butonlar -->
    <button class="btn-secondary" onclick="loadSampleParcel()">Bursa Nilüfer Örnek</button>
    <button class="btn-secondary" onclick="promptGeoJSON()">GeoJSON Yapıştır</button>
  </header>

  <!-- Yükleniyor Uyarısı -->
  <div id="loading-spinner">TKGM Kadastro Verisi Çekiliyor & Hesaplanıyor...</div>

  <!-- 2. TAM EKRAN HARİTA -->
  <div id="map"></div>

  <!-- 3. SAĞ ALT MEKÂNSAL & ULUSAL KOORDİNAT PANELİ (HUD) -->
  <div id="info-panel">
    <div class="panel-header">
      <span class="panel-title" id="info-title">Ada / Parsel Detayı</span>
      <span class="badge" id="info-dom">ITRF96 TM 3°</span>
    </div>

    <div class="info-grid">
      <div class="info-box">
        <span class="info-label">Konum</span>
        <span class="info-value" id="info-location">-</span>
      </div>
      <div class="info-box">
        <span class="info-label">Nitelik / Pafta</span>
        <span class="info-value" id="info-type">-</span>
      </div>
      <div class="info-box">
        <span class="info-label">Tapu Alanı</span>
        <span class="info-value" id="info-deed-area">- m²</span>
      </div>
      <div class="info-box">
        <span class="info-label">Hesaplanan Alan (ITRF)</span>
        <span class="info-value" id="info-calc-area" style="color: #38bdf8;">- m²</span>
      </div>
    </div>

    <!-- Ulusal Metrik Koordinat Listesi -->
    <div>
      <span class="info-label" style="margin-bottom: 4px;">Ulusal Metrik Koordinatlar (ITRF96 - GRS80)</span>
      <div class="coord-table-wrap">
        <table>
          <thead>
            <tr>
              <th>Nokta</th>
              <th>Y (Sağa / m)</th>
              <th>X (Yukarı / m)</th>
            </tr>
          </thead>
          <tbody id="coord-table-body"></tbody>
        </table>
      </div>
    </div>

    <!-- CAD / Dışa Aktarma Butonları -->
    <div style="display: flex; gap: 8px; margin-top: 4px;">
      <button class="btn-primary" style="flex: 1;" onclick="copyCadScript()">
        AutoCAD Komut Kopyala
      </button>
      <button class="btn-secondary" onclick="exportGeoJson()">
        GeoJSON İndir
      </button>
    </div>
  </div>

  <!-- Leaflet JS -->
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <script>
    /* ==========================================================================
       1. HARİTA KURULUMU & UYDU ALTLIKLARI
       ========================================================================== */
    const map = L.map('map', {
      zoomControl: false,
      attributionControl: false
    }).setView([40.23, 28.95], 11);

    L.control.zoom({ position: 'bottomleft' }).addTo(map);

    // Google Hibrit Uydu Katmanı
    const googleSatellite = L.tileLayer('https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}', {
      maxZoom: 22,
      subdomains: ['mt0', 'mt1', 'mt2', 'mt3']
    }).addTo(map);

    // OpenStreetMap Vektör Katmanı
    const osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19
    });

    L.control.layers({
      "Uydu (Hibrit)": googleSatellite,
      "Sokak Haritası": osm
    }, null, { position: 'bottomleft' }).addTo(map);

    let activeParcelLayer = null;
    let currentGeoJsonData = null;
    let currentConvertedPoints = [];

    /* ==========================================================================
       2. GEODEZİK MATEMATİK: WGS84 -> ITRF96 TM 3° DÖNÜŞÜM MOTORU
       ========================================================================== */
    function convertWgs84ToItrf96(lng, lat, forcedDom = null) {
      // Dilim Orta Meridyeni (DOM) Tayini (3° dilimler: 27, 30, 33, 36, 39, 42, 45)
      const dom = forcedDom !== null ? forcedDom : Math.round(lng / 3.0) * 3;

      // Proj4js için ITRF96 / GRS80 TM 3° Tanımı
      const itrfDef = `+proj=tmerc +lat_0=0 +lon_0=${dom} +k=1 +x_0=500000 +y_0=0 +ellps=GRS80 +units=m +no_defs`;
      const wgs84Def = '+proj=longlat +datum=WGS84 +no_defs';

      // Dönüşüm [x, y] -> x = Easting (Sağa), y = Northing (Yukarı)
      const [easting, northing] = proj4(wgs84Def, itrfDef, [lng, lat]);

      return {
        dom: dom,
        y: Number(easting.toFixed(3)),
        x: Number(northing.toFixed(3))
      };
    }

    function calculateMetricPolygonArea(points) {
      let area = 0.0;
      const n = points.length;
      for (let i = 0; i < n; i++) {
        const j = (i + 1) % n;
        area += points[i].y * points[j].x;
        area -= points[j].y * points[i].x;
      }
      return Math.abs(area) / 2.0;
    }

    /* ==========================================================================
       3. PARSEL GEOMETRİSİ RENDER VE ANALİZİ
       ========================================================================== */
    function renderParcel(geoJsonFeature, meta = {}) {
      if (activeParcelLayer) {
        map.removeLayer(activeParcelLayer);
      }

      currentGeoJsonData = geoJsonFeature;

      activeParcelLayer = L.geoJSON(geoJsonFeature, {
        style: {
          color: '#ef4444',
          weight: 2.5,
          opacity: 1,
          fillColor: '#38bdf8',
          fillOpacity: 0.25
        }
      }).addTo(map);

      map.fitBounds(activeParcelLayer.getBounds(), { padding: [80, 80], maxZoom: 19 });

      const coords = extractCoordinates(geoJsonFeature.geometry);
      if (!coords || coords.length === 0) return;

      currentConvertedPoints = [];
      const tableBody = document.getElementById('coord-table-body');
      tableBody.innerHTML = '';

      let domDetected = null;

      coords.forEach((pt, index) => {
        const [lng, lat] = pt;
        const itrf = convertWgs84ToItrf96(lng, lat, domDetected);
        domDetected = itrf.dom;

        currentConvertedPoints.push({
          pt: index + 1,
          y: itrf.y,
          x: itrf.x,
          lng: lng,
          lat: lat
        });

        const tr = document.createElement('tr');
        tr.innerHTML = `<td>${index + 1}</td><td>${itrf.y.toFixed(3)}</td><td>${itrf.x.toFixed(3)}</td>`;
        tableBody.appendChild(tr);
      });

      const calcArea = calculateMetricPolygonArea(currentConvertedPoints);

      document.getElementById('info-title').innerText = `${meta.ada || '-'} Ada / ${meta.parsel || '-'} Parsel`;
      document.getElementById('info-dom').innerText = `ITRF96 DOM ${domDetected}°`;
      document.getElementById('info-location').innerText = `${meta.il || ''} / ${meta.ilce || ''} / ${meta.mahalle || ''}`;
      document.getElementById('info-type').innerText = `${meta.nitelik || 'Arsa'} - Pafta: ${meta.pafta || '-'}`;
      document.getElementById('info-deed-area').innerText = meta.tapuAlani ? `${meta.tapuAlani}` : '-';
      document.getElementById('info-calc-area').innerText = `${calcArea.toLocaleString('tr-TR', { maximumFractionDigits: 2 })} m²`;

      document.getElementById('info-panel').style.display = 'flex';
    }

    function extractCoordinates(geometry) {
      if (geometry.type === 'Polygon') {
        return geometry.coordinates[0];
      } else if (geometry.type === 'MultiPolygon') {
        return geometry.coordinates[0][0];
      }
      return null;
    }

    /* ==========================================================================
       4. CAD VE VERİ DIŞA AKTARIM FONKSİYONLARI
       ========================================================================== */
    function copyCadScript() {
      if (!currentConvertedPoints.length) return;

      const p1 = currentConvertedPoints[0];
      let script = `_LAYER _M PARSEL_SINIRI _C 1 PARSEL_SINIRI\n_PLINE\n`;

      currentConvertedPoints.forEach(p => {
        const dx = (p.y - p1.y).toFixed(3);
        const dy = (p.x - p1.x).toFixed(3);
        script += `${dx},${dy}\n`;
      });
      script += `_C\n_ZOOM _E\n`;

      navigator.clipboard.writeText(script).then(() => {
        alert("AutoCAD komut dizilimi panoya kopyalandı! AutoCAD komut satırına (Command line) doğrudan Ctrl+V ile yapıştırabilirsiniz.");
      });
    }

    function exportGeoJson() {
      if (!currentGeoJsonData) return;
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(currentGeoJsonData, null, 2));
      const downloadAnchor = document.createElement('a');
      downloadAnchor.setAttribute("href", dataStr);
      downloadAnchor.setAttribute("download", `parsel_${Date.now()}.geojson`);
      document.body.appendChild(downloadAnchor);
      downloadAnchor.click();
      downloadAnchor.remove();
    }

    /* ==========================================================================
       5. TKGM / DIŞ SERVİS SORGULAMA MOTORU
       ========================================================================== */
    async function executeParcelQuery() {
      const ilSelect = document.getElementById('select-il');
      const ilceSelect = document.getElementById('select-ilce');
      const mahalleSelect = document.getElementById('select-mahalle');
      const ada = document.getElementById('input-ada').value.trim();
      const parsel = document.getElementById('input-parsel').value.trim();

      if (!mahalleSelect.value || !ada || !parsel) {
        alert("Lütfen İl, İlçe, Mahalle, Ada ve Parsel alanlarını eksiksiz girin.");
        return;
      }

      const mahalleId = mahalleSelect.value;
      const spinner = document.getElementById('loading-spinner');
      spinner.style.display = 'block';

      const targetUrl = `https://cbsapi.tkgm.gov.tr/megsiswebapi.v3.1/api/parsel/${mahalleId}/${ada}/${parsel}`;

      try {
        let response;
        try {
          response = await fetch(targetUrl);
        } catch (corsErr) {
          response = await fetch(`https://api.allorigins.win/raw?url=${encodeURIComponent(targetUrl)}`);
        }

        const data = await response.json();
        spinner.style.display = 'none';

        if (data && data.geometry) {
          renderParcel(data, {
            il: ilSelect.options[ilSelect.selectedIndex].text,
            ilce: ilceSelect.options[ilceSelect.selectedIndex].text,
            mahalle: mahalleSelect.options[mahalleSelect.selectedIndex].text,
            ada: ada,
            parsel: parsel,
            tapuAlani: data.properties?.alan,
            nitelik: data.properties?.nitelik,
            pafta: data.properties?.pafta
          });
        } else {
          alert("Parsel verisi bulunamadı veya kurum servisinden yanıt alınamadı.");
        }
      } catch (err) {
        spinner.style.display = 'none';
        console.error(err);
        alert("Bağlantı hatası: Kamu CBS servisine erişilemedi. 'GeoJSON Yapıştır' veya 'Bursa Nilüfer Örnek' butonunu kullanabilirsiniz.");
      }
    }

    /* ==========================================================================
       6. ÖRNEK VERİ & GEOJSON MANUEL ENTEGRASYON YARDIMCILARI
       ========================================================================== */
    function loadSampleParcel() {
      const sample = {
        "type": "Feature",
        "geometry": {
          "type": "Polygon",
          "coordinates": [[
            [28.93245, 40.24580],
            [28.93385, 40.24570],
            [28.93375, 40.24490],
            [28.93230, 40.24505],
            [28.93245, 40.24580]
          ]]
        },
        "properties": {
          "alan": "10.450 m²",
          "nitelik": "Sanayi Arsası",
          "pafta": "H21-C-05-A"
        }
      };

      document.getElementById('input-ada').value = "1483";
      document.getElementById('input-parsel').value = "8";

      renderParcel(sample, {
        il: "Bursa",
        ilce: "Nilüfer",
        mahalle: "Minareliçavuş",
        ada: "1483",
        parsel: "8",
        tapuAlani: sample.properties.alan,
        nitelik: sample.properties.nitelik,
        pafta: sample.properties.pafta
      });
    }

    function promptGeoJSON() {
      const input = prompt("GeoJSON metnini veya koordinat dizisini yapıştırın:");
      if (!input) return;
      try {
        const parsed = JSON.parse(input);
        const feature = parsed.type === 'FeatureCollection' ? parsed.features[0] : (parsed.type === 'Feature' ? parsed : { type: 'Feature', geometry: parsed });
        renderParcel(feature, {
          il: "Manuel",
          ilce: "Girdi",
          mahalle: "GeoJSON",
          ada: "-",
          parsel: "-",
          tapuAlani: "-",
          nitelik: "Özel Poligon"
        });
      } catch (e) {
        alert("Geçersiz GeoJSON formatı!");
      }
    }

    /* ==========================================================================
       7. İL LİSTESİ VE DİNAMİK DROPDOWN YÖNETİMİ
       ========================================================================== */
    const TURKIYE_ILLERI = [
      { id: 1, ad: "Adana" }, { id: 2, ad: "Adıyaman" }, { id: 3, ad: "Afyonkarahisar" },
      { id: 6, ad: "Ankara" }, { id: 7, ad: "Antalya" }, { id: 16, ad: "Bursa" },
      { id: 34, ad: "İstanbul" }, { id: 35, ad: "İzmir" }, { id: 41, ad: "Kocaeli" },
      { id: 54, ad: "Sakarya" }
    ];

    window.addEventListener('DOMContentLoaded', () => {
      const ilSelect = document.getElementById('select-il');
      TURKIYE_ILLERI.sort((a,b) => a.ad.localeCompare(b.ad, 'tr')).forEach(il => {
        const opt = document.createElement('option');
        opt.value = il.id;
        opt.innerText = il.ad;
        ilSelect.appendChild(opt);
      });

      ilSelect.addEventListener('change', async (e) => {
        const ilId = e.target.value;
        const ilceSelect = document.getElementById('select-ilce');
        const mahalleSelect = document.getElementById('select-mahalle');
        ilceSelect.innerHTML = '<option value="">Yükleniyor...</option>';
        ilceSelect.disabled = true;
        mahalleSelect.innerHTML = '<option value="">Önce İlçe</option>';
        mahalleSelect.disabled = true;

        if (!ilId) return;

        try {
          const url = `https://cbsapi.tkgm.gov.tr/megsiswebapi.v3.1/api/idariYapi/ilceListe/${ilId}`;
          const res = await fetch(`https://api.allorigins.win/raw?url=${encodeURIComponent(url)}`);
          const data = await res.json();
          ilceSelect.innerHTML = '<option value="">İlçe Seçiniz...</option>';
          data.forEach(item => {
            const opt = document.createElement('option');
            opt.value = item.id;
            opt.innerText = item.text || item.ad;
            ilceSelect.appendChild(opt);
          });
          ilceSelect.disabled = false;
        } catch (err) {
          ilceSelect.innerHTML = '<option value="">İlçeler Alınamadı</option>';
        }
      });

      document.getElementById('select-ilce').addEventListener('change', async (e) => {
        const ilceId = e.target.value;
        const mahalleSelect = document.getElementById('select-mahalle');
        mahalleSelect.innerHTML = '<option value="">Yükleniyor...</option>';
        mahalleSelect.disabled = true;

        if (!ilceId) return;

        try {
          const url = `https://cbsapi.tkgm.gov.tr/megsiswebapi.v3.1/api/idariYapi/mahalleListe/${ilceId}`;
          const res = await fetch(`https://api.allorigins.win/raw?url=${encodeURIComponent(url)}`);
          const data = await res.json();
          mahalleSelect.innerHTML = '<option value="">Mahalle Seçiniz...</option>';
          data.forEach(item => {
            const opt = document.createElement('option');
            opt.value = item.id;
            opt.innerText = item.text || item.ad;
            mahalleSelect.appendChild(opt);
          });
          mahalleSelect.disabled = false;
        } catch (err) {
          mahalleSelect.innerHTML = '<option value="">Mahalleler Alınamadı</option>';
        }
      });
    });
  </script>
</body>
</html>
