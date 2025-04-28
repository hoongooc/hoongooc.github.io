// Initialize the map
var map = L.map('mapid').setView([9.702, 105.671], 11);  // Coordinates for Chau Thanh, Hau Giang

// Add OpenStreetMap as the background layer
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
}).addTo(map);

// Add GeoJSON layer for Chau Thanh, Hau Giang communes with population density data
var geojsonLayer = L.geoJSON(null, {
    onEachFeature: function (feature, layer) {
        // Popup showing commune name and population density
        layer.bindPopup("<strong>Commune: </strong>" + feature.properties.name + "<br><strong>Population Density: </strong>" + feature.properties.population_density);
    }
}).addTo(map);

// Fetch the GeoJSON data for Chau Thanh communes (replace with your actual GeoJSON file path)
fetch('data/chau_thanh_communes.geojson')
    .then(response => response.json())
    .then(data => {
        geojsonLayer.addData(data);
        styleCommuneLayer();
    });

// Function to apply graduated symbology for population density
function styleCommuneLayer() {
    geojsonLayer.setStyle(function (feature) {
        var density = feature.properties.population_density;
        return {
            fillColor: getDensityColor(density),
            weight: 1,
            opacity: 1,
            color: 'white',
            dashArray: '3',
            fillOpacity: 0.7
        };
    });
}

// Function to get color based on population density
function getDensityColor(density) {
    return density > 1000 ? '#800026' :
           density > 500  ? '#BD0026' :
           density > 200  ? '#E31A1C' :
           density > 100  ? '#FC4E2A' :
           density > 50   ? '#FD8D3C' :
           density > 20   ? '#FEB24C' :
           density > 10   ? '#FED976' :
                            '#FFEDA0';
}

// Add labels for all communes
var labelLayer = L.geoJSON(null, {
    onEachFeature: function (feature, layer) {
        layer.bindTooltip(feature.properties.name, { permanent: true, direction: 'center' }).openTooltip();
    }
}).addTo(map);

// Fetch and add the label layer (same GeoJSON as above)
fetch('data/chau_thanh_communes.geojson')
    .then(response => response.json())
    .then(data => {
        labelLayer.addData(data);
    });

// Add a legend to the map
var legend = L.control.legend({
    position: 'topright'
});
legend.addTo(map);

// Define the legend content
legend.onAdd = function() {
    var div = L.DomUtil.create('div', 'info legend');
    var grades = [0, 10, 20, 50, 100, 200, 500, 1000];
    var labels = ['<strong>Population Density</strong>'];

    for (var i = 0; i < grades.length; i++) {
        div.innerHTML += 
            '<i style="background:' + getDensityColor(grades[i] + 1) + '"></i> ' +
            grades[i] + (grades[i + 1] ? '&ndash;' + grades[i + 1] : '+') + '<br>';
    }

    return div;
};
