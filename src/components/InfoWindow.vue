<template>
  <div v-if="hasSlotContent" class="info-window-wrapper">
    <div ref="infoWindowRef" v-bind="$attrs">
      <slot />
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  PropType,
  watch,
  ref,
  computed,
  inject,
  onBeforeUnmount,
  Comment,
  onMounted,
  markRaw,
} from "vue";
import equal from "fast-deep-equal";
import { apiSymbol, mapSymbol, markerSymbol } from "../shared/index";

const infoWindowEvents = ["closeclick", "content_changed", "domready", "position_changed", "visible", "zindex_changed"];

export default defineComponent({
  inheritAttrs: false,

  props: {
    options: {
      type: Object as PropType<google.maps.InfoWindowOptions>,
      default: () => ({}),
    },
  },

  emits: infoWindowEvents,

  setup(props, { slots, emit, expose }) {
    const infoWindow = ref<google.maps.InfoWindow>();
    const infoWindowRef = ref<HTMLElement>();

    const map = inject(mapSymbol, ref());
    const api = inject(apiSymbol, ref());
    const anchor = inject(markerSymbol, ref());
    let anchorClickListener: google.maps.MapsEventListener;

    const hasSlotContent = computed(() => slots.default?.().some((vnode) => vnode.type !== Comment));

    const open = (opts?: google.maps.InfoWindowOpenOptions) =>
      infoWindow.value?.open({ map: map.value, anchor: anchor.value, ...opts });
    const close = () => infoWindow.value?.close();

    onMounted(() => {
      watch(
        [map, () => props.options],
        ([_, options], [oldMap, oldOptions]) => {
          const hasChanged = !equal(options, oldOptions) || map.value !== oldMap;

          if (map.value && api.value && hasChanged) {
            if (infoWindow.value) {
              infoWindow.value.setOptions({
                ...options,
                content: hasSlotContent.value ? infoWindowRef.value : options.content,
              });

              if (!anchor.value) open();
            } else {
              infoWindow.value = markRaw(
                new api.value.InfoWindow({
                  ...options,
                  content: hasSlotContent.value ? infoWindowRef.value : options.content,
                })
              );

              if (anchor.value) {
                anchorClickListener = anchor.value.addListener("click", () => {
                  open();
                });
              } else {
                open();
              }

              infoWindowEvents.forEach((event) => {
                infoWindow.value?.addListener(event, (e: unknown) => emit(event, e));
              });
            }
          }
        },
        {
          immediate: true,
        }
      );
    });

    onBeforeUnmount(() => {
      if (anchorClickListener) anchorClickListener.remove();

      if (infoWindow.value) {
        api.value?.event.clearInstanceListeners(infoWindow.value);
        close();
      }
    });

    expose({ infoWindow, open, close });

    return { infoWindow, infoWindowRef, hasSlotContent, open, close };
  },
});
</script>

<style scoped>
.info-window-wrapper {
  display: none;
}

.mapdiv .info-window-wrapper {
  display: inline-block;
}
</style>
