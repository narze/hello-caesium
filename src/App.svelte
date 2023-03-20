<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'
  import { onMount } from "svelte"

  let geolocationState
  let latitude
  let longitude

  const options = {
    enableHighAccuracy: true,
    timeout: 7000,
    maximumAge: 0
  };

  function onError(err) {
    console.warn(`ERROR(${err.code}): ${err.message}`);
  }

  onMount(() => {
    updateGeolocationState()
  })

  function updateGeolocationState() {
    if (navigator.permissions) {

      navigator.permissions.query({name:'geolocation'}).then(function(result) {
        result.onchange = function() {
          console.log(this)
          geolocationState = this.state

          if (this.state === 'granted') {n
            navigator.geolocation.getCurrentPosition(setPosition, onError, options)
          }
        }

        geolocationState = result.state

        if (result.state === 'granted') {
          // geolocation permission has been granted
          navigator.geolocation.getCurrentPosition(setPosition, onError, options)
        } else if (result.state === 'prompt') {
          // geolocation permission has not been granted, but the user may be prompted
        } else {
          // geolocation permission has not been granted
        }
      });
    } else {
      // geolocation permission is not supported by this browser
      alert("บราวเซอร์ไม่รองรับ")
    }
  }

  function grantLocation() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(updateGeolocationState, onError, options);
    } else {
      alert("Geolocation is not supported by this browser.");
    }
  }

  function setPosition(position) {
    latitude = position.coords.latitude;
    longitude = position.coords.longitude;
    // console.log("Latitude: " + latitude + " Longitude: " + longitude);
  }

  function getDistanceFromLatLonInKm(lat1,lon1,lat2,lon2) {
    var R = 6371; // Radius of the earth in km
    var dLat = deg2rad(lat2-lat1);  // deg2rad below
    var dLon = deg2rad(lon2-lon1);
    var a =
      Math.sin(dLat/2) * Math.sin(dLat/2) +
      Math.cos(deg2rad(lat1)) * Math.cos(deg2rad(lat2)) *
      Math.sin(dLon/2) * Math.sin(dLon/2)
      ;
    var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    var d = R * c; // Distance in km
    return d;
  }

  function deg2rad(deg) {
    return deg * (Math.PI/180)
  }

  function getDirection(lat1, lon1, lat2, lon2) {
    const dLat = lat2 - lat1;
    const dLon = lon2 - lon1;
    let direction = '';
    if (Math.abs(dLat) >= Math.abs(dLon)) {
      // Vertical direction (North or South)
      if (dLat > 0) {
        direction += 'เหนือ';
      } else if (dLat < 0) {
        direction += 'ใต้';
      }
      // Add intermediate direction if applicable
      if (Math.abs(dLon) >= Math.abs(dLat) * 2) {
        if (dLon > 0) {
          direction = 'ตะวันออก' + "เฉียง" + direction;
        } else if (dLon < 0) {
          direction = 'ตะวันตก' + "เฉียง" + direction;
        }
      }
    } else {
      // Horizontal direction (East or West)
      if (dLon > 0) {
        direction += 'ตะวันออก';
      } else if (dLon < 0) {
        direction += 'ตะวันตก';
      }
      // Add intermediate direction if applicable
      if (Math.abs(dLat) >= Math.abs(dLon) * 2) {
        if (dLat > 0) {
          direction += "เฉียงเหนือ"
        } else if (dLat < 0) {
          direction += "เฉียงใต้"
        }
      }
    }

    return direction;
  }

  function getGoogleMapsLink(lat, lng) {
    const zoomLevel = 15; // Set the zoom level for the map
    const mapType = 'roadmap'; // Set the type of map to display
    const url = `https://www.google.com/maps/@${lat},${lng},${zoomLevel}z?hl=en&maptype=${mapType}`;
    return url;
  }

  function isSafari() {
    // Check if the browser is Safari
    if (navigator.userAgent.indexOf('Safari') !== -1 && navigator.userAgent.indexOf('Chrome') === -1) {
      // Browser is Safari
      return true
    }

    // Check if the browser is Mobile Safari
    if (navigator.userAgent.match(/(iPod|iPhone|iPad)/) && navigator.userAgent.indexOf('AppleWebKit') !== -1) {
      // Browser is Mobile Safari
      return true
    }

    return false
  }

</script>

<main>
  <h1>Hello Caesium</h1>

  <p>ค้นหาซีเซียม-137 ใกล้คุณ</p>

  <div class="card">
    {#if isSafari()}
      <div style="margin-bottom: 1rem;">(แอปอาจไม่ทำงานบน Safari โปรดใช้บราวเซอร์อื่น หรือช่วยแก้โค้ดให้ใช้ได้บน Safari <a href="https://github.com/narze/hello-caesium">https://github.com/narze/hello-caesium</a>)</div>
    {/if}
    {#if !navigator.geolocation}
      <h2>บราวเซอร์ไม่รองรับ Geolocation</h2>
    {:else if geolocationState == "prompt"}
      <button on:click={grantLocation}>ขอ Location</button>
    {:else if geolocationState == "granted"}
      <!--<div>Latitude: {latitude}</div>
      <div>Longitude: {longitude}</div>-->
      {@const diffInKm = getDistanceFromLatLonInKm(latitude, longitude, 13.9215457, 101.5806728).toFixed(3)}
      <h2>คุณอยู่ห่างจาก<a href={getGoogleMapsLink(13.9215457, 101.5806728)} target="_blank">ซีเซียม</a> ประมาณ <span class="rainbow" style={"font-size: 200%"}> {diffInKm}</span> กิโลเมตร ทางทิศ{getDirection(latitude, longitude, 13.9215457, 101.5806728)}</h2>
    {:else if geolocationState == "denied"}
      <h2>คุณได้ปฏิเสธการให้ Location เราจึงไม่มีข้อมูลมากพอที่จะหาซีเซียมที่อยู่ใกล้ท่าน</h2>
    {:else}
      <h2>Loading...</h2>
    {/if}
  </div>

  <p class="footer">Disclaimer: This is just for fun, please read some news, don't panic (yet.)</p>

</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
  .footer {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    margin: 0;
    padding: 10px;
  }

  .rainbow {
    background-image: linear-gradient(to right, violet, indigo, blue, green, yellow, orange, red);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    animation: rainbow 3s ease infinite;
    -webkit-animation: rainbow 3s ease infinite;
  }

  @keyframes rainbow {
    0% {
      background-position: 0 0;
      -webkit-background-position: 0 0;
    }
    100% {
      background-position: 400% 0;
      -webkit-background-position: 400% 0;
    }
  }

  @-webkit-keyframes rainbow {
    0% {
      background-position: 0 0;
      -webkit-background-position: 0 0;
    }
    100% {
      background-position: 400% 0;
      -webkit-background-position: 400% 0;
    }
  }

</style>
