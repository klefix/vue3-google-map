<script lang="ts">
import { GoogleMap, Marker, CustomControl } from "@/components/index";
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "ServeDev",
  components: {
    Marker,
    GoogleMap,
    CustomControl,
  },
  setup() {
    const greet = () => alert("hi");
    const dist = ref(0);
    const testControlPosition = ref<keyof typeof google.maps.ControlPosition>("LEFT_CENTER");
    const markers = ref([
      { position: { lat: 35 + 0.015, lng: -95 + 0.015 } },
      { position: { lat: 35 + 0.005, lng: -95 + 0.005 } },
    ]);
    const remove = () => (markers.value = []);

    setInterval(() => {
      dist.value += 0.005;
    }, 1000);

    setInterval(() => {
      testControlPosition.value = testControlPosition.value === "LEFT_CENTER" ? "RIGHT_CENTER" : "LEFT_CENTER";
    }, 3000);

    setInterval(() => {
      // test blinking
      markers.value = [
        { position: { lat: 35 + 0.015, lng: -95 + 0.015 } },
        { position: { lat: 35 + 0.005, lng: -95 + 0.005 } },
      ];
    }, 1000);

    // Just to test dynamic custom controls
    const bottomControlQuantity = ref(0);

    return { greet, remove, markers, dist, bottomControlQuantity, testControlPosition };
  },
});
</script>

<template>
  <GoogleMap api-key="" style="width: 100%; height: 80vh" :center="{ lat: 35, lng: -95 }" :zoom="13">
    <Marker :options="{ position: { lat: 35, lng: -95 + dist } }" />
    <Marker :options="{ position: { lat: 35, lng: -95 } }" @click="greet" />
    <Marker v-for="(marker, index) of markers" :key="index" :options="marker" @click="remove" />
    <CustomControl
      v-for="index in Math.floor(bottomControlQuantity)"
      :key="`control-${index}`"
      position="BOTTOM_CENTER"
    >
      <button style="height: 20px; background: orange; color: white" @click="greet">[{{ index }}] Greet</button>
    </CustomControl>

    <CustomControl :position="testControlPosition">
      <button style="height: 20px; background: blue; color: white">testing position change</button>
    </CustomControl>

    <CustomControl position="TOP_CENTER">
      <button style="height: 20px; background: green; color: white" @click="bottomControlQuantity++">
        click me to add a bottom control
      </button>
      <button
        style="height: 20px; background: red; color: white"
        @click="bottomControlQuantity > 0 ? bottomControlQuantity-- : () => null"
      >
        click me to remove the last bottom control
      </button>
    </CustomControl>
  </GoogleMap>
</template>
