
<template>
<!-- Must call this with a v-if on temporaryHighlights
    <HighlightSelectPopup v-if="temporaryHighlights" />
    Otherwise the offsets won't be correct.
    There may be issues when flipping the phone, will have to make sure that works correctly and adjust for it if not
-->
  {{colors}}
  <div class="fixed z-10 inset-0" aria-labelledby="modal-title" role="dialog" aria-modal="true">

    <div class="fixed inset-0" aria-hidden="true" @click="undoTempHighlight()"></div>

    <div class="absolute bg-black rounded-lg p-4 text-left overflow-hidden shadow-xl transform transition-all sm:align-middle"
         :style="`left: ${offsetCenter}px; top: ${offsetTop}px`">
      <div class="flex">
        <button v-for="color in colors"
            :key="color"
            class="cursor-pointer p-1"
            @click.stop="() => chooseHighlight(color)">
          
          <div class="w-8 h-8 rounded-2xl"
               :style="`background-color: ${color};`" />
        </button>
        <button class="cursor-pointer p-1" @click.stop="() => chooseHighlight('white')">
          <div class="h-8 rounded-2xl bg-white flex items-center justify-center px-1">
            Clear
          </div>
        </button>
      </div>
    </div>
    <div class="absolute connecting-triangle"
         :style="`left: ${offsetCenter + triangleOffset}px; top: ${offsetTop + popupHeight}px;`">
    </div>
  </div>
</template>

<script>
import useTextHighlighter from "@/composables/text-highlighter";
import { computed, ref, onMounted, onUnmounted } from "vue";

export default {
  props: {
    scrollTop: {
      type: Number,
      required: true,
    }
  },
  setup (props) {
    console.log("loading popup")
    const { temporaryHighlights, hideHighlightPopup, colors, runHighlight, loadHighlights, textId } = useTextHighlighter()
    const popupHeight = 64;
    const triangleHeight = 10;
    const triangleOffset = ref(20);

    const scrollY = ref(0);
    const updateScroll = function(){
      scrollY.value = window.scrollY;
    }
    onMounted(() => {
      document.addEventListener("scroll", updateScroll)
      updateScroll();
    })
    onUnmounted(() => {
      document.removeEventListener("scroll", updateScroll)
    })

    const offsetTop = computed(() => {
      if(!temporaryHighlights.value) return null;

      return temporaryHighlights.value[0].offsetTop - popupHeight - (triangleHeight * 2) - props.scrollTop - scrollY.value;
    })

    const offsetCenter = computed(() => {
      if(!temporaryHighlights.value) return null;

      const { offsetLeft } = temporaryHighlights.value[0];
      const popupWidth = 290;
      const windowWidth = window.innerWidth;
      const willOverflow = offsetLeft + popupWidth > windowWidth;
      if(willOverflow) {
        // move it so everything's on screen, but triangle is still pointing to highlighted part
        const leftEdgeOfPopup = windowWidth - popupWidth;
        triangleOffset.value = offsetLeft - leftEdgeOfPopup; // mutating properties in a computed property is bad practice - should try to refactor this
        return leftEdgeOfPopup;
      } else {
        return offsetLeft - triangleOffset.value;
      }
    })

    const chooseHighlight = function(color) {
      runHighlight(color)
      hideHighlightPopup();
    }

    const undoTempHighlight = function() {
      loadHighlights(); // writes over the temporary gray highlight from the selection
      hideHighlightPopup();
    }
    
    console.log(colors)
    return {
      temporaryHighlights,
      offsetTop,
      offsetCenter,
      hideHighlightPopup,
      colors,
      chooseHighlight,
      undoTempHighlight,
      popupHeight,
      triangleOffset,
    }
  },
  components: {
  }
}
</script>

<style lang="scss" scoped>
  .connecting-triangle {
    border-top-width: 8px;
  }
  .connecting-triangle::after {
    content: '';
    width: 0px;
    height: 0px;
    border-style: solid;
    border-width: 15px 10px 0 10px;
    border-color:  black transparent transparent transparent;  
    overflow: visible;
    position: absolute;
  }
</style>