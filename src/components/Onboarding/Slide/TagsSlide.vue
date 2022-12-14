<template>
  <div id="tags-slide" class="title" @touchstart="touchStart" @touchend="changeSlide" @mousewheel="wheelMove">
    <div class="tags-title">
      <h1 v-if="show_form" class="tags-h1">SEARCH WITH TAGS</h1>
      <h1 v-if="show_full_tag_page" class="tags-h1">
        <span v-for="(tag, index) in this.selected_tags" :key="index">
          <span v-if="index > 0">{{ tag.toUpperCase() }}</span>
          <span v-if="index < this.selected_tags.length - 1 && index > 0"> & </span>
        </span>
      </h1>
    </div>
    <arrow-up class="justify-arrow clickable" :isFull="isFullSize" :isMobile="isMobile" :text="arrowText"
      @previous-slide="$emit('previousSlide')" />
  </div>
  <div class="form-and-carousel" v-if="show_form">
    <tags-combinaisons-form @updateTagsList="updateTagsList" :isMobile="isMobile" />
    <images-carousel v-if="show_carousel" @showFullTagPage="showFullScreenCarousel()" :isMobile="isMobile"
      :nbImages="this.images_bis.length" :key="this.carousel_key" />
  </div>
  <full-tag-page v-if="this.show_full_tag_page" @exitFullScreen="exitFullScreen()" />
  <!-- It ensures the full display in any case -->
  <div style="padding-bottom: 2vh" />
</template>

<script>
import { mapGetters } from "vuex";
import tags from "@/assets/onboarding/tags.json";
import images from '@/assets/data/images.json';
import ArrowUp from "../Logo/ArrowUp.vue";
import ImagesCarousel from "../ImagesCarousel.vue";
import TagsCombinaisonsForm from "../../FullTags/TagsCombinaisonsForm.vue";
import FullTagPage from '../../FullTags/FullTagPage.vue';

export default {
  components: { ArrowUp, ImagesCarousel, TagsCombinaisonsForm, FullTagPage },
  emits: ["previousSlide", "loadTagView", "showFullTagPage", "updateTagsList", "exitFullScreen"],
  props: {
    isFullSize: Boolean,
    isMobile: Boolean,
    arrowText: Object,
  },
  data() {
    return {
      tags: tags,
      tags_to_display: tags.tags,
      images,
      images_for_carousel: images,
      carousel_key: 0,
      show_carousel: false,
      show_full_tag_page: false,
      show_form: true,
      couldLoadNextSlide: true,
      changeSlideInProgress: false,
      delayBeforeAction: 0,
      startMoveTime: 0,
      startPosition: 0,
    };
  },
  methods: {
    wheelMove(event) {
      if (event.deltaY < 0 && !this.changeSlideInProgress) {
        this.changeSlideInProgress = true;
        this.$emit("previousSlide");
        // This timeout is used to ensure that only one action is take into account when long wheel has been used.
        setTimeout(() => (this.changeSlideInProgress = false), 2000);
      }
    },
    touchStart(event) {
      this.startMoveTime = Date.now();
      this.startPosition = event.changedTouches[0].clientY;
    },
    changeSlide(event) {
      if (Date.now() - this.startMoveTime > 100) {
        const endPosition = event.changedTouches[0].clientY;
        if (this.startPosition < endPosition) {
          this.$emit("previousSlide");
        }
      }
    },
    scrollMove() {
      if (this.$refs["tags_to_display"].scrollTop === 0) {
        this.couldLoadNextSlide = true;
        this.changeSlideInProgress = false;
        this.delayBeforeAction = Date.now();
      } else {
        this.couldLoadNextSlide = false;
      }
    },
    wheelMoveScroll(event) {
      const diffTime = Date.now() - this.delayBeforeAction;
      if (
        this.couldLoadNextSlide &&
        event.deltaY < 0 &&
        diffTime > 1000 &&
        !this.changeSlideInProgress
      ) {
        this.changeSlideInProgress = true;
        this.$emit("previousSlide");
        // This timeout is used to ensure that only one action is take into account when long wheel has been used.
        setTimeout(() => (this.changeSlideInProgress = false), 2000);
      }
    },
    changeSlideScroll() {
      const diffTime = Date.now() - this.delayBeforeAction;
      if (this.couldLoadNextSlide && diffTime > 1000) {
        this.$emit("previousSlide");
      }
    },
    updateTagsList(data) {
      //rebuild the component to prevent array length problems
      this.carousel_key = data.images.length
      if (this.images_bis.length > 0) {
        this.show_carousel = true
        this.images_for_carousel = data.images
      }
      this.handleClickableTagsDivSize()
    },
    showFullScreenCarousel() {
      this.show_form = false;
      this.$store.dispatch('setFullTagPageOrigin', 'tags_slide')
      this.show_full_tag_page = true;
    },
    exitFullScreen() {
      this.show_form = true
      this.show_carousel = true
      this.show_full_tag_page = false
      // //Resize the clickable tags div to let the carousel appear
      setTimeout(() => {
        this.handleClickableTagsDivSize()
      })
    },
    //Is used to let the space for the images carousel in some cases
    handleClickableTagsDivSize() {
      if (this.selected_tags.length > 1) {
        document.querySelector('.clickable-tags-list').classList.add('with-carousel')
      } else {
        document.querySelector('.clickable-tags-list').classList.remove('with-carousel')
      }
    }
  },
  computed: {
    fontSize() {
      return {
        "desktop-font": !this.isMobile,
        "mobile-font": this.isMobile,
      };
    },
    ...mapGetters({
      images_bis: "getImages",
      selected_tags: "getTags"
    }),
  },
};

</script>

<style scoped>
@import "../../shared/pointer.css";

.title {
  min-height: 10vh;
  width: 96vw;
  padding-left: 2vw;
  padding-right: 2vw;
  display: inline-flex;
  align-items: flex-start;
  justify-content: center;
}

.tags-title {
  width: 450px;
}

.justify-arrow {
  display: flex;
  align-content: flex-start;
  margin: 3.75vh 0 0 0;
  width: 82vw;
  justify-content: flex-end;
}

.tags-h1 {
  width: 70vw;
}

.justify-text {
  width: 10vw;
  justify-content: flex-start;
}
</style>
