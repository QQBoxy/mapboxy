<script setup>
import { ref, onMounted } from 'vue';
import { Loader } from '@googlemaps/js-api-loader';
import axios from 'axios';

const container = ref();

const delay = () => {
    return new Promise(function (resolve,) {
        setTimeout(function () {
            resolve();
        }, 100);
    });
}

const getList = async () => {
    const res = await axios({
        method: 'get',
        url: "https://script.google.com/macros/s/AKfycbz3blBPcJRuEIfUeM-IWzOVsiP4PYDkK2S7wkmtO7ZOaCtAAxh7UYZH9Ia-eGgTK_NlRg/exec?method=read_all",
    });
    return res.data;
};

const init = async () => {
    let geocoder = null;
    let marker = null;
    let bounds = null;

    const loader = new Loader({
        apiKey: "AIzaSyAy9XRW3j4sv9gbx4JjbKgwH8LWycwwU8s",
        version: "weekly",
        libraries: ["places"]
    });
    const google = await loader.load();
    const mapOptions = {
        center: {
            lat: 0,
            lng: 0
        },
        zoom: 10
    };
    const map = new google.maps.Map(container.value, mapOptions);

    const { list } = await getList();
    console.log("list", list);

    bounds = new google.maps.LatLngBounds();

    for (let item of list) {
        if (!item.address) {
            continue;
        }
        await delay();

        const infowindow = new google.maps.InfoWindow({});

        marker = new google.maps.Marker({ map });
        geocoder = new google.maps.Geocoder();
        let result = {};
        try {
            result = await geocoder.geocode({ address: item.address });
        } catch (e) {
            console.log(e);
            console.log(`${item.title} not found.`);
            continue;
        }
        const { results } = result;
        marker.setPosition(results[0].geometry.location);
        marker.setMap(map);

        const lat = marker.getPosition().lat();
        const lng = marker.getPosition().lng();
        const url = `https://www.google.com/maps/search/?api=1&query=${lat}%2C${lng}`;

        bounds.extend(marker.position);
        map.fitBounds(bounds);

        const content = `${item.title}<br />${item.address}`;

        google.maps.event.addListener(marker, 'mouseover', ((marker, content, infowindow) => {
            return function () {
                infowindow.setContent(content);
                infowindow.open(map, marker);
            };
        })(marker, content, infowindow));

        google.maps.event.addListener(marker, 'mouseout', ((marker, infowindow) => {
            return function () {
                infowindow.close(map, marker);
            };
        })(marker, infowindow));

        google.maps.event.addListener(marker, 'click', ((url) => {
            return function () {
                window.open(url, '_blank');
            };
        })(url));
    }
};

onMounted(() => {
    init();
});

</script>

<template>
    <div ref="container" class="container"></div>
</template>

<style scoped>
.container {
    margin: 0px auto;
    height: 100%;
}
</style>