<script setup>
import { ref } from 'vue'
import HelloWorld from './components/HelloWorld.vue'
ymaps.ready(init);

let newCoords;
let newAddress = ref();

function init() {
  let map = new ymaps.Map("map", {
    center: [44.63333, 41.93333], // Координаты Москвы
    zoom: 15,
    controls: []
  }, {
    restrictMapArea: [
      [44.562330, 41.860616],
      [44.682021, 42.046242],
    ]
  }),
    myPlacemark;
  var suggestView1 = new ymaps.SuggestView('suggest', {
    boundedBy: [
      [44.682021, 42.046242],
      [44.562330, 41.860616],
    ]
  });

  map.events.add('click', function (e) {
    let coords = e.get('coords');


    ymaps.geocode(coords, {
      kind: 'house', // Поиск ближайшего здания
      results: 1
    }).then(function (res) {
      // Вырвнивание метки по дому
      let firstGeoObject = res.geoObjects.get(0);
      newCoords = firstGeoObject.geometry.getCoordinates();
      newAddress.value = firstGeoObject.getAddressLine();

      console.log(newCoords)

      // Удаление предыдущей метки
      if (myPlacemark) {
        map.geoObjects.remove(myPlacemark);
      }

      // Создание новой метки на полученных координатах
      myPlacemark = new ymaps.Placemark(newCoords, {
        hintContent: 'Метка',
        balloonContent: newAddress.value
      });

      map.geoObjects.add(myPlacemark);
    });
  });

  let input = document.getElementById("suggest");
  let button = document.getElementById("button");


  button.addEventListener("click", function () {
    let query = input.value; // Получаем значение из текстового поля
    if (!query) {
      alert("Пожалуйста, введите адрес!");
      return;
    }

    // Осуществляем геокодирование (поиск) с учетом ограничения по области
    ymaps.geocode(query, {
      results: 1,
      boundedBy: map.getBounds() // Ограничиваем поиск текущей видимой областью карты
    }).then(function (res) {
      let firstGeoObject = res.geoObjects.get(0),
        coords = firstGeoObject.geometry.getCoordinates();

      map.setCenter(coords, 15); // Центрируем карту на найденном объекте и устанавливаем уровень масштабирования

      // Удаляем старую метку, если она существует
      if (myPlacemark) {
        map.geoObjects.remove(myPlacemark);
      }

      newAddress.value = firstGeoObject.getAddressLine();
      // Устанавливаем метку по найденному адресу
      myPlacemark = new ymaps.Placemark(coords, {
        balloonContent: newAddress.value
      });
      map.geoObjects.add(myPlacemark);
    }, function (err) {
      // Обработка случая, когда ничего не найдено
      alert('Ошибка при поиске адреса. Попробуйте снова.');
    });

  });
  let map2 = new ymaps.Map("map2", {
    center:[44.62084431369326,41.947966075646626], // Координаты Москвы
    zoom: 20,
    controls: []
  },
  {
    restrictMapArea: [
      [44.562330, 41.860616],
      [44.682021, 42.046242],
    ]
  })

  let myPlacemark2 = new ymaps.Placemark([44.62084431369326,41.947966075646626], {
    hintContent: 'Метка',
    balloonContent: 'Самовывоз по адресу: улица Гагарина, 110А'
  });

  map2.geoObjects.add(myPlacemark2);

  

  let mapStav = new ymaps.Map("map_stav", {
    center: [45.0375,  41.96623], // Координаты Москвы
    zoom: 20,
    controls: []
  },
  {
    restrictMapArea: [
        [45.1203, 41.87061],
        [44.96334, 42.0536],
    ]
  })
  var myPolygon = new ymaps.Circle([
    [45.0375,  41.96623], 5000
], {
    // Параметры многоугольника
}, {
    draggable: false,
    fillColor: "#DB709350",
    strokeColor: "#990066",
    strokeOpacity: 0.8,
    strokeWidth: 5,
    zIndexActive: -1,
    zIndex: -1,
    zIndexDrag: -1,
    interactiveZIndex: -1,
    zIndexHover: -1,
    fillOpacity: 0.4
});

// Добавляем круг на карту.
mapStav.geoObjects.add(myPolygon);

let myPlacemarkStav;

var myPolygon = new ymaps.Circle([
    [45.0375,  41.96623], 5000
], {
    // Параметры многоугольника
}, {
    draggable: false,
    fillColor: "#DB709350",
    strokeColor: "#990066",
    strokeOpacity: 0.8,
    strokeWidth: 5,
    zIndexActive: -1,
    zIndex: -1,
    zIndexDrag: -1,
    interactiveZIndex: -1,
    zIndexHover: -1,
    fillOpacity: 0.4
});

// Добавляем круг на карту.
mapStav.geoObjects.add(myPolygon);

// let myPlacemarkStav;

var isRequestPending = false;

if (!myPlacemarkStav) {
    myPlacemarkStav = new ymaps.Placemark([0,0], {
        hintContent: 'Метка',
    });
    mapStav.geoObjects.add(myPlacemarkStav);
}

mapStav.events.add('click', async function (e) {
  let coords = e.get('coords');

  // Пропускаем запрос, если точка не внутри круга
  if (!isPointInsideCircle(coords, myPolygon.geometry.getCoordinates(), myPolygon.geometry.getRadius())) {
      console.error("Точка за пределами круга.");
      return;
  }

  // Обновляем положение метки, вместо создания новой
  myPlacemarkStav.geometry.setCoordinates(coords);
});


function isPointInsideCircle(point, circleCenter, radius) {
    var distance = ymaps.coordSystem.geo.getDistance(point, circleCenter);
    return distance <= radius;
}


function isPointInsideCircle(point, circleCenter, radius) {
    var distance = ymaps.coordSystem.geo.getDistance(point, circleCenter);
    return distance <= radius;
}


}


</script>

<template>
  <div style="color: black;">
    <p style="color: white;">{{ newCoords }}</p>
    <p style="color: white;">{{ newAddress }}</p>
    <p style="color: white;">Заказать на адрес</p>
    <input type="text" id="suggest">
    <button id="button" style="margin-left: 10px;">Поиск</button>
    <div id="map" style="width: 600px; height: 400px; color: black; "></div>

    <p style="color: white;">Самовывоз</p>
    <div id="map2" style="width: 600px; height: 400px; color: black; "></div>

    <p style="color: white;">Ставрополь</p>
    <div id="map_stav" style="width: 1000px; height: 1000px; color: black;"></div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
