<template>
    <div id="map"></div>
  </template>
  
  <script setup>
  import { onMounted } from 'vue';
  import L from 'leaflet';
  import 'leaflet/dist/leaflet.css';
  import locationData from '@/assets/location.json';
  import temperatureData from '@/assets/temperatures.json';
  
  const initMap = () => {
    const map = L.map('map').setView([locationData.Latitude[0], locationData.Longitude[0]], 13);
  
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    }).addTo(map);
  
    const coordinates = locationData.Latitude.map((lat, index) => {
      const lng = locationData.Longitude[index];
      if (typeof lat === 'number' && typeof lng === 'number') {
        return [lat, lng];
      } else { 
        return null;
      }
    }).filter(coord => coord !== null);
  
    if (coordinates.length === 0) {
      console.error('No valid coordinates found in location data');
      return;
    }
   
    const polyline = L.polyline(coordinates, { color: 'green' }).addTo(map);
  
    coordinates.forEach((coord, index) => {
      const marker = L.circleMarker(coord, {
        radius: 5,
        color: 'blue'
      }).addTo(map);
  
      const tempIndex = findClosestIndex(locationData.Timestamp[index], temperatureData.Timestamp);
  
      marker.bindTooltip(`Temperature: ${temperatureData.OutsideTemp[tempIndex]}°C`);
    });
  
    polyline.on('mouseover', (e) => {
      const nearestPoint = findNearestPoint(e.latlng, coordinates);
      const tempIndex = findClosestIndex(locationData.Timestamp[nearestPoint.index], temperatureData.Timestamp);
      const temperature = temperatureData.OutsideTemp[tempIndex];
  
      const tooltip = L.tooltip()
        .setLatLng(nearestPoint.coord)
        .setContent(`Temperature: ${temperature}°C`)
        .addTo(map);
  
      polyline.on('mouseout', () => {
        map.removeLayer(tooltip);
      });
    });
  };
  
  const findNearestPoint = (latlng, coordinates) => {
    let nearestIndex = 0;
    let nearestDistance = Infinity;
  
    coordinates.forEach((coord, index) => {
      const distance = latlng.distanceTo(L.latLng(coord));
      if (distance < nearestDistance) {
        nearestDistance = distance;
        nearestIndex = index;
      }
    });
  
    return { index: nearestIndex, coord: coordinates[nearestIndex] };
  };
  
  const findClosestIndex = (targetTimestamp, timestamps) => {
    let closestIndex = 0;
    let closestDiff = Infinity;
  
    timestamps.forEach((timestamp, index) => {
      const diff = Math.abs(new Date(targetTimestamp) - new Date(timestamp));
      if (diff < closestDiff) {
        closestDiff = diff;
        closestIndex = index;
      }
    });
  
    return closestIndex;
  };
  
  onMounted(() => {
    initMap();
  });
  </script>
  
  <style>
  #map {
    height: 100vh;
  }
  </style>
  